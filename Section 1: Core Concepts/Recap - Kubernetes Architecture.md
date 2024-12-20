## Node

A node is a machine, physical or virtual on which k8s is installed.
A node is a worker machine and that is where containers will be launched by k8s.
It was also known as minions in the past.

## Cluster

A cluster is a set of nodes grouped together. If one node fails, you have your application sill accessible from other nodes.

## Master

Master is responsible for managing the cluster, monitor and store the information about members of the cluster.
Master is also responsible for the orchestraton and moving the workload of the failed node to another worker node.

## Components

When you install k8s on a system, you are actually installing API Server, etcd, kubelet, Container Runtime, Controller and Scheduler.

### API Server

The API Server acts as the front-end of the server. The users, management devices and command-line interfaces are all talking to the API server to interact with the k8s cluster.

### etcd

<code>etcd</code> is a distributed, reliable key value store used by k8s to store all data used to manage the cluster.
When we have multiple codes in the cluster in a distributed manner, <code>etcd </code> is responsible for implementing locks within the cluster to ensure that ther are no conflicts between the masters.

### Scheduler

The <code>scheduler</code> is responsible for distributing work or containers across multiple nodes.
It looks for newly created containers and assigns them to nodes.

### Controller

The <code>controler</code> are the brain behind orchestration.
They are responsible for noticing and responding when nodes, containers or endpoints go down.
The controllers make decisions to bring up new containers.

### Container runtime

The container runtime is the underlying software that is used to run containers, aka <code>Docker</code>.
But there other options as well.

### kubelet

And finally, kubelet is the agent that runs on each node in the cluster.
The agent is responsible for making sure that the containers are running on the nodes as expected.

## Master vs Worker Nodes

The master server has the kube-API server and that is what makes it a master that is responsible for interactive with a master to provide health information of the worker node and carry out actions requested by the master on the worker nodes. All the information gathered are stored in a key value store on the master. The Key value store is based on the popular etcd framework. The master also has the control manager and the scheduler and other components as welll.

The worker node or minion, as it is also known, is where the containers are hosted. For example, Docker containers. And to run Docker containers on a system, we need container runtime installed and that's where the container runtime falls.

## kubectl

```
kubectl run hello-minikube
```

```
kubectl cluster-info
```

```
kubectl get nodes
```
