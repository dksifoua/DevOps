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

We then bring an additional layer between the dns server and the cluster like a proxy server that proxies requests on port 80 to port 38080 on our nodes. We then point our dns to the proxy server and users can now access the application by simply visiting `my-online-store.com`. Now this is if the application is hosted on in our data center.

Let's take a step back and see what we could do if we were on a public cloud environment like gcp. In that case, instead of creating a service of type node port, we could set it to type load balancer. When we do that k8s will still do everything that it has to do for a node port. But in addition to that, k8s also sends a request to gcp to provision a network load balancer for the service. On receiving the request, gcp will then auto deploy a load balancer configured to route traffic to the service ports on all nodes and return its information to k8s. The load balancer has an external ip that can be provided to users to access the application. In this case, we set the dns to point to this ip and users can access the application using the url `my-online-store.com`.

Now, the compagny business grows and we have new services for our customers. For eg, a video streaming service. We want users to access the new video streaming service by going to `my-online-store.com/watch`. We'd like to make our old application accessible at `my-online-store.com/wear`. The developers has developed the new streaming service as a completely different application as it has nothing to do with the existing one. However, to share the cluster's resources, we deploy the new application as a separate deployment within the same k8s. We create a `video-service` of type load balancer. K8s provisions the port 38282 for this service and also provision a network load balancer on the cloud. The new load banlancer has a new ip. We have to remember that we must pay for each of these load balancers and having many such load balancers can inversely affect our cloud bill. How do we direct rtaffic between each of theses load balancers based on the url that the user types in? We need yet another proxy or load balancer that redirect traffic based on urls to the diffrent services. 

## Network Policies
