# GitOps

**_GitOps uses GIT as the single source of truth to deliver applications and infrastructure._**

**GitOps Principles**

- Declarative:
  - A system managed by GitOps must have its desired state expressed declaratively (e.g, Kubernetes YAML manifest to manage deployments, namespaces, Ingress, etc.). Whatever is committed to your git repo is the same thing in your cluster.
- Versioned and Immutable:
  - Desired state is stored in a way that enforces immutability, versioning and retains a complete version history.
- Pulled Automatically:
  - Software agents automatically pull the desired state declarations from the source.
- Continuously reconciled:
  - Software agents continuously observe actual system state and attempt to apply the desired state.

This comes with a lot of security advantages, nobody except the GitOps controller is managing your entire Kubernetes cluster. Any unwanted change is overridden by GitOps controller.

**Architecture of ArgoCD**
ArgoCD is a GitOps continues delivery tool for Kubernetes. It has a controller that tracks your Git Repo(Source of Manifests) and keeps it in sync with your destination cluster(s).

When ArgoCD is install on a k8s cluster, all it's components run as Pods.

ArgoCD consists of 3 main components:

- Repo Server: Connects to git, clones remote git repos and generates the needed k8s manifests.
- Application Controller: It's a Kubernetes controller which continuously monitors running applications and compares the current live state against the desired target state.
  - Communicates with the Repo server to get the generated manifests.
  - Communicates with the k8s API to get the actual cluster state.
  - Deploys apps manifests to destination clusters.
  - Detects OutofSync Apps and takes corrective actions(if needed).
  - Invokes user-defined hooks for lifecycle events (PreSync, Sync, PostSync).
- API Server (API + Web Server):
  It is the only component that you will have to interact with, it exposes an API(gRPC/REST) consumed by the Web UI, CLI that's responsible for:
  - application management (Create, Update, Delete)
  - Application operation (ex: Sync, Rollback)
  - Repos and cluster managements.
    Authentication.
    - API Server provides UI/CLI used by users to communicate with ArgoCD. API server should handle authentication, it has the capabilities of single sign on, has the capabilities of integration with your existing OIDC providers. ArgoCD comes with **Dex** servers, Dex is a very light weight OIDC proxy server which can connect with any of your existing providers, consider it as SSO capability for API Server. **Redis** is used for caching, it saves state, in case of cluster crash. **ApplicationSet Controller** automates the generation of ArgoCD Applications.

### Core Concepts

- **Application**: In order to use ArgoCD and start deploying applications, you have to define an **_application_**. (Defines source and destination to deploy group of k8s resources). Application consist of two things:
  - Source: This is your source of manifests. ArgoCD supports Helm charts, Kustomize application, Directory of Yaml files and Jsonnet tools
  - Destination: Cluster and namespace.

- **Project**: provides a logical grouping of applications. Suppose you have application 1,2,3 and 4, all considered to be related to one project. ArgoCD creates a default project. It's mandatory to specify a project for these applications.
  - Projects are useful when ArgoCD is used by multiple teams.
    - You can allow or restrict the sources that a project can use.
    - Allow apps to be deployed into specific destination (clusters) and namespaces.
    - Specify which resources are allowed to be deployed. e.g, deployments, Statefulsets, etc.

- **Desired state**: k8s manifests stored in git, written as bare YAML, Helm or Kustomize.
- **Actual state**: what is actually running in your destination (cluster and namespace)

- **Sync**: The process of making desired state = actual state

- **Refresh or Compare**:
  - Compare the latest code in Git with the live state
  - ArgoCD automatically refreshes every 3 minutes.

### Installation options:

**_Pre-requisite_**

- You need a running k8s cluster (vCluster, EKS, Kind, etc.)

**Installation options:**

- Non High availability setup
  - Suitable for evaluation or dev/testing environments. ArgoCD installs one Pod for each component only.

- High availability setup
  - Recommended for production.
  - You need at least 3 worker nodes in your k8s cluster to achieve this, because ArgoCD distributes the Pods into different nodes.

- Light installation "Core"
  - Suitable if ArgoCD is used by administrators only. UI and API server is not installed for end users.
  - By default it's installed as Non-HA.

**Privileges Options**:
ArgoCD provides two options for in-cluster privileges.

- Cluster-admin privileges: where ArgoCD has the cluster-admin access to deploy into the cluster that it runs in.
- Namespace level privileges: Use this manifest set if you do not need ArgoCD to deploy applications in the same cluster that ArgoCD runs in.

- ArgoCD provides two options for in-cluster privileges.

**_Manifests Installation Options_**

- YAML Manifests:
  [argoCD Installation](https://argo-cd.readthedocs.io/en/stable/getting_started/)

- You can install argoCD using helm.
  [Install argoCD using Helm](https://github.com/argoproj/argo-helm)

[argoCD Example Apps](https://github.com/argoproj/argocd-example-apps)

- Kustomize

**_Get Initial Admin Password_**

- Get the initial admin password that is stored as a secret in ArgoCD namespace.
- Convert the secret from base64 into plain text.

```bash
kubectl get secret -n argocd

kubectl get secret -n argocd argocd-initial-admin-secret -o yaml

echo <password> | base64 --decode
```

**_Accessing ArgoCD server (API + UI)_**

- By default ArgoCD server is not exposed with external endpoint.
  Expose by using three options:
  - Service(LoadBalancer)
    - Change the argocd-server service type to LoadBalancer.
  - Ingress: Use your preferred ingress controller / Gateway API
    - Create an ingress resource that point into argocd-server service.
  - Port-forward: simply use this to access locally on your machine
    - kubectl port-forward svc/argocd-server -n argocd 8080:443

### ArgoCD Application

Application is a Kubernetes resource object representing a deployed application instance in an environment. It's defined and stored in the etcd database in Kubernetes.
Create ArgoCD applications declaratively "YAML". (Recommended). You can store these files into your git repository the same way as your application manifests.

```bash
# Argo application object "CRD"

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook. # Name of application.
  namespace: argocd
spec:
  project: default
  source:   #Source of manifests
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
    #Tools (Directory of files(YAML), Helm, Kustomize). Each Tool has it's own options. ArgoCD can also auto detect tools, without you specifying a tool.
    directory: # Default tools
      recurse: true #deploys and sync all the files and all subdirectories.
    #helm:
      #releaseName: guestbook
    #kustomize:
      #version: v3.5.4
  destination:   #Destination cluster
    server: https://kubernetes.default.svc
    namespace: guestbook
  syncPolicy:
    syncOption:
      - CreateNamespace=true
```

**Multiple Sources for an application**
Before version 2.6 in ArgoCD, you can specify a single source per application, where this source is a git repo or helm repo. We can specify the source using the source field in the application kind.
In the ArgoCD 2.6 version, there is a new feature where we can use multiple sources per application. So we can specify resources stored in multiple repositories whether the repo is git or helm. We can group the resources together in one application from multiple repositories.

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: multiple-sources
  namespace: argocd
spec:
  project: default
  sources:
    - chart: redis
    repoURL: https://charts.bitnami.com/bitnami
    targetRevision: 17.10.2
    - chart: prometheus-redis-exporter
    repoURL: https://prometheus-community.github.io/helm-charts
    targetRevision: 5.3.2
  destination:
    server: https://kubernetes.default.svc
    namespace: default

```

### Projects

**MultiCluster Deployment**
