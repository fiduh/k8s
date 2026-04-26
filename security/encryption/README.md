#### Automate the creation of TLS Certifcates in Kubernetes with Cert-Manager


### Why cert-manager is needed:
Manual creation of TLS certificates in prod is not encouraged
- Certifcates expire: every few months they need to be renewed and doing this manually is risky, your entire application can go offline if you forget.
- Scaling issues: if you have multiple microservices, each with their own hostname, that means mutiple certifcates, managing them manually is not a pratical solution.
- Security: Manually generating certifcates means private keys often sit on your local machines, which is not a safe practice.
- Automate: if something repeats, automate. Repeatitive tasks shouldn't depend on humans.

Cert Manager automates certificate creation, renewal, rotation, secrets update and integration with Gateway API and Ingress. Cert manager uses cluster issuer who issues the certificate, certificate object(what you want issued) and secrets where your TLS secrets are stored.

### Install cert manager

### Create cluster issuer to automatically generate TLS certificates and aviod rotation headaches.

### Enable HTTPS on Gateway API using OpenSSL certificate. 