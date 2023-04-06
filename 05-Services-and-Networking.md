# Services and Networking

## Services

K8s services enable communication between various components within and outside of the application. K8s services helps us connect applications together with othher applications or users. They enable loose coupling between micro-services in our application. There are different type of services

- **NodePort:** The service makes an internal port accessible on a port on the node.
  - **TargetPort:** The port of the container where the service forward the request to.
  - **Port:** The port of the service itself. The service is in fact like a virtual server inside the node. Inside the cluster, it has its own ip address and that ip address is called the cluster ip of the service.
  - **NodePort:** The port we use to access the application externally. This can only be in valid range 30000-32767.

```yaml
# nodeport-service-definition.yaml
apiVersion: v1
kind: Service
metadata:
  name: [service-name]
spec:
  type: NodePort
  ports:
    - targetPort: [target-port] # If not provided, it is assumed to the same value as port.
      port: [port] # The only mandatory field.
      nodePort: [node-port] # If not provided, a free port in the valid range will be automatically allocated.
  selectors:
    [pod-label-key]: [pod-label-value]
```

- **ClusterIP:** The service creates a virtual ip inside the cluster to enable communication between different services.
  - **TargetPort:** The port of the container where the service forward the request to.
  - **Port:** The port of the service itself. The service is in fact like a virtual server inside the node. Inside the cluster, it has its own ip address and that ip address is called the cluster ip of the service.

```yaml
# clusterip-service-definition.yaml
apiVersion: v1
kind: Service
metadata:
  name: [service-name]
spec:
  type: ClusterIP # Default type. If not assigned, k8s will assume it's a cluster ip service type.
  ports:
    - targetPort: [target-port]
      port: [port] # Mandatory.
  selectors:
    [pod-label-key]: [pod-label-value]
``` 

- **LoadBalancer:** The service provisions a load balancer for our application in supported cloud providers.

```yaml
# nodeport-service-definition.yaml
apiVersion: v1
kind: Service
metadata:
  name: [service-name]
spec:
  type: LoadBalancer
  ports:
    - targetPort: [target-port] # If not provided, it is assumed to the same value as port.
      port: [port] # The only mandatory field.
      nodePort: [node-port] # If not provided, a free port in the valid range will be automatically allocated.
  selectors:
    [pod-label-key]: [pod-label-value]
``` 

## Ingress Networking

## Network Policies
