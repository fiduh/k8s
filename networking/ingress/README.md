## Gateway API - Next generation of Kubernetes Ingress, successor to the Ingress API

#### Kubernetes service networking (North-South(External Client to Services in the cluster), East-West(Service to Service))

Why Gateway API:

- Before gateway API, there was ingress, lets first understand what job ingress was doing in kubernetes and whay gateway API is taking its place.
- Ingress is the way services in a kubernetes cluster are exposed to external traffic, but it has it's draw backs;
  - Kubernetes doesn't have a native loadbalancer component, so they came up with the ingress API concept, where vendors/maintainers like (Nginx, AWS, etc.) who already have loadbalancing products, can implement the Ingress controllers, so their loadbalancers can be used in kubernetes.
  - Here's the work flow for using ingress, platform engineer deploys the ingress contoller, devops engineer creates an ingress resource for services they want to expose, ingress controller watches the ingress resource and according to its configuration, ingress controller creates a loadbalancer, using the loadbalancer external clients can reach the service (The Load balancer routes traffic to the relevant services).
  - Kubernetes solved the problem of North - South Traffic using ingress.
  - Drawbacks with ingress:
    - Limited scope of ingress, if you look at the ingress resource, there are only two things you can mention in the specification (spec). 1. The service which should be exposed to the load balancer, 2. The path on which the end user or client can access the service. This simply means through ingress resource kubernetes can tell the loadbalancer, allow request to the e.g login microservice, only if the user/client access the service on the path /login. But this is very basic and applications developers demand more for production level workloads, such as Rate limiting, Web application firewall (WAF), http redirect, canery deployment etc. But the kubernetes Ingress API spec does'nt support any of these.
    - To drive the point home here's an example: Let's say there's an organization who run their services on the AWS echo system, the platform engineer installs ALB controllers(AWS implementation of the ingress controller), the DevOps engineer of service (eg. Login service) creates an ingress resource with spec, where the devops engineer defines login service should be exposed to the internet on path /login, ingress controller watches this specification and it creates a loadbalancer, the end users are happy because through the loadbalancer they can access the login service, but now the devops engineer wants to implement canary model of deployment, WAF, but unfortunately these are not supported in the ingress spec. Note! this draw back is not with the load balancer, majority of the loadbalancers support basic features like canary deployment, integrate with WAF, etc. The problem here is with the ingress spec. That's why the loadbalancer companies created workarounds in the ingress controllers, they started supporting anotations/CRDS. Nginx ingress controllers, ALB ingress controllers, Traofik, etc. started supporting anotations because the spec was limited.
    - Annotations add non standardized complexity because the different ingress controllers implement these in their own ways. Number of annotations grew exponentially (example is the ALB controller has about 30-40 annotations). This makes production grade ingress resource look very complex. When you change to a new controller the annotaions can also change (Lack of portability).
    - Kubernetes spec is very limited, all it says is which service can be exposed and on which path the service should be exposed, it cannot implement rate limiting, WAF, Canary deployment, Natively it cannot implement any production grade features
    - Second drawback: Lack of separation of concerns. Platform engineers, devops enginers and application developers all update the same ingress file. In some companies, ingressClassName is maintained by platform engineers who deployed the ingress controller, the rest of the fields are maintained by devops and application engineers, so there's no clear separation of access.
    - Limited protocol support: it has no native support for non-http protocols like TCP, UDP or GRPC. HTTP is what ingress started with, it was really about webservices which is also the biggest usecase, but there are growing usecases for GRPC, bare TCP and even UDP routing.

Gateway API came into the picture to address these drawbacks of Ingress API spec, Gateway API is solving the same North-South traffic problem ingress solved but by address it's drawbacks.

## How is Gateway API solving these problems

