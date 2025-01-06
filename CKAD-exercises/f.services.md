![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/services&empty)

# Services and Networking (13%)

### Create a pod with image nginx called nginx and expose its port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
```

</p>
</details>

### Confirm that ClusterIP has been created. Also check endpoints

<details><summary>show</summary>
<p>

```bash
kubectl get svc nginx
kubectl get ep
```

</p>
</details>

### Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget

<details><summary>show</summary>
<p>

```bash
kubectl get svc
kubectl run tmp --restart=Never --image=busybox --rm -it -- wget -O- <IP>
```

</p>
or
<p>

```bash
IP=$(kubectl get svc nginx --template={{.sepc.clusterIP}})
kubectl run busybox --restart=Never --image=busybox -it --restart=Never --env="IP=$IP" -- wget -O- $IP:80 --timeout 2
```

</p>
</details>

### Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.

<details><summary>show</summary>
<p>

```bash
kubectl edit svc nginx
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2018-06-25T07:55:16Z
  name: nginx
  namespace: default
  resourceVersion: "93442"
  selfLink: /api/v1/namespaces/default/services/nginx
  uid: 191e3dac-784d-11e8-86b1-00155d9f663c
spec:
  clusterIP: 10.97.242.220
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    run: nginx
  sessionAffinity: None
  type: NodePort # change cluster IP to nodeport
status:
  loadBalancer: {}
```

or

```bash
kubectl patch svc nginx -p '{"spec": {"type":"NodePort"}}'
```

```bash
wget -O- NODE_IP:31931
```

</p>
</details>

### Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

<details><summary>show</summary>
<p>

```bash
kubectl create deployment foo --image=dgkanatsios/simpleapp --replicas=3 --restart=Never -l app=foo --port=8080
```

</p>
</details>

### Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080

<details><summary>show</summary>
<p>

```bash
kubectl get pod -o wide

kubectl run tmp --restart=Never --image=busybox --rm --it -- wget -O- <IP>:8080
```

</p>
</details>

### Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints

<details><summary>show</summary>
<p>

```bash
kubectl expose deployment depl --port=6262 --target-port=8080

kubectl get ep
```

</p>
</details>

### Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster

<details><summary>show</summary>
<p>

```bash
kubectl run tmp --restart=busybox --image=busybox --rm -it -- /bin/sh
wget -O- foo:6262
```

</p>
</details>

### Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the pods in this deployment and apply it

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

> Note that network policies may not be enforced by default, depending on your k8s implementation. E.g. Azure AKS by default won't have policy enforcement, the cluster must be created with an explicit support for `netpol` https://docs.microsoft.com/en-us/azure/aks/use-network-policies#overview-of-network-policy

<details><summary>show</summary>
<p>

```bash
kubectl create deployment nginx --image=nginx --replicas=2 --port=80
kubectl expose deployment nginx --port=80
```

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
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
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- http://nginx:80 --timeout 2 # wont work
kubectl run busybox --image=busybox --rm -it --restart=Never --labels=access=granted s-- wget -O- http://nginx:80 --timeout 2

```

</p>
</details>
