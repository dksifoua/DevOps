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
apiVersion: v1
kind: PersistentVolume
metadata:
  name: [persistent-volume-name]
  labels:
    [label-key]: [label-value]
spec:
  accessmodes:
    - [access-mode] # ReadonlyMany | ReadWriteOne | ReadWriteMany
  capacity:
    storage: [storage-amount] # eg. 1Gi
  persistentVolumeReclaimPolicy: [persistent-volume-reclaim-policy] # What happen when the pvc is delete? Retain (default - the pv is not delete) | Delete (the pv is delete) | Recycle (only the data in pv is deleted)
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

One persistent volume claims are created, k8s binds the persitent volumes to claims based on the requests and properties set on the volumes. Every pvc is bound to a single pv. During the binding process, k8s tries to find a persistent volume that has sufficient capacity as requested by the claim and any other request properties such as access modes, volumes modes, storage classes, etc. However, if there're multiple possible matches for a single claim, and we would like to specifically use a particular one, we could still use labels and selectors to bind to the right volumes. Finally, a smaller claim may get bound to a larger volume if all the other criterias matches and there are no better options. No other claim can utilize the remaining capacity.

If there is no volume available, the pvc will remain in a pending state until newer volumes are made available to the cluster. Once newer volumes are available,  the claim would automatically be bound to the newly available volume.

```yaml
# persistent-volume-claim-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: [persistent-volume-claim-name]
  labels:
    [label-key]: [label-value]
spec:
  accessmodes:
    - [access-mode] # ReadonlyMany | ReadWriteOne | ReadWriteMany
  resources:
    requests:
      storage: [storage-request] # eg. 500Mi
```

```shell
$ kubectl create -f persistent-volume-claim-definition.yaml
$ kubectl get persistentvolumeclaim
$ kubectl delete persistentvolumeclaim [persistent-volume-claim-name]
```

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
      persistentVolumeClaim:
        claimName: [persistent-volume-claim-name]
```

## Storage Classes

## Stateful Sets

## Headless Services
