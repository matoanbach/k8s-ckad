![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)

# Services and Networking (13%)

### Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --port=80

```

```bash
k expose pod nginx --port=80 --type=ClusterIP
```

or

```bash
k run nginx --image=nginx --port=80 --expose
```

</p>
</details>

### Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
k get po -owide
k get svc
k get ep
```

</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
k run tmp --restart=Never --rm --image=nginx:alpine -it -- curl -m 5 <svcIP:port>
k run tmp --restart=Never --rm --image=nginx:alpine -it -- curl -m 5 <podIP>
```

</p>
</details>

### Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

<details><summary>show</summary>
<p>

```bash
k edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2025-01-08T19:51:06Z"
  labels:
    run: nginx
  name: nginx
  namespace: default
  resourceVersion: "959"
  uid: c5bef0db-ab79-49bb-b814-d713b2d5c2b1
spec:
  clusterIP: 10.43.28.250
  clusterIPs:
    - 10.43.28.250
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - nodePort: 30010
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort
```

```bash
k get svc
# result:
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        1d
nginx        NodePort    10.107.253.138   <none>        80:31931/TCP   3m
```

```bash
curl -m 5 <PodIP>
curl -m 5 <svcIP:PORT>
```

</p>
</details>

### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
k create deployment foo --image=dgkanatsios/simpleapp --replicas=3 --dry-run=client -oyaml > foo.yaml
kubectl label deployment foo --overwrite app=foo #This is optional since kubectl create deploy foo will create label app=foo by default

```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: foo
  name: foo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: foo
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: foo
    spec:
      containers:
        - image: dgkanatsios/simpleapp
          name: simpleapp
          ports:
            - containerPort: 8080
          resources: {}
status: {}
```

</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

<details><summary>show</summary>
<p>

```bash
k run tmp --image=busybox --restart=Never --rm -it -- wget -O- 10.42.0.14:8080

# output
Connecting to 10.42.0.14:8080 (10.42.0.14:8080)
writing to stdout
Hello world from foo-6b747fc57f-fb77z and version 2.0
-                    100% |********************************|    54  0:00:00 ETA
written to stdout
pod "tmp" deleted
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>

```bash
k expose deployment foo --name=foo-service --port=6262 --target-port=8080
```

```bash
k get svc
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>
<p>

```bash
k run tmp --image=busybox --restart=Never --rm -it -- wget -O- 10.43.215.40:6262
# output
Connecting to 10.43.215.40:6262 (10.43.215.40:6262)
writing to stdout
Hello world from foo-6b747fc57f-fb77z and version 2.0
-                    100% |********************************|    54  0:00:00 ETA
written to stdout
pod "tmp" deleted
```

</p>
</details>

### Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the pods in this deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

> Note that network policies may not be enforced by default, depending on your k8s implementation. E.g. Azure AKS by default won't have policy enforcement, the cluster must be created with an explicit support for `netpol` https://docs.microsoft.com/en-us/azure/aks/use-network-policies#overview-of-network-policy

<details><summary>show</summary>
<p>

```bash
k create deployment nginx --image=nginx --replicas=2 --port=80 > nginx.yaml
vim nginx.yaml
```

```bash
k expose deployment nginx --name=nginx-service --port=80 --target-port=80
```

```yaml
#netpol.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: granted
```

```bash
k create -f netpol.yaml
```

```bash
k get po -owide
k get svc

k run tmp --image=busybox --restart=Never --rm -it -- wget -O- 10.42.0.21:80 # won't work
k run tmp --image=busybox --restart=Never --rm --labels=access=granted -it -- wget -O- 10.42.0.21:80 # will work
```

</p>
</details>
