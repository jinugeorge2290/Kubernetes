

<h4 align="center"><a href="https://www.cncf.io/certification/cka/">https://www.cncf.io/certification/cka/</a></h3>
<h1 align="center">Certified Kubernetes Administrator (CKA) Notes - Jinu</h1>

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Common Troubleshooting Commands to Remember](#common-troubleshooting-commands-to-remember)
- [TOPIC WISE HINTS](#topic-wise-hints)
  - [Pods and Services](#pods-and-services)
  - [Node Afinitiy](#nodes)
  - [ETCD](#kubernetes-cli-tools)
  - [CRONJOB]()
  - [kubelet]()
- [QUESTIONS](#questions)


## Common Troubleshooting Commands to Remember

**Look at a specific container inside a pod use -c option**

`kubectl logs blue-dp-cka09-trb-xxxx -c init-container`

**Get live logs**

`kubectl logs -f <pod-name>`

**Get logs using the event selector**

`kubectl get event --field-selector involvedObject.name=blue-dp-cka09-trb-xxxxx`

`kubectl get event --field-selector spec.nodeName=<node-name>`

**Sort the events by timecreationstamp**

`kubectl get events --sort-by='.metadata.creationTimestamp' -A`

**Use journactl to  look into the service logs in detail**

`journalctl -u kubelet.service -f`\
`journalctl -u kubelet.service -f | grep -v 'connect: connection refused'`\
`journalctl -r kubelet.service` --> To view logs in a reverese order\
`journalctl -f kubelet.service` --> Help wit live log output on the terminal

`journalctl -u <service name> -n <number of lines> -f`\
Example: journalctl -u kubelet.service -n 100 --no-pager\
`journalctl --unit=kubelet.service | tail -n 300`\
`journalctl -u kubelet.service --since "5 min ago"`\
`journalctl -u nginx.service -u kubelet.service --since today`\
`journalctl -u kubelet.service | tail -n 20`

**Network Policy**

```
To test this, you can run nc/curl as shown below:
controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 middleware-svc 80
middleware-svc (10.105.169.89:80) open

controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 mysql-svc 3306
nc: mysql-svc (10.96.106.81:3306): Operation timed out
command terminated with exit code 1

controlplane ~ ✖ kubectl exec middleware -- nc -zv -w1 mysql-svc 3306
mysql-svc (10.96.106.81:3306) open
```
```
kubectl exec -it cyan-white-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local
kubectl exec -it cyan-black-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local
```

```
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: default
   podSelector:
      matchLabels:
        app: cyan-white-cka28-trb'


ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: default
   podSelector:
      matchLabels:
        app: cyan-white-cka28-trb
```

**Troubleshooting issues with PV,PVC's**

`kubectl get events --sort-by='.metadata.creationTimestamp' -A`
- Warning   VolumeMismatch 

**ETCD CLUSTER ISSUES**

```
`ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints 127.0.0.1:2379 snapshot save /opt/etcd-boot-cka18-trb.db`
```
- Check if there is a problem with the manifest file
- Check if the default etcd ports are correct.
- Best place to look for an error is kubelet.service
- Make use of journalctl command to look at the logs
- Don't forget the restart kubelet if you made changes to manifest files

`journalctl -u kubelet -f | grep -v 'connect: connection refused'`

`systemctl restart kubelet`

***cluster4-controlplane kubelet[2240]: E0923 04:38:15.630925    2240 file.go:187] "Could not process manifest file" err="invalid pod: [spec.containers[0].volumeMounts[1].name: Not found: \"etcd-cert\"]" path="/etc/kubernetes/manifests/etcd.yaml"***


**Node affinity**

- Rememeber Node Afinity is different that Taints and Tolerations
- Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:
  - requiredDuringSchedulingIgnoredDuringExecution: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
  - preferredDuringSchedulingIgnoredDuringExecution: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.


NODE:
`kubectl label node cluster1-node01 node=cluster2-node01`

POD:

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node
            operator: In
            values:
            - cluster2-node01
```

### CRONJOB

Remember:
You will notice that the curl is trying to hit orange-app-cka10-trb directly but it is supposed to hit the relevant service which is orange-svc-cka10-trb so we need to fix the curl command.
Change curl orange-app-cka10-trb (pod name) to curl orange-svc-cka10-trb (service name).

Reason: Pod can change if they are recreated but not service.



**Taints & Tolerations**
```

```


**Check can-i auth command**\
`kubectl auth can-i list pods --namespace target --as system:serviceaccount:dev:dev-sa`

`kubectl auth can-i get deployment --as system:serviceaccount:default:deploy-cka19-trb`

ClusterRole Immperative command

`kubectl create clusterrolebinding deploy-cka19-trb-rolebinding --clusterrole=deploy-cka19-trb-role --serviceaccount=default:deploy-cka19-trb`

ClusterRole Binding Immperative command

`kubectl create clusterrole deploy-cka19-trb-role --verb=get --resource=deployments --cluster cluster1 -oyaml`

### Deployment - Rolling Update

`kubectl set image deployment/frontend www=image:v2`  ----> # Rolling update "www" containers of "frontend" deployment, updating the image

`kubectl rollout history deployment/frontend`   ----> # Check the history of deployments including the revision

`kubectl rollout resume deployment black-cka25-trb`


### EXEC Into a running container

kubectl exec -it cyan-white-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local


kubectl exec -it cyan-black-cka28-trb -- sh
curl cyan-svc-cka28-trb.cyan-ns-cka28-trb.svc.cluster.local


### nslookup to a pod and service


### Issues with Nodes (Not Ready State)
#### Check problem with kubelet.service
**Kubelet Configuration Files**

`/var/lib/kubelet/config.yaml`\
`/etc/kubernetes/kubelet.conf`

```
systemctl start kubelet
journalctl -u kubelet --since "30 min ago" | grep 'Error:'
```




## TOPIC WISE HINTS

### Pods and Services


### Clusterrole Clusterrolebindings Service Accounts




## QUESTIONS


### Question 1

```
For this question, please set the context to cluster1 by running:
kubectl config use-context cluster1. 

The deployment called web-dp-cka17-trb has 0 out of 1 pods up and running. Troubleshoot this issue and fix it. Make sure all required POD(s) are in running state and stable (not restarting)

The application runs on port 80 inside the container and is exposed on the node port 30090.

POD(s) are running and stable?
Issues are fixed?**
```


### Question 2

kubectl apply -f red-probe-cka12-trb.yaml 
error: error validating "red-probe-cka12-trb.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): unknown field "command" in io.k8s.api.core.v1.HTTPGetAction, ValidationError(Pod.spec.containers[0].livenessProbe.httpGet): missing required field "port" in io.k8s.api.core.v1.HTTPGetAction]; if you choose to ignore these errors, turn validation off with --validate=false

Under livenessProbe: you will see the type is httpGet however the rest of the options are command based so this probe should be of exec type.

Change httpGet to exec

kubectl get event --field-selector involvedObject.name=red-probe-cka12-trb

21s         Warning   Unhealthy   pod/red-probe-cka12-trb   Liveness probe failed: cat: can't open '/healthcheck': No such file or directory
Notice the command - sleep 3 ; touch /healthcheck; sleep 30;sleep 30000 it starts with a delay of 3 seconds, but the liveness probe initialDelaySeconds is set to 1 and failureThreshold is also 1. Which means the POD will fail just after first attempt of liveness check which will happen just after 1 second of pod start. So to make it stable we must increase the initialDelaySeconds to at least 5

Change initialDelaySeconds from 1 to 5 and save apply the changes.


### Question 3
*Fine-Grained Network Policies in Kubernetes: Building on Top of a Default Deny Policy*

frontend pods should be able to connect to middleware
middleware pod can connect to mysql
frontend pods should not be able to connect to mysql

To test this, you can run nc/curl as shown below:
controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 middleware-svc 80
middleware-svc (10.105.169.89:80) open

controlplane ~ ➜  kubectl exec frontend-deploy-744dddcc79-nwhdc -- nc -zv -w1 mysql-svc 3306
nc: mysql-svc (10.96.106.81:3306): Operation timed out
command terminated with exit code 1

controlplane ~ ✖ kubectl exec middleware -- nc -zv -w1 mysql-svc 3306
mysql-svc (10.96.106.81:3306) open


NetworkPolicy

*Deny all network rules*
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: []

*create two ingress based network policies to allow frontend-to-middleware and middleware-to-mysql*

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-to-middleware
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: middleware
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: middleware-to-mysql
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mysql
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: middleware
          

### Question 4
```
There is a deployment called nodeapp-dp-cka08-trb created in the default namespace on cluster1. This app is using an ingress resource named nodeapp-ing-cka08-trb.

From cluster1-controlplane host we should be able to access this app using the command: curl http://kodekloud-ingress.app. However, it is not working at the moment. Troubleshoot and fix the issue.
Note: You should be able to ssh into the cluster1-controlplane using ssh cluster1-controlplane command.
Fixed the issue?
App is accessible?

```


### Question 5


## Basic Troubleshooting Steps


initialDelaySeconds - Read about this topic
