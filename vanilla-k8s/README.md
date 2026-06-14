**CKA Objectives**: **_Typical administrator tasks encountered on the job, more specifically, Kubernetes cluster maintenance, networking, storage solutions, and troubleshooting applications and cluster nodes_**.

**_The following overview lists the high-level sections, or domains, of the exam and their scoring weights_**:

- 25%: Cluster Architecture, Installation, and Configuration:
  - This section dives deep into fundamental architecture of K8s, including the distinction between the control plane and worker nodes, high-availability configurations, and tools needed for installing, upgrading, and maintaining a cluster. You will learn practical skills such as installing a cluster from scratch, upgrading its version, and backing up/restoring the etcd database.
    **_The CNCF has also incorporated related topics into this domain. For instance, mastering role-based access control (RBAC) is essential for administrators to effectively manage access to cluster resources. You’ll also become proficient in installing Kubernetes operators and using tools like Kustomize and Helm to discover and deploy cluster components efficiently._**
- 15%: Workloads and Scheduling:
  - Administrators must have a solid understanding of Kubernetes concepts essential for managing cloud native applications. The domain focuses on these critical aspects. It covers key resources such as Deployments, ReplicaSets, and configuration management using ConfigMaps and Secrets.
    **_When a new Pod is created, the Kubernetes scheduler assigns it to an available node based on predefined criteria. Scheduling rules, such as node affinity and taints/tolerations, help fine-tune this process to meet specific requirements._**
- 20%: Servicing and Networking:
  - A cloud native microservice rarely operates in isolation. More often than not, it interacts with other microservices or external systems. For administrators, understanding Pod-to-Pod communication, exposing applications to external clients, and configuring cluster networking is crucial for maintaining a fully functional system.
    **_This domain of the exam evaluates your knowledge of essential Kubernetes networking primitives, including Services, Ingress, NetworkPolicy, and the Gateway API._**
- 10%: Storage:
  - This domain focuses on the various types of volumes used for reading and writing data in Kubernetes. As an administrator, you must understand how to create, configure, and manage these volumes effectively.
    **_PersistentVolumes (PVs) play a crucial role in ensuring data persistence, even after a cluster node restarts. You’ll need to demonstrate the ability to mount a PersistentVolume to a specific path within a container and understand the underlying mechanics. Additionally, it’s essential to grasp the differences between static and dynamic provisioning to manage storage resources efficiently._**
- 30%: Troubleshooting:
  - In production Kubernetes clusters, issues are bound to arise. Applications may misbehave, become unresponsive, or even become completely inaccessible. Additionally, cluster nodes might crash or face configuration problems. Developing effective troubleshooting strategies is critical to quickly identifying and resolving these situations to minimize downtime and disruption.

### Involved Kubernetes Primitives

![Kubernetes Primitives](../assets/cka2_0102.png)

**Documentation**
During the exam, you are permitted to open a well-defined list of web pages as a reference. You can freely browse those pages and copy-paste code to the exam terminal. The official Kubernetes documentation includes the reference manual and the blog. In addition, you can also browse the Helm documentation:

