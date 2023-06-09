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

## Security contexts

<details><summary>show</summary>
<p>

**Docker security**
  
Let us start with a host with docker installed on it.This host has a set of its own processes running such as a number of os processes, the docker daemon itself, etc. Now we run on the host an ubuntu docker container that run a process that sleep for an hour. Unlike VMs, containers are not really isolated from their host. Containers and the host sharethe same kernel. Containers are isolated using namespaces in linux. The host has a namespace and the container has their own namespace. All the processes run by the containers are in fact run on the host itself, but in their own namespace. As far as the docker container is concerned, it is in its own namespace and it can see its own processes only. It cannot see anything outside of it or in any other namespace. So when we list the processes inside the docker container, we see the sleep process with the process ID of 1. For the docker host, all processes of its own as well as those in the child namespaces are visible as just another process in the system. So when we list the processes on the host, we see a list a processes including the sleep command but with a different process ID. This is because the processes can have different process IDs in different namespaces and that's how docker isolates containers within a system.
  
Let us now look at users in context of security. The docker host has a set of users (root and non root user). By default, docker run processes within containers as the root user both inside the container and outside of the container in the host. Now, if we do not want the process within the container to run as the root user, we may set the user using the user option within the docker run command and specify the new user ID `docker run --user 1000 ubuntu sleep 3600`. The process will now run with the new user ID. Another way is to define the user in the docker image itself at the time of creation using the USER instruction then, build the custom image.
  
```
 # Dockerfile
FROM ubuntu

USER 1000
```
When we run process inside a container as root user, it's the same root user on the host and it can do everything the root user can. To prevent that, docker implements a set of security features that limit the abilities of the root user within the container, so the root user within the container isn't really like the root user on the host. Docker uses linux capabilities to implement this. The full list of root user capabilities are at this location `/usr/include/linux/capability.h`. By default, docker containers with a limited set of capabilities and so the processes  running within the container do not have the priviledges to say, reboot the host or perform operations that can disrupt the host or other containers running on the same host. If we wish to override this behavior and provide additionnal privileges, use the `cap-add` option in the docker run command `docker run --cap-add MAC_ADMIN ubuntu`. Similarly we can drop `cap-drop` option to drop privileges or `privileged` to run the container with all the privileges.    
  
**Security contexts**
  
 Security contexts can be configured either at the container level or at the pod level. If we configure security settings at the pod level, the settings will carry over to all the containers within the pod. If we configure security settings at both pod and container, the settings on the container will override the settings on the pod.
  
```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [label-key]: [label-value]
spec:
  securityContext: # pod level
    runAsUser: 1000
  containers:
    - name: [container-name]
      image: [container-image]
      securityContext: # container level
        runAsUser: 1000
        capabilities:
          add: [list-of-capabilities]
```
</p>
</details>

## Service Accounts

<details><summary>show</summary>
<p>
There are two types of accounts in k8s: a user account and a service account. The user account is used by humans (admin, developer, etc) and the service account is used machines (application). When we create a service account, k8s creates a secret that will used as a thentication bearer token. Each namespace in k8s has a default service account which is automatically mounted into every pod create in that namespace. The default service account only has permissions to query k8s api server.

```bash
$ kubectl get serviceaccounts
$ kubectl describe serviceaccounts [service-account-name] 
$ kubectl create serviceaccount [service-account-name]
$ kubectl create -f service-account-definition.yaml
```

```yaml
# service-account-definition.yaml
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
  serviceAccount: [service-account-name]
```
  
 Newer version of k8s (1.24+) don't automatically create a toker for service accounts. We must create it manually
  
 `$ kubectl create token [service-account-name]`
</p>
</details>

## Resources requirements

<details><summary>show</summary>
<p>
Each node in a k8s cluster has a set of cpu, memory, and disc resources available. Every pod consumes a set of resources. Whenever a pod s place on a node (by the scheduler), it consumes resources available to that node. if the node has not enough resources, the scheduler avoids placing a pod on that node instead, places the pod on one node where  sufficient resources are available. If there's no sufficient resources available on any on the nodes within the cluster, k8s holds back scheduling the pod. The pod will be in *pending* state and the pod events will show reason: *insufficient cpu*.

By default, k8s assumes that a pod or container within a pod requires `0.5 cpu`, `256Mi of memory`. These are known as *resource requests* for a container (the minimum amount of cpu and memory requested by the container). We can specify custom values in our pod definition file. the cpu count can be any value as low as `0.1` or `100m` (m for milli). On count of cpu is equivalent of 1 aws vcpu, 1 gcp core, 1 azure core or 1 hyperthread. Similarly with memory, we can specify `M or Mi` or `G or Gi`.

