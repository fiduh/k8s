# X.509 certificate management for Kubernetes with Cert-Manager

cert-manager can obtain certificates from a variety of certificate authorities (public/private Issuers), including: Let's Encrypt, HashiCorp Vault, CyberArk Certificate Manager and private PKI.

### Why cert-manager is needed:

Manual creation of TLS certificates in prod is not encouraged

- Certificates expire: every few months they need to be renewed and doing this manually is risky, your entire application can go offline if you forget.
- Scaling issues: if you have multiple microservices, each with their own hostname, that means multiple certificates, managing them manually is not a practical solution.
- Security: Manually generating certificates means private keys often sit on your local machines, which is not a safe practice.
- Automate: if something repeats, automate. Repetitive tasks shouldn't depend on humans.

Cert Manager automates certificate creation, renewal, rotation, secrets update and integration with Gateway API and Ingress. Cert manager uses cluster issuer who issues the certificate, certificate object(what you want issued) and secrets where your TLS secrets are stored.

### Install cert manager with Helm

[Installing with Helm](https://cert-manager.io/docs/installation/helm/)

```bash
# The latest cert-manager chart is available at the following location:
oci://quay.io/jetstack/charts/cert-manager:v1.20.2

# verify the signature on the chart too, which requires the GPG keyring to be downloaded from this website first.

curl -LO https://cert-manager.io/public-keys/cert-manager-keyring-2021-09-20-1020CF3C033D4F35BAE1C19E1226061C665DF13E.gpg

helm install \
  cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --version v1.20.2 \
  --namespace cert-manager \
  --create-namespace \
  --verify \
  --keyring ./cert-manager-keyring-2021-09-20-1020CF3C033D4F35BAE1C19E1226061C665DF13E.gpg \
  --set crds.enabled=true

```

### Create (cluster)Issuer to automatically generate TLS certificates and avoid rotation headaches.

The first thing you'll need to configure after you've installed cert-manager is an **Issuer** or a **ClusterIssuer**. These are resources that represent certificate authorities (CAs) able to sign certificates in response to certificate signing requests.

Once an Issuer has been configured, you're ready to issue your first certificate!

There are several use cases and methods for requesting certificates through cert-manager:

- [Securing Ingress Resources: A method to secure ingress resources in your cluster.](https://cert-manager.io/docs/usage/ingress/)
- [Enable mTLS on Pods with CSI: Using the cert-manager CSI driver to provide unique keys and certificates that share the lifecycle of pods.](https://cert-manager.io/docs/usage/csi/)
- [Securing Istio Gateway: Secure your Istio Gateway in Kubernetes using cert-manager.](https://istio.io/docs/tasks/traffic-management/ingress/ingress-certmgr/)
- [Securing Istio Service Mesh: Using the cert-manager Istio integration, secure the mTLS PKI for each pod through cert-manager managed certificates.](https://cert-manager.io/docs/usage/istio-csr/)

**Certificate Resource**
[Creating Certificate Resources](https://cert-manager.io/docs/usage/certificate/)

### Enable HTTPS on Gateway API using OpenSSL certificate.

[Deploy cert-manager on AWS Elastic Kubernetes Service (EKS) and use Let's Encrypt to sign a TLS certificate for an HTTPS website](https://cert-manager.io/docs/tutorials/getting-started-aws-letsencrypt/)

**Create Pod Identity Association for cert manager service account and Route53 IAM role**

**Create a ClusterIssuer for Let's Encrypt Staging**
A ClusterIssuer is a custom resource which tells cert-manager how to sign a Certificate. In this case the ClusterIssuer will be configured to connect to the Let's Encrypt staging server, which allows us to test everything without using up our Let's Encrypt certificate quota for the domain name.

```bash
# clusterissuer-lets-encrypt-staging.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: ${EMAIL_ADDRESS}
    profile: tlsserver
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - dns01:
        route53:
          region: ${AWS_DEFAULT_REGION}
          role: arn:aws:iam::${AWS_ACCOUNT_ID}:role/cert-manager-acme-dns01-route53
          auth:
            kubernetes:
              serviceAccountRef:
                name: cert-manager-acme-dns01-route53

```

**_Annotated Gateway resource to automatically generate certificates using the (Cluster)Issuer_**
[generate TLS certificates for Gateway resources](https://cert-manager.io/docs/usage/gateway/)
cert-manager can generate TLS certificates for Gateway resources. This is configured by adding annotations to a Gateway and is similar to the process for Securing Ingress Resources.

```bash
# The annotations cert-manager.io/issuer or cert-manager.io/cluster-issuer tell cert-manager to create a Certificate for a Gateway. For example, the following Gateway will trigger the creation of a Certificate with the name letsencrypt-staging:

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example
  annotations:
    cert-manager.io/issuer: letsencrypt-staging
spec:
  gatewayClassName: foo
  listeners:
    - name: http
      hostname: example.com
      port: 443
      protocol: HTTPS
      allowedRoutes:
        namespaces:
          from: All
      tls:
        mode: Terminate
        certificateRefs:
          - name: letsencrypt-staging

# A few moments later, cert-manager will create a Certificate. The Certificate is named after the Secret name letsencrypt-staging. The dnsNames field is set with the hostname field from the Gateway spec.

```

**Create a production ready certificate**
Now that everything is working with the Let's Encrypt staging server, we can switch to the production server and get a trusted certificate.

Create a Let's Encrypt production Issuer by copying the staging ClusterIssuer YAML and modifying the server URL and the names, then apply it:

```bash
# clusterissuer-lets-encrypt-production.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: $EMAIL_ADDRESS
    profile: tlsserver
    privateKeySecretRef:
      name: letsencrypt-production
    solvers:
    - dns01:
        route53:
          region: ${AWS_DEFAULT_REGION}
          role: arn:aws:iam::${AWS_ACCOUNT_ID}:role/cert-manager-acme-dns01-route53
          auth:
            kubernetes:
              serviceAccountRef:
                name: cert-manager-acme-dns01-route53

```

### Developers can create their own certificates without relying on the Cluster operators.

In cert-manager, the typical operator-managed flow is:

- Operator installs cert-manager, creates ClusterIssuers (or namespace-scoped Issuers), and those are cluster-wide/namespace-wide resources that developers can reference or simply annotates Gateway object to auto generate certificates.
- Developers can also self-service and auto generate certificates by annotating HTTPRoute object.

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: my-team-ns
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod

# cert-manager will then issue a cert scoped to that route. So developers aren't blocked waiting for the operator to wire up TLS — they just annotate their own HTTPRoute and cert-manager handles the rest, as long as a ClusterIssuer exists
```
