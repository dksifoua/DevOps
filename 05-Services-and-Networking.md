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

Let's start with a simple scenario. We are deploying an application in k8s for a company that has an online store selling products. The application will be available at `www.my-online-store.com`. We build the application into a docker image and deploy it on the k8s cluster as a pod in a deployment. The application needs a db, so we deploy a mysql db as a pod an create a service of type cluster ip called `mysql-service` to make it accessible to our application. The application is now working. To make the application accessible to the outside world, we create another service, this time of type node port and make the application available on a `38080` port on the nodes in the cluster. The user can now access the application using the url `http://<any-node-ip>:38080`. Whenever the traffic increases, we increases the number of replicas of the pod to handle the additional traffic and the node port service takes care of splitting traffic between the pods. However, in a production grade application, there are many more things involve in addition to simply splitting the traffic between the pods. For example, we do not want hte user to type the ip address every time. So we configure out dns server to point to the ip of the node. The users can now access the application using the url `http://my-online-store.com:38080`. Now we don't want users to remember port the  number either. However, service node port can only allocate high-numbered ports which are greater than 30000.

## Network Policies
