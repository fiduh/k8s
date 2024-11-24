#### Prerequisite: Install eksct, kubectl, aws-cli

#### Create Kubernetes Cluster with EKSCTL
    - Addons: CoreDNS and eks-pod-identity-agent

```bash 
export CLUSTER_NAME=basic-cluster
export AGW_AWS_REGION=us-east-1

eksctl create cluster -f cluster.yml
```

#### Create NodeGroup
```bash
eksctl create nodegroup -f nodegroup.yml
```

#### Create pod identity association for Cilium operator service account, giving it the required permissions
```bash
eksctl create podidentityassociation -f pod-identity.yml
```

#### Install Cilium CNI on the cluster using helm and replace Kube-proxy.
[Helm install Cilium docs](https://docs.cilium.io/en/stable/installation/k8s-install-helm/)

# You can install Cilium in either ENI mode or Overlay mode on an EKS cluster.
    - In case of ENI mode, Cilium will manage ENIs instead of the VPC CNI, so the aws-node DaemonSet has to be patched to prevent conflict behavior.
    - set your API_SERVER_IP and API_SERVER_PORT by using `kubectl cluster-info`

# Before we install Cilium with Gateway API, we need to make sure we install the Gateway API CRDs
[Install standard channels crds](https://gateway-api.sigs.k8s.io/guides/?h=crds#getting-started-with-gateway-api)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml

helm repo add cilium https://helm.cilium.io/ && helm repo update cilium

API_SERVER_IP=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.endpoint" --output text | sed 's|https://||')
API_SERVER_PORT=443

helm install cilium cilium/cilium --version 1.16.4 \
  --namespace kube-system \
  --set eni.enabled=true \
  --set ipam.mode=eni \
  --set egressMasqueradeInterfaces=eth0 \
  --set operator.serviceAccount.name=cilium-operator \
  --set operator.serviceAccount.create=false \
  --set routingMode=native \
  --set kubeProxyReplacement=true \
  --set nodePort.enabled=true \
  --set l7Proxy=true \
  --set endpointRoutes.enabled=true \
  --set k8sServiceHost=${API_SERVER_IP} \
  --set k8sServicePort=${API_SERVER_PORT} \
  --set cluster.name=${CLUSTER_NAME} \
  --set gatewayAPI.enabled=true \
  
```


#### Deploy the AWS Load Balancer Controller
[Install ALB Controller with Helm](https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html)

```bash

    - ## Get EKS cluster VPC ID
    export AGW_VPC_ID=$(aws eks describe-cluster \
    --name $CLUSTER_NAME \
    --region $AGW_AWS_REGION  \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

    helm repo add eks https://aws.github.io/eks-charts && helm repo update eks

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=$CLUSTER_NAME \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=$AGW_AWS_REGION \
    --set vpcId=$AGW_VPC_ID

    kubectl get deployment -n kube-system aws-load-balancer-controller
```

## Service Networking
#### Gateway API (Cilium implementation) (North/South Trafffic) - accepting traffic into the cluster, create using Helm. This creates an NLB (Network Load Balancer) that accepts external traffic
    - Install Gateway API CRDs 
    - GatewayClass
    - Gateway
    - HTTPRoute --> Service



#### Deploy ACK Controller for API Gateway
[Installing AWS Service Controllers](https://aws-controllers-k8s.github.io/community/docs/user-docs/install/)
***Helm charts for ACK service controllers can be found in the*** [ACK registry within the Amazon ECR Public Gallery](https://gallery.ecr.aws/aws-controllers-k8s) To find a Helm chart for a specific service, you can go to gallery.ecr.aws/aws-controllers-k8s/$SERVICENAME-chart. For example, the link to the ACK service controller Helm chart for Amazon Simple Storage Service (Amazon S3) is gallery.ecr.aws/aws-controllers-k8s/s3-chart.

*You can use the Helm CLI to log into the ECR public Helm registry and install the chart.*

```bash
    export ACK_SYSTEM_NAMESPACE=kube-system
    export SERVICE=apigatewayv2
    export RELEASE_VERSION=$(curl -sL https://api.github.com/repos/aws-controllers-k8s/${SERVICE}-controller/releases/latest | jq -r '.tag_name | ltrimstr("v")')
    export AWS_REGION=us-east-1
    export CHART_REPO=oci://public.ecr.aws/aws-controllers-k8s
    export CHART_REF="$CHART_REPO/$SERVICE-chart --version $RELEASE_VERSION --set=aws.region=$AWS_REGION"

    aws ecr-public get-login-password --region us-east-1 | helm registry login --username AWS --password-stdin public.ecr.aws

    helm install --create-namespace -n $ACK_SYSTEM_NAMESPACE ack-$SERVICE-controller $CHART_REF


```

## Create API Gateway resources
    - Create security group for the VPC link:
```bash
    AGW_VPCLINK_SG=$(aws ec2 create-security-group \
    --description "SG for VPC Link" \
    --group-name SG_VPC_LINK \
    --vpc-id $AGW_VPC_ID \
    --region $AGW_AWS_REGION \
    --output text \
    --query 'GroupId')
```

#### Create a VPC Link for the internal NLB:
```bash
    cat > vpclink.yaml<<EOF
    apiVersion: apigatewayv2.services.k8s.aws/v1alpha1
    kind: VPCLink
    metadata:
    name: nlb-internal
    spec:
    name: nlb-internal
    securityGroupIDs: 
        - $AGW_VPCLINK_SG
    subnetIDs: 
        - $(aws ec2 describe-subnets \
            --filter Name=tag:kubernetes.io/role/internal-elb,Values=1 \
            --query 'Subnets[0].SubnetId' \
            --region $AGW_AWS_REGION --output text)
        - $(aws ec2 describe-subnets \
            --filter Name=tag:kubernetes.io/role/internal-elb,Values=1 \
            --query 'Subnets[1].SubnetId' \
            --region $AGW_AWS_REGION --output text)
        - $(aws ec2 describe-subnets \
            --filter Name=tag:kubernetes.io/role/internal-elb,Values=1 \
            --query 'Subnets[2].SubnetId' \
            --region $AGW_AWS_REGION --output text)
    EOF

    kubectl apply -f vpclink.yaml

    aws apigatewayv2 get-vpc-links --region $AGW_AWS_REGION
```


#### Create an API with VPC Link integration: - private integration with AWS VPC ***K8S CRD***

```bash

    cat > apigw-api.yaml<<EOF
    apiVersion: apigatewayv2.services.k8s.aws/v1alpha1
    kind: API
    metadata:
    name: apitest-private-nlb
    spec:
    body: '{
                "openapi": "3.0.1",
                "info": {
                    "title": "ack-apigwv2-import-test-private-nlb",
                    "version": "v1"
                },
                "paths": {
                "/\$default": {
                    "x-amazon-apigateway-any-method" : {
                    "isDefaultRoute" : true,
                    "x-amazon-apigateway-integration" : {
                    "payloadFormatVersion" : "1.0",
                    "connectionId" : "$(kubectl get vpclinks.apigatewayv2.services.k8s.aws \
    nlb-internal \
    -o jsonpath="{.status.vpcLinkID}")",
                    "type" : "http_proxy",
                    "httpMethod" : "GET",
                    "uri" : "$(aws elbv2 describe-listeners \
    --load-balancer-arn $(aws elbv2 describe-load-balancers \
    --region $AGW_AWS_REGION \
    --query "LoadBalancers[?contains(DNSName, '$(kubectl get service authorservice \
    -o jsonpath="{.status.loadBalancer.ingress[].hostname}")')].LoadBalancerArn" \
    --output text) \
    --region $AGW_AWS_REGION \
    --query "Listeners[0].ListenerArn" \
    --output text)",
                "connectionType" : "VPC_LINK"
                    }
                    }
                },
                "/meta": {
                    "get": {
                        "x-amazon-apigateway-integration": {
                        "uri" : "$(aws elbv2 describe-listeners \
    --load-balancer-arn $(aws elbv2 describe-load-balancers \
    --region $AGW_AWS_REGION \
    --query "LoadBalancers[?contains(DNSName, '$(kubectl get service echoserver \
    -o jsonpath="{.status.loadBalancer.ingress[].hostname}")')].LoadBalancerArn" \
    --output text) \
    --region $AGW_AWS_REGION \
    --query "Listeners[0].ListenerArn" \
    --output text)",
                        "httpMethod": "GET",
                        "connectionId": "$(kubectl get vpclinks.apigatewayv2.services.k8s.aws \
    nlb-internal \
    -o jsonpath="{.status.vpcLinkID}")",
                        "type": "HTTP_PROXY",
                        "connectionType": "VPC_LINK",
                        "payloadFormatVersion": "1.0"
                        }
                    }
                    }
                },
                "components": {}
            }'
    EOF

    kubectl apply -f apigw-api.yaml

```

#### API Gateway - to make the application accessible from the internet ***K8S CRD***
    - Install the ACK for AWS API Gateway using helm
    - Use the service account name for API Gateway created using eksctl


#### Create a stage:
```bash

    echo "
    apiVersion: apigatewayv2.services.k8s.aws/v1alpha1
    kind: Stage
    metadata:
    name: "apiv1"
    spec:
    apiID: $(kubectl get apis.apigatewayv2.services.k8s.aws apitest-private-nlb -o=jsonpath='{.status.apiID}')
    stageName: api
    autoDeploy: true
    " | kubectl apply -f -

    kubectl get api apitest-private-nlb -o jsonpath="{.status.apiEndpoint}"

```

