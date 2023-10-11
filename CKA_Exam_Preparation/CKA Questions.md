

## Create Sidecar container

Q1. Create Sidecar container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["/bin/bash"]
    args: [ "-c", "while true; do echo $(date) 'Hi I am a sidecar container' >> /var/log/index.html; sleep 5; done"]
  restartPolicy: OnFailure

```


Q2. 