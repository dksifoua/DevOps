## Multi-Container Pods

Let's say we want to deploy a web server with a logging service agent. We don't want to merge and blow the code the code of the tool services as each of them targets different functionalities we would still like them to be developped and deployed separately. We only need the two fonctionalities to work together. We need one agent per web server instance paired together that can scale up and down together. That's why we have multi-container pods that share the same lifecycle which means they are created together and destoyed together. They share the same network space which means they can refer to each other as localhost and they have access to the same storage volumes. This way,  we do not have to establish volume sharing or services between the pods to enable communication between them.

There are different patterns when it comes to designing multi-container pods in k8s such as ambassador, adapter and sidecar.

**Design Patterns - Sidecar**

A good example of sidecar pattern is deploying a logging agent alonside a web server to collect logs and forward them to a central log server.

**Design Patterns - Adapter**

Building on the previous example, let's say we have multiple applications generating logs in different formats. It would be hard to process the various formats on the central logging server. So before sending the logs to the central server, we would like to them to convert them to a common format. For this, we deploy an adapter container. The adapter container processes the logs before sending them to the central server.

**Design Patterns - Ambassador**

Let's say our application communicates with different database instances at different stages (dev, staging, prod) of development. We must make sure to modify this connectivity in the application code depending on the environment we're deploying the application to. We may choose to outsource such logic to a separate container within our pod so that the application can always refer to a db at localhost and the new container will proxy that request to the right db. This is known as an ambassador container.

Note: These are different patterns in designing a multi-container pod. When it comes to implementing them using a pod definition file, it is always the same.

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
    - name: [container-name-1]
      image: [container-image-1]
    - name: [container-name-2]
      image: [container-image-2]
```

## Init Containers

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.

But at times we may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts. That's where initContainers comes in. When a POD is first created the initContainer is launch, and the process in the initContainer must run to a completion before the real container hosting the application starts.

We can configure multiple such initContainers as well. In that case each init container is launch one at a time in sequential order. If any of the initContainers fail to complete, k8s restarts the pod repeatedly until the Init Container succeeds.

```yaml
# pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod-name]
  labels:
    [label-key]: [label-value]
spec:
  initContainers:
    - name: [init-container-name-1]
      image: [init-container-image-1]
    - name: [init-container-name-2]
      image: [init-container-image-2]
  containers:
    - name: [container-name]
      image: [container-image]
```

