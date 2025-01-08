![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)

# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

```bash
k run multipod --image=busybox --restart=Never --dry-run=client -oyaml --command -- /bin/sh -c 'echo "hello"; sleep 3600' > multipod.yaml
vim multipod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multipod
  name: multipod
spec:
  containers:
    - command:
        - /bin/sh
        - -c
        - echo "hello"; sleep 3600
      image: busybox
      name: container1
      resources: {}

    - command:
        - /bin/sh
        - -c
        - echo "hello"; sleep 3600
      image: busybox
      name: container2
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
k exec multipod -c container2 -it -- ls
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using 'echo "Test" > /work-dir/index.html'. Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --port=80 --restart=Never --dry-run=client -oyaml > init.yaml
vim init.yaml
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
  initContainers:
    - image: busybox
      name: init-con
      command:
        - sh
        - -c
        - 'echo "Test" > /work-dir/index.html'
      volumeMounts:
        - name: data-mount
          mountPath: /work-dir
  containers:
    - image: nginx
      name: nginx
      ports:
        - containerPort: 80
      resources: {}
      volumeMounts:
        - name: data-mount
          mountPath: /usr/share/nginx/html
  volumes:
    - name: data-mount
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
k get po -owide
k run tmp --restart=Never --image=nginx:alpine --rm -it -- wget -O- 10.42.0.30
```

</p>
</details>
