#### Create Namespaces for seperation of concerns.

Implementing Pod Security Standards (PSS) with Pod Security Admission (PSA)
PSA is a built in admission controller that implements the the security controls outlined in the PSS
As an alternative to PSA, you can use Policy-as-Code (PaC) open-sourse solutions.

PSS define three different policies to broadly cover the security spectrum. These policies are cumulative and range from highly-permisive to highly-restrictive.

Policy levels:
    - Privileged: Unrestricted policy, providing the wildest possible level of permissions. This policy allows for known privilege escalations.
    - Baseline: Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) pod configuration.
    - Restricted: Heavily restricted policy, following current pod hardening best practices.

PSA admission controller implements the controls, outlined by the PSS policies, via three modes of operation, which are provided in the following list:
    - enforce: Policy violations will cause the pod to be rejected.
    - audit: Policy violations trigger the addition of an audit annotation to the event recorded in audit log, but are otherwise allowed
    - warn: Policy violations will trigger a user-facing warning, but are otherwise allowed.

PSA labels for Namespaces


#### Network Policy
[Network Policy Editor by - ISOVALENT](https://editor.networkpolicy.io/)
OSI Layer 3&4 Network rules using IP Addresses and Ports, It's a builtin kubernetes feature, but it's implementation part of it lies in the CNI which is installed in your cluster. A CNI like Cilium gives you the capabilitie to create custom resources to control layer 7 (HTTP) traffic.

 ```bash
 kubectl get networkpolicy -A
 ```

By default: all communication (to and from Pods) is allowed in a kubernetes cluster even across namespaces and each pod gets it's own IP Address. This might pose a challenge in a multi-tenent cluster.

Network policy component is the way to control traffic flow at the IP address and port level.
It sets rules for Pods to communicate, these are strict rules for inter cluster communication.

Senario we are solving for.
 - FRONTEND send ingress traffic to WORKER, WORKER can receive ingress traffic from only FRONTEND and send ingress traffic to DATABASE, DATABASE can receive ingress traffic from only WORKER and block all egress traffic. 

 ```bash

 kubectl apply -f security/network-policy/.

 kubectl exec <PodName> -- sh -c 'nc -v <PodIP PORT>'
 ```

 kubectl exec backend-pod -- sh -c 'nc -v 100.44.0.1 6379'