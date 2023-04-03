# Pod Design

## Labels, Selectors & Annotations

Labels and selectors are a stanard method to group things together and filter them based on a criteria. Labels are properties attached to each item. Selectors help to filter these items.

```
$ kubectl get pods --selector [label-key]=[label-value]
```

While labels and selectors are used to group and select objects, annotations are used to record other details for informatry purpose. For example tool details like name, version, build information, etc or contact details, phone numbers, emails IDs, etc that may be used for some kind of integration purpose.

```yaml
# deployemnt-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [deployemnt-name]
  labels:
    [deployemnt-label-key]: [deployemnt-label-value]
  annotations:
    [annotation-label-key]: [annotation-label-value]
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

## Rollng Updates & Rollbacks in Deployments

When we first create a deployment, it triggers a rollout. A new rollout creates a new deployment revision (revision 1). In the future, when the application is upgraded, meaning when the container version is updated to a new one, a new rollout is triggered and a new deployment revision is created (revision 2). This helps us keep track of the changes made to our deployments and enables us to rollback to a previous version of deployment if necessary.

```
$ kubectl rollout status deployment [deployment-name]
$ kubectl rollout history deployment [deployment-name] <--revision [revision-number]>
```

There are two types of deployment strategies:

- **Recreate Strategy:** First destroy the running instances and then deploy the new instances of the new application version. The problem with this strategy is that during the period after the older versions are down and before any newer version is up, the application is down and inaccessible to users.
- **Rolling Update Strategy:** Take down the older version and bring up the newer version one by one. This way, the application never goes down and the upgrade is seamles. Rolling update is the default deployment strategy in k8s.

```
# After modifying the deployment definition file, run:
$ kubectl apply -f deployemnt-definition.yaml
# Or (The deployment definition file is not modified) run:
$ kubectl set image deployment [deployment-name] [container-name]=[container-image]
```

```
$ kubectl rollout undo deployment [deployment-name] <--to-revision [revision-number]> # Rollback to the previous version
$ kubectl rollout history deployment [deployment-name]
```

## Jobs & CronJobs

There are different types of workloads that a container can serve like web server, application server or database. These workloads are meant to continue to run for a long period period of time until manually taken down. There are other kind of workloads such as batch processing, analytics or reporting that are meant to carry out a  specific task and then finish. For example, performing a computation, processing and image, performing some kind of analytics on a large dataset, generating a report and sending an email, etc. These are workloads that are meant to live for a short period of time, perform a set of tasks and then finish.

With pods, after the container executed the task, it exits. But it will be recreated again and again until a thresold is reached. K8s want the application to live forever. That is the default behavior of pods (`restartPolicy: Always`). We can override that property by setting its to `Never` or `OnFailure`. That way, k8s does not restart the container one the job is finished. That works just fine.

Now let's say we have a new use case for batch processing where we have large datasets that required multiple pods to process the data in parallel. We want to make sure that all pods perform the task assigned to them successfully and then exit. So we need a manager that can create as many pods as we want to get the work done and ensure that work gets done succesfully. While replicasets is used to make sure a specified number of pods are running at all times, a job is used to run a set of pods to perform a given task to completion.

```yaml
# job-definition.yaml
apiVersion: apps/v1
kind: Job
metadata:
  name: [job-name]
spec:
  completions: [number-of-pods] # Pods are created one by one
  parallelism: [number-of-pods] # Pods are created in parallel
  template:
      spec:
        containers:
          - name: [container-name]
            image: [container-image]
        restartPolicy: [restart-policy]
```

```
$ kubectl create -f job-definition.yaml
$ kubectl delete job [job-name]
```

A cron job is a job that can be scheduled just like cron tab in linux. 

```yaml
# cronjob-definition.yaml
apiVersion: apps/v1
kind: CronJob
metadata:
  name: [job-name]
spec:
  schedule: [cron schedule]
  jobTemplate:
    spec:
      completions: [number-of-pods] # Pods are created one by one
      parallelism: [number-of-pods] # Pods are created in parallel
      template:
          spec:
            containers:
              - name: [container-name]
                image: [container-image]
            restartPolicy: [restart-policy]
```

```
$ kubectl create -f cronjob-definition.yaml
$ kubectl get cronjob
$ kubectl delete cronjob [cronjob-name]
```
