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

```yaml
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

```yaml
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

**Kube Config**

Let say a client wants to query a k8s api server using certificates. To do so, the client passes arguments with its request.

```bash
$ curl https://[kube-apiserver]:[port]/api/v1/pods \
  --key [client-key].key
  --cert [client-certificate].crt
  --cacert [certificate-authority].crt
```

With the kubectl command, we can specify the same information using the option `server`, `client-key`, `client-certificate` and `certificate-authority`

```bash
$ kubectl get pods \
  --server [kube-apiserver]:[port]
  --client-key [client-key].key
  --client-certificate [client-certificate].crt
  --certificate-authority [certificate-authority].crt
```

Obviously, typing those in every time is a tedious task. So we move these information to a configuration file called as `kubeconfig` and then specify this file as the kube config option.

`$ kubeclt get pods --kubeconfig [kubecconfig-file]`

By default, the kubectl tool looks for a file named `.kube/config`. So if move configurations in that file, we don't have to specify the path to file explicitely in the kubectl command.

The kubeconig file has three options: 

- **clusters:** are the various k8s clusters that we need access to.
- **Users:** are just the different users.
- **Contexts:** define which user accounts will be used to access which cluster.


```yaml
# kubeconfig-definition file. We don't need to create any k8s object. The kubectl will read it
apiVersion: v1
kind: Config
current-context: [context-name] # Default context to be use by kubectl
clusters:
  - name: [server-name]
    cluster:
      certificate-authority: [certificate-authority].crt
      certificate-authority-data: [certificate-authority-content] # Alternative to certificate-authority field
      server: [kube-apiserver]:[port]
contexts:
  - name: [context-name] # eg. [username]@[server-name]
    context:
      cluster: [server-name]
      user: [username]
      namespace: [namespace] # When switch to the context, we will auto get to the namespace
users:
  - name: [username]
    user:
      client-certificate: [client-certificate].crt
      client-key: [client-key].key
```

```bash
$ kubectl config view # view the config file
$ kubectl config view --kubeconfig=[kubeconfig-filepath]
$ kubectl config use-context [context-name] # Use a different context. This will edit the kubeconfig file
```

**API Groups**

The k8s api is grouped into multiple groups based on their purpose. `/metrics`, `/healthz`, `/version`, `/api`, `/apis`, `/logs`. The version api is for viewing the version of the cluster. The metrics and healthz apis are used to monitor the health of the cluster. The logs are used for integrating with third party logging applications. The /apis and /api are responsible for the cluster functionalities.

- **/api** The core group. It's where all core functionalities exist such as `v1/namespaces`, `v1/pods`, `v1/rc`, `v1/events`, `v1/endpoints`, `v1/nodes`, `v1/bindings`, `v1/pv`, `v1/pvc`, `configmaps`, `v1/secrets`, `v1/services`, etc. 
- **/apis** The named group. This group is more organized. And going forward, all the newer features are going to be made available through these named groups. It has groups under it for `/apps`, `extensions`, `networking.k8s.io`, `storage.k8s.io`, `authentication.k8s.io`, `certificates.k8s.io`, etc. Under them there are resources that we can `list`, `get`, `create`, `delete`, `update`, `watch`. These are known as verbs.

We can get all the api groups by just querying the k8s cluster like this: `$ curl https://[kube-apiserver]:[port] -k`

## Authorization

There are different authorization mechanisms supported by k8s:

- **Node based authorization:** This is access from within the cluster
- **Attributes based authorization - ABAC:** We associate a user or a group of users with a set of permissions. We do this by creating a policy file
- **Roles based authorization - RBAC:** We define a role for each type of users (groups). RBAC provides a more standard approach to manage accesses within the k8s cluster. 
- **Webhook authorization:** To outsource authorization mechanism.

There are two more authorization modes in addition to those above: **AlwaysAllow** (default) and **AlwaysDeny**.

Those modes are configured using the authorization-mode on the kube apiserver: `--authorization-mode=Node,RBAC,WebHook`. When we have multiple modes configured, the request is authorized using each mode in the order it is specified. When a mode denies a request, it gets to the next mode. As soon as a mode approuves the request, no more checks are done and the user is granted permission.

```yaml
# role-definition-file.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: [role-name]
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "list", "get"]
    resourcesNames: ["resource-name-1", "resource-name-2"] # If we want to give access only to certain pods.
...

# role-bindings-definition-file.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: [role-binding-name]
subjects:
  - kind: User
    name: [username]
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: [role-name]
  apiGroup: rbac.authorization.k8s.io
```


```bash
$ kubectl create -f role-definition-file.yaml
$ kubectl create -f role-bindings-definition-file.yaml
$ kubectl get roles
$ kubectl get rolebindings
$ kubectl describe role [role-name]
$ kubectl describe role [role-binding-name]
$ kubectl auth can-i [verb] [resource] <--namespace [namespace]> # Check if I as a user can perform a kubectl command
$ kubectl auth can-i [verb] [resource] <--namespace [namespace]> --as [username] # Check for another user 
```

## Cluster Roles

## Admission Controllers

