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

Now, the compagny business grows and we have new services for our customers. For eg, a video streaming service. We want users to access the new video streaming service by going to `my-online-store.com/watch`. We'd like to make our old application accessible at `my-online-store.com/wear`. The developers has developed the new streaming service as a completely different application as it has nothing to do with the existing one. However, to share the cluster's resources, we deploy the new application as a separate deployment within the same k8s. We create a `video-service` of type load balancer. K8s provisions the port 38282 for this service and also provision a network load balancer on the cloud. The new load banlancer has a new ip. We have to remember that we must pay for each of these load balancers and having many such load balancers can inversely affect our cloud bill. How do we direct rtaffic between each of theses load balancers based on the url that the user types in? We need yet another proxy or load balancer that redirect traffic based on urls to the diffrent services. Every time we introduce a new service, we have to reconfigure the load balancer.

And finally, we also need to enable ssl for the applications so users can access the applications using https. Where to configure that? It can be done at different levels either at the application level itself, or at the load balancer level, or at the proxy server level, but which one? We don't our developer to implement it on the applications as they would do it in different ways and it's an additional burden for them to develop an additional code to handle that. We want it to be configured in one place with minimal maintenance. Now, that's a lot of different configurations and all of these becomes difficult to manage when your applications scale. It required involving different individuals in different teams. we need to configure our firewall rules for each new service and it's expensive as well, as  for each new service, a new cloud native load balancer needs to be provisioned. Wouldn't it be nice if we could manage all that within the k8s cluster and hace all that configuration as just as another k8s definition filethat lives along with the rest of the applicatio ndeployment files?

That where **ingress** comes in. Ingress helps users access our application using a single externally accessible url that we can configure to route traffic to different services with the cluster based on the url path. At the same time, implement ssl security as well. We can think of ingress a layout 7 load balancer built into the k8s cluster that caan be configured using natives k8s primitives  just like anu other object in k8s. Now, even with ingress, we still need to expose (node port or load balancer) it to make accessible outside the cluster. But that's just a one time configuration. Going forward, we rae going to perform all our load balancing of ssl and url based routing configurations on the ingress controller.

### Ingress Controllers

Ingress controllers are solutions deployed to applu configurations. K8s doesn't come with ingress controller by default. So we must delploy one.There are number of soluttions available for ingress like gce (supported and maintain by the k8s project), nginx (supported and maintain by the k8s project), istio, contour, haproxy, taefik.

These ingress controllers are not just another load balancer. The load balancer components are just a part of it. Ingress controllers have additional intelligence built into them to monitor the k8s cluster for new definitions or ingress resources and configure the nginx server accordingly. An nginx controller is deployed as just another deployment in k8s.

```yaml
# nginx-controller-definition-file.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller # The nginx progra is stored here
            - --configmap=$(POD_NAMESPACE)/nginx-configuration # For nginx configurations
          envs:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

```yaml
# nginx-configmap-definition-file.yaml
apiVersion: v1
kind: Configmap
metadata:
  name: nginx-configuration
```

```yaml
# nginx-service-definition-file.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-inngress
spec:
  type: NodePort
  selector:
    name: nginx-ingress
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
```

```yaml
# nginx-service-account-definition-file.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-inngress-service-account
spec:
  type: NodePort
  selector:
    name: nginx-ingress
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
```

### Ingress Resources

Ingress Resources are set of rules & configurations applied to on the ingress controller.

```yaml
# ingress-definition-file.yaml - route all traffic to a service
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: [ingress-name]
spec:
  backend:
    serviceName: [service-name]
    servicePort: [service-port]
```

```yaml
# ingress-definition-file.yaml - route all traffic based on rules
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: [ingress-name]
spec:
  rules:
  - host: [host] # wear.my-online-store.com or use path
    http:
    paths:
      - path: [path] # my-online-store.com/wear or use host
        backend:
          serviceName: [service-name]
          servicePort: [service-port]
```

## Network Policies

A network policy is another object in k8s linked to one or more pods where we can define policy rules.

```yaml
# network-policy-definition-file.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: [network-policy-name]
spec:
  podSelector:
    matchLabels:
      [pod-label-key]: [pod-label-value]
  policyTypes:
    - [policy-type] # Ingres or Egress
  Egress:
    - to:
      - podSelector:
          matchLabels:
            [destination-pod-label-key]: [destination-pod-label-value]
      <-> namespaceSelector: # With a dash (-), it means OR. Without, it means AND
          matchLabels:
            name: [destination-pod-namespace]
        ipBlock:
          cidr: [cidr] # eg. 192.168.0.1/32
      ports:
        - protocol: [protocol]
          port: [port]
  Ingress: # Add this if it's present in policyTypes
    - from
      - podSelector:
          matchLabels:
            [origin-pod-label-key]: [origin-pod-label-value]
      ports:
        - protocol: [protocol]
          port: [port]
```

**Note:** Network policies are enforced by the network solution implemented on the k8s cluster, and not all network solutions support network policies. A few of them that are supported are Cube Router, Calico, Romana and Weave-net. Flannel doesn't support (enforce) network policies.
