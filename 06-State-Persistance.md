# State Persistance

## Volumes

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [label-key]: [label-value]
spec:
  containers:
    - name: [container-name]
      image: [container-image]
      volumeMounts:
        - name: [volume-name]
          mountPath: [container-path]
  volumes:
    - name: [volume-name]
      hostPath: # Not recommended in a multi-node cluster because data will be different in each node.
        type: Directory
        path: [host-path]
      awsElasticBlockStore: # Use network file systems alternatives like NFS, GlusterFS, Flocker, AWS, GCP, Azure
        volumeID: [volume-id]
        fsType: [file-system-type]
```

## Persistant Volumes

## Persistant Volume Claims

## Storage Classes

## Stateful Sets

## Headless Services
