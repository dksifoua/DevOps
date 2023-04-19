# Secutity Primitives

Of course, all access to the hosts that form a k8s cluster must be secured, root access disabled, password based authentication disabled, and only ssh key based authentication to be made available. And of course, any other measures we need to take to secure the physical or virtual infrastructure that host k8s. If that is compromised, everything is compromised. Our focus on this part is more on k8s related security. What are the risks and what measures do we need to take to secure the k8s cluster?

As we've seen alrealdy, the kube api server is at the centre of all operations with k8s. We interact with it through the kubectl utility or by accessing the api directly. And through that we can perform almost any operation on the cluster. So that's the first line of defense. *Controlling access to the api-server itself*. We need to make two types of decisions:

- **Who can access the cluster?** It is defined by the **authentication** mechanism. There are different ways to authentication to the api server:
  - User ID and password stored in a static file
  - User ID and token stored in a static file
  - Certificates
  - External authentication providers - LDAP
  - Service Accounts for machines
- **What can they do?** It is defined by the **authorization** mechanism. Authorization is implemented through:
  - Role Based Access Control - RBAC where users are associated to groups with specific permissions.
  - Attribute Based Access Control - ABAC
  - Node Authorization
  - Webhook Mode

All communication with the cluster between the various component such as the ectd cluste, the kube controller manager, the scheduler, the api server, as well as those running on the worker nodes such as the kubelet and the kube proxy is secured using tls encryption

For communication between application within the cluster, by default all ports can access all other ports within the cluster. We can restrict access between them using network policies.
