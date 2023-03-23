# Configuration

## Command & Arguments in Kubernetes
In k8s, `command` replaces docker `ENTRYPOINT` and `args` replaces docker `CMD`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
spec:
  containers:
    - name: [container-name]
      image: [container-image]
      command: [docker-entry-point-list]
      args: [ddocker-cmd-list]
```
