![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/observability&empty)

# Observability (18%)

## Liveness, readiness and startup probes

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

### Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml --command -- ls > nginx.yaml
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
      livenessProbe:
        exec:
          command:
            - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
k get po
# output
NAME              READY   STATUS    RESTARTS   AGE
nginx             1/1     Running   0          29s
simple-webapp-1   1/1     Running   0          2m13s
```

</p>
</details>

### Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

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
      resources: {}
      livenessProbe:
        exec:
          command:
            - ls
        initialDelaySeconds: 5
        periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
k describe pod/nginx | grep -i liveness
# output
    Liveness:       exec [ls] delay=5s timeout=1s period=5s #success=1 #failure=3
```

</p>
</details>

### Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

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
      ports:
        - containerPort: 80 # Note: Readiness probes runs on the container during its whole lifecycle. Since nginx exposes 80, containerPort: 80 is not required for readiness to work.
      readinessProbe: # declare the readiness probe
        httpGet: # add this line
          path: / #
          port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Lots of pods are running in `qa`,`alan`,`test`,`production` namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of `<namespace>/<pod name>` per line.

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
      resources: {}
      readinessProbe:
        exec:
          command:
            - ls
        initialDelaySeconds: 5
        periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Logging

### Create a busybox pod that runs `i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done`. Check its logs

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --command -- sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
```

```bash
k logs busybox  -f
# output
0: Thu Jan  9 18:58:37 UTC 2025
1: Thu Jan  9 18:58:38 UTC 2025
2: Thu Jan  9 18:58:39 UTC 2025
3: Thu Jan  9 18:58:40 UTC 2025
4: Thu Jan  9 18:58:41 UTC 2025
5: Thu Jan  9 18:58:42 UTC 2025
6: Thu Jan  9 18:58:43 UTC 2025
7: Thu Jan  9 18:58:44 UTC 2025
8: Thu Jan  9 18:58:45 UTC 2025
9: Thu Jan  9 18:58:46 UTC 2025
```

</p>
</details>

## Debugging

### Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never -- /bin/sh -c 'ls /notexist'
```

```bash
k logs busybox
# output
ls: /notexist: No such file or directory
```

</p>
</details>

### Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never -- /bin/sh -c 'notexist'
```

```bash
k logs busybox
# output
/bin/sh: notexist: not found
```

```bash
k get error
```

</p>
</details>

### Get CPU/memory utilization for nodes ([metrics-server](https://github.com/kubernetes-incubator/metrics-server) must be running)

<details><summary>show</summary>
<p>

```bash
k top nodes
```

</p>
</details>
