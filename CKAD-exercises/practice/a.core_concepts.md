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
k create ns mynamespace
```

```bash
k run nginx --image=nginx  --restart=Never -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never -n mynamespae --dry-run=client -oyaml > nginx.yaml

k create -f nginx.yaml
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --rm -it -- env
# output

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=busybox
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.43.0.1:443
HOME=/root
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --dry-run=client -oyaml -- env > busybox.yaml
vim busy.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
    - args:
        - env
      image: busybox
      name: busybox
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
k create -f busy.box
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
k create ns myns --dry-run=client -oyaml > myns.yaml
```

</p>
</details>

### Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
k create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -oyaml > myrq.yaml
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
k get pods -A
k get pods --all-namespaces
```

</p>
</details>

### Create a pod with image nginx called nginx and expose traffic on port 80

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --port=80 --expose
```

```bash
k run tmp --restart=Never --rm --image=nginx:alpine -it -- wget -O- nginx:80
```

</p>
</details>

### Change pod's image to nginx:1.24.0. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

```bash
k set image pod nginx nginx=nginx:1.24.0
```

```bash
k describe pod nginx | grep -i image
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
k get pod nginx -owide
```

```bash
k run tmp --restart=Never --rm --image=nginx:alpine -it -- wget -O- nginx:80
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
k get pod nginx -oyaml > nginx.yaml
cat nginx.yaml
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
k get pod nginx
k describe pod nginx
```

</p>
</details>

### Get pod logs

<details><summary>show</summary>
<p>

```bash
k logs nginx
```

</p>
</details>

### If pod crashed and restarted, get logs about the previous instance

<details><summary>show</summary>
<p>

```bash
k logs nginx -p
k logs nginx --previous
```

</p>
</details>

### Execute a simple shell on the nginx pod

<details><summary>show</summary>
<p>

```bash
k exec nginx -it -- /bin/sh
```

</p>
</details>

### Create a busybox pod that echoes 'hello world' and then exits

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never -it -- /bin/sh -c 'echo "Hello world"'
```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --rm -it -- /bin/sh -c 'echo "Hello world"'
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --env=var1=val1
```

```bash
k exec nginx -it -- sh -c 'echo $var1'
# output
val1
```

</p>
</details>
```
