![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)

# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --dry-run=client -oyaml --command -- /bin/sh 'echo "hello"; sleep 3600' > busybox.yaml
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
        - /bin/sh
        - -c
        - echo "hello"; sleep 3600
      image: busybox
      name: busybox1
      resources: {}

    - command:
        - /bin/sh
        - -c
        - echo "hello"; sleep 3600
      image: busybox
      name: busybox2
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}s
```

```bash
k exec busybox -c busybox2 -it -- sh -c 'ls'
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using 'echo "Test" > /work-dir/index.html'. Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --port=80 --restart=Never --dry-run=client -oyaml > nginx.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: multicon
spec:
  initContainers:
    - image: busybox
      name: init-con
      command:
        - sh
        - -c
        - echo "Test" > /work-dir/index.html
      volumeMounts:
        - name: test-mount
          mountPath: /work-dir
  containers:
    - image: nginx
      name: nginx
      ports:
        - containerPort: 80
      resources: {}

      volumeMounts:
        - name: test-mount
          mountPath: /usr/share/nginx/html
  volumes:
    - name: test-mount
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
k get po -owide
# output
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
multicon   1/1     Running   0          13s   10.42.0.18   controlplane   <none>           <none>
```

```bash
k run tmp --restart=Never --image=nginx:alpine --rm -it -- sh
# output
If you don't see a command prompt, try pressing enter.
/ # wget -O- 10.42.0.18:80
Connecting to 10.42.0.18:80 (10.42.0.18:80)
writing to stdout
Test
-                    100% |************************************************************************************************************|     5  0:00:00 ETA
written to stdout
/ # exit
pod "tmp" deleted
```

</p>
</details>
