
### CNI Plugins
CNIs are third-party Network Plugins that provide critical networking functionalities for Kubernetes clusters. These include creating and managing network interfaces for containers, allocating IP addresses to pods, and enabling communication between pods across different nodes in the cluster.

**Understanding and Implementing CNIs in Kubernetes**
```Weave - Helps enforcement of Network Policies
Weave is a networking solution designed for containerized applications. It provides a networking model that is scalable and works across multiple clouds and data centers.
One of the key benefits of Weave CNI is it supports network policies. Network policies are a set of rules that govern how traffic is allowed between pods in a cluster.
Using network policies, you ensure that only authorized traffic is allowed to flow within the cluster. This helps to improve security and ensure that your applications are isolated from one another.

Installing Weave
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
```
Flannel - Flannel does not support Network Policies
Flannel is a CNI that is used to provide networking between containers in cluster. Flannel is primarily focused on establishing the pod network.
However, Flannel by itself does not contain the mechanisms that will enforce network policies. To implement network polcies, we also need to add additional tools such as Calico or Canal. These CNIs integrate with Kubernetes to provide a complete networking and security.

Installing Flannel
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

**Summary: Flannel vs Weave for Network Policies**
When it comes to network policies, Weave has a clear advantage as it provides built-in support. Flannel, on the other hand, requires an additional third-party tool to enforce network policies.

Choosing between Flannel and Weave depends on your specific needs. If network policies are a crucial part of your Kubernetes security strategy and you prefer a simple built-in solution, then Weave is a good choice. However, if you are focused on simplicity and are willing to integrate additional tools for network policy support, then Flannel may be suitable.

### Default Deny All Ingress Trafic Policy

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress-null
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
### Allow all ingress traffic Policy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:
  - {}
  policyTypes:
  - Ingress
```
### Building on Top of a Default Deny Policy

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: frontend-to-middleware
    namespace: default
spec:
    podselector:
        matchlabels:


### Basic Network Troublshooting and checking connectivity
***UseNetcat command****
Basic Commands: 
nc 
The -z flag can be used to tell nc to report open ports, rather than initiate a connection
v' Have nc give more verbose output
-w timeout

```
#Check connectivity from one pod to another using the service
kubectl exec frontend-deploy-744dddcc79-lmkzd -- nc -zv -w1 middleware-svc 80

controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 middleware-svc 80
middleware-svc (10.105.169.89:80) open

controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 mysql-svc 3306
nc: mysql-svc (10.96.106.81:3306): Operation timed out
command terminated with exit code 1

controlplane ~ ✖ kubectl exec middleware -- nc -zv -w1 mysql-svc 3306
mysql-svc (10.96.106.81:3306) open
```
### Network Polciy Using PodSelctor

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-middleware
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: middleware
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
```

### Namespace-Based Isolation with Kubernetes Network Policies


```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-alpha-logger
  namespace: alpha-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              function: logging
```
*** Single network policy called allow-alpha-logger that would allow all pods from the alpha-logger namespace to the alpha-prod namespace ***
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-beta-logger-1
  namespace: beta-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              function: logging
        - podSelector:
            matchLabels:
              role: logger-1
```

*** Single network policy called darkarts-magic in the darkarts that will allow only ingress trafic from dumbledore pod and egress to TCP/UDP port 53 ***

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: darkarts-magic
  namespace: darkarts
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector:
        matchLabels:
          i-am: dumbledore
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: default
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53

```