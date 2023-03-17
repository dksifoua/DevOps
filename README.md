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


## Pods

A pod is a single instance of an application (container). It's the smallest object that you can create in k8s. Pods usually have a one-to-one relationship with containers running your application. So to scale out add new pods, and to scale in remove pods. Not add additional containers to an existing pod to scale your application.

However, we are not restricted to having a single container in a single pod. A single pod can have multiple containers except for the fact that they are usually not multiple containers at the same time. As said before, if the intention is to scale the application, then we would to create additional pods. But sometimes we might have a scenario where we have a helper container that might be doing some kind of supporting task for our application, such as processing a user, enter data, processing a file, uploader by a user, etc. and we want these helper containers to live alongside our application. In that scenarion, we can have both of these containers part of the same pod. The two containers can also communicate with each other directly by referring to each other as localhost since they share the same network space. Plus they can easily share the same storage space as well.

**Pods definiton**
- `$ kubectl [pod-name] --image [image-name]` - create a pod using cli aarguments

```yaml
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














