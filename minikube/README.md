## MiniKube
[MiniKube Docs](https://minikube.sigs.k8s.io/docs/)

*Create local kubernetes cluster using Containers or Virtual machines
*

***Prequirest***: First install a “driver”
Docker or VirtualBox

*Minikube needs to create a virtual Linux environment, minikube creates a Linux Box (either a VM or a Container) with any of the virtualization technologies available in your local machine.*

***Install MiniKube***

***macOS***:
```sh
brew install minikube
```

***Windows***:
```bash
choco install minikube
```

***Start your cluster***

*If you don’t specify any configuration or flag, minikube will chose the best driver options for you.*
```bash
minikube start
```

*List all minikube clusters created on the local machine*
```bash
minikube profile list
```

***Stop/delete/clean up minikube***
```bash
minikube stop
Minikube stop -all
Minikube delete -all
Minikube delete -all -purge
```

***HA cluster***
[Multiple Control Plane HA Cluster](https://minikube.sigs.k8s.io/docs/tutorials/multi_control_plane_ha_clusters/)

```bash
minikube start --ha --driver=docker --container-runtime=containerd --profile ha-demo
```

***Add a worker node***
```bash
minikube node add -p ha-demo
```