[Reference manual](https://kubernetes.io/docs)
[Blog](https://kubernetes.io/blog)
[Helm](https://helm.sh/docs)

**kubectl** - (for interacting with the Kubernetes cluster)

**kubeadm** - (for installing a Kubernetes cluster from scratch and upgrading the Kubernetes version of an existing cluster)

**etcdctl and etcdutl** - (for backing up and restoring the etc database)
Static Pods

**vi or vim** - text editor in linux
**grep, awk, sed\*** - text processing in linux
**cat, less, head, tail** - file operations
**netstat, curl, wget** - basic networking commands

**Setting a Context and Namespace**
The exam environment comes with multiple Kubernetes clusters already set up for you.
Each of the exam tasks need to be solved on a designated cluster.

```bash
kubectl config set-context <context-of-question> \
  --namespace=<namespace-of-question>

kubectl config use-context <context-of-question>

# Using the Alias for kubectl
alias k=kubectl
k version

# Using kubectl Command Auto-Completion

```

**Control Plane Node Components**
The control plane node requires a specific set of components to perform its job. The following list of components will give you an overview:

- API server: The API server exposes the API endpoints clients use to communicate with the Kubernetes cluster. For example, if you execute the tool kubectl, a command-line based Kubernetes client, you will make a RESTful API call to an endpoint exposed by the API server as part of its implementation. The API processing procedure inside of the API server will ensure aspects like authentication, authorization, and admission control.
- Schedular: The scheduler is a background process that watches for new Kubernetes Pods with no assigned nodes and assigns them to a worker node for execution
- Controller manager: The controller manager watches the state of your cluster and implements changes where needed. For example, if you make a configuration change to an existing object, the controller manager will try to bring the object into the desired state.
- etcd: Cluster state data needs to be persisted over time so it can be reconstructed upon a node restart or even a full cluster restart. That’s the responsibility of etcd, an open source software Kubernetes integrates with. At its core, etcd is a key-value store used to persist all data related to the Kubernetes cluster.

**Common Node Components**:
Kubernetes employs components that are leveraged by all nodes independent of their specialized responsibility:

- Kubelet: The kubelet runs on every node in the cluster; however, it makes the most sense on a worker node. This is because the control plane node usually doesn’t execute workload, and the worker node’s primary responsibility is to run workload. The kubelet is an agent that makes sure that the necessary containers are running in a Pod. You could say that the kubelet is the glue between Kubernetes and the container runtime engine and ensures that containers are running and healthy.
- Kube-proxy: The kube-proxy is a network proxy that runs on each node in a cluster to maintain network rules and enable network communication. In part, this component is responsible for implementing the Service concept.
- Container runtime: the container runtime is the software responsible for managing containers. The kubelet can be configured to choose from a range of different container runtime engines. While you can install a container runtime engine on a control plane, it’s not necessary, as the control plane node usually doesn’t handle workload.

**API Primitives and Objects**
Every Kubernetes primitive follows a general structure, which you can observe if you look deeper at a manifest of an object.
![API Primitives](../assets/cka2_0302.png)

**Using kubectl**
kubectl is the primary tool for interacting with the Kubernetes clusters from the command line. The exam is exclusively focused on the use of kubectl.

```bash
kubectl [command] [TYPE] [NAME] [flags]
```

The command specifies the operation you’re planning to run. Typical commands are verbs like create, get, describe, or delete. Next, you’ll need to provide the resource type you’re working on, either as a full resource type or its short form. For example, you could work on a service here or use the short form, svc.
The name of the resource identifies the user-facing object identifier, effectively the value of metadata.name in the YAML representation.
Finally, you can provide zero to many command-line flags to describe additional configuration behavior. A typical example of a command-line flag is the --port flag, which exposes a Pod’s container port.

![kubectl](../assets/cka2_0303.png)

A crucial time-saving tool during the exam is kubectl explain, which provides instant access to resource specifications without needing to search documentation. For example, kubectl explain pods.spec.containers shows all available container configuration fields, while kubectl explain deployment.spec.strategy.rolling​Up⁠date details rolling update parameters.

**Imperative Object Management**
Imperative object management does not require a manifest definition. You’ll use kubectl to drive the creation, modification, and deletion of objects with a single command and one or many command-line options.

**_Creating objects_**
Use the run or create command to create an object on the fly. Any configuration needed at runtime is provided by command-line options.

```bash
# The following run command creates a Pod named frontend that executes the container image nginx:1.29.0 in a container with the exposed port 80:
kubectl run frontend --image=nginx:1.29.0 --port=80
```

**_Updating objects_**
The configuration of live objects can still be modified. kubectl supports this use case by providing the edit and patch commands.

```bash
kubectl edit pod frontend
```

**_Deleting objects_**
You can delete a Kubernetes object at any time. During the exam, the need may arise if you made a mistake while solving a problem and want to start from scratch to ensure a clean slate.
Upon execution of the delete command, Kubernetes tries to delete the targeted object gracefully so that there’s minimal impact on the end user.
During the exam, end-user impact is not a concern. The most important goal is to complete all tasks in the time granted to the candidate. Therefore, waiting on an object to be deleted gracefully is a waste of time. You can force an immediate deletion of an object with the command-line option --now.

```bash
#The following command terminates the Pod named nginx using a SIGKILL signal:
$ kubectl delete pod nginx --now
```

### Creating and Managing a Kubernetes Cluster

Set up the underlying infrastructure for installing a Kubernetes cluster:
**_The infrastructure required by Kubernetes cluster nodes consist of servers, networks, and storage._**

- 3 EC2 (Control Plane and 2 Worker nodes) and 2 different Security Groups
- Check the Kubernetes documentation for the minimum CPU and Memory requirements

### What applications do you need to install

- Deploy a container runtime on every node(Control Plane / Worker), so that containers can be scheduled on them.
- Install the Kubelet application also on every node, which will basically run as a regular Linux process, just like the container runtime. They will both be installed from a package repository, just like we install any application on Linux servers.
- With the Container runtime and Kubelet in place, we can now deploy Pods for other K8s components.
  - On the Control plane nodes (We need to now deploy pods that run the master applications):
    - Api Server, Scheduler, Controller Manager, and ETCD applications will all be deployed as Pods on the Master nodes
  - The Kube Proxy application will be deployed on every node and runs as a Pod.
  - There are two things we need in order to deploy these Pod applications:
    - Since all these applications are pods, we need the Kubernetes manifest files for each application.
    - We must also make sure these applications are deployed securely: This means unauthorized users shouldn’t be able to access these applications, and the applications should talk to each other in an encrypted way.

**Chicken and Egg Situation**: All Master components will be deployed as Pods, but we need the Master components to deploy Pods.

- Because when deploying a Pod, we need to send a request to the API server > the Scheduler component will decide where to run the Pod > and the Pod data will be written into etcd.
- As you see, when scheduling one Pod, all the master components are involved, so how do we schedule these components without having them in the cluster? So we have the chicken and the egg problem; the answer to this is Static Pods.
- Static Pods are just like any other pods, but they are directly scheduled by the kublet without the need for API server, scheduler, or etcd.
- Once we have the Container Runtime and Kublet running, we can actually schedule these static pods.

### Regular Pod Scheduling:

- API Server gets the request > Scheduler (decides which node to place the Pod) > API server contacts Kubelet on the Node Scheduler selected to schedule the Pod.

### Static Pod Scheduling:

- Kubelet schedules Pod: How does it happen? Kubelete continuously watches a specific location on the Node it is running on. That location is /etc/kubernetes/manifests. It watches the folder for any Kubernetes manifest files. It schedules Pods as static Pods with no master processes required.
- Kubelet (Not Controller Manager) watches static Pod and restarts if it fails
- Pod names are suffixed with the node hostname
  **_This all means that when installing a Kubernetes cluster, we would first need to generate static Pod manifests for the API Server, Controller Manager, Scheduler, and etcd applications, and put these manifest files in the “/etc/kubernetes/manifests” folder, where Kubelet would find them and schedule them._**

### Certificates

When deploying these applications, we need to ensure that they will run securely. The question is, how do they talk to each other securely, and how can each component identify the other components, and also establish a mutual TLS communication so that communication between them is encrypted? We need certificates for that.

Who talks to whom in a Kubernetes cluster? And what certificate do we need for that?

- The API server is at the center of everything and all the communication. Almost every other component will talk to the API server; every component that wants to talk to the API server will need to provide a certificate to be able to authenticate itself with the API server.
- The way it works is, we need to first generate a self-signed CA certificate for Kubernetes (“cluster root CA”), which we will use to sign all the client and server certificates for each component in the cluster.
- These certificates are stored in the “/etc/kubernetes/pki” folder. The API server will have a server certificate, and the Scheduler and Controller Manager will both have their client certificate to talk to the API server. Same way API server talks to etcd and kubelet applications, which means etcd and kubelet will need their own server certificates, and since in this case API server is the client, it will need its own client certificate.
- PKI certificates for Authentication. Each component gets a certificate, signed by the same certificate authority.
- There’s one more client that we need to give a certificate or cluster access to, and that’s ourselves as Admins of the cluster, because we as Admins also need to talk to the server to administer it. All the queries and updates in the cluster go through the API server, and this means we also need our own client certificate for the Admin user to authenticate to the API server. To be valid and accepted by the API server, this certificate also needs to be signed by the CA that we created in Kubernetes.

This entire process is complex and time-consuming when done manually. The command-line tool Kubeadm bootstraps all these, providing “fast paths” for creating K8s clusters.

- It cares only about bootstrapping, not about provisioning machines.
  [Use the Kubernetes documentation as a guide to set up a cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

- Once the underlying infrastructure is provisioned, we need to check our prerequisites and configure a couple of things on those servers so that Kubernetes can work on them properly after installation.
- 2 GB or more RAM per machine (any less will leave little room for your apps).
- 2 CPUs or more for control plane machines.
- All machines must be on the same network
- Unique hostnames, Mac address, and product_uuid for every node.
- Your firewall should allow certain ports.
- Swap disabled. You must disable swap in order for kubelet to work properly.

### Swap configuration:

- The default behavior of a kubelet is to fail to start if swap memory is detected on a node. This means that swap should either be disabled or tolerated by kubelet on every server(node).
  - To disable swap, sudo swapoff -a can be used to disable swapping temporarily. To make this change persistent across reboots, make sure swap is disabled in config files like /etc/fstab, systemd.swap, depending how it was configured on your system.
- [List of ports to be allowed on the different nodes (Control Plane / Worker nodes)](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)

Control plane

| Protocol | Direction | Port Range | Purpose                 | Used By              |
| -------- | --------- | ---------- | ----------------------- | -------------------- |
| TCP      | Inbound   | 6443       | Kubernetes API server   | All                  |
| TCP      | Inbound   | 2379-2380  | etcd server client API  | kube-apiserver, etcd |
| TCP      | Inbound   | 10250      | Kubelet API             | Self, Control plane  |
| TCP      | Inbound   | 10259      | kube-scheduler          | Self                 |
| TCP      | Inbound   | 10257      | kube-controller-manager | Self                 |

Although etcd ports are included in the control plane section, you can also host your own etcd cluster externally or on custom ports.

| Protocol | Direction | Port Range  | Purpose            | Used By              |
| -------- | --------- | ----------- | ------------------ | -------------------- |
| TCP      | Inbound   | 10250       | Kubelet API        | Self, Control plane  |
| TCP      | Inbound   | 10256       | kube-proxy         | Self, Load balancers |
| TCP      | Inbound   | 30000-32767 | NodePort Services† | All                  |
| UDP      | Inbound   | 30000-32767 | NodePort Services† | All                  |

Default port range for [NodePort Services](https://kubernetes.io/docs/concepts/services-networking/service/)

All default port numbers can be overridden. When custom ports are used those ports need to be open instead of defaults mentioned here.

One common example is API server port that is sometimes switched to 443. Alternatively, the default port is kept as is and API server is put behind a load balancer that listens on 443 and routes the requests to API server on the default port.

- Optimize the process by giving the servers(nodes) a more human readable names.

```bash
sudo /etc/hosts
Map Private IP address(172.31.44.219) - name(Master)
Make an entry for all nodes


Sudo hostnamectl set-hostname control_plane

```

### Install a container runtime on every node

Install Containerd using Docker’s apt repository.

Create a Bash script to make this easier. (install.sh)

Make the file executable; chmod u+x install.sh

```bash
# Add Docker's official GPG key:
sudo apt -y update
sudo apt -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc


# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF


sudo apt -y update


sudo apt -y install containerd.io
sudo systemctl --no-pager status containerd.service
sudo systemctl is-active containerd.service


# Check if cri is disabled
cat /etc/containerd/config.toml


# Remove config
sudo rm /etc/containerd/config.toml


sudo mkdir -p /etc/containerd && sudo containerd config default | sudo tee /etc/containerd/config.toml


#Configure containerd to use systemd cgroup driver for compatibility with kubeadm:
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml


sudo systemctl restart containerd
sudo systemctl status containerd --no-pager
#Verify CRI is working
sudo crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock version


#Enable IPV4 packet forwarding
#To manually enable IPV4 packet forwarding
# sysctl params required by setup, params persist across reboots


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF


# Apply sysctl params without reboot
sudo sysctl --system


#Verify that net.ipv4.ip_forward is set to 1 with:
sysctl net.ipv4.ip_forward

```

### Install Kubernetes Processes

- Install kubeadm and kubelet on each Node, kubectl on the control-plane node.
- Execute kubeadm init (once): this is to initialize a control-plane Node, which will orchestrate the whole Kubernetes setup.
  - It will generate /etc/kubernetes folder
  - Generates a self-signed CA to set up identities for each component
  - Generates static Pod manifest files into /etc/kubernetes/manifests
  - Makes all necessary configurations
  - Kubelet will detect manifest files and start the Pods with the suffix of the hostname
  - Kubeadm does not install or manage kubelet for you
  - Before the cluster is initialized, first install Kubelet: it starts pods and containers
  - When we create a cluster, we need to interact with it to administer it, so we install a Kubernetes client command line tool, Kubectl.

**_These instructions are for Kubernetes v1.34._**

```bash
#1. Update the apt package index and install packages needed to use the Kubernetes apt repository:
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg


#2. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories, so you can disregard the version in the URL:
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.


# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


#3. Add the Kubernetes apt repository.
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list


echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


#4. Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


#5. (Optional) Enable the kubelet service before running kubeadm:
sudo systemctl enable --now kubelet
```

To initialize the control-plane node run:

```bash
sudo kubeadm init <args>
```

Kubeadm init phases [preflight, certs, kubeconfig, kubelet-start, control-plane]

- Addons: kubeadm installs a DNS server (CoreDNS) and the kube-proxy addon components via the API server.

Kubelete runs as a system process and lives in the /var/lib/kubelet/ folder.

### How do we access the cluster once configured?

Kubeconfig & kubectl: Connect to cluster

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### [Kubectl auto completion](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#enable-shell-autocompletion) and set an alias

```bash
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc


source ~/.bashrc
```

Regenerate the command to join nodes to the cluster.

```bash
kubeadm token create –print-join-command
```

### Pod Networking

- Every Pod has a unique IP address across the cluster.
- IP address reachable from all other Pods in the K8s cluster
- Kubernetes doesn’t have an in-built solution for pod networking
- The cluster operator is expected to implement a networking solution (CNI), Container Networking Interface, eg, Cilium, flannel, etc.

### [Install Cilium as CNI and Kube-proxy replacement](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/)

- Install the Cilium CLI

```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum

sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin

rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}

```

### [Initializing Cluster without Kube-proxy and installing Cilium using Helm](https://docs.cilium.io/en/stable/network/kubernetes/kubeproxy-free/)

### List resources that are cluster-scoped or namespaced

```bash
Kubectl api-resources –namespaced=false
```

### Explore available fields for every Kubernetes resource with the explain command.

As a parameter, provide the JSONPath of the object you’d like to render details for.

```bash
​​kubectl explain pods.spec
```

### DNS

Kubelet supplies each Pod with the DNS server location at /etc/resolv.conf
Configuration for kubelet is located at /var/lib/kubelet/config.yaml

- Service fully qualified domain name(FQDN):
  <servicename>.<namespace>.svc.cluster.local
  nginx-svc.default.svc.cluster.local

- Configure Service IP Address:
  - How Service Names are mapped to IP Addresses:
    Services IP address range is defined in Kube API Server Configuration located at /etc/kubernetes/manifests/kube-apiserver.yaml
    --service-cluster-ip-range=10.96.0.0/12
    These are internal service IP Addresses that only the applications in the cluster can use to access the services.
  - Kubeadm provisions and creates the Kube API server configuration, so how does Kubeadm actually get the Service AIP Address range?
    If we look at the kubeadm init --help command and it's options, one of the options we see is --service-cidr and its default range is "10.96.0.0/12", you can also specify an alternative IP Address range.
  - Check for the Kubeadm default configuration values:
    kubeadm config print init-defaults
    Gives you a list of all the default values kubeadm will initialize your cluster with.

### Upgrading a Cluster Version

Over time, you will want to upgrade the Kubernetes version of an existing cluster to pick up bug fixes and new features. The upgrade process has to be performed in a controlled manner to avoid the disruption of workload currently in execution and to prevent the corruption of cluster nodes.
**_It is recommended to upgrade from a minor version to a next higher one (e.g., from 1.31.0 to 1.32.0), or from a patch version to a higher one (e.g., from 1.31.0 to 1.31.3). Abstain from jumping up multiple minor versions to avoid unexpected side effects._**

- Determine upgrade version, Install new kubeadm version, kubeadm upgrade plan, kubeadm upgrade apply, Drain workload on node, Upgrade kubelet and kubectl, Restart kubelet process, Allow new workload on node.

**Upgrading Control Plane Nodes**

```bash
sudo apt update
sudo apt-cache madison kubeadm

# Upgrade kubeadm to a target version. Say you’d want to upgrade to version 1.31.5-1.1.
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install \
  -y kubeadm=1.31.5-1.1 && sudo apt-mark hold kubeadm

# Check which versions are available to upgrade to and validate whether your current cluster is upgradable.
sudo kubeadm upgrade plan

sudo kubeadm upgrade apply v1.31.5

# Drain the control plane node by evicting the workload. Any new workload won’t be schedulable on the node until uncordoned:
kubectl drain kube-control-plane --ignore-daemonsets

# Upgrade the kubelet and the kubectl tool to the same version:
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo \
  apt-get install -y kubelet=1.31.5-1.1 kubectl=1.31.5-1.1 && sudo apt-mark \
  hold kubelet kubectl

# Restart the kubelet process:
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Reenable the control plane node so that the new workload can become schedulable:
kubectl uncordon kube-control-plane

# The control plane node should now show the usage of Kubernetes 1.31.5:
kubectl get nodes


```

**Upgrading Worker Nodes**

```bash
# Upgrade kubeadm to a target version. This is the same command you used for the control plane node
sudo apt-mark unhold kubeadm && sudo apt-get update && sudo apt-get install \
  -y kubeadm=1.31.5-1.1 && sudo apt-mark hold kubeadm

# Upgrade the kubelet configuration:
sudo kubeadm upgrade node

# Drain the worker node by evicting the workload. Any new workload won’t be schedulable on the node until uncordoned:
kubectl drain kube-worker-1 --ignore-daemonsets

# Upgrade the kubelet and the kubectl tool with the same command used for the control plane node:
sudo apt-mark unhold kubelet kubectl && sudo apt-get update && sudo apt-get \
  install -y kubelet=1.31.5-1.1 kubectl=1.31.5-1.1 && sudo apt-mark hold kubelet \
  kubectl

# Restart the kubelet process:
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Reenable the worker node so that the new workload can become schedulable:
kubectl uncordon kube-worker-1
```

### Backing Up and Restoring etcd

Kubernetes stores both the declared and observed states of the cluster in the distributed etcd key-value store. It’s important to have a backup plan in place that can help you with restoring the data in case of data corruption. Backing up the data should happen periodically in short time frames to avoid losing as much historical data as possible.

**Using the etcd Administration Utilities**

The backup process stores the etcd data in a snapshot file. This snapshot file can be used to restore the etcd data at any given time. You can encrypt the snapshot file to protect sensitive information. The tool etcdctl is used to create a backup snapshot file. Restoring the etcd data from the snapshot file requires the use of the tool etcdutl.

**Backing Up etcd**
The describe command reveals the configuration of the etcd service. Look for the value of the option --listen-client-urls for the endpoint URL. In the following output, the host is localhost, and the port is 2379. The server certificate is located at /etc/kubernetes/pki/etcd/server.crt defined by the option --cert-file. The CA certificate can be found at /etc/kubernetes/pki/etcd/ca.crt specified by the option --trusted-ca-file:

Use the etcdctl command to create the backup with version 3 of the tool.
Provide the mandatory command-line options --cacert, --cert, and --key. The option --endpoints is not needed as we are running the command on the same server as etcd. After running the command, the file /opt/etcd-backup.db has been created:

```bash
sudo ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/etcd-backup.db
```

**Restoring etcd**
To restore etcd from the backup, use the etcdutl snapshot restore command. At a minimum, provide the --data-dir command-line option.

```bash
sudo ETCDCTL_API=3 etcdutl --data-dir=/var/lib/from-backup snapshot restore \
  /opt/etcd-backup.db

# Edit the YAML manifest of the etcd Pod, which can be found at /etc/kubernetes/manifests/etcd.yaml. Change the value of the attribute spec.volumes.hostPath with the name etcd-data from the original value /var/lib/etcd to /var/lib/from-backup:
```

### Authentication, Authorization, and Admission Control

The API server is the gateway to the Kubernetes cluster. Any human user, client (e.g., kubectl), cluster component, or service account will access the API server by making a RESTful API call via HTTPS. It is the central point for performing operations like creating a Pod or deleting a Service.

The first stage of request processing is authentication. Authentication validates the identity of the caller by inspecting the client certificates or bearer tokens. If the bearer token is associated with a service account, then it will be verified here.
![Processing an API Request](../assets/cka2_0601.png)

The second stage determines if the identity provided in the first stage can access the verb and HTTP path request. Therefore, stage two deals with authorization of the request, which is implemented with the standard Kubernetes RBAC model. Here, we ensure that the service account is allowed to list Pods or create a new Service object if that has been requested.

The third stage of request processing deals with admission control. Admission control verifies whether the request is well formed or potentially needs to be modified before the request is processed. An admission control policy could, for example, ensure that the request for creating a Pod includes the definition of a specific label. If the request doesn’t define the label, then it is rejected.

**Authentication with kubectl**
Developers interact with the Kubernetes API by running the kubectl command-line tool. Whenever you execute a command with kubectl, the underlying HTTPS call to the API server needs to authenticate.

**_The Kubeconfig_**
Credentials for the use of kubectl are stored in the file $HOME/.kube/config, also known as the kubeconfig file. The kubeconfig file defines the API server endpoints of the clusters we want to interact with, as well as a list of users registered with the cluster, including their credentials in the form of client certificates. The mapping between a cluster and user for a given namespace is called a context. kubectl uses the currently selected context to know which cluster to talk to and which credentials to use.

User management is handled by the cluster administrator. The administrator creates a user representing the developer and hands the relevant information (username and credentials) to the human wanting to interact with the cluster via kubectl. Alternatively, it is also possible to integrate with external identity providers for authentication purposes, e.g., via OpenID Connect.

**Managing Kubeconfig Using kubectl**
You do not have to manually edit the kubeconfig file(s) to change or add configuration. kubectl provides commands for reading and modifying its contents.

```bash
# To view the merged contents of the kubeconfig file(s), run the following command:
kubectl config view

# To render the currently selected context, use the current-context subcommand.
kubectl config current-context

# To change the context, provide the name with the use-context subcommand.
kubectl config use-context bmuschko

# To register a user with the kubeconfig file(s), use the set-credentials subcommand. We are choosing to assign the username myuser and point to the client certificate by providing the corresponding CLI flags:
kubectl config set-credentials myuser \
  --client-key=myuser.key --client-certificate=myuser.crt \
  --embed-certs=true


```

**Create user using x509 certificates**

```bash
# Log into the Kubernetes control plane node and create a temporary directory that will hold the generated keys. Navigate into the directory:
mkdir cert && cd cert

# 1. Create a private key using the openssl executable. Provide an expressive file name, such as <username>.key:
 openssl genrsa -out dev-tom.key 2048

# 2. Create a certificate sign request (CSR) in a file with the extension .csr. You need to provide the private key from the previous step. The -subj option provides the username (CN) and the group (O). The following command uses the username dev-tom and the group named cka-study-guide. To avoid assigning the user to a group, leave off the /O component of the assignment:
openssl req -new -key dev-tom.key -subj "/CN=tom/O=cka-study-guide" -out dev-tom.csr

# Lastly, sign the CSR with the Kubernetes cluster certificate authority (CA). The CA can usually be found in the directory /etc/kubernetes/pki and needs to contain the files ca.crt and ca.key. The following command signs the CSR and makes it valid for 364 days:

openssl x509 -req -in dev-tom.csr -CA /etc/kubernetes/pki/ca.crt -CAkey \
  /etc/kubernetes/pki/ca.key -CAcreateserial -out dev-tom.crt -days 364

  # OR

# 3. Send CSR using K8s Certificates API
kubectl apply -f - <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: dev-tom
spec:
  request:
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1V6Q0NBVHNDQVFBd0RqRU1NQW9HQTFVRUF3d0RkRzl0TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQwpBUThBTUlJQkNnS0N
BUUVBanQrY3g0YjYwK3QxcVZjcDRZVlEzTC9oTWJSNVdvRDFqdnk4M0EvdDlFMUJYMDcxCm9Fd1JIeVlVd2tmeUJBcFBiQm5BZlZEZjR3TGhYMDBXa2NtMXBiekp1S0EyM3IyTGFZUkxGR1MxTHk1VmpFNzAKKzhUO
VVpenU1c2EvZzE4VlJ2SWlXaCtiZzR5SHovWitFNE11eUppMFJkdUVTSERGMkFZbzVISW0veUIrL2hOaQp4bEpJSHVOMFFLeENqVlZCTlRiM0hxZVZYMUg3TWFPektWdzQ5WTJYWGdqM1hYMGFESDY3Ny8rZFlud2l
oYk1xClJzY1IyenZwbTlOL0RXQllNZWZDNWNOWnBKOGZIQ2RXMzlWc1NDNUkyNkliaVBnYy9SbnBpZjdoR1ZFQk5HT0YKa2lvazN0TGZ2cWRVZmhFdWpYQ2VNOEhocXAyL1JEYUxBekl6QVFJREFRQUJvQUF3RFFZS
ktvWklodmNOQVFFTApCUUFEZ2dFQkFEc040K29uUjdrWkNyeFRQODUvY2VhMTFscXQ5NEF5WFlBR2R5c1VBcXJneW5DYVlSY0pvZWUyCjNiZGpBRGtwWUNJM1JMQ0VDM0pVWnVpUlN1ZUtDQVRjQ0FTdk1QZVBHYlZ
IZTdhUCtlR1dzdndab0NQRnJPb3kKWDdtUDhpM2ZQaExZUnUzK0tPV2xNNGM1aThVQTFIVTZhTWJOT1A4b1hJS2t2bVZRaFBjTEtIMWxxZzlOOGZKeQpNUHFFMjFhbDM4L2JnMWQxbWxpUGZHYWVsQzdOdThELzFxW
VFxWVh4VEZzWk1vamNZRUVyVnhPT2paZjM1OWdBCkdWdkF6UEQvVmtPVnYxLzBHb0VCVFdNUFRjd1hQTUVtcEZzRWdteVZldkNqVUJDWmdpS0VGcHVmR1hyMzRldUkKN25xWWozbHZVbDA1c3ZPbHVWYlUrS1l3czR
DRWlwTT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400 # one day
  usages:
  - client auth

EOF

# 4. K8s signs certificates for you
kubectl get certificatesigningrequests.certificates.k8s.io

# 5. K8s admin approves certificate
kubectl certificate approve dev-tom

kubectl get csr dev-tom -o yaml

echo 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4ekNDQWR1Z0F3SUJBZ0lRTEtiN2RtRkdiU2FFS3hzOUVUM1ZpakFOQmdrcWhraUc5dzBCQVFzRkFEQVYKTV
JNd0VRWURWUVFERXdwcmRXSmxjbTVsZEdWek1CNFhEVEkwTURreU1qRTBOVEExTlZvWERUSTBNRGt5TXpFMApOVEExTlZvd0RqRU1NQW9HQTFVRUF4TURkRzl0TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FR
OEFNSUlCCkNnS0NBUUVBanQrY3g0YjYwK3QxcVZjcDRZVlEzTC9oTWJSNVdvRDFqdnk4M0EvdDlFMUJYMDcxb0V3Ukh5WVUKd2tmeUJBcFBiQm5BZlZEZjR3TGhYMDBXa2NtMXBiekp1S0EyM3IyTGFZUkxGR1MxTH
k1VmpFNzArOFQ5VWl6dQo1c2EvZzE4VlJ2SWlXaCtiZzR5SHovWitFNE11eUppMFJkdUVTSERGMkFZbzVISW0veUIrL2hOaXhsSklIdU4wClFLeENqVlZCTlRiM0hxZVZYMUg3TWFPektWdzQ5WTJYWGdqM1hYMGFE
SDY3Ny8rZFlud2loYk1xUnNjUjJ6dnAKbTlOL0RXQllNZWZDNWNOWnBKOGZIQ2RXMzlWc1NDNUkyNkliaVBnYy9SbnBpZjdoR1ZFQk5HT0ZraW9rM3RMZgp2cWRVZmhFdWpYQ2VNOEhocXAyL1JEYUxBekl6QVFJRE
FRQUJvMFl3UkRBVEJnTlZIU1VFRERBS0JnZ3JCZ0VGCkJRY0RBakFNQmdOVkhSTUJBZjhFQWpBQU1COEdBMVVkSXdRWU1CYUFGS00xLzJQdGZxSGJQL2VTTU1jZlYzZGwKV2s1cE1BMEdDU3FHU0liM0RRRUJDd1VB
QTRJQkFRQmRPcTAzOXIyQ1BuUERJUkg1ajZRZUhkVXVua2c1Uk1UYgp1NVNRR00zRno0NnVYV1haN3VyRXA1c0Z1NXNEQ3JHRHNqNGZOVGFCWGpzVVF1cjdBNndQY0pQYUtzUmVUNUhnCllIaEhtVjVBTWVEU3B4N2
p2VHhLQU92Q1Q2WXBsQUd3cjFNc0hwQ0RSY2grNm5yNSt3Q09wc2pPSjFlNG5JSTcKZ0ZxZ21zODJtbGk3UytZa3F2NHpMeFJMRWg4aVVMRVFKR2pVSnhKWFpjb202RHNlU2JDeVFRd00yN1ZPeUVwNApvaUtPaHNT
UkZoaDR6OVhCaWU5SkJ5MTdvVUJ2b2NicERTNVUyeFcxOXRZenhaVlVHZm1HcnYxRlhiVmZPaEFRClV2cnh5UlpoYnZydXl1WXJMb1UxUTVpdmxWTXFFOFlFdmJ1OTdBWWptZjByakM4UENwelcKLS0tLS1FTkQgQ0
VSVElGSUNBVEUtLS0tLQo=' | base64 --decode > dev-tom.crt


# 6. Create the user in Kubernetes by setting a user entry in kubeconfig for dev-tom. Point to the CRT and key file. Set a context entry in kubeconfig for dev-tom:
kubectl config set-credentials dev-tom \
  --client-certificate=dev-tom.crt --client-key=dev-tom.key

kubectl config set-context dev-tom-context --cluster=<cluster-name> \
  --user=dev-tom

# To switch to the user, use the context named dev-tom-context. You can check the current context using the command config current-context:

kubectl config use-context dev-tom-context

kubectl config current-context
```

**ServiceAccount**

A user represents a real person who commonly interacts with the Kubernetes cluster using the kubectl executable or the UI dashboard. Some service applications like Helm running inside of a Pod need to interact with the Kubernetes cluster by making requests to the API server via RESTful HTTP calls. For example, a Helm chart would define multiple Kubernetes objects required for a business application. Kubernetes uses a ServiceAccount to authenticate the Helm service process with the API server through an authentication token. This ServiceAccount can be assigned to a Pod and mapped to RBAC rules.

A Kubernetes cluster already comes with a ServiceAccount, the default ServiceAccount that lives in the default namespace. Any Pod that doesn’t explicitly assign a ServiceAccount uses the default ServiceAccount.

To create a custom ServiceAccount imperatively, run the create serviceaccount command:

```bash

# Imperative way to create a service account
kubectl create serviceaccount build-bot

# Declarative way to create a service account

kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot
EOF

# Assigning a ServiceAccount to a Pod imperatively
kubectl run build-observer --image=alpine --restart=Never \
  --serviceaccount=build-bot

# Alternatively, you can directly assign the ServiceAccount in the YAML manifest of a Pod, Deployment, Job, or CronJob using the field serviceAccountName.

kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: build-observer
spec:
  serviceAccountName: build-bot
...
EOF
```

### Authorization with Role-Based Access Control

We’ve learned that the API server will try to authenticate any request sent using kubectl by verifying the provided credentials. An authenticated request will then need to be checked against the permissions assigned to the requestor. The authorization phase of the API processing workflow checks if the operation is permitted against the requested API resource.

In Kubernetes, those permissions can be controlled using role-based access control (RBAC). In a nutshell, RBAC defines policies for users, groups, and service accounts by allowing or disallowing access to manage API resources. Enabling and configuring RBAC is mandatory for any organization with an emphasis on security.
Setting permissions is the responsibility of a cluster administrator.

**Manage Role-Based Access Control (RBAC)**

In Kubernetes, you need to be authenticated before you are allowed to make a request to an API resource.

**RBAC**: defines policies for users, groups, and processes by allowing or disallowing access to manage API resources.

**_RBAC consists of three key building blocks_**:
**Subject**: The user or process that wants to access a resource
**Resource**: The Kubernetes API resource type (e.g., a Deployment or node)
**Verb**: The operation that can be executed on the resource (e.g., creating a Pod or deleting a Service)

![RBAC building blocks](../assets/cka2_0602.png)

When you need to quickly determine which operations (verbs) are supported for a specific Kubernetes resource during the exam, kubectl api-resources -o wide is an invaluable command that displays all available API resources along with their supported verbs (like get, list, create, update, patch, watch, delete).

**_Users and groups are not stored in etcd, the Kubernetes database, and are meant for processes running outside of the cluster. Service accounts exist as objects in Kubernetes and are used by processes running inside of the cluster._**

**_Kubernetes does not represent a user as with an API resource. The user is meant to be managed by the administrator of a Kubernetes cluster, which then distributes the credentials of the account to the real person or to be used by an external process._**

Calls to the API server with a user need to be authenticated. Kubernetes offers a variety of authentication methods for those API requests.

| Authentication strategy  | Description                                                           |
| ------------------------ | --------------------------------------------------------------------- |
| X.509 client certificate | Uses an OpenSSL client certificate to authenticate                    |
| Basic authentication     | Uses username and password to authenticate                            |
| Bearer tokens            | Uses OpenID (a flavor of OAuth2) or webhooks as a way to authenticate |

The following steps demonstrate the creation of a user that uses an OpenSSL client certificate to authenticate. Those actions have to be performed with the cluster-admin Role object.

**Understanding RBAC API Primitives**
[Kubernetes API Groups Documentation](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/)

Kubernetes API primitives that implement the RBAC functionality:

**Role**
The Role API primitive declares the API resources and their operations this rule should operate on. For example, you may want to say “allow listing and deleting of Pods,” or you may express “allow watching the logs of Pods,” or even both with the same Role. Any operation that is not spelled out explicitly is disallowed as soon as it is bound to the subject.

**RoleBinding**
The RoleBinding API primitive binds the Role object to the subject(s). It is the glue for making the rules active. For example, you may want to say “bind the Role that permits updating Services to the user John Doe.”

![RBAC API Primitives](../assets/cka2_0603.png)

**Default User-Facing Roles**
Kubernetes defines a set of default Roles. You can assign them to a subject via a RoleBinding or define your own, custom Roles depending on your needs.

| Default ClusterRole | Description                                                                                                       |
| ------------------- | ----------------------------------------------------------------------------------------------------------------- |
| cluster-admin       | Allows read and write access to resources across all namespaces.                                                  |
| admin               | Allows read and write access to resources in namespace including Roles and RoleBindings.                          |
| edit                | Allows read and write access to resources in namespace except Roles and RoleBindings. Provides access to Secrets. |
| view                | Allows read-only access to resources in namespace except Roles, RoleBindings, and Secrets.                        |

To define new Roles and RoleBindings, you will have to use a context that allows for creating or modifying them, that is, cluster-admin or admin.

**Creating Roles**
Roles can be created imperatively with the create role command. The most important options for the command are --verb for defining the verbs aka operations, and --resource for declaring a list of API resources. The following command creates a new Role for the resources Pod, Deployment, and Service with the verbs list, get, and watch:

```bash
kubectl create role read-only --verb=list,get,watch \
  --resource=pods,deployments,services
```

The command-line option --resource-name spells out one or many object names that the policy rules should apply to. A name of a Pod could be nginx and listed here with its name. Providing a list of resource names is optional. If no names have been provided, then the provided rules apply to all objects of a resource type.

**Creating RoleBindings**
The imperative command creating a RoleBinding object is create rolebinding. To bind a Role to the RoleBinding, use the --role command-line option. The subject type can be assigned by declaring the options --user, --group, or --serviceaccount. The following command creates the RoleBinding with the name read-only-binding to the user called dev-tom:

```bash
kubectl create rolebinding read-only-binding --role=read-only --user=dev-tom
```

At any given time, you can check a user’s permissions with the auth can-i command. The command gives you the option to list all permissions or check a specific permission:

```bash
kubectl auth can-i --list --as dev-tom


kubectl auth can-i list pods --as dev-tom
```

**Namespace-wide and Cluster-wide RBAC**
Roles and RoleBindings apply to a particular namespace. You will have to specify the namespace at the time of creating both objects. Sometimes, a set of Roles and Rolebindings needs to apply to multiple namespaces or even the whole cluster. For a cluster-wide definition, Kubernetes offers the API resource types ClusterRole and ClusterRoleBinding. The configuration elements are effectively the same. The only difference is the value of the kind attribute:

- To define a cluster-wide Role, use the imperative subcommand clusterrole or the kind ClusterRole in the YAML manifest.
- To define a cluster-wide RoleBinding, use the imperative subcommand clusterrolebinding or the kind ClusterRoleBinding in the YAML manifest.

**Aggregating RBAC Rules**

Existing ClusterRoles can be aggregated to avoid having to redefine a new, composed set of rules that likely leads to duplication of instructions. For example, say you wanted to combine a user-facing role with a custom Role. An aggregated ClusterRole can merge rules via label selection without having to copy-paste the existing rules into one.

Say we defined two ClusterRoles. The ClusterRole list-pods allows for listing Pods and the ClusterRole delete-services allows for deleting Services.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: list-pods
  namespace: rbac-example
  labels:
    rbac-pod-list: "true"
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - list

  ---

  apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: delete-services
  namespace: rbac-example
  labels:
    rbac-service-delete: "true"
rules:
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - delete

# To aggregate those rules, ClusterRoles can specify an aggregationRule. This attribute describes the label selection rules.

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pods-services-aggregation-rules
  namespace: rbac-example
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac-pod-list: "true"
  - matchLabels:
      rbac-service-delete: "true"
rules: []

```

### Admission Control

The last phase of processing a request to the API server is admission control. Admission control is implemented by admission controllers. An admission controller provides a webhook to approve, deny, or mutate a request before it takes effect.

Admission controllers can be registered with the configuration of the API server. By default, the configuration file can be found at /etc/kubernetes/manifests/kube-apiserver.yaml. It is the cluster administrator’s job to manage the API server configuration. The following command-line invocation of the API server enables the admission control plug-ins named NamespaceLifecycle, PodSecurity, and LimitRanger:

```bash
kube-apiserver --enable-admission-plugins=NamespaceLifecycle,PodSecurity,LimitRanger
```

### Operators and Custom Resource Definitions (CRDs)

Kubernetes comes with a core feature set to fulfill the basic needs for running application stacks with a standard set of primitives. For custom use cases, Kubernetes allows for installing extensions to the platform, operators.

A Custom Resource Definition (CRD) is a Kubernetes extension mechanism (often bundled with an operator) for introducing custom API primitives to fulfill requirements not covered by built-in primitives.

**Working with Operators**
Operators extend the core behavior of the Kubernetes cluster without actually changing the Kubernetes code. You can think of the operator as a plug-in to the platform. Operators typically automate tasks that would have to be performed by humans, such as deploying, configuring, scaling, upgrading, and managing applications.

**The Operator Pattern**
An operator usually consists of multiple components: one or more CRDs, a controller, and often additional components like RBAC rules for permissions. CRDs can be understood as the schema that defines the blueprint for a custom object, and the instantiation of those objects with the newly introduced type, also called the Custom Resources (CR).

For a CRD to be useful, it has to be backed by a controller. Controllers interact with the Kubernetes API and implement the reconciliation logic that interacts with CRD objects.

The combination of CRDs and controllers is commonly referred to as the operator pattern.

A prominent operator is the External Secrets Operator that helps with integrating external Secret managers, like Amazon Web Services (AWS) Secrets Manager and HashiCorp Vault, with Kubernetes. Another is the Crossplane operator, which helps with creating and managing cloud resources using declarative syntax.

### Helm and Kustomize

Kubernetes objects can be created, modified, and deleted by using imperative kubectl commands or by running a kubectl command against a manifest file declaring the desired state of an object, called a manifest. The primary definition language of a manifest is YAML.
It’s recommended that development teams commit and push those manifest files to version control repositories, as it will help with tracking and auditing changes over time.

Modeling an application in Kubernetes often requires a set of supporting objects, each of which can have its own manifest. For example, you may want to create a Deployment that runs the application on five Pods, a ConfigMap to inject configuration data as environment variables, and a Service for exposing network access.

It is not practical to manage a full application stack by running individual kubectl commands. That is where open source tools like Helm and Kustomize come into play. They allow you conveniently manage the lifecycle of application stacks and cluster components as one single unit, while at the same time allowing for contextual parameter adjustment at the time of deployment.

**Working with Helm**
Helm is package manager for a set of Kubernetes manifests; it also provides a templating engine. At runtime, it replaces placeholders in YAML template files with actual, end-user-defined values. The artifact produced by the Helm executable is a chart file that bundles the manifests that comprise the API resources of an application in the form of a TAR file. You can upload the chart file to a chart repository so that other teams can use it to deploy the bundled manifests. The Helm ecosystem offers a wide range of reusable charts for common use cases searchable on Artifact Hub (for example, for running Grafana or PostgreSQL).

**Managing an Existing Chart**
As a developer, you want to reuse existing functionality instead of putting in the work to define and configure it yourself. For example, you may want to install the open source monitoring service Prometheus on your cluster.

Prometheus requires the installation of multiple Kubernetes primitives. Thankfully, the Kubernetes community provided a Helm chart, making it very easy to install and configure all the moving parts in the form of a Kubernetes operator.

```bash
# Add repository
helm repo add jenkinsci https://charts.jenkins.io/

# List repositories
helm repo list

# Searching for a Chart in a Repository, a repo can have many charts
helm search repo jenkinsci

# Install chart
helm install my-jenkins jenkinsci/jenkins --version 5.8.25

helm show values jenkinsci/jenkins

# Upgrading an Installed Chart
helm repo update

helm upgrade my-jenkins jenkinsci/jenkins --version 5.8.26

# Uninstalling a Chart
helm uninstall my-jenkins
```

**Working with Kustomize**
Kustomize is a tool introduced with Kubernetes 1.14 that aims to make manifest management more convenient. It supports three different use cases:

- Generating manifests from other sources. For example, creating a ConfigMap and populating its key-value pairs from a properties file.
- Adding common configuration across multiple manifests. For example, adding a namespace and a set of labels for a Deployment and a Service.
- Composing and customizing a collection of manifests. For example, setting resource boundaries for multiple Deployments.

The central file needed for Kustomize to work is the kustomization file. The standardized name for the file is kustomization.yaml and cannot be changed. A kustomization file defines the processing rules Kustomize works upon.

Kustomize is fully integrated with kubectl and can be executed in two modes: rendering the processing output on the console or creating the objects. Both modes can operate on a directory, tarball, Git archive, or URL as long as they contain the kustomization file and referenced resource files:

**_Rendering the produced output_**
The first mode uses the kustomize subcommand to render the produced result on the console but does not create the objects. This command works similarly to the dry-run option you might know from the run command:

```bash
kubectl kustomize <target>
```

**_Creating the objects_**
The second mode uses the apply command in conjunction with the -k command-line option to apply the resources processed by Kustomize, as explained in the previous section:

```bash
kubectl apply -k <target>
```

**Composing Manifests**

One of the core functionalities of Kustomize is to create a composed manifest from other manifests. Combining multiple manifests into a single one may not seem that useful by itself, but many of the other features described later will build upon this capability. Say you wanted to compose a single YAML definition with multiple manifests separated by “---” from a Deployment and a Service resource file. All you need to do is to place the resource files into the same folder as the kustomization file:

```bash
kustomization.yaml
web-app-deployment.yaml
web-app-service.yaml

# The kustomization file lists the resources in the resources section.

# A kustomization file combining two manifests
resources:
- web-app-deployment.yaml
- web-app-service.yaml

# As a result, the kustomize subcommand renders the combined manifest containing all of the resources separated by three hyphens (---) to denote the different object definitions

kubectl kustomize ./
```

### Workloads and Scheduling

The Workloads and Scheduling domain encompasses the foundational Kubernetes primitives used for deploying applications in enterprise environments. It also covers the key concepts and mechanisms that drive how workloads are efficiently admitted, scheduled, and distributed across worker nodes, ensuring optimal resource utilization and performance.

**Pods and Namespaces**
The most important primitive in the Kubernetes API is the Pod. A Pod lets you run a containerized application.
In addition to running a container, a Pod can consume other services like storage, configuration data, and much more.

**Creating Pods**
The run command is the central entry point for creating Pods imperatively.

```bash
kubectl run hazelcast --image=hazelcast/hazelcast:5.1.7 \
  --port=5701 --env="DNS_DOMAIN=cluster" --labels="app=hazelcast,env=prod"

  # The run command offers a wealth of command-line options. Execute the kubectl run --help

# Container-Level Restarts
#Every Pod provides container-level restart capabilities. If a container fails, the kubelet cluster component will restart it based on its configured restart policy.
#The restart policy of a Pod can be set using the attribute spec.restartPolicy. Possible values for this attribute include Always, OnFailure, and Never.

# Accessing Logs of a Pod
kubectl logs hazelcast
# You can stream the logs with the command-line option -f. This option is helpful if you want to see logs in real time.

# Upon a container restart, you won’t have access to the logs of the previous container; the logs command renders the logs only for the current container. However, you can still get back to the logs of the previous container by adding the -p command-line option.
```

**Executing a Command in Container**
Some situations require you to get the shell to a running container and explore the filesystem. Maybe you want to inspect the configuration of your application or debug its current state. You can use the exec command to open a shell in the container to explore it interactively.

```bash
kubectl exec -it hazelcast -- /bin/sh

# It’s also possible to execute a single command inside of a container. Say you wanted to render the environment variables available to containers without having to be logged in. Just remove the interactive flag -it and provide the relevant command after the two dashes:
kubectl exec hazelcast -- env

```

**Creating a Temporary Pod**
Under certain conditions, you’ll want to execute a command in a Pod just for troubleshooting. This use case doesn’t require a Pod object to run beyond the execution of the command. That’s where temporary Pods come into play.

Under certain conditions, you’ll want to execute a command in a Pod just for troubleshooting. This use case doesn’t require a Pod object to run beyond the execution of the command. That’s where temporary Pods come into play.

```bash
kubectl run busybox --image=busybox:1.36.1 --rm -it --restart=Never -- env
```

**Configuring Pods**
The curriculum expects you to feel comfortable with editing YAML manifests either as files or as live object representations.

**Declaring environment variables**
Applications need to expose a way to make their runtime behavior configurable. For example, you may want to inject the URL to an external web service or declare the username for a database connection. Environment variables are a common option to provide this runtime configuration.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: spring-boot-app
spec:
  containers:
  - name: spring-boot-app
    image: springio/gs-spring-boot-docker
    env:
    - name: SPRING_PROFILES_ACTIVE
      value: dev
    - name: VERSION
      value: '1.5.3'
```

**Deleting a Pod**
A graceful deletion operation can take anywhere from 5 to 30 seconds, time you don’t want to waste during the exam.

To save time during the exam, you can circumvent the grace period by adding the --now option to the delete command. Avoid using the --now flag in production Kubernetes environments.

**Working with Namespaces**
Namespaces are an API construct used to avoid naming collisions, and they represent a scope for object names. A good use case for namespaces is to isolate the objects by team or responsibility.

**Setting a Namespace Preference**
Providing the --namespace or -n command-line option for every command is tedious and error-prone. You can set a permanent namespace preference if you know that you need to interact with a specific namespace you are responsible for. The first command shown sets the permanent namespace code-red. The second command renders the currently set permanent namespace:

```bash
kubectl config set-context --current --namespace=code-red

kubectl config view --minify | grep namespace:
```

### ConfigMaps and Secrets

Kubernetes dedicates two primitives to defining configuration data: the ConfigMap and the Secret. Both primitives are completely decoupled from the lifecycle of a Pod, which enables you to change their configuration data values without necessarily having to redeploy the Pod.

In essence, ConfigMaps and Secrets store a set of key-value pairs. Those key-value pairs can be injected into a container as environment variables, or they can be mounted as a volume.
![ConfigMaps and Secrets](../assets/cka2_1001.png)

The ConfigMap and Secret may look almost identical in purpose and structure on the surface; however, there is a slight but significant difference. A ConfigMap stores plain-text data, for example, connection URLs, runtime flags, or even structured data like a JSON or YAML content. Secrets are better suited for representing sensitive data like passwords, API keys, or Secure Sockets Layer (SSL) certificates and store the data in Base64-encoded form.

**Working with ConfigMaps**
It’s not unusual that the same configuration data needs to be made available to multiple Pods. Instead of copy-pasting the same key-value pairs across multiple Pod definitions, you can choose to centralize the information in a ConfigMap object. The ConfigMap object holds configuration data and can be consumed by as many Pods as you want. Therefore, you will need to modify the data in only one location should you need to change it.

**Creating a ConfigMap**
You can create a ConfigMap by emitting the imperative create configmap command. This command requires you to provide the source of the data as an option. Kubernetes distinguishes the four different options.

| Option          | Example                     | Description                                                                       |
| --------------- | --------------------------- | --------------------------------------------------------------------------------- |
| --from-literal  | --from-literal=locale=en_US | Literal values, which are key-value pairs as plain text                           |
| --from-env-file | --from-env-file=config.env  | A file that contains key-value pairs and expects them to be environment variables |
| --from-file     | --from-file=app-config.json | A file with arbitrary contents                                                    |
| --from-file     | --from-file=config-dir      | A directory with one or many files                                                |

```bash
kubectl create configmap db-config --from-literal=DB_HOST=mysql-service \
  --from-literal=DB_USER=backend

```

**Consuming a ConfigMap as Environment Variables**
With the ConfigMap created, you can now inject its key-value pairs as environment variables into a container.
Use spec.containers[].envFrom[].configMapRef to reference the ConfigMap by name.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    envFrom:
    - configMapRef:
        name: db-config

# Inspect the environment variables available in the container by running the env Unix command:
kubectl exec backend -- env
```

**Mounting a ConfigMap as a Volume**
Another way to configure applications at runtime is by processing a machine-readable configuration file. Say we have decided to store the database configuration in a JSON file named db.json with the structure.

```bash
# A JSON file used for configuring database information

{
    "db": {
      "host": "mysql-service",
      "user": "backend"
    }
}

# Given that we are not dealing with literal key-value pairs, we need to provide the option --from-file when creating the ConfigMap object:

kubectl create configmap db-config --from-file=db.json

# ConfigMap YAML manifest defining structured data
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  db.json: |-                        1
    {
       "db": {
          "host": "mysql-service",
          "user": "backend"
       }
    }

# The multiline string syntax (|-) used in this YAML structure removes the line feed and removes the trailing blank lines.

# The Pod mounts the ConfigMap as a volume to a specific path inside of the container with read-only permissions. The assumption is that the application will read the configuration file when starting up.

apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    volumeMounts:
    - name: db-config-volume
      mountPath: /etc/config
  volumes:
  - name: db-config-volume
    configMap:                      1
      name: db-config

# To verify the correct behavior, open an interactive shell to the container.
kubectl exec -it backend -- /bin/sh
ls -1 /etc/config
cat /etc/config/db.json

# The application code can now read the file from the mount path and configure the runtime behavior as needed.
```

**Working with Secrets**
Data stored in ConfigMaps represents arbitrary plain-text key-value pairs. In comparison to the ConfigMap, the Secret primitive is meant to represent sensitive configuration data. A typical example for Secret data is a password or an API key for authentication.

**_Values stored in a Secret are only encoded, not encrypted_**
Secrets expect the value of each entry to be Base64-encoded. Base64 encodes a value, but it doesn’t encrypt it. Anyone with access to its value can decode it without problems. Therefore, storing Secret manifests in the source code repository alongside other resource files should be avoided.

**Creating a Secret**
You can create a Secret with the imperative command create secret. In addition, a mandatory subcommand needs to be provided that determines the type of Secret.

| CLI option      | Description                                                                                                           | Internal type           |
| --------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| generic         | Creates a Secret from a file, directory, or literal value                                                             | Opaque                  |
| docker-registry | Creates a Secret for use with a Docker registry, e.g., to pull images from a private registry when requested by a Pod | kubernetes.io/dockercfg |
| tls             | Creates a TLS Secret                                                                                                  | kubernetes.io/tls       |

To demonstrate the functionality, let’s create a Secret of type generic. The command sources the key-value pairs from the literals provided as a command-line option:

```bash
kubectl create secret generic db-creds --from-literal=pwd=s3cre!
```

**Consuming a Secret as Environment Variables**
Consuming a Secret as environment variables works similar to the way you’d do it for ConfigMaps. Here, you’d use the YAML expression spec.containers[].env⁠From​[].secretRef to reference the name of the Secret.

```bash
# Injecting Secret key-value pairs into the container

apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    envFrom:
    - secretRef:
        name: secret-basic-auth
```

**Remapping Environment Variable Keys**
Sometimes, key-value pairs stored in a Secret do not conform to typical naming conventions for environment variables or can’t be changed without impacting running services. You can redefine the keys used to inject an environment variable into a Pod with the spec.containers[].env[].valueFrom attribute.

```bash
# Remapping environment variable keys for Secret entries

apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    env:
    - name: USER
      valueFrom:
        secretKeyRef:
          name: secret-basic-auth
          key: username
    - name: PWD
      valueFrom:
        secretKeyRef:
          name: secret-basic-auth
          key: password
```

The same mechanism of reassigning environment variables works for ConfigMaps. You’d use the attribute spec.containers[].env[].valueFrom.configMapRef instead.

**Mounting a Secret as a Volume**
To demonstrate mounting a Secret as a volume, we’ll create a new Secret of type kubernetes.io/ssh-auth. This Secret type captures the value of an SSH private key that you can view using the command cat ~/.ssh/id_rsa. To process the SSH private key file with the create secret command, it needs to be available as a file with the name ssh-privatekey:

```bash
cp ~/.ssh/id_rsa ssh-privatekey

kubectl create secret generic secret-ssh-auth --from-file=ssh-privatekey \
  --type=kubernetes.io/ssh-auth
```

Mounting the Secret as a volume follows the two-step approach: define the volume first, and then reference it as a mount path for one or many containers. The volume type is called secret.

```bash
# Mounting a Secret as a volume

apiVersion: v1
kind: Pod
metadata:
  name: backend
spec:
  containers:
  - image: bmuschko/web-app:1.0.1
    name: backend
    volumeMounts:
    - name: ssh-volume
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: ssh-volume
    secret:
      secretName: secret-ssh-auth

# Application runtime behavior can be controlled either by injecting configuration data as environment variables or by mounting a volume to a path.
```

### Deployments and ReplicaSets

A major strength of Kubernetes lies in its ability to scale applications seamlessly and manage replication with ease. To enable these capabilities, Kubernetes provides two primitives: Deployments and ReplicaSets.

Deployments also make it easy to roll out updates to your application and, if needed, roll back to a previous version—all with minimal downtime and maximum control.

- Understand application deployments and how to perform rolling update and rollbacks
- Understand the primitives used to create robust, self-healing, application deployments

**Working with Deployments**
A ReplicaSet is a Kubernetes API resource that controls multiple, identical instances of a Pod running the application, called replicas. It has the capability of scaling the number of replicas up or down on demand.

A Deployment abstracts the functionality of ReplicaSet and manages it internally. In practice, this means you do not have to create, modify, or delete ReplicaSet objects yourself. The Deployment keeps a history of application versions and can delegate toward the ReplicaSet to roll back to an older version to counteract a blocking or potentially costly production issue. Furthermore, it offers the capability of scaling the number of replicas.

**Creating Deployments**
You can create a Deployment using the imperative command create deployment. The command offers a range of options, some of which are mandatory. At a minimum, you need to provide the name of the Deployment and the container image. The Deployment passes this information to the ReplicaSet, which uses it to manage the replicas. The default number of replicas created is one; however, you can define a higher number of replicas using the option --replicas.

```bash
kubectl create deployment app-cache --image=memcached:1.6.8 --replicas=4

# A YAML manifest for a Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-cache
  labels:
    app: app-cache
spec:
  replicas: 4
  selector:
    matchLabels:
      app: app-cache
  template:
    metadata:
      labels:
        app: app-cache
    spec:
      containers:
      - name: memcached
        image: memcached:1.6.8
```

For label selection to work properly, the assignment of spec.selector.matchLabels and spec.template.metadata needs to match.

![Deployment label selection](/assets/cka2_1102.png)

**Replica Replacement**
The ReplicaSet in Kubernetes ensures that a specified number of replicas are running at all times. If a Pod fails or is deleted, Kubernetes automatically creates a replacement Pod to maintain the desired replica count. This behavior is one of Kubernetes’ key self-healing capabilities.

**Performing Rolling Updates and Rollbacks**
A Deployment fully abstracts rollout and rollback capabilities by delegating this responsibility to the ReplicaSet(s) it manages. Once a user changes the definition of the Pod template in a Deployment, Kubernetes will create a new ReplicaSet that applies the changes to the replicas it controls and then shut down the previous ReplicaSet.

**Rolling Out a New Revision**
The Deployment primitive employs rolling update as the default deployment strategy, also referred to as ramped deployment. It’s called “ramped” because the Deployment gradually transitions replicas from the old version to a new version in batches. The Deployment automatically creates a new ReplicaSet for the desired change after the user updates the Pod template.

Kubernetes keeps track of the changes you make to a Deployment over time in the rollout history. Every change is represented by a revision. When changing the Pod template of a Deployment—for example, by updating the image—the Deployment triggers the creation of a new ReplicaSet. The Deployment will gradually perform the migrating by decreasing the replica count of the old ReplicaSet and increasing the count on the new ReplicaSet. You can check the rollout history by running the following command. You will see two revisions listed:

```bash
kubectl rollout history deployment app-cache
```

By default, a Deployment persists for a maximum of 10 revisions in its history. You can change the limit by assigning a different value to spec.revisionHistoryLimit.

**Adding a Change Cause for a Revision**
The rollout history renders the column CHANGE-CAUSE. You can populate the information for a revision to document why you introduced a new change or which kubectl command you use to make the change.

By default, changing the Pod template does not automatically record a change cause. To add a change cause to the current revision, add an annotation with the reserved key kubernetes.io/change-cause to the Deployment object. The following imperative annotate command assigns the change cause “Image updated to 1.6.10”:

```bash
kubectl annotate deployment app-cache kubernetes.io/change-cause=\
"Image updated to 1.6.10"
```

**Rolling Back to a Previous Revision**
Problems can arise in production that require swift action. For example, the container image you just rolled out contains a crucial bug. Kubernetes gives you the option to roll back to one of the previous revisions in the rollout history. You can achieve this by using the rollout undo command. To pick a specific revision, provide the command-line option --to-revision. The command rolls back to the previous revision if you do not provide the option. Here, we are rolling back to revision 1:

```bash
kubectl rollout undo deployment app-cache --to-revision=1
```

### Scaling Workloads

There are several reasons why scaling a workload becomes necessary, particularly to maintain optimal performance under increasing demand. For example, an application may experience a surge in users as it gains popularity, or it may need to process larger volumes of data over time.

In Kubernetes, scaling a workload can be achieved in two primary ways: by increasing the resources allocated to each Pod (vertical scaling), or by adjusting the number of Pods running concurrently (horizontal scaling). Horizontal scaling is especially effective for handling fluctuating workloads, ensuring the application remains responsive and resilient under varying levels of demand, such as back pressures on CPU, memory, and I/O.

**Manual Scaling of Workload**
Manual scaling of workloads requires specifying a fixed number of Pods to run. This number should be informed by real-world usage metrics gathered from production environments or estimated through load testing during development.

**Manually Scaling a Deployment**
Scaling (up or down) the number of replicas controlled by a Deployment is a straightforward process. You can either manually edit the live object using the edit deployment command and change the value of the attribute spec.replicas, or you can use the imperative scale deployment command.

```bash
kubectl scale deployment app-cache --replicas=6
```

**Autoscaling of Workload**
Another way to scale a Deployment is with the help of a HorizontalPodAutoscaler (HPA). The HPA is an API primitive that defines rules for automatically scaling the number of replicas under certain conditions. Common scaling conditions include a target value, an average value, or an average utilization of a specific metric (e.g., for CPU and/or memory).

![Autoscaling a Deployment](/assets/cka2_1201.png)

**Prerequisites for Autoscaling**
An HPA only works on scalable resources like the Deployment, ReplicaSet, and StatefulSet. It cannot scale standalone Pods. For an HPA to work, a couple of prerequisites need to be fulfilled:

- The Metrics Server needs to be installed. Without it, the HPA cannot retrieve the necessary metrics to evaluate Pod performance. Collecting metrics may take a couple of minutes initially after installing the component.
- The container resource requests need to be defined. For CPU-based autoscaling, your Pods must define spec.contain⁠ers[].resources.​requests.cpu. For memory-based autoscaling, spec.contain⁠ers[].resources.​requests.memory is required. These values provide the baseline for calculating utilization.
- The cluster must have sufficient CPU and memory resources to schedule new Pods.

**Creating Horizontal Pod Autoscalers**
Let’s say you want to define average CPU utilization as the scaling condition. At runtime, the HPA checks the metrics collected by the Metrics Server to determine if the average maximum CPU or memory usage across all replicas of a Deployment is less than or greater than the defined threshold.

![Autoscaling a Deployment horizontally](/assets/cka2_1202.png)

You can use the autoscale deployment command to create an HPA for an existing Deployment. The option --cpu-percent defines the average maximum CPU usage threshold. At the time of writing, the imperative command doesn’t offer an option for defining the average maximum memory utilization threshold. The options --min and --max provide the minimum number of replicas to scale down to and the maximum number of replicas the HPA can create to handle the increased load, respectively:

```bash
kubectl autoscale deployment app-cache --cpu-percent=80 --min=3 --max=5

#A YAML manifest for an HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-cache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-cache
  minReplicas: 3
  maxReplicas: 5
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
```

**Defining Multiple Scaling Metrics**
You can define more than a single resource type as a scaling metric.
We are inspecting CPU and memory utilization to determine if the replicas of a Deployment need to be scaled up or down.

```bash
# A YAML manifest for a HPA with multiple metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-cache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-cache
  minReplicas: 3
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
```

### Resource Requirements, Limits, and Quotas

A workload executed in Pods will consume a certain amount of resources (e.g., CPU and memory). You should define resource requirements for those applications. On a container level, you can define a minimum amount of resources needed to run the application, as well as the maximum amount of resources the application is allowed to consume. Application developers should determine the right sizing with load tests or at runtime by monitoring the resource consumption.

Kubernetes measures CPU resources in millicores (m), also called millicpu, and memory resources in bytes. That’s why you might see resources defined as 600 m or 100 Mi.

- Configure Pod admission and scheduling (limits, node affinity, etc.)

Kubernetes administrators can put measures in place to enforce the use of available resource capacity. There are two Kubernetes primitives in this realm: the ResourceQuota and the LimitRange.

A LimitRange is a policy that constrains or defaults the resource allocations for a single object of a specific type (such as for a Pod or a PersistentVolumeClaim).

**Working with Resource Requirements**
It’s recommended practice that you specify resource requests and limits for every container. Determining those resource expectations is not always easy, specifically for applications that haven’t been exercised in a production environment yet. Load testing the application early during the development cycle can help with analyzing the resource needs. Further adjustments can be made by monitoring the application’s resource consumption after deploying it to the cluster.

Quality of Service (QoS) classes in Kubernetes automatically categorize Pods into Guaranteed, Burstable, or BestEffort tiers based on their resource requests and limits, determining their eviction priority when nodes experience resource pressure. While understanding QoS is valuable for production workloads, it’s not explicitly covered in the CKA exam, which focuses more on practical resource management and Pod scheduling rather than the underlying eviction priorities.

**Defining Container Resource Requests**
One metric that comes into play for workload scheduling is the resource request defined by the containers in a Pod. Commonly used resources that can be specified are CPU and memory. The scheduler ensures that the node’s resource capacity can fulfill the resource requirements of the Pod. More specifically, the scheduler determines the sum of resource requests per resource type across all containers defined in the Pod and compares them with the node’s available resources.
Each container in a Pod can define its own resource requests.

```bash
# Setting container resource requests
apiVersion: v1
kind: Pod
metadata:
  name: rate-limiter
spec:
  containers:
  - name: business-app
    image: bmuschko/nodejs-business-app:1.0.0
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "256Mi"
        cpu: "1"
  - name: ambassador
    image: bmuschko/nodejs-ambassador:1.0.0
    ports:
    - containerPort: 8081
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
```

**Defining Container Resource Limits**
Another metric you can set for a container is the resource limits. Resource limits ensure that the container cannot consume more than the allotted resource amounts. For example, you could express that the application running in the container should be constrained to 1,000 m of CPU and 512 Mi of memory.

```bash
# Setting container resource limits
apiVersion: v1
kind: Pod
metadata:
  name: rate-limiter
spec:
  containers:
  - name: business-app
    image: bmuschko/nodejs-business-app:1.0.0
    ports:
    - containerPort: 8080
    resources:
      limits:
        memory: "256Mi"
  - name: ambassador
    image: bmuschko/nodejs-ambassador:1.0.0
    ports:
    - containerPort: 8081
    resources:
      limits:
        memory: "64Mi"
```

**Working with Resource Quotas**
The Kubernetes primitive ResourceQuota establishes the usable maximum amount of resources per namespace. Once put in place, the Kubernetes scheduler will take care of enforcing those rules. The following list should give you an idea of the rules that can be defined:

- Setting an upper limit for the number of objects that can be created for a specific type (e.g., a maximum of three Pods)
- Limiting the total sum of compute resources (e.g., 3 Gi of RAM)
- Expecting a Quality of Service (QoS) class for a Pod (e.g., BestEffort to indicate that the Pod must not make any memory or CPU limits or requests)

```bash
# Defining hard resource limits with a ResourceQuota

apiVersion: v1
kind: ResourceQuota
metadata:
  name: awesome-quota
  namespace: team-awesome
spec:
  hard:
    pods: 2
    requests.cpu: "1"
    requests.memory: 1024Mi
    limits.cpu: "4"
    limits.memory: 4096Mi
```

**Working with Limit Ranges**
In the previous section, you learned how a resource quota can restrict the consumption of resources within a specific namespace in aggregate. For individual Pod objects, the resource quota cannot set any constraints. That’s where the limit range comes in. The enforcement of LimitRange rules happens during the admission control phase when processing an API request.

The LimitRange is a Kubernetes primitive that constrains or defaults the resource allocations for specific object types:

- Enforces minimum and maximum compute resource usage per Pod or container in a namespace
- Enforces minimum and maximum storage request per PersistentVolumeClaim in a namespace
- Enforces a ratio between request and limit for a resource in a namespace
- Sets default requests/limits for compute resources in a namespace and automatically injects them into containers at runtime

**Creating LimitRanges**

```bash
# A LimitRange defining multiple constraint criteria
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - type: Container   1
    defaultRequest:   2
      cpu: 200m
    default:          3
      cpu: 200m
    min:              4
      cpu: 100m
    max:              4
      cpu: "2"
```

### Pod Scheduling

The scheduler is the cluster component responsible for deciding which node to select for running a Pod.

**Pod Scheduling Algorithm**
Initially a Pod created by an end user does not have a node assigned. That’s the job of the scheduler cluster component. The scheduler runs in a scheduling loop, watching for unscheduled Pods. It will then evaluate available nodes to choose the most suitable one for the Pod.

**Pod Scheduling Options**
The scheduler does a reasonably good job of assigning a Pod to a feasible node. Under certain conditions, you may want to restrict which node a Pod can run on, or define a preferred selection criteria. This is usually expressed by using label selection.

- Node selector: A hard requirement to determine which node the Pod needs to run on
- Node affinity and anti-affinity: A more flexible requirement for defining hard or soft requirements for node selection
- Taints and tolerations: A way to safeguard specific nodes from scheduling Pods on them based on conditions and requirements
- Pod topology spread constraints: Defines how to spread Pods across different topologies, i.e., regions and zones

**Working with Node Selectors**
The node selector defines a hard requirement for scheduling a Pod on specific nodes. To use the node selector, label one or many nodes with a specific label key-value pair. When defining a Pod in a YAML manifest, select the label from the Pod via the attribute spec.nodeSelector.

```bash
# Labeling a Node
kubectl label node multi-node-m03 disk=ssd

kubectl get nodes --show-labels

# Assigning a Node Selector to a Pod
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  nodeSelector:
    disk: ssd
  containers:
  - name: nginx
    image: nginx:1.27.1

# The node selector is not limited to a single label key-value pair. Selecting a map of labels is completely valid.
```

**Working with Node Affinity and Anti-Affinity**
For more flexible and expressive scheduling rules, you should use node affinity, defined under spec.affinity.nodeAffinity in the Pod specification. Node affinity allows you to match nodes using label selector expressions, enabling more complex criteria such as logical operators (In, NotIn, Exists, etc.) and prioritized preferences.

Below image shows the node affinity concept in action. In this scenario, Pod 1 can be scheduled on Node 1 or Node 2 based on the defined set-based label selection.

![Node affinity scenarios](/assets/cka2_1403.png)

Node anti-affinity is useful in situations where you want to prevent Pods from being scheduled on specific nodes. This is particularly helpful in high-availability situations where you want to spread Pods across different zones or regions.

**Assigning a Node Affinity to a Pod**
In short, node affinity effectively replaces the node selector for most use cases, offering greater precision and control over workload placement.

```bash
# Assigning node affinity

apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
          operator: In
          values:
          - ssd
          - hdd
  containers:
  - name: nginx
    image: nginx:1.27.1
```

**Node Affinity Types**
Beyond supporting set-based expressions, node affinity also introduces specific types that control when the rules are applied. One commonly used type is requiredDuringSchedulingIgnoredDuringExecution. This setting means that the affinity rules are strictly enforced only at the time of scheduling—when the Pod is initially assigned to a node. Once the Pod is running, any changes to the node affinity rules are ignored and will not trigger rescheduling.

Types starting with requiredDuringScheduling express a hard requirement, and types starting with preferredDuringScheduling express a soft requirement, a preference which the schedule doesn’t have to adhere to in case no fitting node can be determined.

- requiredDuringSchedulingIgnoredDuringExecution: Rules that must be met for a Pod to be scheduled onto a node
- preferredDuringSchedulingIgnoredDuringExecution: Rules that specify preferences that the scheduler

**Node Affinity Operators**

| Operator     | Behavior                                                                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| In           | A node has an assigned label value in the given set of strings.                                                                                               |
| NotIn        | Only those nodes are selected that do not have an assigned label value in the given set of strings.                                                           |
| Exists       | A node has a label key assigned to it that matches the given string.                                                                                          |
| DoesNotExist | A node does not have a label key assigned to it that matches the given string.                                                                                |
| Gt           | Selects nodes where the label’s value is numerically greater than the specified value, like selecting nodes with more than 8 CPUs using “cpu-count Gt 8.”     |
| Lt           | Selects nodes where the label’s value is numerically less than the specified value, like selecting nodes with less than 16 GB memory using “memory-gb Lt 16.” |

There are two operators, NotIn and DoesNotExist, that negate the selective effects of their counterparts In and Exists. Those operators are used to define a node anti-affinity behavior.

**Assigning a Node Anti-Affinity to a Pod**
Node anti-affinity rules are typically used to prevent certain Pods from being scheduled on the same nodes, based on labels. An essential tool for defining anti-affinity behavior is the operator.
uses the NotIn operator to repel Pods from a set of nodes with the given label values.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: NotIn                                1
            values:
            - ssd
            - ebs
  containers:
  - name: nginx
    image: nginx:1.27.1
```

In essence, node anti-affinity does not require you to learn a new API or new attributes when defining a Pod. Its usage primary boils down to the operator you select to define the node affinity rule.

**Working with Taints and Tolerations**
Similar to node anti-affinity, taints and tolerations represent another way in Kubernetes to influence where Pods can be scheduled, but they serve different purposes and work in fundamentally different ways. Node anti-affinity is meant to be used to spread or separate workload across nodes, whereas taints and tolerations are used for node isolation and workload protection.

The main purpose of taints and tolerations is to prevent Pods from being scheduled on a node unless they explicitly tolerate that node’s taint. You’d add the taint to a node to say, “don’t schedule anything here unless it tolerates the taint.” Then in the Pod, you’d add a toleration if you want it to become schedulable on the tainted node.

A typical use case for using taints and tolerations is the need to ensure that Pods are not scheduled on control plane nodes. Control plane nodes use the taint node-role.kubernetes.io/control-plane:NoSchedule to prevent Pods from being scheduled on them unless they provide a corresponding toleration.

**Tainting a Node**
A taint on a node marks it as unsuitable for certain Pods unless those Pods explicitly state they can tolerate it. A taint consists of three parts—key, value, and effect— formatted as key=value:effect. The key and value portions represent a simple, free-form key-value pair, similar to a label assignment. The effect describes the runtime treatment of the taint by the scheduler.

Use the imperative kubectl taint command to add a taint to a node. The following command adds the taint special=true:NoSchedule to the node named multi-node-m02:

```bash
kubectl taint node multi-node-m02 special=true:NoSchedule

#The scheduler now considers this taint during Pod placement.
```

**Taint Effects**

- NoSchedule: Unless a Pod has matching toleration, it won’t be scheduled on the node.
- PreferNoSchedule: Try not to place a Pod that does not tolerate the taint on the node, but it is not required.
- NoExecute: Evict Pod from node if already running on it. No future scheduling on the node.

**Assigning a Toleration to a Pod**
To enable a Pod to run on a tainted node, you must add a toleration to the Pod specification that precisely matches the key, value, and effect of the node’s taint. This tells the scheduler that the Pod is allowed to tolerate the taint and may be placed on the node despite the restriction.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  tolerations:
  - key: "special"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx:1.27.1
```

**Working with Pod Topology Spread Constraints**
Pod topology spread constraints control how Pods are distributed across your cluster to improve availability, resilience, and resource utilization. They define rules for how Pods of a certain group (typically in the same Deployment) should be spread across topology domains (like zones, nodes, or racks). You can define the Pod topology spread constraint in the Pod API with the attribute spec.topologySpread​Con⁠straints.

**Assigning a Topology Spread Constraint to a Pod**
Below shows an example for a Deployment YAML definition with six replicas of an app and ensures that they are evenly spread across availability zones.

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 6
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: web
      containers:
      - name: nginx
        image: nginx:1.27.1
```

### Storage

This critical area ensures applications can reliably store and access data beyond the lifecycle of individual Pods.

**Volumes**

When container images are instantiated as containers, the container needs context—context to CPU, memory, and I/O resources. Pods provide the network and the filesystem context for the containers within. The network is provided as the Pod’s virtual IP address, and the filesystem is mounted to the hosting node’s filesystem.

Applications running in the container can interact with the filesystem as part of the Pod context. A container’s temporary filesystem is isolated from any other container or Pod and is not persisted beyond a Pod restart.

Essentially, a volume is a directory that’s shareable between multiple containers of a Pod.

Different volume types and the process for defining and mounting a volume in a container.

**Volume Types**
Every volume needs to define a type. The type determines the medium that backs the volume and its runtime behavior.

| Type                  | Description                                                                                                                                                   |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| emptyDir              | Empty directory in Pod with read/write access. Persisted only for the lifespan of a Pod. Good for caches or data exchange between containers in the same Pod. |
| hostPath              | File or directory from the host node’s filesystem.                                                                                                            |
| configMap / secret    | Inject configuration data into Pods. See Chapter 10 for examples.                                                                                             |
| nfs                   | An existing Network File System (NFS) share. Preserves data after Pod restart.                                                                                |
| persistentVolumeClaim | Claims a PersistentVolume for use by a Pod. See “Creating PersistentVolumeClaims”.                                                                            |

**Creating and Accessing Volumes**
Defining a volume for a Pod requires two steps. First, you need to declare the volume itself using the attribute spec.volumes[]. As part of the definition, you provide the name and the type. Just declaring the volume won’t be sufficient, though. Second, the volume also needs to be mounted to a path of the consuming container via spec.​con⁠tainers[].volumeMounts[]. The mapping between the volume and the volume mount occurs by the matching name.

```bash
# A Pod defining and mounting a volume

apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx:1.27.1
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
  - name: sidecar
    image: busybox:1.37.0
    volumeMounts:
    - name: shared-data
      mountPath: /data

# Specifies a volume of type emptyDir. The curly braces mean that we don’t want to provide additional configuration, e.g., a size limit.
```

**Read-Only Volume Mounts**
Some data is only meant for consumption, for example, configuration data provided through a volume. You can mark a volume mount to be read-only. Kubernetes will prevent any write operation on that volume mount. It’s important to understand that other containers may use the same volume in read/write mode.

To make a volume mount read-only, assign the value true to the attribute spec.​con⁠tainers[].volumeMounts[].readOnly

```bash
# A mount path marked as read-only
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx:1.27.1
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html
      readOnly: true

# Prevents write operations for this mount path
```

### Persistent Volumes

Persistent volumes are a specific category of the wider concept of volumes with the capability of persisting data beyond a Pod lifecycle. The mechanics for persistent volumes are slightly more complex. The persistent volume is the resource that actually persists the data to an underlying physical storage.

The persistent volume claim represents the connecting resource between a Pod and a persistent volume responsible for requesting the storage.

Finally, the Pod needs to claim the persistent volume and mount it to a directory path available to the containers running inside of the Pod.

- Implement storage classes and dynamic volume provisioning
- Configure volume types, access modes, and reclaim policies
- Manage persistent volumes and persistent volume claims

![Claiming a persistent volume from a Pod](/assets/cka2_1601.png)

**Working with Persistent Volumes**
Data stored on volumes outlive a container restart. In many applications, the data lives far beyond the lifecycles of the applications, container, Pod, nodes, and even the clusters themselves. Data persistence ensures the lifecycles of the data are decoupled from the lifecycles of the cluster resources. A typical example would be data persisted by a database. That’s the responsibility of a persistent volume. Kubernetes models persist data with the help of two primitives: the PersistentVolume and the PersistentVolumeClaim.

The PersistentVolume is the primitive representing a piece of storage in a Kubernetes cluster. It is completely decoupled from the Pod and therefore has its own lifecycle. The object captures the source of the storage (e.g., storage made available by a cloud provider). A PersistentVolume is either provided by a Kubernetes administrator or assigned dynamically by mapping to a storage class.

The PersistentVolumeClaim requests the resources of a PersistentVolume—for example, the size of the storage and the access type. In the Pod, you will use the type persistentVolumeClaim to mount the abstracted PersistentVolume by using the PersistentVolumeClaim.

**Volume Types**
Kubernetes supports several types of persistent volumes to accommodate different storage backends and use cases. Each type has its own characteristics and is suitable for specific scenarios.

| Volume Type | Description                                                                                                                                                                                                     |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| hostPath    | Mounts a file or directory from the host node’s filesystem into the Pod. Useful for development and testing but not recommended for production multi-node clusters, as it ties the Pod to a specific node.      |
| local       | Represents a mounted local storage device such as a disk, partition, or directory. Provides better performance than remote storage but requires node affinity to ensure Pods are scheduled on the correct node. |
| nfs         | Allows multiple Pods to share the same Network File System (NFS) mount. Supports ReadWriteMany access mode and is useful for sharing data between Pods across nodes.                                            |
| csi         | Container Storage Interface (CSI) driver that provides a standardized way to expose storage systems to containerized workloads. Most modern storage solutions use CSI drivers.                                  |
| fc          | Fibre Channel (FC) volume that allows existing FC storage to be attached to Pods. Requires FC hardware and proper configuration on nodes.                                                                       |
| iscsi       | Internet Small Computer Systems Interface (iSCSI) volume that allows existing iSCSI storage to be mounted to Pods. Provides block-level storage over IP networks.                                               |

The choice of volume type depends on your infrastructure, performance requirements, and whether you need the storage to be shared across multiple Pods or nodes. For cloud environments, CSI drivers are typically provided by cloud providers (like AWS EBS CSI driver, GCE Persistent Disk CSI driver, or Azure Disk CSI driver) to integrate with their native storage services.

**Static Versus Dynamic Provisioning**
A PersistentVolume can be created statically or dynamically. If you go with the static approach, then you first need to create a storage device and then reference it by explicitly creating an object of kind PersistentVolume. The dynamic approach doesn’t require you to create a PersistentVolume object. It will be automatically created from the PersistentVolumeClaim by setting a storage class name using the attribute spec.storageClassName.

A storage class is an abstraction concept that defines a class of storage device (e.g., storage with slow or fast performance) used for different application types. It’s the job of a Kubernetes administrator to set up storage classes.

**Creating PersistentVolumes**
When you create a PersistentVolume object yourself, we refer to the approach as static provisioning. A PersistentVolume can be created only by using the manifest-first approach. At this time, kubectl doesn’t allow the creation of a PersistentVolume using the create command. Every PersistentVolume needs to define the storage capacity using spec.capacity and an access mode set via spec.accessModes.

Example below creates a PersistentVolume named db-pv with a storage capacity of 1 Gi and read/write access by a single node. The attribute hostPath mounts the directory /data/db from the host node’s filesystem.

```bash
#YAML manifest defining a PersistentVolume

apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/db
```

**Configuration Options for a PersistentVolume**
A PersistentVolume offers a variety of configuration options that determine its innate runtime behavior. For the exam, it’s important to understand the volume mode, access mode, reclaim policy, and node affinity configuration options.

**Volume Mode**
The volume mode handles the type of device. That’s a device either meant to be consumed from the filesystem or backed by a block device. The most common case is a filesystem device. You can set the volume mode using the attribute spec.volumeMode

- Filesystem: Default. Mounts the volume into a directory of the consuming Pod. Creates a filesystem first if the volume is backed by a block device and the device is empty.
- Block: Used for a volume as a raw block device without a filesystem on it.

**Access Mode**
Each PersistentVolume can express how it can be accessed using the attribute spec.accessModes. For example, you can define that the volume can be mounted only by a single Pod in a read or write mode or that a volume is read-only but accessible from different nodes simultaneously.

| Type             | Short form | Description                               |
| ---------------- | ---------- | ----------------------------------------- |
| ReadWriteOnce    | RWO        | Read/write access by a single node        |
| ReadOnlyMany     | ROX        | Read-only access by many nodes            |
| ReadWriteMany    | RWX        | Read/write access by many nodes           |
| ReadWriteOncePod | RWOP       | Read/write access mounted by a single Pod |

The following command parses the access modes from the PersistentVolume named db-pv. As you can see, the returned value is an array underlining the fact that you can assign multiple access modes at once:

```bash
kubectl get pv db-pv -o jsonpath='{.spec.accessModes}'
```

ReadWriteOnce (RWO) allows mounting by a single node as read-write, ideal for databases like MySQL or PostgreSQL and stateful applications using block storage (AWS EBS, GCE Persistent Disk). ReadOnlyMany (ROX) enables multiple nodes to mount read-only simultaneously, perfect for serving static web content or shared configuration files across multiple Pods. ReadWriteMany (RWX) permits concurrent read-write access from multiple nodes, requiring file-based storage like NFS or AWS EFS, is essential for shared upload directories or content management systems where multiple Pods process the same files. ReadWriteOncePod (RWOP) ensures only one Pod cluster-wide can mount the volume, providing stronger guarantees than RWO for applications like etcd or during StatefulSet migrations where absolute single-writer semantics are critical. Storage provider support varies: block storage typically supports RWO/RWOP while file systems are needed for ROX/RWX.

**Reclaim Policy**
Optionally, you can also define a reclaim policy for a PersistentVolume. The reclaim policy specifies what should happen to a PersistentVolume object when the bound PersistentVolumeClaim is deleted.
For dynamically created PersistentVolumes, the reclaim policy can be set via the attribute .reclaimPolicy in the storage class. For statically created PersistentVolumes, use the attribute spec.​per⁠sistentVolumeReclaimPolicy in the PersistentVolume definition.

| Type    | Description                                                                                                        |
| ------- | ------------------------------------------------------------------------------------------------------------------ |
| Retain  | Default. When PersistentVolumeClaim is deleted, the PersistentVolume is “released” and must be manually reclaimed. |
| Delete  | Deletion removes PersistentVolume and its associated storage.                                                      |
| Recycle | This value is deprecated. You should use one of the other values.                                                  |

This command retrieves the assigned reclaim policy of the PersistentVolume named db-pv:

```bash
kubectl get pv db-pv -o jsonpath='{.spec.persistentVolumeReclaimPolicy}'
```

**Node Affinity**
Node affinity allows you to constrain which nodes a PersistentVolume can be accessed from. This is particularly important for local storage types like local and hostPath volumes, which are physically tied to specific nodes. By defining node affinity rules, you ensure that Pods using the PersistentVolume are scheduled only on nodes that can actually access the underlying storage.

The node affinity is specified using the spec.nodeAffinity field in the PersistentVolume definition. It uses the same syntax as Pod node affinity, with required rules that must be satisfied for the volume to be accessible.

```bash
# Defining node affinity for a PersistentVolume

apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node01
          - node02
```

When a PersistentVolumeClaim binds to a PersistentVolume with node affinity, any Pod using that claim will be scheduled according to these constraints. If no suitable node is available, the Pod will remain in Pending status.

Important considerations:

- Node affinity is required for local volume types.
- The scheduler considers both the Pod’s node selectors and the PersistentVolume’s node affinity.
- Changes to node labels after binding don’t affect existing mounted volumes.
- For high availability, avoid overly restrictive node affinity rules.

**Creating PersistentVolumeClaims**
Its purpose is to bind the PersistentVolume to the Pod.

```bash
# Definition of a PersistentVolumeClaim

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 256Mi
```

What that’s saying is: “Give me a PersistentVolume that can fulfill the resource request of 256 Mi and provides the access mode ReadWriteOnce.”

Static provisioning should use an empty string for the attribute spec.storageClassName if you do not want it to automatically assign the default storage class. The binding to an appropriate PersistentVolume happens automatically based on those criteria.

**Binding by Volume Name**
When creating a PersistentVolumeClaim, you can optionally specify the exact PersistentVolume you want to bind to using the spec.volumeName attribute. This creates a direct binding between the PersistentVolumeClaim and a specific PersistentVolume, bypassing the normal matching algorithm that considers storage size, access modes, and storage class.

```bash
#Binding a PersistentVolume to a PersistentVolumeClaim by name

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: specific-pvc
spec:
  volumeName: db-pv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Mounting PersistentVolumeClaims in a Pod**
All that’s left is to mount the PersistentVolumeClaim in the Pod that wants to consume it.
Use spec.volumes[].persistentVolumeClaim and provide the name of the PersistentVolumeClaim.

```bash
# A Pod referencing a PersistentVolumeClaim

apiVersion: v1
kind: Pod
metadata:
  name: app-consuming-pvc
spec:
  volumes:
  - name: app-storage
    persistentVolumeClaim:
      claimName: db-pvc
  containers:
  - image: alpine:3.22.2
    name: app
    command: ["/bin/sh"]
    args: ["-c", "while true; do sleep 60; done;"]
    volumeMounts:
      - mountPath: "/mnt/data"
        name: app-storage
```

**Storage Classes**
A StorageClass is a Kubernetes primitive that defines a specific type or “class” of storage. One typical storage characteristic is the type (e.g., fast SSD storage versus remote cloud storage or the backup policy for storage). The storage class is used to provision a PersistentVolume dynamically based on its criteria.

In practice, this means that you do not have to create the PersistentVolume object yourself. The provisioner assigned to the storage class takes care of it. Most Kubernetes cloud providers come with a list of existing provisioners.

**Creating Storage Classes**
Storage classes can be created declaratively only with the help of a YAML manifest. At a minimum, you need to declare the provisioner. All other attributes are optional and use default values if not provided upon creation. Most provisioners let you set parameters specific to the storage type.

The example below defines a storage class on Google Compute Engine denoted by the provisioner kubernetes.io/gce-pd.

```bash
# Definition of a storage class

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd

# The storage class can be listed using the get storageclass command
kubectl get storageclass
```

**Using Storage Classes**
Provisioning a PersistentVolume dynamically requires assigning of the storage class when you create the PersistentVolumeClaim.

```bash
#Using a storage class in a PersistentVolumeClaim

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi
  storageClassName: standard

```

### Servicing and Networking

The domain Servicing and Networking covers the Kubernetes primitives important for establishing and restricting communication between microservices running in the cluster, or outside consumers. More specifically, this domain covers the primitives Services and Ingresses, as well as network policies.

### Services

In “Using a Pod’s IP Address for Network Communication”, we learned that you can communicate with a Pod by its IP address. A restart of a Pod will automatically assign a new virtual ClusterIP address. Therefore, other parts of your system cannot rely on the Pod’s IP address if they need to talk to one another.

Building a microservices architecture, where each of the components runs in its own Pod with the need to communicate with each other through a stable network interface, requires a different primitive, the Service.

The Service implements an abstraction layer on top of Pods, assigning a fixed virtual IP fronting all the Pods with matching labels, and that virtual IP is called ClusterIP.

**Working with Services**
In a nutshell, Services provide discoverable names and load balancing to a set of Pods. The Service remains agnostic from IP addresses with the help of the Kubernetes DNS control-plane component.

Similar to a Deployment, the Service determines the Pods it works on with the help of label selection.

![Service traffic routing based on label selection](/assets/cka2_1701.png)

**_Services and Deployments_**
Services are a complementary concept to Deployments. Services route network traffic to a set of Pods, and Deployments delegate to a ReplicaSet to manage a set of Pods, the replicas. While you can use both concepts in isolation, it is recommended to use Deployments and Services together. The primary reason is the ability to scale the number of replicas and at the same time be able to expose an endpoint to funnel network traffic to those Pods.

**Service Types**
Every Service defines a type. The type is responsible for exposing the Service inside and/or outside of the cluster.

| Type         | Description                                                                                                                                                                         |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ClusterIP    | Exposes the Service on a cluster-internal IP. Reachable only from within the cluster. Kubernetes uses a round-robin algorithm to distribute traffic evenly among the targeted Pods. |
| NodePort     | Exposes the Service on each node’s IP address at a static port. Accessible from outside of the cluster. The Service type does not provide any load balancing across multiple nodes. |
| LoadBalancer | Exposes the Service externally using a cloud provider’s load balancer.                                                                                                              |

Other Service types, e.g., ExternalName or the headless Service, can be defined.

**Service type inheritance**
The Service types just mentioned, ClusterIP, NodePort, and LoadBalancer, make a Service accessible with different scopes of exposure. It’s imperative to understand that those Service types also build on top of each other.

![Network accessibility characteristics for Service types](/assets/cka2_1702.png)

For example, creating a Service of type NodePort means that the Service will bear the network accessibility characteristics of a ClusterIP Service type as well. In turn, a NodePort Service is accessible from within and from outside of the cluster.

**When to use which Service type?**
When building a microservices architecture, the question arises of which Service type to choose to implement certain use cases.

The ClusterIP Service type is suitable for use cases that call for exposing a microservice to other Pods within the cluster. Say you have a frontend microservice that needs to connect to one or many backend microservices. To properly implement the scenario, you’d stand up a ClusterIP Service that routes traffic to the backend Pods. The frontend Pods would then talk to that Service.

The NodePort Service type is often mentioned as a way to expose an application to consumers external to the cluster. Consumers will have to know the node’s IP address and the statically assigned port to connect to the Service. That’s problematic for multiple reasons. First, the node port is usually allocated dynamically. Therefore, you won’t typically know it in advance. Second, providing the node’s IP address will funnel the network traffic only through a single node so you will not have load balancing at your disposal. Finally, by opening a publicly available node port, you are at risk of increasing the attack surface of your cluster. For all these reasons, a NodePort Service is primarily used for development or testing purposes, and less so in production environments.

The LoadBalancer Service type makes the application available to outside consumers through an external IP address provided by an external load balancer. Network traffic will be distributed across multiple nodes in the cluster. This solution works great for production environments, but keep in mind that every provisioned load balancer will accrue costs and can lead to an expensive infrastructure bill. A more cost-effective solution is the use of an Ingress.

**Creating Services**
You can create Services in a variety of ways, some of which are more appropriate for the exam as they provide a fast turnaround. Let’s discuss the imperative approach first.

A Service needs to select a Pod with a matching label. The Pod created by the following run command is called echoserver, which exposes the application on the container port 8080. Internally, it automatically assigns the label key-value pair run=echoserver to the object:

```bash
kubectl run echoserver --image=k8s.gcr.io/echoserver:1.10 --restart=Never \
  --port=8080

```

You can create a Service object using the create service command. Make sure to provide the Service type as a mandatory argument. Here we are using the type clusterip. The command-line option --tcp specifies the port mapping. Port 80 exposes the Service to incoming network traffic. Port 8080 targets the container port exposed by the Pod:

```bash
kubectl create service clusterip echoserver --tcp=80:8080

# An even faster workflow of creating a Pod and Service together can be achieved with a run command and the --expose option.
kubectl run echoserver --image=k8s.gcr.io/echoserver:1.10 --restart=Never \
  --port=8080 --expose
```

It’s actually more common to use a Deployment and Service that work together. The following set of commands creates a Deployment with five replicas and then uses the expose deployment command to instantiate the Service object. The port mapping can be provided with the options --port and --target-port:

```bash
kubectl create deployment echoserver --image=hashicorp/http-echo:1.0.0 \
  --replicas=5

kubectl expose deployment echoserver --port=80 --target-port=8080
```

An endpoint is a resolvable network endpoint, which serves as the virtual IP address and container port of a Pod. If a Service does not render any endpoints, then you are likely dealing with a misconfiguration. Use the EndpointSlice API to interact with the endpoints.

EndpointSlice is Kubernetes’ scalable service discovery mechanism that maps Services to their backing Pod network endpoints. Instead of storing all endpoints in a single object like the deprecated Endpoints API, EndpointSlice distributes this information across multiple smaller objects, with each slice containing up to 100 endpoints by default. The following command lists the endpoints for the Service named echoserver with the assigned label app=echoserver:

```bash
kubectl get endpointslices -l app=echoserver
```

**Discovering the Service by DNS lookup**
Kubernetes registers every Service by its name with the help of its DNS service named CoreDNS. Internally, CoreDNS will store the Service name as a hostname and maps it to the ClusterIP address. Accessing a Service by its DNS name instead of an IP address is much more convenient and expressive when building microservice architectures because IP addresses are ephemeral and unpredictable, whereas the labels are declarative and known.

The full hostname for a Service is echoserver.default.svc.cluster.local. The string svc describes the type of resource we are communicating with. CoreDNS uses the default value cluster.local as a domain name (which is configurable if you want to change it). You do not have to spell out the full hostname when communicating with a Service.

### Ingresses

Once there’s a need to expose the application to external consumers, selecting an appropriate Service type becomes crucial. The most practical choice often involves creating a Service of type LoadBalancer. Such a Service offers load balancing capabilities by assigning an external IP address accessible to consumers outside the Kubernetes cluster.

However, opting for a LoadBalancer Service for each externally reachable application has drawbacks. In a cloud provider environment, each Service triggers the provisioning of an external load balancer, resulting in increased costs. Additionally, managing a collection of LoadBalancer Service objects can lead to administrative challenges, as a new object must be established for each externally accessible microservice.

To mitigate these issues, the Ingress primitive comes into play, offering a singular, load-balanced entry point to an application stack. An Ingress possesses the ability to route external HTTP(S) requests to one or more Services within the cluster based on an optional, DNS-resolvable host name and URL context path.

- Know how to use Ingress controllers and Ingress resources

**Working with Ingresses**
The Ingress exposes HTTP (and optionally HTTPS) routes to clients outside of the cluster through an externally-reachable URL. The routing rules configured with the Ingress determine how the traffic should be routed. Cloud provider Kubernetes environments will often deploy an external load balancer. The Ingress receives a public IP address from the load balancer. You can configure rules for routing traffic to multiple Services based on specific URL context paths

![Managing external access to the Services via HTTP(S)](/assets/cka2_1801.png)

The scenario depicted above instantiates an Ingress as the sole entry point for HTTP(S) calls to the domain name next.example.com. Based on the provided URL context, the Ingress directs the traffic to either of the fictional Services: one designed for a business application and the other for fetching metrics related to the application.

Specifically, the URL context path /app is routed to the App Service responsible for managing the business application. Conversely, sending a request to the URL context /metrics results in the call being forwarded to the Metrics Service, which is capable of returning relevant metrics.

**Installing an Ingress Controller**
For Ingress to function, an Ingress controller is essential. This controller assesses the set of rules outlined by an Ingress, dictating the routing of traffic. The choice of Ingress controller often depends on the specific use cases, requirements, and preferences of the Kubernetes cluster administrator. Noteworthy examples of production-grade Ingress controllers include the F5 NGINX Ingress Controller or the AKS Application Gateway Ingress Controller.

**Deploying Multiple Ingress Controllers**
Certainly, deploying multiple Ingress controllers within a single cluster is a feasible option, especially if a cloud provider has preconfigured an Ingress controller in the Kubernetes cluster. The Ingress API introduces the attribute spec.ingressClassName to facilitate the selection of a specific controller implementation by name.

Kubernetes determines the default Ingress class by scanning for the annotation ingressclass.kubernetes.io/is-default-class: "true" within all Ingress class objects. In scenarios where Ingress objects do not explicitly specify an Ingress class using the attribute spec.ingressClassName, they automatically default to the Ingress class marked as the default through this annotation. This mechanism provides flexibility in managing Ingress classes and allows for a default behavior when no specific class is specified in individual Ingress objects.

**Configuring Ingress Rules**
When creating an Ingress, you have the flexibility to define one or multiple rules. Each rule encompasses the specification of an optional host, a set of URL context paths, and the backend responsible for routing the incoming traffic. This structure allows for fine-grained control over how external HTTP(S) requests are directed within the Kubernetes cluster, catering to different services based on specified conditions.

An Ingress controller can optionally define a default backend that is used as a fallback route should none of the configured Ingress rules match.

**Creating Ingresses**
You can create an Ingress with the imperative create ingress command. The main command-line option you need to provide is --rule, which defines the rules in a comma-separated fashion. The notation for each key-value pair is <host>/<path>=<service>:<port>. Let’s create an Ingress object with two rules:

```bash
kubectl create ingress next-app \
  --rule="next.example.com/app=app-service:8080" \
  --rule="next.example.com/metrics=metrics-service:9090"

# If you look at the output of the create ingress --help command, more fine-grained rules can be specified.
```

Port 80 for HTTP traffic is implied, as we didn’t specify a reference to a TLS Secret object. If you have specified tls=mysecret in the rule definition, then the port 443 would be listed here as well.

### Gateway API

The Gateway API was introduced to standardize and build upon the lessons learned from Ingress and service mesh frameworks like Istio, Contour, and Linkerd, which had demonstrated the need for more sophisticated traffic management capabilities beyond what Ingress could provide. As a more expressive and extensible alternative to the traditional Ingress resource, the Gateway API offers role-oriented design, support for multiple protocols beyond HTTP/HTTPS, and enhanced traffic-routing features.

The Gateway API is the successor of the Ingress primitive and is becoming increasingly important as organizations adopt this modern approach to managing external traffic.

- Use the Gateway API to manage Ingress traffic

**Why Is the Ingress Primitive Not Sufficient?**
The Ingress API is the standard Kubernetes way to configure external HTTP/HTTPS load balancing for Services, but it has limitations. While the Ingress supports TLS termination and simple content-based request routing of HTTP traffic, real-world use cases call for more advanced features. Enhancing the existing Ingress API model of the Ingress wouldn’t allow for adding those features easily for a variety of reasons:

- Ingress controller-specific extensibility: Advanced features like traffic splitting, rate limiting, and request/response manipulation are provided by nonportable annotations for specific Ingress implementations.
- Insufficient permission model: The Ingress API is not well suited for multitenant environments that call for a strong permission model.

**Working with the Gateway API**
You can think of the Gateway API as a unified and standardized API for managing traffic into and out of a Kubernetes cluster instead of having to choose from individual Ingress implementations. Product-specific annotations are not needed anymore to configure routing options. The Gateway API offers a flexible way to incorporate similar features. Effectively, the Gateway API is a universal specification supported by a wide range of different implementations.

Instead of handling one single primitive like the Ingress, the Gateway API splits up responsibilities into multiple primitives.

**Gateway API Resources**
The Gateway API introduces a layered approach to traffic management through four primary resource types that work together to handle incoming traffic:

- Gateway: Defines an instance of traffic-handling infrastructure, such as a cloud load balancer.
- GatewayClass: Each Gateway is associated with a GatewayClass, which describes the actual kind of gateway controller that will handle traffic for the Gateway.
- HTTPRoute/GRPCRoute: Defines HTTP- or GRPC-specific rules for mapping traffic from a Gateway listener to a representation of backend network endpoints. These endpoints are often represented as a Service.
- ReferenceGrant: Can be used to enable cross-namespace references within the Gateway API, e.g., routes may forward traffic to backends in other namespaces.

Managing those Gateway API resources falls within the responsibility of different personas.

The GatewayClass is provided by platform providers, e.g., the cloud provider. The Gateway and ReferenceGrant are installed by the Kubernetes cluster administrator. Lastly, the HTTPRoute and GRPCRoute are created by application developers.

**Installing the Gateway API CRDs**
At the time of writing, the Gateway API resources do not ship with the standard set of Kubernetes API resources. You will have to install the Gateway API in the form of Custom Resource Definitions. The following command installs version 1.3.0 of the CRDs:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/\
download/v1.3.0/standard-install.yaml

# You will now be able to list the CRDs by searching for the API group used by the Gateway API resources:
kubectl get crds | grep gateway.networking.k8s.io
```

**Deploying a Gateway Controller**
The Gateway API requires a controller implementation to function. Different controllers offer various features and performance characteristics. For this example, we’ll use the Envoy Gateway implementation installed by Helm:

```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.4.2 \
  -n envoy-gateway-system --create-namespace
```

**Creating GatewayClasses**
Depending on the Kubernetes environment you are operating in, you may or may not have to create a GatewayClass. Cloud provider Kubernetes clusters should already come with one.

In case you want to create your own GatewayClass, you will first have to determine the Gateway controller name. The controller name depends on the controller implementation installed. You can usually look up the controller name in its documentation.

The controller name for Envoy is gateway.envoyproxy.io/gatewayclass-controller. Create the file gateway-class.yaml

```bash
# A GatewayClass that uses the Envoy GatewayClass controller

apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller

# List GatewayClass objects
kubectl get gatewayclasses
```

**Creating Gateways**
With the GatewayClass instantiated, create a Gateway resource to handle incoming traffic.

```bash
# A Gateway exposing an HTTP listener

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: hello-world-gateway
spec:
  gatewayClassName: envoy
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

There’s only one additional object to set up to finalize the HTTP traffic routing: the HTTPRoute object.

**Creating HTTPRoutes**

```bash
# An HTTPRoute routing traffic to a Service backend
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: hello-world-httproute
spec:
  parentRefs:
    - name: hello-world-gateway
  hostnames:
    - "hello-world.exposed"
  rules:
    - backendRefs:
        - group: ""
          kind: Service
          name: web
          port: 3000
          weight: 1
      matches:
        - path:
            type: PathPrefix
            value: /
```

**Network Policies**
The uniqueness of the IP address assigned to a Pod is maintained across all nodes and namespaces. This is accomplished by allocating a dedicated subnet to each registered node during its creation. The Container Network Interface (CNI) plug-in handles the leasing of IP addresses from the assigned subnet when a new Pod is created on a node. Consequently, Pods on a node can seamlessly communicate with all other Pods running on any node within the cluster.

By default, Kubernetes allows unrestricted Pod-to-Pod communication across all namespaces, which poses a significant security risk, as a compromised Pod in one namespace could potentially access sensitive services in another namespace.

Network policies in Kubernetes function similarly to firewall rules, specifically designed for governing Pod-to-Pod communication. These policies include rules specifying the direction of network traffic (ingress and/or egress) for one or multiple Pods within a namespace or across different namespaces. Additionally, these rules define the targeted ports for communication. This fine-grained control enhances security and governs the flow of traffic within the Kubernetes cluster.

**Working with Network Policies**
Within a Kubernetes cluster, any Pod can talk to any other Pod without restrictions using its IP address or DNS name, even across namespaces. Not only does unrestricted inter-Pod communication pose a potential security risk, it also makes it harder to understand the mental communication model of your architecture. A network policy defines the rules that control traffic from and to a Pod.

For example, there’s no good reason to allow a backend application running in a Pod to talk directly to the frontend application running in another Pod. The communication should be directed from the frontend Pod to the backend Pod.

**Installing a Network Policy Controller**
A network policy cannot work without a network policy controller. The network policy controller evaluates the collection of rules defined by a network policy.

Some CNIs like flannel do not include a network policy controller and focus solely on providing basic Pod-to-Pod connectivity, meaning NetworkPolicy resources will be accepted by the API but completely unenforced.

Cilium is a CNI that implements a network policy controller. You can install Cilium on cloud-provider and on-prem Kubernetes clusters.

Once it is installed, you should find at least two Pods running Cilium and the Cilium Operator in the kube-system namespace.

**Creating a Network Policy**
Label selection plays a crucial role in defining which Pods a network policy applies to. We’ve already seen this concept in action in other contexts (e.g., the Deployment and the Service). Furthermore, a network policy defines the direction of the traffic to allow or disallow. In the context of a network policy, incoming traffic is called ingress, and outgoing traffic is called egress. For ingress and egress, you can allowlist the sources of traffic like Pods, IP addresses, or ports.

You cannot create a new network policy with the imperative create command. Instead, you will have to use the declarative approach.

```bash
# Declaring a NetworkPolicy with YAML

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: payment-processor
      role: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: coffee-shop
```

A network policy defines a couple of important attributes, which together form its set of rules. The table below shows the attributes on the spec level.

Spec attributes of a network policy

| Attribute   | Description                                                                              |
| ----------- | ---------------------------------------------------------------------------------------- |
| podSelector | Selects the Pods in the namespace to apply the network policy to.                        |
| policyTypes | Defines the type of traffic (i.e., ingress and/or egress) the network policy applies to. |
| ingress     | Lists the rules for incoming traffic. Each rule can define from and ports sections.      |
| egress      | Lists the rules for outgoing traffic. Each rule can define to and ports sections.        |

You can specify ingress and egress rules independently using spec.ingress.from[] and spec.egress.to[]. Each rule consists of a Pod selector, an optional namespace selector, or a combination of both.

Attributes of a network policy to and from selectors

| Attribute                         | Description                                                                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| podSelector                       | Selects Pods by label(s) in the same namespace as the network policy that should be allowed as ingress sources or egress destinations |
| namespaceSelector                 | Selects namespaces by label(s) for which all Pods should be allowed as ingress sources or egress destinations                         |
| namespaceSelector and podSelector | Selects Pods by label(s) within namespaces by label(s)                                                                                |

**Visualizing network policies**
Defining the rules of network policies correctly can be challenging. The page networkpolicy.io provides a visual editor for network policies that renders a graphical representation in the browser.

**Applying Default Network Policies**
The principle of least privilege is a fundamental security concept, and it’s highly recommended when it comes to restricting Pod-to-Pod network traffic in Kubernetes. The idea is to initially disallow all traffic and then selectively open up only the necessary connections based on the application’s architecture and communication requirements.

You can lock down Pod-to-Pod communication with the help of a default network policy. Default network policies are custom policies set up by administrators to enforce restrictive communication patterns by default.

The example below defines a default network policy that denies all ingress and egress network traffic in the namespace.

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: internal-tools
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

# The curly braces for spec.podSelector mean “apply to all Pods in the namespace.”

# Defines the types of traffic the rule should apply to, in this case ingress and egress traffic.

# The network policy prevents any network communication between the Pods in the internal-tools namespace
```

With those default deny constraints in place, you can define more detailed rules and loosen restrictions gradually. Network policies are additive. It’s common practice to now set up additional network policies that will open up directional traffic, but only the ones that are really required.

**Restricting Access to Specific Ports**
Controlling access at the port level is a critical aspect of network security in Kubernetes. If not explicitly defined by a network policy, all ports are accessible, which can pose security risks. For instance, if you have an application running in a Pod that exposes port 80 to the outside world, leaving all other ports open widens the attack vectors unnecessarily. Port rules can be specified for ingress and egress as part of a network policy.

```bash
# Definition of a network policy allowing ingress access on port 80

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: port-allow
  namespace: internal-tools
spec:
  podSelector:
    matchLabels:
      app: api
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: consumer
    ports:
    - protocol: TCP
      port: 80

# Only allows incoming traffic on port 80

# When defining network policies, only allow those ports that are required for implementing your architectural needs. All other ports should be locked down.
```

### Troubleshooting

The Troubleshooting domain of the exam focuses on diagnosing and resolving issues across the entire Kubernetes stack, from cluster-level problems to application-specific failures. Key competencies include identifying and fixing problems with cluster components, troubleshooting node failures and resource constraints, monitoring resource utilization for both cluster infrastructure and applications, analyzing container logs and output streams for debugging, and resolving service connectivity and networking issues such as DNS problems, network policies, and service endpoint configurations.

**Troubleshooting Applications**
**Troubleshooting Pods**
In most cases, creating a Pod is no issue. You simply emit the run, create, or apply commands to instantiate the Pod. If the YAML manifest is formed properly, Kubernetes accepts your request, so the assumption is that everything works as expected. To verify the correct behavior, the first thing you’ll want to do is to check the Pod’s high-level runtime information. The operation could involve other Kubernetes objects like a Deployment responsible for rolling out multiple replicas of a Pod.

**Retrieving High-Level Information**
After working with Kubernetes for a while, you’ll automatically recognize common error conditions. The table below lists some of those error statuses and explains how to fix them.

Common Pod error statuses

|                           Status | Root cause                                                   | Potential fix                                                                                                                                   |
| -------------------------------: | :----------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| ImagePullBackOff or ErrImagePull | Image could not be pulled from registry.                     | Check correct image name, ensure the image exists in the registry, verify network access from the node to the registry, and ensure proper auth. |
|                 CrashLoopBackOff | Application or command run in container crashes.             | Check the container's command/entrypoint and logs; verify the image can run locally (e.g., with Docker) and fix application errors.             |
|       CreateContainerConfigError | ConfigMap or Secret referenced by container cannot be found. | Verify the correct name of the ConfigMap/Secret and ensure the configuration object exists in the same namespace.                               |

**Inspecting Events**
You might not encounter any of those error statuses. But there’s still a chance of the Pod having a configuration issue. You can retrieve detailed information about the Pod using the kubectl describe pod command to inspect its events.

Another helpful command is kubectl get events. The output of the command lists the events across all Pods for a given namespace. You can use additional command-line options to further filter and sort events:

```bash
kubectl get events
```

Sometimes troubleshooting won’t be enough. You may have to dig into the application runtime behavior and configuration in the container.

**Troubleshooting Containers**
You can interact with the container for a deep-dive into the application’s runtime environment.
The next sections will discuss how to inspect logs, open an interactive shell to a container, and debug containers that do not provide a shell.

**Inspecting Logs**
When troubleshooting a Pod, you can retrieve the next level of details by downloading and inspecting its logs. You may or may not find additional information that points to the root cause of a misbehaving Pod, but it’s definitely worth a look.

The logs command provides two helpful options. The option -f streams the logs, meaning you’ll see new log entries as they’re being produced in real time. The option --previous gets the logs from the previous instantiation of a container, which is helpful if the container has been restarted.

**Opening an Interactive Shell**
If any of the previous commands don’t point you to the root cause of the failing Pod, it’s time to open an interactive shell to a container. Application developers will know best what behavior to expect from the application at runtime. Inspect the running processes by using the Unix or Windows utility tools, depending on the image run in the container.

The exec command opens an interactive shell to troubleshoot in a hands-on fashion.

**Interacting with a Distroless Container**
Some images run in containers are designed to be very minimal for security reasons. For example, the Google distroless images don’t have any Unix utility tools preinstalled. You can’t even open a shell to a container, as it doesn’t come with a shell.

Kubernetes offers the concept of ephemeral containers. Those containers are meant to be disposable and have no resilience features like probes. You can deploy an ephemeral container for troubleshooting minimal containers that would usually not allow the use of the exec command.

The debug command can inject an ephemeral container to a running Pod for debugging purposes.

The following command adds the ephemeral container running the image busybox to the Pod named minimal-pod and opens an interactive shell for it:

```bash
kubectl debug -it minimal-pod --image=busybox:1.37.0
```

**Troubleshooting Services and Networking**
A Service provides a unified network interface for Pods.

Service and networking problems in Kubernetes are among the most challenging issues to diagnose because they involve multiple layers: Pod networking, service discovery, DNS resolution, network policies, and ingress controllers. These issues typically manifest as connection timeouts, refused connections, or intermittent failures that can cripple application functionality.

**Diagnosing Service-to-Pod Label Selection**
In case you can’t reach the Pods that should map to the Service, start by ensuring that the label selector matches with the assigned labels of the Pods. You can query the information by describing the Service and then render the labels of the available Pods with the option --show-labels.

```bash
kubectl describe service myservice

kubectl get pods --show-labels
```

**Diagnosing Service-to-Pod Port Mapping**
Services must correctly map ports between the Service definition and the selected Pod’s containers. Check if the port mapping from the target port of the Service to the container port of the Pod is configured correctly. Both ports need to match, or the network traffic wouldn’t be routed properly:

```bash
kubectl get service myapp -o yaml | grep targetPort:

kubectl get pods myapp-68bf896d89-qfhlv -o yaml | grep containerPort:
```

**Inspecting the Service’s Endpoints**
A straightforward way to check if label selection and port mapping are set up properly is to inspect the Service’s endpoints. Service endpoints are API objects that represent the network addresses (IP addresses and ports) of the Pods backing a Service.

Use the get endpoints command to render a Service’s endpoints. Ensure that the output lists the number of Pod IP addresses and container ports expected to be selected by the Service:

```bash
kubectl get endpoints myservice
```

If the output of the command does not render any endpoints, then you know that either label selection or port mapping has not been configured correctly in the Service.

**Verifying Accessibility Scope**
Different service types (ClusterIP, NodePort, LoadBalancer) have different accessibility scopes. By default, the Service type is ClusterIP, which means that a Pod can be reached through the Service only if queried from inside of the cluster.

**DNS Resolution Problems**
Instead of using the cluster-internal IP address to access a ClusterIP Service, you will likely want to use the Service’s DNS name instead:

```bash
kubectl run tmp --image=busybox:1.37.0 -it --rm -- wget \
  myservice.default.svc.cluster.local:80
```

DNS issues can prevent Pods from resolving service names, causing application failures. It’s possible that the CoreDNS Pods are currently not running. You can check by executing the following command:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns

# To scan the CoreDNS logs for error messages, run the following command:

kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50
```

CoreDNS logs reveal DNS-related problems through various error patterns that help identify the root cause of service discovery failures. Common issues include upstream connectivity problems where CoreDNS cannot reach external DNS servers, resolution failures for Services that don’t exist or are incorrectly referenced, communication breakdowns with the Kubernetes API server preventing Service discovery, configuration problems causing routing loops or processing errors, and performance issues from either excessive query loads or insufficient resources. By recognizing these patterns in the logs, you can quickly determine whether the problem lies in network connectivity, configuration, or resource allocation.

**Network Policy Restrictions**
Network policies in Kubernetes act as a firewall at the Pod level, controlling which Pods can communicate with each other and what external traffic is allowed. When network policies are implemented, they follow a “default deny” model—once any network policy selects a Pod, that Pod becomes isolated and can only receive traffic explicitly allowed by policies.

This security feature, while essential for production environments, frequently causes mysterious connectivity issues where applications that previously worked suddenly experience timeouts or connection refused errors. Common symptoms include Pods being unable to reach services they depend on, health checks failing, cross-namespace communication breaking, or external traffic being blocked unexpectedly.

The challenge with troubleshooting network policies lies in their implicit nature—there’s no immediate error message indicating that a network policy is blocking traffic; instead, connections simply fail silently.

Debugging requires systematically checking if network policies exist in the namespace, understanding which Pods they select through their podSelector, examining the ingress and egress rules to see what traffic is permitted, and testing connectivity from different source Pods to identify exactly where the policy enforcement is occurring.

**Inspecting Resource Metrics**
Deploying software to a Kubernetes cluster is only the start of operating an application long-term. Developers need to understand their applications’ resource consumption patterns and behaviors, with the goal of providing a scalable and reliable service.

In the Kubernetes world, monitoring tools like Prometheus and Datadog help with collecting, processing, and visualizing information over time. The exam does not expect you to be familiar with third-party monitoring, logging, tracing, and aggregation tools; however, it is helpful to have a basic understanding of the underlying Kubernetes infrastructure responsible for collecting usage metrics. The following are examples of typical metrics:

- Number of nodes in the cluster
- Health status of nodes
- Node performance metrics such as CPU, memory, disk space
- Pod-level performance metrics such as CPU and memory consumption

This responsibility falls to the Metrics Server, a cluster-wide aggregator of resource usage data.
kubelets running on nodes collect metrics and send them to the Metrics Server.

The Metrics Server stores data in memory and does not persist data over time. If you are looking for a solution that keeps historical data, then you need to look into commercial or self-hosted options.

With Metrics server installed, you can now query for metrics of cluster nodes and Pods with the top command:

```bash
kubectl top nodes

kubectl top pod frontend
```

It takes a couple of minutes after the installation of the Metrics Server before it has gathered information about resource consumption. Rerun the kubectl top command if you receive the error message error: Metrics API not available.

**Troubleshooting Clusters**
When a Kubernetes cluster experiences infrastructure-level failures, the impact can be catastrophic. Pods refuse to schedule, applications become unreachable, and entire nodes disappear from the cluster, potentially affecting hundreds of workloads simultaneously. Unlike application-specific issues that might affect a single service, problems with cluster components and nodes strike at the very foundation of your Kubernetes environment, making their swift diagnosis and resolution critical for maintaining system availability.

- Troubleshoot clusters and nodes
- Troubleshoot cluster components

**Inspecting the Status of Cluster Components**
Among those components available on the control plane node are the following:

- kube-apiserver: Exposes the Kubernetes API used by clients like kubectl for managing objects
- etcd: A key-value store for storing the cluster data
- kube-scheduler: Selects nodes for Pods that have been scheduled but not created
- corde-dns: Serves as the cluster’s DNS server, automatically creating DNS records that allow Pods and Services to discover each other by name rather than IP addresses, while also handling external DNS resolution for workloads
- kube-controller-manager: Runs controller processes (e.g., the job controller responsible for Job object execution)
- cloud-controller-manager (optional): Links cloud provider–specific APIs to the Kubernetes cluster. This controller is not available in on-premise cluster installations of Kubernetes.

In addition to the components running specifically on the control plane nodes, every node (control plane and worker nodes) can contain the following components:

- kubelet: Ensures that all Pods are running, including their containers
- kube-proxy (optional): Maintains the network rules on nodes to implement Services
- Container runtime: The software component responsible for running containers

To discover those components and their status, list the Pods available in the namespace kube-system. Be aware that some components do not run in a Pod, e.g., the kubelet or the container runtime.

```bash
kubectl get pods -n kube-system
```

Any status that does not show Running should be inspected further. You can retrieve the logs for control plane component Pods in the same fashion you do for any other Pod, using the logs command. The following command downloads the logs for the kube-apiserver component:

```bash
kubectl logs kube-apiserver -n kube-system
```

**Troubleshooting Node Issues**
Control plane nodes are the critical components for keeping a cluster operational.

```bash
# Rendering Cluster Information
kubectl cluster-info

# For a detailed view of the cluster logs, append the dump subcommand.
kubectl cluster-info dump

```

**Node Showing NotReady Status**
Worker nodes are responsible for managing the workload. Make sure you have a sufficient number of worker nodes available to distribute the load.

```bash
# Checking Available Resources
kubectl describe node worker-1

# To check on memory and the number of processes running, use the top command:
top

# To check on the available disk space, use the command df:
df -h

# Checking the kubelet Process
systemctl status kubelet

# Use journalctl to take a look at the log files of the process:
journalctl -u kubelet.service

# You will want to restart the process once you have identified the issue in the logs and fixed it:
systemctl restart kubelet
```

**Checking Certificate Validity**
Sometimes, the certificate used by the kubelet can expire. Make sure that the values for the attributes Issuer and Not After are correct:

```bash
openssl x509 -in /var/lib/kubelet/pki/kubelet.crt -text

# For a quick rundown of all certificates in the cluster’s Public Key Infrastructure (PKI), you can use the following command:
kubeadm certs check-expiration

# You can renew all certificates necessary to run the control plane with the following command:
kubeadm certs renew all
```

You must restart the kube-apiserver, kube-controller-manager, kube-scheduler, and etcd, so that they can use the new certificates.

**Checking the kube-proxy Pod**
The kube-proxy components run in a set of dedicated Pods in the namespace kube-system. You can clearly identify the Pods by their naming prefix kube-proxy and the appended hash. Verify if any of the Pods states a different status than Running. Each of the kube-proxy Pods runs on a dedicated worker node. You can add the -o wide option to render the node the Pod is running on in a new column:

```bash
kubectl get pods -n kube-system
```
