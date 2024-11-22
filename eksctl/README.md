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

#### Create pod identity association for cilium operator service account
```bash
eksctl create podidentityassociation -f pod-identity.yml
```

#### Install Cilium CNI on the cluster using helm and replace Kube-proxy.
[K8s install helm docs](https://docs.cilium.io/en/stable/installation/k8s-install-helm/)

# You can install Cilium in either ENI mode or Overlay mode on an EKS cluster.
    - In case of ENI mode, Cilium will manage ENIs instead of VPC CNI, so the aws-node DaemonSet has to be patched to prevent conflict behavior.
    ```bash
    kubectl -n kube-system patch daemonset aws-node --type='strategic' -p='{"spec":{"template":{"spec":{"nodeSelector":{"io.cilium/aws-node-enabled":"true"}}}}}'

    ```
    - set your API_SERVER_IP and API_SERVER_PORT by using `kubectl cluster-info`

```bash

helm repo add cilium https://helm.cilium.io/

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
    --set clusterName=my-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=$AGW_AWS_REGION \
    --set vpcId=$AGW_VPC_ID

    kubectl get deployment -n kube-system aws-load-balancer-controller
```


#### Deploy ACK Controller for API Gateway
```bash
    export HELM_EXPERIMENTAL_OCI=1
    export SERVICE=apigatewayv2
    export RELEASE_VERSION=v0.0.2
    export CHART_REPO=oci://public.ecr.aws/aws-controllers-k8s
    export CHART_REF="$CHART_REPO/$SERVICE --version $RELEASE_VERSION"

```

#### VPC Link - private integration with AWS VPC ***K8S CRD***

#### API Gateway - to make the application accessible from the internet ***K8S CRD***
    - Install the ACK for AWS API Gateway using helm
    - Use the service account name for API Gateway created using eksctl

#### NLB (Network Load Balancer) - to route traffic to the Gateway API (Ingress Controller) ***K8S CRD***
    - Install the ACK for AWS Load Balancer using helm
    - Use the service account name for API Gateway created using eksctl

#### Gateway API (Cilium) (North/South Trafffic) - accepting traffic into the cluster, create using Helm.
    - Install Gateway API CRDs
    - Gateway
    - HTTPRoute --> Service

#### GAMA East/West Traffic