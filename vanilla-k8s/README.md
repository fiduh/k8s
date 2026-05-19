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

### Role-Based Access Control (RBAC)

In Kubernetes, you need to be authenticated before you are allowed to make a request to an API resource.

**RBAC**: defines policies for users, groups, and processes by allowing or disallowing access to manage API resources.

**_RBAC consists of three key building blocks_**:
**Subject**: The user or process that wants to access a resource
**Resource**: The Kubernetes API resource type (e.g., a Deployment or node)
**Verb**: The operation that can be executed on the resource (e.g., creating a Pod or deleting a Service)

![RBAC building blocks](../assets/cka2_0602.png)

**_Users and groups are not stored in etcd, the Kubernetes database, and are meant for processes running outside of the cluster. Service accounts exist as objects in Kubernetes and are used by processes running inside of the cluster._**

**_Kubernetes does not represent a user as with an API resource. The user is meant to be managed by the administrator of a Kubernetes cluster, which then distributes the credentials of the account to the real person or to be used by an external process._**

Calls to the API server with a user need to be authenticated. Kubernetes offers a variety of authentication methods for those API requests.

| Authentication strategy  | Description                                                           |
| ------------------------ | --------------------------------------------------------------------- |
| X.509 client certificate | Uses an OpenSSL client certificate to authenticate                    |
| Basic authentication     | Uses username and password to authenticate                            |
| Bearer tokens            | Uses OpenID (a flavor of OAuth2) or webhooks as a way to authenticate |

The following steps demonstrate the creation of a user that uses an OpenSSL client certificate to authenticate. Those actions have to be performed with the cluster-admin Role object.

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
