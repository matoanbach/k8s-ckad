![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/configuration&empty)

# Configuration (18%)

[ConfigMaps](#configmaps)

[SecurityContext](#securitycontext)

[Requests and Limits](#requests-and-limits)

[Secrets](#secrets)

[Service Accounts](#serviceaccounts)

<br>#Tips, export to variable<br>
<br>export ns="-n secret-ops"</br>
<br>export do="--dry-run=client -oyaml"</br>

## ConfigMaps

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Create a configmap named config with values foo=lala,foo2=lolo

<details><summary>show</summary>
<p>

```bash
k create cm config --from-literal=foo=lala --from-literal=foo2=lolo

k get cm config

k describe cm config
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
k describe cm config
```

</p>
</details>

### Create and display a configmap from a file

Create the file with

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>show</summary>
<p>

```bash
k create cm config --from-file=config.txt
```

</p>
</details>

### Create and display a configmap from a .env file

Create the file with the command

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>show</summary>
<p>

```bash
k create cm-env --from-env-file=config.env
```

</p>
</details>

### Create and display a configmap from a file, giving the key 'special'

Create the file with

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>show</summary>
<p>

```bash
k create cm cm-special --from-file=special=config4.txt
```

</p>
</details>

### Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

<details><summary>show</summary>
<p>

```bash
k create cm options --from-literal=var5=val5
```

```bash
k run nginx --image=nginx --dry-run=client -oyaml > options.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      env:
        - name: option
          valueFrom:
            configMapKeyRef:
              name: options
              key: var5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec nginx -it  -- /bin/sh
# env
    WEBAPP_COLOR_SERVICE_PORT_8080_TCP_ADDR=10.43.14.77
    WEBAPP_COLOR_SERVICE_SERVICE_HOST=10.43.14.77
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_PORT=tcp://10.43.0.1:443
    WEBAPP_COLOR_SERVICE_PORT_8080_TCP_PORT=8080
    HOSTNAME=nginx
    WEBAPP_COLOR_SERVICE_PORT_8080_TCP_PROTO=tcp
    HOME=/root
    WEBAPP_COLOR_SERVICE_PORT=tcp://10.43.14.77:8080
    WEBAPP_COLOR_SERVICE_SERVICE_PORT=8080
    PKG_RELEASE=1~bookworm
    DYNPKG_RELEASE=1~bookworm
    WEBAPP_COLOR_SERVICE_PORT_8080_TCP=tcp://10.43.14.77:8080
    TERM=xterm
    NGINX_VERSION=1.27.3
    KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    NJS_VERSION=0.8.7
    KUBERNETES_PORT_443_TCP_PORT=443
    option=val5
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    NJS_RELEASE=1~bookworm
    KUBERNETES_SERVICE_PORT_HTTPS=443
    KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
    KUBERNETES_SERVICE_HOST=10.43.0.1
    PWD=/
    # exit
```

</p>
</details>

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
k create cm anotherone --from-literal=var6=val6 --from-literal=var7=val7
```

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      envFrom:
        - configMapRef:
            name: anotherone
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory.

<details><summary>show</summary>
<p>

```bash
k create cm cmvolume --from-literal=var8=val8 --from-literal=var9=val9
```

```bash
k run nginx --image=nginx --dry-ru=client -oyaml > nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      volumeMounts:
        - name: config
          mountPath: /etc/lala
  volumes:
    - name: config
      configMap:
        name: cmvolume
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
controlplane ~ ➜  k exec nginx -it -- sh
    >_ cd /etc/lala
    >_ ls
        var8  var9
    >_ cat var8
        val8
```

</p>
</details>

## SecurityContext

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > 101.yaml
vim 101.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      securityContext:
        runAsUser: 101
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added to its single container

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > 101.yaml
vim NET_ADMIN.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      securityContext:
        runAsUser: 101
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Resource requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --dry-run=client -oyaml > nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
        limits:
          cpu: 200m
          memory: 512Mi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

## Limit Ranges

kubernetes.io > Documentation > Concepts > Policies > Limit Ranges (https://kubernetes.io/docs/concepts/policy/limit-range/)

### Create a namespace named limitrange with a LimitRange that limits pod memory to a max of 500Mi and min of 100Mi

<details><summary>show</summary>
<p>

```bash
k create ns limitrange
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lm
  namespace: limitrange
spec:
  limits:
    - max: # max and min define the limit range
        memory: 500Mi
      min:
        memory: 100Mi
      type: Pod
```

```bash
k create -f lm.yaml
```

</p>
</details>

### Describe the namespace limitrange

<details><summary>show</summary>
<p>

```bash
k describe limitranges lm -n limitrange
```

</p>
</details>

### Create an nginx pod that requests 250Mi of memory in the limitrange namespace

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --dry-run=client -oyaml > nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: limitrange
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          memory: 250Mi
        limits:
          memory: 500Mi # max and min define the limit range
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create -f nginx.yaml
    Error from server (Forbidden): error when creating "nginx.yaml": pods "nginx" is forbidden: maximum memory usage per Pod is 500Mi.  No limit is specified
```

</p>
</details>

## Resource Quotas

kubernetes.io > Documentation > Concepts > Policies > Resource Quotas (https://kubernetes.io/docs/concepts/policy/resource-quotas/)

### Create ResourceQuota in namespace `one` with hard requests `cpu=1`, `memory=1Gi` and hard limits `cpu=2`, `memory=2Gi`.

<details><summary>show</summary>
<p>

```bash
k create quota myquota --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi -n one
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myquota
  namespace: one
spec:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
status: {}
```

```bash
k get quota myquota
```

</p>
</details>

### Attempt to create a pod with resource requests `cpu=2`, `memory=3Gi` and limits `cpu=3`, `memory=4Gi` in namespace `one`

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "2"
          memory: "3Gi"
        limits:
          cpu: "3"
          memory: "4Gi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k run nginx --image=nginx --dry-run=client -oyaml > nginx.yaml
    Error from server (Forbidden): error when creating "nginx.yaml": pods "nginx" is forbidden: exceeded quota: myquota, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi
```

</p>
</details>

### Create a pod with resource requests `cpu=0.5`, `memory=1Gi` and limits `cpu=1`, `memory=2Gi` in namespace `one`

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "0.5"
          memory: "1Gi"
        limits:
          cpu: "1"
          memory: "2Gi"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k get po
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          46s
```

</p>
</details>

## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Create a secret called mysecret with the values password=mypass

<details><summary>show</summary>
<p>

```bash
k create secret generic mysecret --from-literal=password=mypass
```

</p>
</details>

### Create a secret called mysecret2 that gets key/value from a file

Create a file called username with the value admin:

```bash
echo -n admin > username
```

<details><summary>show</summary>
<p>

```bash
k create secret generic mysecret2 --from-file=username
```

</p>
</details>

### Get the value of mysecret2

<details><summary>show</summary>
<p>

```bash
k get secrets mysecret2 -oyaml
>_
apiVersion: v1
data:
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2025-01-08T18:31:03Z"
  name: mysecret2
  namespace: one
  resourceVersion: "1347"
  uid: d38dda46-4033-4129-8575-e7683571e857
type: Opaque
```

```bash
echo YWRtaW4= | base64 -d
>_
admin
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --dry-run=client -oyaml > mypod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      volumeMounts:
        - name: mysecret2-vol
          mountPath: /etc/foo
  volumes:
    - name: mysecret2-vol
      secret:
        secretName: mysecret2
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec nginx -it -- sh
>_ env
k exec nginx -it -- sh
# ls /etc/foo
username
# cd /etc/foo
# ls
username
# cat username
admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
k delete --force pod nginx
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
    - image: nginx
      name: nginx
      resources: {}
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret2
              key: username

  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

### Create a Secret named 'ext-service-secret' in the namespace 'secret-ops'. Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal.

<details><summary>show</summary>
<p>

```bash
k create secret generic ext-service-secret -n secret-ops --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T
```

</p>
</details>

### Consuming the Secret. Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops' and consume the Secret as an environment variable. Then, open an interactive shell to the Pod, and print all environment variables.

<details><summary>show</summary>
<p>

```bash
k run consumer --image=nginx -n secret-ops --dry-run=client -oyaml > consumer.yaml
vim consumer.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
    - image: nginx
      name: consumer
      resources: {}
      envFrom:
        - secretRef:
            name: ext-service-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec consumer -it -- sh
>_ env
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_PORT=tcp://10.43.0.1:443
    HOSTNAME=consumer
    HOME=/root
    PKG_RELEASE=1~bookworm
    DYNPKG_RELEASE=1~bookworm
    TERM=xterm
    NGINX_VERSION=1.27.3
    KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    NJS_VERSION=0.8.7
    KUBERNETES_PORT_443_TCP_PORT=443
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    NJS_RELEASE=1~bookworm
    API_KEY=LmLHbYhsgWZwNifiqaRorH8T
    KUBERNETES_SERVICE_PORT_HTTPS=443
    KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
    KUBERNETES_SERVICE_HOST=10.43.0.1
```

</p>
</details>

### Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory.

<details><summary>show</summary>
<p>

```bash
k create secret generic my-secret --type=kubernetes.io/ssh-auth --from-file=ssh-privatekey=id_rsa --dry-run=client -oyaml > sc.yaml
vim sc.yaml
```

```yaml
apiVersion: v1
data:
  ssh-privatekey: ""
kind: Secret
metadata:
  creationTimestamp: null
  name: my-scret
type: kubernetes.io/ssh-auth
```

```bash
k create -f sc.yaml
```

</p>
</details>

### Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file.

<details><summary>show</summary>
<p>

```bash
k run consumer --image=nginx -n secret-ops --dry-run=client -oyaml > consumer.yaml
vim consumer.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
    - image: nginx
      name: consumer
      resources: {}
      volumeMounts:
        - name: secret-volume
          mountPath: /var/app
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec consumer -it -- sh
# cd /var/app
# ls
ssh-privatekey
# cat ssh-privatekey
so8y12hp9edmih1123w1ds1d
```

</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
k get sa -A
k get sa --all-namespaces
```

</p>
</details>

### Create a new serviceaccount called 'myuser'

<details><summary>show</summary>
<p>

```bash
k create sa myuser
```

</p>
</details>

### Create an nginx pod that uses 'myuser' as a service account

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --dry-run=client -oyaml > nginx.yaml
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
spec:
  serviceAccountName: myuser
  containers:
    - image: nginx
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create -f nginx.yaml
k get po
k describe po nginx
```

</p>
</details>

### Generate an API token for the service account 'myuser'

<details><summary>show</summary>
<p>

```bash
k create token myuser > mytoken.token
    eyJhbGciOiJSUzI1NiIsImtpZCI6IkZkRWNHaUZzMHNYdFVTMDhObjZjM1dpdngzcFVCT19TTmtVcTNIaFNKX1UifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiLCJrM3MiXSwiZXhwIjoxNzM2MzY3MTQyLCJpYXQiOjE3MzYzNjM1NDIsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiOTg0YzlhMzEtOTY4Yi00MDNkLTg1YjAtMjQwMTU5ZGQ1NGVlIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJzZWNyZXQtb3BzIiwic2VydmljZWFjY291bnQiOnsibmFtZSI6Im15dXNlciIsInVpZCI6IjFmNDM0OGE4LTc5NTktNGI0OS1iOTgxLTg1NDAzNWY4OTNhMCJ9fSwibmJmIjoxNzM2MzYzNTQyLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6c2VjcmV0LW9wczpteXVzZXIifQ.K2Vv4LLXzHCNHMqIhZ9QdO7Z5RIofG9erLD4dpnH9TwK3Ovq4Qxw4CAVLmj1fpr6GVi3lCnn_ZTOV6cp0tPQpqEjuPb9-2B_MeN4uAYYeH0avcMqJK8SkUbTFIn2IhFCAg1I5OOrMMhyjg3GvbKQYWl7olB6cgLpHUmu3AAVP_OJ4inMkN45Uyu7XWiKSRjSe-LA0Ez0vPaWh_GfBAoE0_31h2V5oWmaIuJhXxRCJCD7uXmP4io8Y_dtlDwzkUf5WgI7WtUXJYzCjWlqEcsIkXym509UovIsMjb8v1eoJUYnFPWY4jfEkkWLbJUEFgx9fo5evz34Vus7mUuugKQBYQ
```

</p>
</details>
