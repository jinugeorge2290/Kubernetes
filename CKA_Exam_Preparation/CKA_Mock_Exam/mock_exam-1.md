

Sections:

ARCHITECTURE, INSTALL AND MAINTENANCE
TROUBLESHOOTING
 


Revision of topics needed:
Cluster Role and Cluster Role Bindings 
JSONPATH QUESTIONS
Network Policy - Ingress & Egress

nodeAffinity
nodeSelectors


### Authorization can-i as service account
```yaml
student-node ~ ➜ kubectl edit clusterrole green-role-cka22-arch --context cluster1
kubectl auth can-i get namespaces --as=system:serviceaccount:default:green-sa-cka22-arch
```

```bash

kubectl get secret beta-sec-cka14-arch --context cluster3 -n beta-ns-cka14-arch -o json | jq .data.secret | tr -d '"' | base64 -d

```



```bash

### Find Top nodes 
student-node ~ ➜  kubectl top node --context cluster1 --no-headers | sort -nr -k2 | head -1
### Find the node across all clusters that consumes the most memory
student-node ~ ➜  kubectl top node --context cluster1 --no-headers | sort -nr -k4 | head -1

```


```bash

### ETCD BACKUP
### Install etcd utility (if not installed already) and take etcd backup
cluster1-controlplane ~ ➜ cd /tmp
cluster1-controlplane ~ ➜ export RELEASE=$(curl -s https://api.github.com/repos/etcd-io/etcd/releases/latest | grep tag_name | cut -d '"' -f 4)
cluster1-controlplane ~ ➜ wget https://github.com/etcd-io/etcd/releases/download/${RELEASE}/etcd-${RELEASE}-linux-amd64.tar.gz
cluster1-controlplane ~ ➜ tar xvf etcd-${RELEASE}-linux-amd64.tar.gz ; cd etcd-${RELEASE}-linux-amd64
cluster1-controlplane ~ ➜ mv etcd etcdctl  /usr/local/bin/
cluster1-controlplane ~ ➜ ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/cluster1_backup.db

```



```bash

---------------
### Questions 
For this question, please set the context to cluster1 by running:
kubectl config use-context cluster1
We deployed an app using a deployment called web-dp-cka06-trb. it's using the httpd:latest image. There is a corresponding service called web-service-cka06-trb that exposes this app on the node port 30005. However, the app is not accessible!

Troubleshoot and fix this issue. Make sure you are able to access the app using curl http://kodekloud-exam.app:30005 command.

---------------
Solution:

List the deployments to see if all PODs under web-dp-cka06-trb deployment are up and running.
kubectl get deploy
You will notice that 0 out of 1 PODs are up, so let's look into the POD now.

kubectl get pod
You will notice that web-dp-cka06-trb-xxx pod is in Pending state, so let's checkout the relevant events.

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxx
You should see some error/warning like this:

Warning   FailedScheduling   pod/web-dp-cka06-trb-76b697c6df-h78x4   0/1 nodes are available: 1 persistentvolumeclaim "web-cka06-trb" not found. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
Let's look into the PVCs

kubectl get pvc
You should see web-pvc-cka06-trb in the output but as per logs the POD was looking for web-cka06-trb PVC. Let's update the deployment to fix this.

kubectl edit deploy web-dp-cka06-trb
Under volumes: -> name: web-str-cka06-trb -> persistentVolumeClaim: -> claimName change web-cka06-trb to web-pvc-cka06-trb and save the changes.

Look into the POD again to make sure its running now

kubectl get pod
You will find that its still failing, most probably with ErrImagePull or ImagePullBackOff error. Now lets update the deployment again to make sure its using the correct image.

kubectl edit deploy web-dp-cka06-trb
Under spec: -> containers: -> change image from httpd:letest to httpd:latest and save the changes.
Look into the POD again to make sure its running now

kubectl get pod
You will notice that POD is still crashing, let's look into the POD logs.

kubectl logs web-dp-cka06-trb-xxxx
If there are no useful logs then look into the events

kubectl get event --field-selector involvedObject.name=web-dp-cka06-trb-xxxx --sort-by='.lastTimestamp'
You should see some errors/warnings as below

Warning   FailedPostStartHook   pod/web-dp-cka06-trb-67dccb7487-2bjgf   Exec lifecycle hook ([/bin -c echo 'Test Page' > /usr/local/apache2/htdocs/index.html]) for Container "web-container" in Pod "web-dp-cka06-trb-67dccb7487-2bjgf_default(4dd6565e-7f1a-4407-b3d9-ca595e6d4e95)" failed - error: rpc error: code = Unknown desc = failed to exec in container: failed to start exec "c980799567c8176db5931daa2fd56de09e84977ecd527a1d1f723a862604bd7c": OCI runtime exec failed: exec failed: unable to start container process: exec: "/bin": permission denied: unknown, message: ""
Let's look into the lifecycle hook of the pod

kubectl edit deploy web-dp-cka06-trb
Under containers: -> lifecycle: -> postStart: -> exec: -> command: change /bin to /bin/sh
Look into the POD again to make sure its running now

kubectl get pod
Finally pod should be in running state. Let's try to access the webapp now.

curl http://kodekloud-exam.app:30005
You will see error curl: (7) Failed to connect to kodekloud-exam.app port 30005: Connection refused
Let's look into the service

kubectl edit svc web-service-cka06-trb
Let's verify if the selector labels and ports are correct as needed. You will note that service is using selector: -> app: web-cka06-trb
Now, let's verify the app labels:

kubectl get deploy web-dp-cka06-trb -o yaml
Under labels you will see labels: -> deploy: web-app-cka06-trb
So we can see that service is using wrong selector label, let's edit the service to fix the same

kubectl edit svc web-service-cka06-trb
Let's try to access the webapp now.

curl http://kodekloud-exam.app:30005
Boom! app should be accessible now.

```


