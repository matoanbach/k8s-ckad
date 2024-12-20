## YAML in Kubernetes

- <code>pod-definition.yml</code>

|    Kind    | Version |
| :--------: | :-----: |
|    POD     |   v1    |
|  Service   |   v1    |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
        app: my-app
        type: front-end
spec:
    containers:
        -   name: nginx-container
            image: nginx
```

```
kubectl create -f pod-definition.yml
```

```
Node
├── Pod (myapp-pod)
│   ├── Container (nginx-container)
│   │   ├── Image (nginx)
│   │   └── Application (NGINX Web Server)
│   ├── Shared Storage (None in this configuration)
│   └── Shared Network Namespace
└── Kubernetes Components (e.g., kubelet, kube-proxy)
```

```
kubectl get pods
```

```
kubectl describe pod myapp-pod
```

```
tieuma@Tieus-MacBook-Pro Section 1: Core Concepts % kubectl describe pod myapp-pod
Name:             myapp-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Fri, 20 Dec 2024 13:36:56 -0500
Labels:           app=my-app
                  type=front-end
Annotations:      <none>
Status:           Running
IP:               10.244.0.3
IPs:
  IP:  10.244.0.3
Containers:
  nginx-container:
    Container ID:   docker://5a2b1703442de74bf77535e29c37eebd1c225eec48f38e804238c90b0a307669
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:fb197595ebe76b9c0c14ab68159fd3c08bd067ec62300583543f0ebda353b5be
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 20 Dec 2024 13:37:13 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b4xfx (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-b4xfx:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  13m   default-scheduler  Successfully assigned default/myapp-pod to minikube
  Normal  Pulling    13m   kubelet            Pulling image "nginx"
  Normal  Pulled     13m   kubelet            Successfully pulled image "nginx" in 15.618s (15.618s including waiting). Image size: 197054391 bytes.
  Normal  Created    13m   kubelet            Created container nginx-container
  Normal  Started    13m   kubelet            Started container nginx-container
```

## Running k8s cluster with minikube

### Before applying the myapp-pod.yaml

```
Minikube
└── Kubernetes Cluster
    ├── Control Plane
    │   ├── API Server
    │   ├── Scheduler
    │   ├── Controller Manager
    │   └── etcd (Cluster State Storage)
    └── Node
        ├── System Pods
        │   ├── kube-dns (for internal DNS resolution)
        │   ├── storage-provisioner (for dynamic PV provisioning)
        │   ├── coredns (if enabled)
        │   ├── kube-proxy (for networking rules)
        │   └── Any Minikube-specific Addons (e.g., metrics-server if enabled)
        ├── Shared Storage (Empty or default configuration)
        └── Kubernetes Components
            ├── kubelet
            └── kube-proxy
```

### After applying

```
Minikube
└── Kubernetes Cluster
    ├── Control Plane
    │   ├── API Server
    │   ├── Scheduler
    │   ├── Controller Manager
    │   └── etcd (Cluster State Storage)
    └── Node
        ├── System Pods
        │   ├── kube-dns
        │   ├── storage-provisioner
        │   └── kube-proxy
        ├── User Pods
        │   ├── Pod (myapp-pod)
        │   │   ├── Container (nginx-container)
        │   │   │   ├── Image (nginx)
        │   │   │   └── Application (NGINX Web Server)
        │   │   ├── Shared Storage (None in this configuration)
        │   │   └── Shared Network Namespace
        └── Kubernetes Components
            ├── kubelet
            └── kube-proxy
```

### After applying another pod

```
Minikube
└── Kubernetes Cluster
    ├── Control Plane
    │   ├── API Server
    │   ├── Scheduler
    │   ├── Controller Manager
    │   └── etcd (Cluster State Storage)
    └── Node
        ├── System Pods
        │   ├── kube-dns
        │   ├── storage-provisioner
        │   └── kube-proxy
        ├── User Pods
        │   ├── Pod (myapp-pod)
        │   │   ├── Container (nginx-container)
        │   │   │   ├── Image (nginx)
        │   │   │   └── Application (NGINX Web Server)
        │   │   ├── Shared Storage (None in this configuration)
        │   │   └── Shared Network Namespace
        │   ├── Pod (another-app-pod)
        │   │   ├── Container (another-container)
        │   │   │   ├── Image (httpd)
        │   │   │   └── Application (Apache HTTP Server)
        │   │   ├── Shared Storage (None in this configuration)
        │   │   └── Shared Network Namespace
        └── Kubernetes Components
            ├── kubelet
            └── kube-proxy
```
