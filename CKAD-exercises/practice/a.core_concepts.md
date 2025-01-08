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
k run nginx --image=nginx -n mynamespace
```

</p>
</details>

### Create the pod that was just described using YAML

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx -n mynamespace --restart=Never --dry-run=client -oyaml > nginx.yaml
vim nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

### Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never -it --rm --command -- sh -c env
k run busybox --image=busybox --restart=Never --command -- sh -c env
```

```bash
k get po
k logs busybox
```

</p>
</details>

### Create a busybox pod (using YAML) that runs the command "env". Run it and see the output

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --dry-run=client --restart=Never -oyaml --command -- sh -c env > busybox.yaml
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
    - command:
        - sh
        - -c
        - env
      image: busybox
      name: busybox
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

```bash
k create -f busybox.yaml
k logs busybox
```

</p>
</details>

### Get the YAML for a new namespace called 'myns' without creating it

<details><summary>show</summary>
<p>

```bash
k create ns myns --dry-run=client --restart=Never -oyaml > myns.yaml
vim myns.yaml # check the yaml file
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: myns
spec: {}
status: {}
```

</p>
</details>

### Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

<details><summary>show</summary>
<p>

```bash
k create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -oyaml
```

</p>
</details>

### Get pods on all namespaces

<details><summary>show</summary>
<p>

```bash
k get pods --all-namespaces
```

or

```bash
k get pods -A
```

</p>
</details>

### Create a pod with image nginx called nginx and expose traffic on port 80

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --port=80 --restart=Never
```

</p>
</details>

### Change pod's image to nginx:1.24.0. Observe that the container will be restarted as soon as the image gets pulled

<details><summary>show</summary>
<p>

```bash
k set image pod nginx nginx=nginx:1.24.0
```

</p>
</details>

### Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

<details><summary>show</summary>
<p>

```bash
k get pod nginx -owide
k run tmp --restart=Never --rm --image=busybox -it -- wget -O- "IP"
```

```html
Connecting to 10.42.0.17 (10.42.0.17:80) writing to stdout
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      html {
        color-scheme: light dark;
      }
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.
    </p>

    <p>
      For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br />
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
- 100% |********************************| 615 0:00:00 ETA written to stdout pod
"tmp" deleted
```

</p>
</details>

### Get pod's YAML

<details><summary>show</summary>
<p>

```bash
k get pod busybox -oyaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-01-08T13:37:59Z"
  labels:
    run: nginx
  name: nginx
  namespace: default
  resourceVersion: "1248"
  uid: 674ea041-aa02-4913-b7cb-35939aa8d9c6
spec:
  containers:
    - image: nginx:1.24.0
      imagePullPolicy: Always
      name: nginx
      ports:
        - containerPort: 80
          protocol: TCP
      resources: {}
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-mwr8m
          readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Never
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
    - effect: NoExecute
      key: node.kubernetes.io/not-ready
      operator: Exists
      tolerationSeconds: 300
    - effect: NoExecute
      key: node.kubernetes.io/unreachable
      operator: Exists
      tolerationSeconds: 300
  volumes:
    - name: kube-api-access-mwr8m
      projected:
        defaultMode: 420
        sources:
          - serviceAccountToken:
              expirationSeconds: 3607
              path: token
          - configMap:
              items:
                - key: ca.crt
                  path: ca.crt
              name: kube-root-ca.crt
          - downwardAPI:
              items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
                  path: namespace
    ...
```

</p>
</details>

### Get information about the pod, including details about potential issues (e.g. pod hasn't started)

<details><summary>show</summary>
<p>

```bash
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

#or

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
k run busybox --image=busybox --restart=Never --rm -it --command -- echo "hello world"

#or

k run busybox --image=busybox --restart=Never --rm -it --command -- /bin/sh -c 'echo "hello world"'

```

</p>
</details>

### Do the same, but have the pod deleted automatically when it's completed

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --rm -it --command -- echo "hello world"

#or

k run busybox --image=busybox --restart=Never --rm -it --command -- /bin/sh -c 'echo "hello world"'
```

</p>
</details>

### Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --env=var1=val1 --env=var2=val2
```

or

```bash
k exec nginx -it -- env
#or
k exec nginx -it -- /bin/sh
-> echo $var1
```
</p>
</details>
```
