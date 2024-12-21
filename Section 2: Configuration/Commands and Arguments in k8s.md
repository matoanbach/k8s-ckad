#### What is the command used to run the pod <code>ubuntu-sleeper</code>

```
k describe pod ubuntu-sleeper
```

<img src="https://github.com/matoanbach/k8s-ckad/blob/main/assets/sec%201/6.png"/>

#### Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file <code>ubuntu-sleeper-2.yaml</code>.

<img src="https://github.com/matoanbach/k8s-ckad/blob/main/assets/sec%201/7.png"/>

- Another way:

<img src="https://github.com/matoanbach/k8s-ckad/blob/main/assets/sec%201/8.png"/>

#### Update pod <code>ubuntu-sleeper-3</code> to sleep for 2000 seconds.

```
kubectl replace -force -f /tmp/kubectl-edit-1238102310.yaml
```

