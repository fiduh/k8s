**CKA Objectives**: **_Typical administrator tasks encountered on the job, more specifically, Kubernetes cluster maintenance, networking, storage solutions, and troubleshooting applications and cluster nodes_**.

**Kubectl** - (for interacting with the Kubernetes cluster)

**Kubeadm** - (for installing a Kubernetes cluster from scratch and upgrading the Kubernetes version of an existing cluster)

**Etcdctl** - (for backing up and restoring the etc database)
Static Pods

### Creating and Managing a Kubernetes Cluster

Set up the underlying infrastructure:

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
