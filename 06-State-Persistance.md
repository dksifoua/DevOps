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

Before a pv is created, we must have created the storage. Every time an application requires storage, we have to first manually provision the disk before create the pv using the same name as that of created disk. That's called static provisioning volumes. It would've been nice if the volume gets provisionned automatically when required. That's where storage classes come in.

With storage classes, we can define a provisioner such as google storage than can automatically provision storage on google cloud and attach that to pods when a claim is made. That's called dynamic provisioning of volumes.

```yaml
# storage-class-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: [storage-class-name]
provisioner: [provisioner] # eg. for google storage: kubernetes.io/gce-pd
parameters: # optional -very specific to the provisioner we are using
```

Now we have a storage class, we no longer need (to manually create) a pv because the pv and any associated storage is going to be automatically created when the storage class is created. The pvc becomes:

```yaml
# persistent-volume-claim-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: [persistent-volume-claim-name]
  labels:
    [label-key]: [label-value]
spec:
storageClassName: [storage-class-name]
  accessmodes:
    - [access-mode] # ReadonlyMany | ReadWriteOne | ReadWriteMany
  resources:
    requests:
      storage: [storage-request] # eg. 500Mi
```

## Stateful Sets

We are tasked to deploy a db server. We install and set up mysql on a server and create a db. To withstand failures, we are tasked to deploy a high availability solution. So we deploy additional servers and install mysql on those as well. We have a blank db on the new servers. To replicate data from the initial server into the new server, we can set up a master slave architecture where all writes are done on the master node and reads are done on the slave nodes. To set up this architecture, we have to set up the master node first, then set up the first slave node, copy the data from the master node into the slave node and enable continuous replication from the master. To add more slave nodes, repeat the process but instead of copying the data from the master node, we copy the data from a set up slave node and enable continuous replication from the master node. 

With the scenario above, there is an order that we follow in order the set up the architecture. That order cannot be guaranteed with k8s deployment since the pods in a deployment come up at the same time. Another problem with implementing this architecture using deployment is the need to differentiate the master and slaves in order to copy data and enable continuous replication. That is something we cannot do with pods in a deployment because pod ip addresses are dynamically assigned and may change when the pods get recreated. So we definitely need a static hostname. But again, the pods in a deplyment come up with random names that may change when the pods get recreated. So that won't help us here. So the architecture above cannot be implemented with deployments. That's where stateful sets come in.

Stateful sets are similar to deployment sets as they create pods based on a template, they can scale up and down, they can perform rolling updates and rollbacks, but there are some differences. With stateful sets, pods are created in a sequential order. After the firs pod is deployed, it must be in a running and ready state before the next pod is deployed. So that helps us ensure that the master  is deplyed first and then slave 1 and the slave 2. Stateful sets also assigns a unique original index to each pod, a number starting from 0 for the first pod and increments by one. Each pod gets a unique name derived from this index combined the stateful set name. Stateful sets maintain a sticky identity for each of their pods.

```yaml
# statefulset-definition.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: [statefulset-name]
  labels:
    [statefulset-label-key]: [statefulset-label-value]
spec:
  serviceName: [headless-service-name]
  podManagementPolicy: Parallel # optional - if we don't a stack order for the statefulset lifecycle
  replicas: [number-of-replicas]
  selector:
    matchLabels:
      [pod-label-key]: [pod-label-value]
  template:
      metadata:
        name: [pod-name]
        labels:
          [pod-label-key]: [pod-label-value]
      spec:
        containers:
          - name: [container-name]
            image: [container-image]
```

## Headless Services

In our master slave architure, we said that we wanted to point all the writes requests to the master node and the reads on the slave node. To balance the read traffic, we can use a k8s service (node port or load balancer). For the write traffic we use a headless service.

A headless service is created like a normal service, but it does not have an ip on its own like a cluste ip. It does not perform any load balancing. All it does is create dns entries for each pod using the pod name and a subdomain. Each pod in a stateful set get dns like [pod-name].[headless-service-name].[namespace].svc.cluster.local

```yaml
# headless-service-definition.yaml
apiVersion: v1
kind: Service
metadata:
  name: [headless-service-name]
spec:
  clusterIP: None
  ports:
    - port: [port]
  selectors:
    [pod-label-key]: [pod-label-value]
```

When a headless service is created, the dns entries are created for pods only of the two conditions are met:

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [label-key]: [label-value]
spec:
  subdomain: [headless-service-name] # first condition
  hostname: [hostname] # second condition
  containers:
    - name: [container-name]
      image: [container-image]
```

When we used that fields in deployment (subdomain and hostname), all pod within the deployment get the same name. With stateful sets, we do not need to specify a subdomain or a hostname. The stateful set automatically assigns the right hostname for each pod, based on the pod name and it automatically assigns the right subdomain based on the headless service name.

## Volume Claim Templates

If we want a different storage for each pod in our stateful set, we need to use a volume claim template instead of a pvc. A pvc will be automatically created for each pod based on the volume claim template.

```yaml
# deployemnt-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [deployemnt-name]
  labels:
    [deployemnt-label-key]: [deployemnt-label-value]
spec:
  replicas: [number-of-replicas]
  selector:
    matchLabels:
      [pod-label-key]: [pod-label-value]
  template:
      metadata:
        name: [pod-name]
        labels:
          [pod-label-key]: [pod-label-value]
      spec:
        containers:
          - name: [container-name]
            image: [container-image]
            volumeMounts:
              - name: [volume-name]
                mountPath: [container-path]
  volumeClaimTemplates:
    - metadata:
        name: [persistent-volume-claim-name]
      spec:
      storageClassName: [storage-class-name]
        accessmodes:
          - [access-mode] # ReadonlyMany | ReadWriteOne | ReadWriteMany
        resources:
          requests:
            storage: [storage-request] # eg. 500Mi
```
