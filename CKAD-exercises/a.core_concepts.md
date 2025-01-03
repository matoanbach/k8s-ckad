![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/core_concepts&empty)

# Core Concepts (13%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

### Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

<details><summary>show</summary>
<p>

```bash
kubectl create namespace mynamespace
```

```bash
kubectl run nginx --image=nginx -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx -n mynamespace --dry-run=client -o yaml > nginx.yaml
kubectl create -f nginx.yaml -n mynamespace
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -it  --rm --command -- env
kubectl run busybox --image=busybox --restart=Never --command -- env
kubectl logs busybox
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never --command -- env --dry-run=client -o yaml > busybox.yaml
cat busybox.yaml
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create ns myns --dry-run=client -o yaml > myns.yaml
cat myns.yaml
```

</p>
</details>

### Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: myrq
spec:
  hard:
    cpu: "1"
    memory: 1Gi
    pods: "2"
  scopeSelector:
    matchExpressions:
      - operator: In
    scopeName: PriorityClass
    values: ["medium"]
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
kubectl get pod --all-namespaces
kubectl get pod -A
```

</p>
</details>

### Create a pod with image nginx called nginx and expose traffic on port 80

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

</p>
</details>

### Change pod's image to nginx:1.24.0. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

```bash
kubectl set image pod mypod nginx=nginx:1.24.0
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
kubectl get po -o wide
kubectl run tmp --restart=Never --rm --image=busybox:alpine -it -- wget -O- [ip_address]
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
kubectl get pod mypod -o yaml
kubectl get pod mypod -oyaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
kubectl describe pod mypod
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
kubectl logs mypod
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
kubectl logs nginx -p
kubectl logs nginx --privious
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
kubectl exec mypod -it -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
kubectl run mypod --restart=Never --image=busybox --command -- /bin/sh -c 'echo "hello world"'
# or
kubectl run mypod --restart=Never --image=busybox --command -- echo "hello world"
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
kubectl run mypod --restart=Never -rm --image=busybox --command -- /bin/sh -c 'echo "hello world"'
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
kubectl run mypod --image=nginx --restart=Never --env=var1=val1
kubectl exet -it nginx -- env
kubectl exet -it nginx -- /bin/sh -c  'echo $var1'
```

</p>
</details>
```
