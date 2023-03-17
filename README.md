# Kubernetes - k8s

## Kubernetes architecture & components

- **Node:** It is a worker machine where containers will be launched by k8s. It was also known as *minions* in the past.
- **Cluster:** It is a set of nodes grouped together. A cluster is useful to sharing load. In addition to that, If a node fails, the application still accessible from other nodes.
- **Master Node:** It is responsible of the orchestration of containers on the worker nodes.
- **API server:** It acts as a frontend for k8s. The users, management devices, CLIs, all talk to the API server to interact with the k8s cluster.
- **Etcd:** It is distributed reliable key-value store used by k8s to store all data used to manage the cluster. When you have multiple nodes and multiple masters in your cluster, etcd stores all that information on all the nodes in a disttributed manner. Etcd is responsible for implementing locks with the cluster to ensure that there  are no conflicts between the masters.
- **Scheduler:** It is responsible for distributing work or containers across multiple nodes. It looks for newly created containers and assign them to nodes.
- **Controller:** It is the brain behind the orchestration. It is responsible for noticing and responding when nodes, containers or endpoints go down. It makes decision to bring up new containers in such cases.
- **Container runtime:** It is the underlying software that is used to run containers. It can be Docker, rkt or cri-o.
- **Kubeet:** It is the agent that runs on each node in the cluster. The agent is responsible for making sure that the containers are running on the nodes as expected.
- **Kubectl:** It is used to deploy and manage applications on a k8s cluster.
