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

## Jobs & CronJobs
