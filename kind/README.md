## Create Kubernetes Cluster locally using Kind.
[Kind Docs](https://kind.sigs.k8s.io/docs/user/quick-start/)

*Kind (kubernetes in Docker): a tool that’s used to run kubernetes cluster using Docker container nodes.*

> On the backend, it spins up some Docker containers, each of these Docker containers are treated as a separate node, the nodes can be used as control plane or worker node. 

***Prerequisite***: have Go and Docker installed locally. 
You can install Kind locally with package managers:

***Windows***: 
```bash
choco install kind
```
***macOS***:
```bash
brew install kind
````
***Create cluster***:
```bash
kind create cluster
```
With optional flags: image or name of the cluster:
> --image flag – kind create cluster --image=... 

> --name flag – kind create cluster --name kind-2

More usage can be discovered with 
```bash
kind create cluster --help
```
When you list your kind clusters, you will see something like the following:

```bash
kind get clusters
kind
kind-2
```
***Deleting a Cluster***
```sh
kind delete cluster
```
*If the flag --name is not specified, kind will use the default cluster context name kind and delete that cluster.*

***[Configuring Your kind Cluster 🔗︎](https://kind.sigs.k8s.io/docs/user/quick-start/#configuring-your-kind-cluster)***

```sh
kind create cluster --config kind-example-config.yaml
```

***[Multi-node clusters 🔗︎](https://kind.sigs.k8s.io/docs/user/quick-start/#multi-node-clusters)***
```yaml
# three node (two workers) cluster config

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
    - role: control-plane
    - role: worker
    - role: worker
```

***Control-plane HA***
```yml
# a cluster with 3 control-plane nodes and 3 workers
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
    - role: control-plane
    - role: control-plane
    - role: control-plane
    - role: worker
    - role: worker
    - role: worker
```