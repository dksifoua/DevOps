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
