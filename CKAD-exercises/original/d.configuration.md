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
k create cm config --from-literal=foo=lala --from-literal=foo2=loo
```

</p>
</details>

### Display its values

<details><summary>show</summary>
<p>

```bash
k get cm config  -oyaml
apiVersion: v1
data:
  foo: lala
  foo2: loo
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-09T17:10:38Z"
  name: config
  namespace: default
  resourceVersion: "772"
  uid: 39cccd60-fb4d-4b69-a420-38fba8fe66f4Í
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
k create cm config-file --from-file=config.txt
```

```bash
k get cm config-file -oyaml
# output
apiVersion: v1
data:
  config.txt: |
    foo3=lili
    foo4=lele
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-09T17:12:01Z"
  name: config-file
  namespace: default
  resourceVersion: "798"
  uid: e4a3d35e-cd54-4789-9299-31a2771420e5
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
k create cm config-env --from-env-file=config.env
```

```bash
k get cm config-env -oyaml
# output
apiVersion: v1
data:
  var1: val1
  var2: val2
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-09T17:13:25Z"
  name: config-env
  namespace: default
  resourceVersion: "825"
  uid: 3dfb6047-24e4-4113-af6b-2221b3254f50
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
k create cm config4-file --from-file=special=config4.txt
```

```bash
k get cm config4-file -oyaml
# output
apiVersion: v1
data:
  special: |
    var3=val3
    var4=val4
kind: ConfigMap
metadata:
  creationTimestamp: "2025-01-09T17:14:22Z"
  name: config4-file
  namespace: default
  resourceVersion: "841"
  uid: a0b8e287-401e-4563-8761-9d4062128656s
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
      env:
        - name: option
          valueFrom:
            configMapKeyRef:
              name: options
              key: var5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>cat

### Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod

<details><summary>show</summary>
<p>

```bash
k create cm anotherone --from-literal=var6=val6 --from-literal=var7=val7
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
status: {}s
```

```bash
k exec nginx -it -- sh -c 'echo $var6'
# output
val6
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
k exec nginx -it -- sh -c 'ls /etc/lala'
# output
var8  var9
```

</p>
</details>

## SecurityContext

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod

<details><summary>show</summary>
<p>

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
  securityContext: # insert this line
    runAsUser: 101 # UID for the user
  containers:
    - image: nginx
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added to its single container

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
      imagePullPolicy: IfNotPresent
      name: nginx
      resources: {}
    securityContext: # insert this line
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
k create nginx --image=nginx --restart=Never --dry-run=client -oyaml > nginx.yaml
vim nginx.yaml
```

```yaml

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
k config set-context --current --namespace=limitrange
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mylimitrange
spec:
  limits:
    - max: # max and min define the limit range
        memory: 500Mi
      min:
        memory: 100Mi
      type: Pod
```

```bash
k describe limitrange

# output
Name:       mylimitrange
Namespace:  default
Type        Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---    ---    ---------------  -------------  -----------------------
Pod         memory    100Mi  500Mi  -                -              -
```

</p>
</details>

### Describe the namespace limitrange

<details><summary>show</summary>
<p>

```bash
k describe ns limitrange
# output
Name:         limitrange
Labels:       kubernetes.io/metadata.name=limitrange
Annotations:  <none>
Status:       Active

No resource quota.

Resource Limits
 Type  Resource  Min    Max    Default Request  Default Limit  Max Limit/Request Ratio
 ----  --------  ---    ---    ---------------  -------------  -----------------------
 Pod   memory    100Mi  500Mi  -                -              -
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
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          memory: 250Mi
        limits:
          memory: 500Mi # required <= max
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

## Resource Quotas

