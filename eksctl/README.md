#### Prerequisite: Install eksct, kubectl, aws-cli

#### Create Kubernetes Cluster with EKSCTL
    - Addons: CoreDNS and eks-pod-identity-agent

```bash 
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

```bash

helm repo add cilium https://helm.cilium.io/ && helm repo update cilium

API_SERVER_IP=B2B616832E36D6C5AB99D8EB7300C44F.gr7.us-east-1.eks.amazonaws.com
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
  --set k8sServiceHost=${API_SERVER_IP} \
  --set k8sServicePort=${API_SERVER_PORT} \
  --set cluster.name=basic-cluster
```


#### Deploy the AWS Load Balancer Controller
[Install ALB Controller with Helm](https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html)

```bash

    - ## Get EKS cluster VPC ID
    export AGW_VPC_ID=$(aws eks describe-cluster \
    --name $AGW_EKS_CLUSTER_NAME \
    --region $AGW_AWS_REGION  \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

    helm repo add eks https://aws.github.io/eks-charts && helm repo update eks

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=basic-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=$AGW_AWS_REGION \
    --set vpcId=$AGW_VPC_ID

    kubectl get deployment -n kube-system aws-load-balancer-controller
```


#### Deploy ACK Controller for API Gateway
[Installing AWS Service Controllers](https://aws-controllers-k8s.github.io/community/docs/user-docs/install/)
***Helm charts for ACK service controllers can be found in the*** [ACK registry within the Amazon ECR Public Gallery](https://gallery.ecr.aws/aws-controllers-k8s) To find a Helm chart for a specific service, you can go to gallery.ecr.aws/aws-controllers-k8s/$SERVICENAME-chart. For example, the link to the ACK service controller Helm chart for Amazon Simple Storage Service (Amazon S3) is gallery.ecr.aws/aws-controllers-k8s/s3-chart.

*You can use the Helm CLI to log into the ECR public Helm registry and install the chart.*

```bash
    export ACK_SYSTEM_NAMESPACE=ack-system
    export SERVICE=apigatewayv2
    export RELEASE_VERSION=$(curl -sL https://api.github.com/repos/aws-controllers-k8s/${SERVICE}-controller/releases/latest | jq -r '.tag_name | ltrimstr("v")')
    export AWS_REGION=us-east-1
    export CHART_REPO=oci://public.ecr.aws/aws-controllers-k8s
    export CHART_REF="$CHART_REPO/$SERVICE-chart --version $RELEASE_VERSION --set=aws.region=$AWS_REGION"

    aws ecr-public get-login-password --region us-east-1 | helm registry login --username AWS --password-stdin public.ecr.aws

    helm install --create-namespace -n $ACK_SYSTEM_NAMESPACE ack-$SERVICE-controller $CHART_REF


```

#### VPC Link - private integration with AWS VPC ***K8S CRD***

#### API Gateway - to make the application accessible from the internet ***K8S CRD***
    - Install the ACK for AWS API Gateway using helm
    - Use the service account name for API Gateway created using eksctl

#### NLB (Network Load Balancer) - to route traffic to the Gateway API (Ingress Controller) ***K8S CRD***
    - Install the ACK for AWS Load Balancer using helm
    - Use the service account name for API Gateway created using eksctl

#### Gateway API (Cilium implementation) (North/South Trafffic) - accepting traffic into the cluster, create using Helm.
    - Install Gateway API CRDs
    - Gateway
    - HTTPRoute --> Service

#### GAMA East/West Traffic