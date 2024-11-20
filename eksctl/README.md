#### Prerequisite: Install eksct, kubectl, aws-cli

#### Create Kubernetes Cluster with EKSCTL

```bash 
eksctl create cluster -f cluster.yml
```
#### Roles & Policies

#### Cilium CNI

#### API Gateway - to make the application accessible from the internet ***K8S CRD***

#### NLB (Network Load Balancer) - to route traffic to the Gateway API (Ingress Controller) ***K8S CRD***

#### Gateway API (Cilium) (North/South Trafffic) - accepting traffic into the cluster.

#### GAMA East/West Traffic