In a docker world, a container has no limit to the resources it can consume on a node. However, we can set a limit for the resource usage of a pod in k8s by adding a limit section under the resource section in the pod definition file. Requests and limits are set for each container within a pod. When a pod try to exceed the resource limit of cpu , k8s throttles the cpu so that it doesn't go beyond the specified limit. The container cannot use more cpu resources than its limit. However, this is not the case with the memory; a container can use more memory resource than its limit. If a pod tries to use more memory than its limit constantly, the pod will be terminated.

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
      reqources:
        requests:
          cpu: [cpu-request] 
          memory: [memory-request]
        limits:
          cpu: [cpu-limit]
          memory: [memory-limit]
 ```
It's also possible to modify the default value of request and limit by creating a LimitRange in that namespace.
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/
 
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
---
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```
</p>
</details>


## Taints & Tolerations

<details><summary>show</summary>
<p>
The concept of taints and tolerations are used to set restrictions on what pods can ba schedule on a node. To prevent pods to be place on a node, we place a taint on that node. By default, pod have no tolerations wich means unless specified otherwise, none of the pods in the k8s cluster can tolerate any taint. If we want to enable certain pods to be place on a tainted node, we must have a toleration to theses pods. The scheduler will schedule pods with respect to node taints and pods tolerations. By default, k8s apply a taint to the master node. This is why the scheduler never place a pod on the master node. This is just a best practice that can be modified.

```
$ kubectl taint nodes [node-name] [key-taint]=[value-taint]:[taint-effect]
$ kubectl taint nodes [node-name] [key-taint]- # Remove taint on node with label key [key-taint]
$ kubectl describe node [node-name] | grep Taint
```

The taint effect defines what would happen to the pods if they do not tolerate the taint. There're three taint effects:
- `NoSchedule`: The pod won't be scheduled on the node.
- `PreferNoSchedule`: The scheduler will try to avoid placing a pod on the node with no guarantee.
- `NoExecute`: New pods won't be scheduled on the node and existing pods on the node if any, will be evicted if the do not tolerate the taint. These pods may have be scheduled on the node before the taint was applied to node.

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
  tolerations:
    - key: "[key-taint]"
      operator: "Equal"
      value: "[value-taint]"
      effect: "[taint-effect]"
```

To summary, taints & tolerations do not tell the pod to go to a particular node. Instead it tells the node to only accept pods with certain tolerations. If the requirements is to restrict a pod to certain nodes, it is achieved through another concept called as node selectors & affinity.
</p>
</details>

## Node Selectors & Node Affinity

<details><summary>show</summary>
<p>

Node selectors & Node Affinity are used to set a limitation on a pod so that it only runs on particular nodes. They are two ways to achieve this.

**Node Selectors**

```
$ kubectl label node [node-name] [node-label-key]=[node-label-value]
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
  nodeSelector:
    [node-label-key]: [node-label-value]
```

This is the simple and easier method. It serves our purpose, but has some limitations. We used a single label and selector to achieve our goal here. But what if our requirements are much more complex. For example, we would like to say something like *place the pod on a large or medium labeled node" or "place the pod on any node that is not small*. We cannot achieve this using node selectors. For this node affinity and anti-affinity featrures are the way to go.

**Node Afffinity**

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
  affinity:
    nodeAffinity:
      [type-of-node-affinity]:
        nodeSelectorTerms:
          - matchExpressions:
            - key: [node-label-key]
              operator: [operator] # In | NotIn | Exists (no need values)
              values:
                - [node-label-value]
```

The type of node affinity defines the behavior of the scheduler with respect to node affinity and the stages in the lifecycle of the pod. There are currently three types of node affinity available:
- `requiredDuringSchedulingIgnoredDuringExecution`:
  - requiredDuringScheduling: The pod won't be scheduled if there's not any node with affinity rules.
  - ignoredDuringExecution: Pods will continue to run and any changes in node affinity won't impact them.
- `preferredDuringSchedulingIgnoredDuringExecution`:
  - preferredDuringScheduling: If there's not any node with affinity rules, the scheduler will place the pod on any available node.
  - ignoredDuringExecution: Pods will continue to run and any changes in node affinity won't impact them.
- `requiredDuringSchedulingRequiredDuringExecution`
  - requiredDuringScheduling: The pod won't be scheduled if there's not any node with affinity rules.
  - requiredDuringExecution: Any pod that is running on nodes that doesn't meet affinity rules will be terminated.
</p>
</details>
