![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/multi_container&empty)

# Multi-container Pods (10%)

### Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

<details><summary>show</summary>
<p>

```bash
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit
kubectl delete po busybox
```

</p>
</details>

### Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using 'echo "Test" > /work-dir/index.html'. Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

<details><summary>show</summary>
<p>

```bash
containers:
  image: busybox
  imagePullPolicy: IfNotPresent
  name: busybox
  resources: {}
  ports:
  - containerPort: 80
  volumeMounts:
    - name: work-volume
      mountPath: /usr/share/nginx/html
initContainers:
  - name: busybox2
    image: busybox
    command: 
    - "/bin/sh"
    - -c
    - "echo "Test" > /work-dir/index.html"
    volumeMounts:
    - name: work-volume
      mountPath: /work-dir
volumes:
- name: work-volume
  emptyDir: {}
```

```bash
kubectl get pod pod1 -owide
kubectl run tmp --restart=Never --rm --image=busybox -- wget -O- <IP>
```


</p>
</details>
