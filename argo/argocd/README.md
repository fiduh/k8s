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

- Repo Server: Connects to git and the the state
- Application Controller: Connects to Kubernetes and gets the state
- API Server: Provides UI/CLI used by users to communicate with ArgoCD. API server should handle authentication, it has the capabilities of single sign on, has the capabilities of integration with your existing OIDC providers. ArgoCD comes with Dex servers, Dex is a very light weight OIDC proxy server which can connect with any of your existing providers, consider it as SSO capability for API Server. Redis is used for caching, it saves state, in case of cluster crash

[argoCD Installation](https://argo-cd.readthedocs.io/en/stable/getting_started/)
You can install argoCD using helm.
[Install argoCD using Helm](https://github.com/argoproj/argo-helm)

[argoCD Example Apps](https://github.com/argoproj/argocd-example-apps)

**MultiCluster Deployment**
