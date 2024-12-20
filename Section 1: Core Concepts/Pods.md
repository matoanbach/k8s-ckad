## POD

```txt
Node
├── Pod
│   ├── Container
│   │   ├── Image
│   │   └── Application (Running inside the container)
│   ├── Shared Storage (if any)
│   └── Shared Network Namespace
└── Kubernetes Components (e.g., kubelet, kube-proxy)
```

<img src="https://github.com/matoanbach/k8s-ckad/blob/main/assets/sec%201/3.png"/>

- If there is an increase in number of users, we scale up and down by adding or removing pods and nodes.

- Pod has a one-to-one relationship with the containers, but we aren't restricted to having a single container in a single POD. We don't add containers of the same kind to an existing POD.

- The two containers can also communicate with each other directly by referring to each other as localhost, since they share the same network space. And they can easily share the same storage space as well.

## kubectl

```
kubectl run ngix --image nginx
```

```
kubectl get pods
```