```

Question

The blue-dp-cka09-trb deployment is having 0 out of 1 pods running. Fix the issue to make sure that pod is up and running.

List the pods
kubectl get pod
Most probably you see Init:Error or Init:CrashLoopBackOff for the corresponding pod.

Look into the logs
kubectl logs blue-dp-cka09-trb-xxxx -c init-container
You will see an error something like

sh: can't open 'echo 'Welcome!'': No such file or directory
Edit the deployment
kubectl edit deploy blue-dp-cka09-trb
Under initContainers: -> - command: add -c to the next line of - sh, so final command should look like this
   initContainers:
   - command:
     - sh
     - -c
     - echo 'Welcome!'
If you will check pod then it must be failing again but with different error this time, let's find that out

kubectl get event --field-selector involvedObject.name=blue-dp-cka09-trb-xxxxx
You will see an error something like

Warning   Failed      pod/blue-dp-cka09-trb-69dd844f76-rv9z8   Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config" to rootfs at "/etc/nginx/nginx.conf": mount /var/lib/kubelet/pods/98182a41-6d6d-406a-a3e2-37c33036acac/volumes/kubernetes.io~configmap/nginx-config:/etc/nginx/nginx.conf (via /proc/self/fd/6), flags: 0x5001: not a directory: unknown
Edit the deployment again
kubectl edit deploy blue-dp-cka09-trb
Under volumeMounts: -> - mountPath: /etc/nginx/nginx.conf -> name: nginx-config add subPath: nginx.conf and save the changes.
Finally the pod should be in running state.

```

```

The controlplane node called cluster4-controlplane in the cluster4 cluster is planned for a regular maintenance. In preparation for this maintenance work, we need to take backups of this cluster. However, something is broken at the moment!


Troubleshoot the issues and take a snapshot of the ETCD database using the etcdctl utility at the location /opt/etcd-boot-cka18-trb.db.


Note: Make sure etcd listens at its default port. Also you can SSH to the cluster4-controlplane host using the ssh cluster4-controlplane command from the student-node.


Solution
SSH into cluster4-controlplane host.
ssh cluster4-controlplane
Let's take etcd backup

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-boot-cka18-trb.db
It might stuck for forever, let's see why that would happen. Try to list the PODs first

kubectl get pod -A
There might an error like

The connection to the server cluster4-controlplane:6443 was refused - did you specify the right host or port?
There seems to be some issue with the cluster so let's look into the logs

journalctl -u kubelet -f
You will see a lot of connect: connection refused erros but that must be because the different cluster components are not able to connect to the api server so try to filter out these logs to look more closly

journalctl -u kubelet -f | grep -v 'connect: connection refused'
You should see some erros as below

cluster4-controlplane kubelet[2240]: E0923 04:38:15.630925    2240 file.go:187] "Could not process manifest file" err="invalid pod: [spec.containers[0].volumeMounts[1].name: Not found: \"etcd-cert\"]" path="/etc/kubernetes/manifests/etcd.yaml"
So seems like there is some incorrect volume which etcd is trying to mount, let's look into the etcd manifest.

vi /etc/kubernetes/manifests/etcd.yaml 
Search for etcd-cert, you will notice that the volume name is etcd-certs but the volume mount is trying to mount etcd-cert volume which is incorrect. Fix the volume mount name and save the changes. Let's restart kubelet service after that.

systemctl restart kubelet
Wait for few minutes to see if its good now.

kubectl get pod -A
You should be able to list the PODs now, let's try to take etcd backup now:

ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-boot-cka18-trb.db
It should work now.


```



