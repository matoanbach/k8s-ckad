![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)

# Services and Networking (13%)

### Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --expose=80
```

</p>
</details>

### Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
k get ep
# output
NAME         ENDPOINTS              AGE
kubernetes   192.168.243.144:6443   5m14s
nginx        10.42.0.10:80          26s
```

</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
k run tmp --restart=Never --image=nginx:alpine --rm -it -- sh
# output
If you don't see a command prompt, try pressing enter.
/ # wget -O- 10.42.0.10:80
Connecting to 10.42.0.10:80 (10.42.0.10:80)
writing to stdout
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>s
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
  creationTimestamp: "2025-01-09T19:07:02Z"
  name: nginx
  namespace: default
  resourceVersion: "766"
  uid: 8191546a-ef50-4532-821b-195afe76a2d8
spec:
  clusterIP: 10.43.180.10
  clusterIPs:
    - 10.43.180.10
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - port: 80
      protocol: TCP
      nodePort: 30019
      targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

</p>
</details>

### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
k create deployment foo --image=dgkanatsios/simpleapp --restart=Never -l app=foo --port=8080
```

</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

<details><summary>show</summary>
<p>

```bash
k get po -owide
# output
NAME                   READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
foo-6b747fc57f-6cn62   1/1     Running   0          18s     10.42.0.12   controlplane   <none>           <none>
nginx                  1/1     Running   0          7m48s   10.42.0.10   controlplane   <none>           <none>
webapp-1               1/1     Running   0          8m10s   10.42.0.9    controlplane   <none>           <none>
```

```bash
k run tmp --restart=Never --image=nginx:alpine --rm -it -- curl -m 5 10.42.0.12:8080
# output
Hello world from foo-6b747fc57f-6cn62 and version 2.0
pod "tmp" deleted
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>

```bash
k expose deployment foo --name=foo-service --port=6262
```

```bash
k get svc
# output
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
foo-service   ClusterIP   10.43.95.108   <none>        6262/TCP       2s
kubernetes    ClusterIP   10.43.0.1      <none>        443/TCP        17m
nginx         NodePort    10.43.180.10   <none>        80:30019/TCP   13m
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>
<p>

```bash
k run busybox --restart=Never --image=busybox --rm -it -- /bin/sh
# output
If you don't see a command prompt, try pressing enter.
/ # for i in `seq 1 5`; do wget -O- 10.43.95.108:6262; done
Connecting to 10.43.95.108:6262 (10.43.95.108:6262)
writing to stdout
Hello world from foo-6b747fc57f-6cn62 and version 2.0
-                    100% |************************************************************************************************************|    54  0:00:00 ETA
written to stdout
Connecting to 10.43.95.108:6262 (10.43.95.108:6262)
writing to stdout
Hello world from foo-6b747fc57f-6cn62 and version 2.0
-                    100% |************************************************************************************************************|    54  0:00:00 ETA
written to stdout
Connecting to 10.43.95.108:6262 (10.43.95.108:6262)
writing to stdout
Hello world from foo-6b747fc57f-6cn62 and version 2.0
-                    100% |************************************************************************************************************|    54  0:00:00 ETA
written to stdout
Connecting to 10.43.95.108:6262 (10.43.95.108:6262)
writing to stdout
Hello world from foo-6b747fc57f-6cn62 and version 2.0
-                    100% |************************************************************************************************************|    54  0:00:00 ETA
written to stdout
Connecting to 10.43.95.108:6262 (10.43.95.108:6262)
writing to stdout
Hello world from foo-6b747fc57f-6cn62 and version 2.0
-                    100% |************************************************************************************************************|    54  0:00:00 ETA
written to stdout
```

</p>
</details>

### Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the pods in this deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

> Note that network policies may not be enforced by default, depending on your k8s implementation. E.g. Azure AKS by default won't have policy enforcement, the cluster must be created with an explicit support for `netpol` https://docs.microsoft.com/en-us/azure/aks/use-network-policies#overview-of-network-policy

<details><summary>show</summary>
<p>

```bash
k create deployment mydeploy --image=nginx --replicas=2
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mydeploy
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              access: granted
```

```bash
k get po -owide
#output
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
foo-6b747fc57f-6cn62        1/1     Running   0          15m     10.42.0.12   controlplane   <none>           <none>
mydeploy-5b9b9b667b-g8ntq   1/1     Running   0          2m46s   10.42.0.21   controlplane   <none>           <none>
mydeploy-5b9b9b667b-xbczq   1/1     Running   0          2m46s   10.42.0.22   controlplane   <none>           <none>
nginx                       1/1     Running   0          22m     10.42.0.10   controlplane   <none>           <none>
webapp-1                    1/1     Running   0          23m     10.42.0.9    controlplane   <none>           <none>
```

```bash
k run busybox --restart=Never --image=busybox --label=access=granted --rm -it -- wget -O- 10.42.0.21:80 # won't work

k run busybox --restart=Never --image=busybox --labels=access=granted --rm -it -- wget -O- 10.42.0.21:80 # will work
# output
If you don't see a command prompt, try pressing enter.
writing to stdout
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |************************************************************************************************************|   615  0:00:00 ETA
written to stdout
pod "busybox" deleted
```

</p>
</details>
