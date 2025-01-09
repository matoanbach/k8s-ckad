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
k run busybox --image=busybox --restart=Never --dry-run=client -oyaml --command -- /bin/sh -c 'sleep 3600' > busybox.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: mybusybox
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
        - name: shared-volume
          mountPath: /etc/foo

    - command:
        - /bin/sh
        - -c
        - sleep 3600
      image: busybox
      name: busybox2
      resources: {}
      volumeMounts:
        - name: shared-volume
          mountPath: /etc/foo
  volumes:
    - name: shared-volume
      emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd # instead of cut command you can use awk -F ":" '{print $1}'
cat /etc/foo/passwd # confirm that stuff has been written successfully
exit
```

```bash
kubectl exec -it busybox -c busybox -- /bin/sh
mount | grep foo # confirm the mounting
cat /etc/foo/passwd
exit
kubectl delete po busybox
```

</p>
</details>

### Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster

<details><summary>show</summary>
<p>

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
    path: "/etc/foo"s
```

```bash
k create -f pv.yaml
# output
persistentvolume/myvolume created
```

```bash
k get pv
# output
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
myvolume   10Gi       RWO,RWX        Retain           Available           normal         <unset>
```

</p>
</details>

### Create a PersistentVolumeClaim for this PersistentVolume, called 'mypvc', a request of 4Gi and an accessMode of ReadWriteOnce, with the storageClassName of normal, and save it on pvc.yaml. Create it on the cluster. Show the PersistentVolumeClaims of the cluster. Show the PersistentVolumes of the cluster

<details><summary>show</summary>
<p>

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
    path: "/etc/foo"
```

```bash
k get pv
# output
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM           STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
myvolume   10Gi       RWO,RWX        Retain           Bound    default/mypvc   normal         <unset>
```

</p>
</details>

### Create a busybox pod with command 'sleep 3600', save it on pod.yaml. Mount the PersistentVolumeClaim to '/etc/foo'. Connect to the 'busybox' pod, and copy the '/etc/passwd' file to '/etc/foo/passwd'

<details><summary>show</summary>
<p>

```bash
k run busybox --image=busybox --restart=Never --dry-run=client -oyaml --command -- /bin/sh -c 'sleep 3600' > busybox.yaml
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
        - sleep 3600
      image: busybox
      name: busybox
      resources: {}
      volumeMounts:
        - name: mount-volume
          mountPath: /etc/foo
  volumes:
    - name: mount-volume
      persistentVolumeClaim:
        claimName: mypvc
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
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
    - command:
        - /bin/sh
        - -c
        - sleep 3600
      image: busybox
      name: busybox
      resources: {}
      volumeMounts:
        - name: mount-volume
          mountPath: /etc/foo
  volumes:
    - name: mount-volume
      persistentVolumeClaim:
        claimName: mypvc
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl exec busybox -it -- cat /etc/passwd
# output
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
```

</p>
</details>

### Create a busybox pod with 'sleep 3600' as arguments. Copy '/etc/passwd' from the pod to your local folder

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
```

```bash
k cp busybox:/etc/passwd .passwd
```

```bash
cat .passwd
# output
root:x:0:0:root:/root:/bin/sh
daemon:x:1:1:daemon:/usr/sbin:/bin/false
bin:x:2:2:bin:/bin:/bin/false
sys:x:3:3:sys:/dev:/bin/false
sync:x:4:100:sync:/bin:/bin/sync
mail:x:8:8:mail:/var/spool/mail:/bin/false
www-data:x:33:33:www-data:/var/www:/bin/false
operator:x:37:37:Operator:/var:/bin/false
nobody:x:65534:65534:nobody:/home:/bin/false
```

</p>
</details>