```
We recently deployed a DaemonSet called logs-cka26-trb under kube-system namespace in cluster2 for collecting logs from all the cluster nodes including the controlplane node. However, at this moment, the DaemonSet is not creating any pod on the controlplane node.
Troubleshoot the issue and fix it to make sure the pods are getting created on all nodes including the controlplane node.


Solution
Check the status of DaemonSet

kubectl --context2 cluster2 get ds logs-cka26-trb -n kube-system
You will find that DESIRED CURRENT READY etc have value 2 which means there are two pods that have been created. You can check the same by listing the PODs

kubectl --context2 cluster2 get pod  -n kube-system
You can check on which nodes these are created on

kubectl --context2 cluster2 get pod <pod-name> -n kube-system -o wide
Under NODE you will find the node name, so we can see that its not scheduled on the controlplane node which is because it must be missing the reqiured tolerations. Let's edit the DaemonSet to fix the tolerations

kubectl --context2 cluster2 edit ds logs-cka26-trb -n kube-system
Under tolerations: add below given tolerations as well

- key: node-role.kubernetes.io/control-plane
  operator: Exists
  effect: NoSchedule
Wait for some time PODs should schedule on all nodes now including the controlplane node.



```


```
We want to deploy a python based application on the cluster using a template located at /root/olive-app-cka10-str.yaml on student-node. However, before you proceed we need to make some modifications to the YAML file as per details given below:


The YAML should also contain a persistent volume claim with name olive-pvc-cka10-str to claim a 100Mi of storage from olive-pv-cka10-str PV.


Update the deployment to add a sidecar container, which can use busybox image (you might need to add a sleep command for this container to keep it running.)

Share the python-data volume with this container and mount the same at path /usr/src. Make sure this container only has read permissions on this volume.


Finally, create a pod using this YAML and make sure the POD is in Running state.
info_outline
Solution
Update olive-app-cka10-str.yaml template so that it looks like as below:

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: olive-pvc-cka10-str
spec:
  accessModes:
  - ReadWriteMany
  storageClassName: olive-stc-cka10-str
  volumeName: olive-pv-cka10-str
  resources:
    requests:
      storage: 100Mi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: olive-app-cka10-str
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: olive-app-cka10-str
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - cluster1-node01
      containers:
      - name: python
        image: poroko/flask-demo-app
        ports:
        - containerPort: 5000
        volumeMounts:
        - name: python-data
          mountPath: /usr/share/
      - name: busybox
        image: busybox
        command:
          - "bin/sh"
          - "-c"
          - "sleep 10000"
        volumeMounts:
          - name: python-data
            mountPath: "/usr/src"
            readOnly: true
      volumes:
      - name: python-data
        persistentVolumeClaim:
          claimName: olive-pvc-cka10-str
  selector:
    matchLabels:
      app: olive-app-cka10-str

---
apiVersion: v1
kind: Service
metadata:
  name: olive-svc-cka10-str
spec:
  type: NodePort
  ports:
    - port: 5000
      nodePort: 32006
  selector:
    app: olive-app-cka10-str
Apply the template:

kubectl apply -f olive-app-cka10-str.yam

```



```

John is setting up a two tier application stack that is supposed to be accessible using the service curlme-cka01-svcn. To test that the service is accessible, he is using a pod called curlpod-cka01-svcn. However, at the moment, he is unable to get any response from the application.



Troubleshoot and fix this issue so the application stack is accessible.



While you may delete and recreate the service curlme-cka01-svcn, please do not alter it in anyway.

info_outline
Solution
Test if the service curlme-cka01-svcn is accessible from pod curlpod-cka01-svcn or not.


kubectl exec curlpod-cka01-svcn -- curl curlme-cka01-svcn

.....
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0


We did not get any response. Check if the service is properly configured or not.


kubectl describe svc curlme-cka01-svcn ''

....
Name:              curlme-cka01-svcn
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          run=curlme-ckaO1-svcn
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.45.180
IPs:               10.109.45.180
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>


The service has no endpoints configured. As we can delete the resource, let's delete the service and create the service again.

To delete the service, use the command kubectl delete svc curlme-cka01-svcn.
You can create the service using imperative way or declarative way.


Using imperative command:
kubectl expose pod curlme-cka01-svcn --port=80


Using declarative manifest:


apiVersion: v1
kind: Service
metadata:
  labels:
    run: curlme-cka01-svcn
  name: curlme-cka01-svcn
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: curlme-cka01-svcn
  type: ClusterIP


You can test the connection from curlpod-cka-1-svcn using following.


kubectl exec curlpod-cka01-svcn -- curl curlme-cka0


```



