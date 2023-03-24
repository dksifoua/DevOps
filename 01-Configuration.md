# Configuration

## Commands & Arguments
<details><summary>show</summary>
<p>
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
</p>
</details>
  
## ConfigMaps
<details><summary>show</summary>
<p>
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
</p>
</details>

## Secrets
<details><summary>show</summary>
<p>
The config map stores configuration data in plain text format. It is not the right to store password or any credential data. Secrets are used to store sensitive information but in an encoded format.

```bash
$ kubectl get secrets
$ kubectl describe secret [configmap-name] 
$ kubectl create secret generic [secret-name] --from-literal=[key]=[value]
$ kubectl create secret generic [secret-name] --from-file=[path-to-file]
$ kubectl create -f secret-definition.yaml
```

```yaml
# secret-definition.yaml
apiVersion: v1
kind: Secret
metadata:
  name: [secret-name]
data:
  [secret-key]: [secret-value-encoded] 
  # echo -n 'password' | base64
  # echo -n 'cGFzc3dvcmQ=' | base64 --decode
  
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
            secretKeyRef:
              name: [configmap-name]
              key: [cm-key]
      envFrom:
        - secretRef:
            name: [secret-name]
  volumes:
    - name: [volume-name]
      secret:
        secretName: [configmap-name]
```

Here are somethings to keep in mind when working with secrets.
1. Secrets are not encrypted, they're only encoded, meaning anyone can lookup the secret file and decoded it. So, DO NOT check-in secret objects to scm along with code.
2. Secrets are not encoded in etcd. None of the data in etcd is encrypted by default. So consider enabling encryption at rest. https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
3. Anyone able to create pods/deployments in the same namespace can access the secrets. So consider configure least-priviledge access to secrets through RBAC - Role Based Access Controls
4. Consider third-party secrets store providers like AWS, GCP, Azure, Vault 
</p>
</details>
