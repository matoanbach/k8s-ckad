![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/state&empty)

# State Persistence (8%)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Define volumes

### Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

<details><summary>show</summary>
<p>

_This question is probably a better fit for the 'Multi-container-pods' section but I'm keeping it here as it will help you get acquainted with state_

```bash
k run multipod --image=busybox  --dry-run=client -oyaml --command -- /bin/sh -c 'sleep 3600' > multipod.yaml
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
        - sleep 3600
      image: busybox
      name: busybox1
      resources: {}

      volumeMounts:
        - name: data-vol
          mountPath: /etc/foo

    - command:
        - /bin/sh
        - -c
        - sleep 3600
      image: busybox
      name: busybox2
      resources: {}
      volumeMounts:
        - name: data-vol
          mountPath: /etc/foo

  volumes:
    - name: data-vol
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

</p>
</details>

### Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

<details><summary>show</summary>
<p>

```bash
vim pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myvolume
  labels:
    type: local
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: "etc/foo"
```

```bash
k get pv
```

</p>
</details>

### Create a PersistentVolumeClaim for this PersistentVolume, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show</summary>
<p>

```bash
vim pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

</p>
</details>

### Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --dry-run=client -oyaml --command -- /bin/sh -c 'sleep 3600' > pod.yaml
vim pod.yaml
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
        - /bin/sh
        - -c
        - sleep 3600
      image: busybox
      imagePullPolicy: IfNotPresent
      name: busybox
      resources: {}
      volumeMounts: #
        - name: myvolume #
          mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
    - name: myvolume #
      persistentVolumeClaim: #
        claimName: mypvc #
status: {}
```

```bash
kubectl create -f pod.yaml
```

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### Create a second pod which is identical with the one you just created (you can easily do it by changing the 'name' property on pod.yaml). Connect to it and verify that '/etc/foo' contains the 'passwd' file. Delete pods to cleanup. Note: If you can't see the file from the second pod, can you figure out why? What would you do to fix that?

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox2
spec:
  containers:
    - args:
        - /bin/sh
        - -c
        - sleep 3600
      image: busybox
      imagePullPolicy: IfNotPresent
      name: busybox
      resources: {}
      volumeMounts: #
        - name: myvolume #
          mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
    - name: myvolume #
      persistentVolumeClaim: #
        claimName: mypvc #
status: {}
```

</p>
</details>

### Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl cp busybox:/etc/password ./passwd
cat passwd
```
</p>
</details>
