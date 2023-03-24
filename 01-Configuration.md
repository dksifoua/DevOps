# Configuration

## Command & Arguments
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
      command: [docker-entry-point-list] # string type
      args: [ddocker-cmd-list] # string type
```

## ConfigMaps
ConfigMaps are used to pass configuration data in a form of key value pairs in k8s. When a pod is created, we inject the configmap into the pod so the key value are available as environment variables for the application  hosted inside a container in a pod.

```bash
$ kubectl get configmaps
$ kubectl describe configmap [configmap-name] 
$ kubectl create configmap [configmap-name] --from-literal=[key]=[value]
$ kubectl create configmap [configmap-name] --from-file=[path-to-file]
$ kubectl create -f configmap-definition.yaml
```

```yaml
# configmap-definition.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: [configmap-name]
data:
  [cm-key]: [cm-value]
  
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [pod-label-key]: [pod-label-value]
spec:
  containers:
    - name: [container-name]
      image: [container-image]
      env:
        - name: [key]
          valueFrom:
            configMapKeyRef:
              name: [configmap-name]
              key: [cm-key]
      envFrom:
        - configMapRef:
            name: [configmap-name]
  volumes:
    - name: [volume-name]
      configMap:
        name: [configmap-name]
```
