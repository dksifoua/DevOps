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

## Authentication

The k8s cluster consists on multiple physical or virtual nodes and various components that work together. We have users like administrators that access the cluster to perform administrative tasks, the developers that access the cluster to test or deploy their applications, end users who access applications deployed on the cluster, and we have third-party applications accessing the cluster for integration purposes.

The security of end users who access the applications deployed on the cluster is managed by the applications themselves internally. So our focus is on user's accessing the cluster: User (administrators or developers) & Service Account (machines or robots).

K8s doesn't manage user accounts natively. It relies on an external source like a file with user details or certificates or third party identity services like LDAP to manage theses users. So we cannot create user in a k8s cluster or view the list of users. However, in case of service accounts, k8s can manage them.

So let's focus on users.

All user access is managed by the api server. Whether the request comes from the kubectl tool or the api directly. All of these requests go through the kube api server which authenticates the requests before processing them. There are different authentication mechanisms that can be configured:

**Auth mechanism - Basic**

We create a list of users and passwords in a csv file and use that as the source for user information. The file has three columns: password, username and user id with an optional fouth column for groups. We then pass the filename as an option to the kube api server. We then add that file to kube api server definition file.

````yaml
# kube-apiserver-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - ...
    - --basic-auth-file=[basic-csv-file-path]
...
```

The kube api server will automatically restart after we apply modifications on its definition file.

To authenticate using the basic credentials while accessing the api server:

`$ curl -v -k https://[master-node-ip]:[port]/api/v1/pods -u "[user]=[password]"`

Instead of a static password file, we can use a static token file. We just replace the passwords by the tokens.

````yaml
# kube-apiserver-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - ...
    - --token-auth-file=[token-csv-file-path]
...
```

`$ curl -v -k https://[master-node-ip]:[port]/api/v1/pods --header "Authorization: Bearer [token]"`

**Notes**: The basic and token authentications are not recommended as they are insecure
