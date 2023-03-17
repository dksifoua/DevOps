# Kubernetes - k8s

## Kubernetes architecture & components

- **Node:** It is a worker machine where containers will be launched by k8s. It was also known as *minions* in the past.
- **Cluster:** It is a set of nodes grouped together. A cluster is useful to sharing load. In addition to that, If a node fails, the application still accessible from other nodes.
- **Master Node:** It is responsible of the orchestration of containers on the worker nodes.
- **API server:** It acts as a frontend for k8s. The users, management devices, CLIs, all talk to the API server to interact with the k8s cluster.
- **Etcd:** It is distributed reliable key-value store used by k8s to store all data used to manage the cluster. When you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all the nodes in a disttributed manner. Etcd is responsible for implementing locks with the cluster to ensure that there  are no conflicts between the masters.
- **Scheduler:** It is responsible for distributing work or containers across multiple nodes. It looks for newly created containers and assign them to nodes.
- **Controller:** It is the brain behind the orchestration. It is responsible for noticing and responding when nodes, containers or endpoints go down. It makes decision to bring up new containers in such cases.
- **Container runtime:** It is the underlying software that is used to run containers. It can be Docker, rkt or cri-o.
- **Kubeet:** It is the agent that runs on each node in the cluster. The agent is responsible for making sure that the containers are running on the nodes as expected.
- **Kubectl:** It is used to deploy and manage applications on a k8s cluster.


## Kubernetes concepts

### 1. Pods
<details><summary>show</summary>
<p>

A pod is a single instance of an application (container). It's the smallest object that you can create in k8s. Pods usually have a one-to-one relationship with containers running your application. So to scale out add new pods, and to scale in remove pods. Not add additional containers to an existing pod to scale your application.

However, we are not restricted to having a single container in a single pod. A single pod can have multiple containers except for the fact that they are usually not multiple containers at the same time. As said before, if the intention is to scale the application, then we would to create additional pods. But sometimes we might have a scenario where we have a helper container that might be doing some kind of supporting task for our application, such as processing a user, enter data, processing a file, uploader by a user, etc. and we want these helper containers to live alongside our application. In that scenarion, we can have both of these containers part of the same pod. The two containers can also communicate with each other directly by referring to each other as localhost since they share the same network space. Plus they can easily share the same storage space as well.

**Pods definiton**
- `$ kubectl [pod-name] --image [image-name]` - create a pod using cli aarguments

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
```

- `$ kubectl create -f pod-definition.yaml` - create a pod from its yaml definition file
- `$ kubectl get pods` - get the list of pods created
- `$ kubectl descripe pod [pod-name]` - get all the information about the pod when it was created
</p>
</details>

### 2. Replication contollers & ReplicaSets
<details><summary>show</summary>
<p>

The replication controller helps us run multiple instances of a single pod in the k8s cluster, thus providing high availability. So, does that mean we can't use a relication controller is we plan to have a single pod? No, even if we have a single pod, the replication controller can help by automatically bringing up a new pod when the existing one fails. Thus, the replication controller ensures that the specified number of pods are running at all times even if it's just one or one hundred.

Another reason we need a replication controller is to create multiple pods to share the load accross them. For example, if we have a single pod serving a set of users, when the number of users increase, we deploy additional pods to balance the load. If the demand futher increases and if we were to run out of resources on the first  node, we could deploy pods across the other nodes in the cluster. The replication controller spans accross multiple nodes in the cluster.

Replcation controllers and Replicasets have the same purpose. Replication controller is the older technology that is being replaced by replicaset. Replicaset is the new recommended way to set up replication.

**Replication controllers & Replicasets definiton**
```yaml
# replication-controller-definition.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: [replication-controller-name]
  labels:
    [rc-label-key]: [rc-label-value]
spec:
  replicas: [number-of-replicas]
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

```yaml
# replicaset-definition.yaml
apiVersion: apps/v1
kind: ReplicationSet
metadata:
  name: [replicationset-name]
  labels:
    [rs-label-key]: [rs-label-value]
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
```

- `$ kubectl create -f replication-controller-definition.yaml` - create a replica controller and underlying pods from its yaml definition file
- `$ kubectl create -f replicaset-definition.yaml` - create a replicaset and underlying pods from its yaml definition file
- `$ kubectl get replicaset`
- `$ kubectl replace -f replicaset-definition.yaml` - scale replicaset after modifying the number of replicas in its definition file
- `$ kubeclt scale --replicas=[number-of-replicas] -f replicaset-definition.yaml`- scale the replicaset without changing it definition file
- `$ kubeclt scale --replicas=[number-of-replicas] -f replicaset [replicaset-name]`- scale the replicaset without changing it definition file
- `$ kubectl delete replicaset [replicaset-name]` - delete a replicaset and the underlying pods
</p>
</details>

### 3. Deployements
<details><summary>show</summary>
<p>

Let say we have a web server that needs to be deployed in production environment. We need not one but many such instances of the web server running for obvious reasons. Secondly, whenever newer versions of application builds become available on the docker registry, we would like to upgrade our docker instance seamlessly. However, when we upgrade our instances, we do not want to upgrade all of them at once (**Recreate strategy**) because it will impact users accessing the application, so we might want to upgrade them one after the other (**Rolling update strategy**), and that kind of upgrade is known as **rolling updates**.

Now, let suppose one of upgrades performed resulted in an unexcepted error and we're asked to undo the recent changes, we would like to be able to **rollback** the changes that were recently carried out.

Finally, say  we would like to make multiple changes to our environment  qsuch as upgrading the underlying webserver versions as well as scaling the environment and also modifying the resource allocations, etc. we do not want to apply each change immediatly after the command is run, instead we like to apply a pause to our environment, make the changes and then resume so that all the changes are **rolled out** together.

All these capabilities arte available with the k8s deployments.

**Deployment definiton**
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
```

- `$ kubectl create -f deployment-definition.yaml` - create deployment and underlying replicatset and pods its yaml definition file
- `$ kubectl get deployments`
- `$ kubectl rollout status deployment/[deployemnt-name]` - get the status of the rollout
- `$ kubectl rollout history deployment/[deployemnt-name]` - get the revisions and history of rollout
- `$ kubectl apply -f deployment-definition.yaml` - upgrade the deployment after modifying the yaml definition
- `$ kubectl set image deployment/[deployemnt-name] [container-name]=[container-image]` - upgrade the deployment without modifying the yaml definition
- `$ kubectl rollout undo deployment/[deployemnt-name]` - rollout the deployment to the previous version
</p>
</details>






