# Observability

## Readiness Probes

A pod has a status and some conditions. The pod status tells us where the pod is in its lifecycle. When the pod is first created, it is in a `Pending` state. This when the scheduler tries to figure out where to place the pod. If the scheduler cannot find the node to place the pod, it remains in a pending state. To find out why it stucks in a pending state, run the `kubectl describe pod [pod-name]` command. One the pod is scheduled, it goes into a `ContainerCreating` status where the images required for the application are pulled and all the containers start. Once all containers in pod start, the pod goes into a `Running` state where it continues to be until the program commpletes successfully or is terminated. The pod status is displayed in the output of the command `kubectl get pods`. At any point in time, the pod status can only be one these values: Pending, ContainerCreating or Running, and only gives us a high level summary of a pod. 

However, at times, we may want additional informations. Conditions complement pod status. It is an array of true or false values that tells us the state of the a pod. When a pod is scheduled on a node, the pod scheduled condition (`PodScheduled`) is set to true. When the pod is initialized  (`Initialized`), its value is set to true. When all the containers in the pod are ready, the container's ready condition (`ContainersReady`) is set to true. And finally, the pod itself is considerated to be ready (`Ready`). To see the state of pod conditions, run the command: `kubectl describe pod [pod-name]` and look for the condition section. The ready state of the pod can be also viewe in the output of the command `kubectl get pods`. 

The ready condition indicates that the application inside the pod is running and is ready to accept user traffic. However, the application running on the pod may take additional time to warm up. Therefore, the ready condition of the pod is not very true. This is because k8s assumes that as soon as the pod is running, it ready to serve user traffic. There are different ways to define if an application inside a container is actually ready. We can set different kind of tests or *probes* (appropriate term). In case of a web app, it could be when the api server is up and running. So we can run an http test to see f the api server responds. In case of a db, we may test if a particular tcp socket is listenning; or we may simply execute a command within the container to run a custom script that would exit successfully if the application is ready. 

Now when the container is created, k8s does not immediately set the ready condition to true. Instead, it performs a test to see if the application responds positively. Until then, no traffic will be forward to the pod.

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
      readinessProbe:
        initialDelaySeconds: [initial-delay-seconds] # (optional) Give the application enough time to warm up before testing the its readiness
        periodSeconds: [period-seconds] # (optional) How often to test the application readiness
        failureThreshold: [failure-threshold] # By default, if the app is no ready after 03 attempts, the probe will stop. Use this to make more attempts
        # Can be one of the three
        httpGet:
          path: [http-path]
          port: [http-port]
        tcpSocket:
          port: [tcp-port]
        grpc:
          port: [grpc-port]
        exec:
          command:
            - [cmd]
            - [args]
```

## Liveness Probes

Let's say while the pod is running, for some reasons, the application crashes and the pod continues to stay alive (for eg, the app may stuck in an infinite loop due to bug). As far as k8s is concerned, the container is up and running so the application is assumed to be up. But the users hitting the container are not served. In that case, the container need to be restarted or destroyed and a new container is to be brought up. That is where the liveness probe can help us. A liveness probe can be configured on the container to periodically test whether the application within the container is actually healthy. if the test fails, the container is considered unhealthy and is destroyed and recreated. But again, as a developer we get to define what it means for the application to be healthy. It can be an http test, tcp test or command execution test.

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
      livenessProbe:
        initialDelaySeconds: [initial-delay-seconds] # (optional) Give the application enough time to warm up before testing the its readiness
        periodSeconds: [period-seconds] # (optional) How often to test the application readiness
        failureThreshold: [failure-threshold] # By default, if the app is no ready after 03 attempts, the probe will stop. Use this to make more attempts
        # Can be one of the three
        httpGet:
          path: [http-path]
          port: [http-port]
        tcpSocket:
          port: [tcp-port]
        grpc:
          port: [grpc-port]
        exec:
          command:
            - [cmd]
            - [args]
```
