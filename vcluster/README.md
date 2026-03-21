# VIND (vCluster in Docker)

Run virtual Kubernetes Clusters as Docker containers, these Clusters are pretty lightweight in nature. Use for Dev, Testing and CI/CD pipelines purposes.

### Hows it different from KIND if they both allow you run local Kubernetes clusters?

- vCluster is very feature rich:
  - You can Pause and Resume your Kubernetes clusters any time unlike KIND
  - It has a built in UI
  - Comes with a built-in loadBalancer service type, you get an external IP with which you can access services in the Cluster.
  - Supports Docker natively, if you have images on your machine, you don't have to push those images to a Docker registry to use the images within your Kubernetes cluster.
  - Ability to connect your local Kubernetes cluster with external Nodes(e.g., join an EC2 instances as one of the nodes of your Kubernetes cluster), helps in scaling your local Kubernetes Cluster.

[vCluster Repo](https://github.com/loft-sh/vcluster)
[vCluster](https://www.vcluster.com/install)

Setup a fully functional Kubernetes cluster within a docker container.
Traditional approach to setting up Kubernetes clusters are time consuming and resource intensive processes.

**Prerequisites**
Have Docker running on your machine and Kubectl installed on your machine

[vCluster Docs](https://www.vcluster.com/docs)

- Works with any Kubernetes distro - cloud, on-prem, or private (EKS, GKE, Bare Metal)
- Choose a Tenancy Model(how you want to isolate workloads within your cluster): Pick a tenancy mode and drop it into a vcluster.yaml file (Shared Nodes, Dedicated Nodes, Private Nodes)

```bash
# vcluster.yaml (optional for shared nodes)
sync:
  fromHost:
    nodes:
      # set to true for real node specs
      enabled: false
```

- Deploy vCluster: Install the CLI and launch your virtual cluster in seconds.

```bash
# Install vCluster CLI
brew install loft-sh/tap/vcluster
vcluster --version

# Tell vCluster to use Docker as a driver
vcluster use driver docker

# To use the UI
vcluster platform start

# Deploy vCluster
sudo vcluster create vc-tenant-1


# The -f vcluster.yaml flag is optional — use it only when you need a custom setup. Without it, vCluster will create a cluster with defaults.

# values.yaml
experimental:
      docker:
            nodes:
            - name: "worker-1"
              ports:
              - "9090:9090"
            - name: "worker-2"
              volumes:
              - "/tmp/data:/data"
              env:
              - "NODE_ROLE=worker"

# Make sure volume location is present on the Host
mkdir -p /tmp/data

# Deploy cluster with custom setup
sudo vcluster create vc-tenant-1  --values values.yaml

# Make Changes
vcluster create --upgrade vc-tenant-1 -f vcluster.yaml
# If you make any changes to your vcluster.yaml, you can use the --upgrade flag to upgrade the running virtual cluster and apply your changes.
```

### Deploy vCluster AddOns

[Addons](https://www.vcluster.com/docs/vcluster/configure/vcluster-yaml/deploy)

**CNI**
vCluster installs Flannel by default, but you can install your own CNI with the following:

```bash
# Disable default Flannel CNI on vcluster.yaml
deploy:
  cni:
    flannel:
      enabled: false
```

**Metrics Server**
vCluster can install metrics server into the cluster. This can be enabled via:

```bash
# Enable Metrics Server on vcluster.yaml
deploy:
  metricsServer:
    enabled: true
```

### Pause and Resume Cluster

```bash
# Pause Cluster
vcluster pause <name-of-cluster>

# Resume Cluster
vcluster resume <name-of-cluster>
```

### Adding external node to vCluster

```bash
sudo vcluster create my-cluster \
--set privateNodes.vpn.enable=true
--set privateNodes.vpn.nodeToNode.enable=true

# copy
vcluster token create

# Create a basic EC2 instance, ssh into the instance and paste the output of vcluster token create to join the instance to the cluster.
```