kubernetes.io > Documentation > Concepts > Policies > Resource Quotas (https://kubernetes.io/docs/concepts/policy/resource-quotas/)

### Create ResourceQuota in namespace `one` with hard requests `cpu=1`, `memory=1Gi` and hard limits `cpu=2`, `memory=2Gi`.

<details><summary>show</summary>
<p>

```bash
k create quota myquota --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi --dry-run=client -oyaml > myquota.yaml
vim myquota.yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  creationTimestamp: null
  name: myquota
spec:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
status: {}
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
  namespace: one
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "2"
          memory: 3Gi
        limits:
          cpu: "3"
          memory: 4Gi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create -f nginx.yaml
# output
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
  namespace: one
spec:
  containers:
    - image: nginx
      name: nginx
      resources:
        requests:
          cpu: "0.5"
          memory: 1Gi
        limits:
          cpu: "1"
          memory: 2Gi
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
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

```yaml
k get secret mysecret -oyaml
# output
apiVersion: v1
data:
  password: bXlwYXNz
kind: Secret
metadata:
  creationTimestamp: "2025-01-09T18:09:26Z"
  name: mysecret
  namespace: default
  resourceVersion: "784"
  uid: cac808d5-abad-48fc-9fae-90f224cc62a6
type: Opaque
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
k get secret mysecret2 -oyaml
# outpu
apiVersion: v1
data:
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2025-01-09T18:11:17Z"
  name: mysecret2
  namespace: default
  resourceVersion: "817"
  uid: 88faddf8-6485-42b1-a88b-65e9f7a06055
type: Opaque
```

</p>
</details>

### Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > nginx.yaml
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
  containers:
    - image: nginx
      name: nginx
      resources: {}
      volumeMounts:
        - name: secret-mount
          mountPath: /etc/foo
  volumes:
    - name: secret-mount
      secret:
        secretName: mysecret2
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
k exec nginx -it -- cat /etc/foo/username
# output
admin
```

</p>
</details>

### Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

<details><summary>show</summary>
<p>

```bash
k delete pod nginx
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
  restartPolicy: Never
status: {}
```

```bash
k exec nginx -it -- env
# output
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx
NGINX_VERSION=1.27.3
NJS_VERSION=0.8.7
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
USERNAME=admin
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_SERVICE_PORT=443
TERM=xterm
HOME=/root
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
k exec consumer -it -- env
# output
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=consumer
NGINX_VERSION=1.27.3
NJS_VERSION=0.8.7
NJS_RELEASE=1~bookworm
PKG_RELEASE=1~bookworm
DYNPKG_RELEASE=1~bookworm
API_KEY=LmLHbYhsgWZwNifiqaRorH8T
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
TERM=xterm
HOME=/root
```

</p>
</details>

### Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory.

<details><summary>show</summary>
<p>

```bash
k create secret generic my-secret --type=kubernetes.io/ssh-auth --namespace=secret-ops --from-file=ssh-privatekey=id_rsa
```

</p>
</details>

### Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file.

<details><summary>show</summary>
<p>

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
        - name: secret-vol
          mountPath: /var/app
  volumes:
    - name: secret-vol
      secret:
        secretName: my-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec consumer -it -- cat /var/app/ssh-privatekey
# output
1kf492uwq
```

</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### See all the service accounts of the cluster in all namespaces

<details><summary>show</summary>
<p>

```bash
k get sa --all-namespaces
k get sa -A
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
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > nginx.yaml
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
  restartPolicy: Never
status: {}
```

```bash
k describe po nginx | grep -i service
# output
Service Account:  myuser
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-z6bsg (ro)
```


</p>
</details>

### Generate an API token for the service account 'myuser'

<details><summary>show</summary>
<p>

```bash
k create token myuser
# output
eyJhbGciOiJSUzI1NiIsImtpZCI6Ikc3NXJ6cDBRUkp4UjVLWWZEMmR5UWEzbHFURzVnOXUwNGN4RVFwSDRUYkUifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM2NDUxMzYxLCJpYXQiOjE3MzY0NDc3NjEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwianRpIjoiYTFiZTllNGQtZTk1ZS00YjVjLTk3ZDctMmQzYjdkY2ViOTY0Iiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6Im15dXNlciIsInVpZCI6IjdmOTcxYTNhLTRiNDAtNGY3Zi1hYzVjLTVkOTAxNDdmMTdjMyJ9fSwibmJmIjoxNzM2NDQ3NzYxLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpteXVzZXIifQ.kEx2wMyFWAufbeTD6wEOS6N-x6lvc6HvMwDZXks1QZ4fFcjPuWlNQwEyXsutKOJ25RlgJAORHFL7IiljSRNR6UmHXBaSEkPWNqIwe2TK2BMpQQX4fn2PQ1vgWMFJMPqlE05RP7nJTUrrOdGB9FBBw04xBbT-TKTX60X5u0t7LBWbIpeiEyWHxMkOzfHIoEG09Y8wFhirveQhU5-dWRcRdHcGUzSMy2M_ElUFLpAwhCqCMg24KMwiCxToasB9bvYq39qu7cN2_XMja6JMRE8MjbCYdW70EEQMW4VGAYWTmnilCcay_QG9CDPYiW47yuRnNw5aUsWd-mq6TpXFcSP_Cg
```

</p>
</details>