- Gateway API is broken down into 3 different custom resources
  - GatewayClass: When ever the platform engineer deploys a Gateway Controller, e.g, Cilium, they will create a Gateway class resource
  - Gateway: Created by devOps engineers, this resource defines the gateway class to be used
  - Routes Resources: Created by application developers, this resource defines the backend service configuration. That is on which port the service should be accessed, on which path, WAF support, HTTP Redirect, Canary model of deployment are all supported by Routes. Routes is now capable of implementing production grade features natively, replacing complex annotations.
    The Routes resources are what application developers actually care about, it asnswers the questions:
    - Which URL paths do you want to route to which services
    - Do you need to route to services mainted by other teams in namespaces you don't?control?
    - Do you need to reference any TLS certificates or other namespaced resources?

  - There's now clear separation of concerns as Gateway API resources are modeled after roles in organizations, so you have role based access control.
  - Portability: Since we now have standazied CRDs, it becomes easier to switch Gateway API Controllers
  - Native support for non-http protocols like TCP, UDP, etc, available protocols a Gateway controller implementation supports is intended to be extensible, and each supported protocol will have an associated Route resources. e.g. HTTPRoute, TLSRoute, GRPCRoute, TCPRoute, etc.

#### Steps to use Ingress API

```bash
# Ingress TLS terminate
```

```bash
# Ingress TLS Passthrough
```

#### Steps to use Gateway API

- Install Gateway API CRDs + Install a Gateway Controller (e.g. Cilium)

```bash
# Install Cilium in an EKS cluster using helm
# Enable Gateway API Controller and it's CRDs

```

## Gateway API TLS termination at the Gateway level

    - Flow: The platform engineer deploys a Gateway API Controller, GatewayClass is created that points to the deployed controller, Gateway is configured to use the GatewayClass and accept from clients on port 443, the secret hold the TLS certificate info is added to the spec.termination to terminate the TLS Traffic, the issueance of these certificates can be auto mated using Cert Manager or done manually- check {point to the cert manager README} for implementation.
    - Note when TLS is terminated at the Gateway level, the data now becomes plain text in the cluster and this is not considered true E2EE.
    - Certificates are handled at the Gateway

![https://gateway-api.sigs.k8s.io/reference/spec/](Gateway API Reference)

```bash

apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tls-terminate
spec:
  gatewayClassName: cilium
  listeners:
  - name: app-https
    protocol: HTTPS # Other possible value is `TLS`
    port: 443
    hostname: test.acekloudnative.com
    tls:
      mode: Terminate # If protocol is `TLS`, `Passthrough` is a possible mode
      certificateRefs:
      - kind: Secret
        group: ""
        name: default-cert


# To then direct traffic to the different services you can use HTTPRoute

```

#### How routing is handled when using TLS passthrough

## Gateway API (TLS Passthrough) - End to end encryption(E2EE) with TLSRoute

    - Decrypting incoming HTTPS traffic directly within the pod—either by the containerized application itself or a sidecar proxy (e.g., Envoy, Nginx)—ensuring end-to-end encryption from the client to the pod, rather than terminating at a Gateway
    - Flow:

```bash
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: tls-passthrough-gateway
spec:
  gatewayClassName: cilium
  listeners:
  - name: tls
    protocol: TLS
    port: 443
    hostname: test.acekloudnative.com
    tls:
      mode: Passthrough

```

#### How routing is handled when using TLS passthrough

#### Terminating the TLS at the Application / Pod level.

## Gateway API - for East/West (service to service) routing (Mesh Service Binding)

You can implement service mesh features with the same language you did for ingress, colapse alot of the complexities around service mesh implementation into these durable route resources.

- Gateway API handles East/West routing by extending Route ParentRef fields to allow kind: Service instead of assuming kind: Gateway
- Makes it possible to use Route resources to instruct service mesh controllers to route traffic between service facets inside your cluster.

## ReferenceGrants:

The resources used by application teams to coordinate with each other and allow cross-namespace routing. TLS Certs are things that you might want to have access to but from a policy stand point they are controlled in a different namespace, now you can actaually have teams granting access to resources, not just certs, you can grant access to lets say you are allowd to route a url into this other namesapce that I control.
