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
