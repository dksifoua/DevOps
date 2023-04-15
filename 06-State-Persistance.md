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

When we created volumes, we configured them within the pod definition file so every configuration information required to configure storage for the volume goes within the pod definition file. Now, with a large environment with a lot of users deploying a lot of pods, the users will have to configure storage every time for each pod. Whatever storage solution is used, the user who deploys the pod will have to configure that on all pod definition files. Every time a change is to be made, the user would have to make them on all of his pods. Instead, we would like to manage storage more centrally. We would like to configure it in a way that an administrator can create a large pool of storage and then have users carve out pieces from it, as required. That is where persitent volumes can help us.

A persistent volume is a cluster wide pool of volumes configured by and administrator to be used by users deploying applications on the cluster. The users can now select storage from this pool using persistent volume claims.

```yaml
# persistent-volume-definition.yaml
apiVersion: PersistentVolume
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [label-key]: [label-value]
spec:
  accessmodes:
    - [access-mode] # ReadonlyMany | ReadWriteOne | ReadWriteMany
  capacity:
    storage: [storage-amount] # eg. 1Gi
  hostPath: # Not recommend in a production environment
    path: [host-path]
  awsElasticBlockStore: # Alternative
        volumeID: [volume-id]
        fsType: [file-system-type]
```

```shell
$ kubectl create -f persistent-volume-definition.yaml
$ kubectl get persistentvolume
```

## Persistant Volume Claims

## Storage Classes

## Stateful Sets

## Headless Services
