![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)

# Pod design (20%)

[Labels And Annotations](#labels-and-annotations)

[Deployments](#deployments)

[Jobs](#jobs)

[Cron Jobs](#cron-jobs)

## Labels and Annotations

kubernetes.io > Documentation > Concepts > Overview > Working with Kubernetes Objects > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

### Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

<details><summary>show</summary>
<p>

```bash
k create pod nginx1 --image=nginx --restart=Never -l app=v1
k create pod nginx2 --image=nginx --restart=Never -l app=v1
k create pod nginx3 --image=nginx --restart=Never -l app=v1
```

or

```bash
for i in `seq 1 3`; do k run nginx$i --image=nginx --restart=Never -l app=v1; done
```

</p>
</details>

### Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
k get pods --show-labels
```

</p>
</details>

### Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
k label pod nginx2 app=v2 --overwrite
```

</p>
</details>

### Get the label 'app' for the pods (show a column with APP labels)

<details><summary>show</summary>
<p>

```bash
k get pod -L app
k get pod --label-columns=app
```

</p>
</details>

### Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
k get pod -l app=v2
k get pod --selector=app=v2
```

</p>
</details>

### Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels

<details><summary>show</summary>
<p>

```bash
k label pod -l "app in (v1,v2)" tier=web --overwrite
```

</p>
</details>

### Add an annotation 'owner: marketing' to all pods having 'app=v2' label

<details><summary>show</summary>
<p>

```bash
k annotate pods -l app=v2 owner=marketing
```

</p>
</details>

### Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
k label pods -l app app-
```

</p>
</details>

### Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>

```bash
k annotate pods nginx{1..3} description='my description'
```

</p>
</details>

### Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
k annotate pod nginx1 --list
k describe pod nginx1 | grep -i annotation
k get po nginx1 -o custom-columns=NAME:metadata.name.ANNOTATIONS:metadata.annotations.description
```

</p>
</details>

### Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
k annotate pods nginx{1..3} description- owner-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
k delete pods --all
k delete pods nginx{1..3}
```

</p>
</details>

## Pod Placement

### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

```bash
k label node mynode accelerator=nvidia-tesla-p100
```

```bash
k run pod --image=nginx --restart=Never --dry-run=client -oyaml > mypod.yaml
vim my.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  nodeSelector:
    accelerator: nvidia-tesla-p100
  containers:
    - image: nginx
      name: pod
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

### Taint a node with key `tier` and value `frontend` with the effect `NoSchedule`. Then, create a pod that tolerates this taint.

<details><summary>show</summary>
<p>

```bash
k taint node mynode tier=frontend:NoSchedule
```

```bash
k run nginx --iamge=nginx --restart=Never --dry-client=client -oyaml > nginx.yaml
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
  tolerations:
  - key: "tier"
    value: "frontend"
    effect: "NoSchedule"
    operator: "Equal"
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}s
```

</p>
</details>

### Create a pod that will be placed on node `controlplane`. Use nodeSelector and tolerations.

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
  tolerations:
    - key: "tier"
      value: "frontend"
      effect: "NoSchedule"
      operator: "Equal"
  nodeSelector:
    kubernetes.io/hostname: controlplane
  containers:
    - image: nginx
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Workload Resources > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
k create deployment nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

</p>
</details>

### View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
k get deployment nginx -oyaml > nginx.deploy.yaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
k get rs
#output
NAME               DESIRED   CURRENT   READY   AGE
nginx-7b4cd9f47b   2         2         2       30s
```

```bash
k get rs nginx-7b4cd9f47b -oyaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
k get po
#output
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7b4cd9f47b-4rdbc   1/1     Running   0          75s
nginx-7b4cd9f47b-h8mfg   1/1     Running   0          75s
```

```bash
k get po nginx-7b4cd9f47b-4rdbc -oyaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
k rollout status deployment nginx
# output
deployment "nginx" successfully rolled out
```

</p>
</details>

### Update the nginx image to nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
k set image deployment nginx nginx=nginx:1.19.9
```

```bash
k describe deployment nginx | grep -i image
# output
    Image:         nginx:1.19.9
```

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
k rollout history deployment nginx
#output
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

<details><summary>show</summary>
<p>

```bash
k rollout undo deployment nginx
```

```bash
k describe deployment nginx | grep -i image
# output
    Image:         nginx:1.18.0
```

</p>
</details>

### Do an on-purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
k set image deployment/nginx nginx=nginx:1.91
```

```bash
k rollout status deployment/nginx
#output
Waiting for deployment "nginx" rollout to finish: 1 out of 2 new replicas have been updated...
```

```bash
k get po
NAME                     READY   STATUS             RESTARTS   AGE
nginx-7b4cd9f47b-t6nrk   1/1     Running            0          67s
nginx-7b4cd9f47b-zpjh2   1/1     Running            0          69s
nginx-b9d99dcff-sq5r9    0/1     ImagePullBackOff   0          18s
```

```bash
k describe po nginx-b9d99dcff-sq5r9 | grep -i error:
  Warning  Failed     42s (x3 over 85s)  kubelet            Failed to pull image "nginx:1.91": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:1.91": failed to resolve reference "docker.io/library/nginx:1.91": docker.io/library/nginx:1.91: not found
  Warning  Failed     42s (x3 over 85s)  kubelet            Error: ErrImagePull
  Warning  Failed     4s (x5 over 84s)   kubelet            Error: ImagePullBackOffs
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
k rollout status deployment/nginx
#output
Waiting for deployment "nginx" rollout to finish: 1 out of 2 new replicas have been updated...
```

</p>
</details>

### Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
k rollout undo deployment nginx --to-revision=2
```

```bash
k rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx" rollout to finish: 1 old replicas are pending termination...
deployment "nginx" successfully rolled out
```

```bash
k get pod
# output
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6fd547885b-56qx4   1/1     Running   0          26s
nginx-6fd547885b-qq2kx   1/1     Running   0          25s
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
k rollout history deployment nginx --revision=4
#output
deployment.apps/nginx with revision #4
Pod Template:
  Labels:       app=nginx
        pod-template-hash=b9d99dcff
  Containers:
   nginx:
    Image:      nginx:1.91
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
  Node-Selectors:       <none>
  Tolerations:  <none>
```

</p>
</details>

### Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
k scale deployment nginx --replicas=5
```

</p>
</details>

### Autoscale the deployment, pods between 5 and 10, targeting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
k autoscale deployment nginx --min=5 --max=10 --cpu-percent=80
```

</p>
</details>

### Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
k rollout pause deployment nginx
```

</p>
</details>

### Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
k set image deployment nginx nginx=nginx:1.19.9
```

</p>
</details>

### Resume the rollout and check that the nginx:1.19.9 image has been applied

<details><summary>show</summary>
<p>

```bash
k rollout resume deployment nginx
```

</p>
</details>

### Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
k delete deploy/nginx hpa/nginx --force
```

</p>
</details>

### Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

<details><summary>show</summary>
<p>

```bash
k create deploy myapp --image=nginx --dry-run=client -oyaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: v1
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: v2
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: myapp
        version: v2
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

```bash
k expose deployment myapp --port=80 --name=myapp-service --dry-run=client -oyaml > myapp-service.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: myapp
  name: myapp-service
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: myapp
    # version: v1 # remove this
status:
  loadBalancer: {}
```

```bash
k get ep
# output
NAME            ENDPOINTS                                               AGE
kubernetes      192.168.81.39:6443                                      31m
myapp-service   10.42.0.21:80,10.42.0.22:80,10.42.0.23:80 + 1 more...   108s
```

</p>
</details>

## Jobs

### Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
k create job pi --image=perl:5.34 --restart=Never -it -- /bin/sh -c "perl -Mbignum=bpi -wle 'print bpi(2000)'"
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
k logs job/pi
# output
3.1415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679821480865132823066470938446095505822317253594081284811174502841027019385211055596446229489549303819644288109756659334461284756482337867831652712019091456485669234603486104543266482133936072602491412737245870066063155881748815209209628292540917153643678925903600113305305488204665213841469519415116094330572703657595919530921861173819326117931051185480744623799627495673518857527248912279381830119491298336733624406566430860213949463952247371907021798609437027705392171762931767523846748184676694051320005681271452635608277857713427577896091736371787214684409012249534301465495853710507922796892589235420199561121290219608640344181598136297747713099605187072113499999983729780499510597317328160963185950244594553469083026425223082533446850352619311881710100031378387528865875332083814206171776691473035982534904287554687311595628638823537875937519577818577805321712268066130019278766111959092164201989380952572010654858632788659361533818279682303019520353018529689957736225994138912497217752834791315155748572424541506959508295331168617278558890750983817546374649393192550604009277016711390098488240128583616035637076601047101819429555961989467678374494482553797747268471040475346462080466842590694912933136770289891521047521620569660240580381501935112533824300355876402474964732639141992726042699227967823547816360093417216412199245863150302861829745557067498385054945885869269956909272107975093029553211653449872027559602364806654991198818347977535663698074265425278625518184175746728909777727938000816470600161452491921732172147723501414419735685481613611573525521334757418494684385233239073941433345477624168625189835694855620992192221842725502542568876717904946016534668049886272327917860857843838279679766814541009538837863609506800642251252051173929848960841284886269456042419652850222106611863067442786220391949450471237137869609563643719172874677646575739624138908658326459958133904780275901
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
k create job myjob --image=busy --dry-run=client -oyaml -- /bin/sh -c  'echo hello;sleep 30;echo world' > myjob.yaml
vim myjob.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: myjob
spec:
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - command:
            - /bin/sh
            - -c
            - echo hello;sleep 30;echo world
          image: busy
          name: myjob
          resources: {}
      restartPolicy: Never
status: {}
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
k logs myjob-4cxkt -f
# output
hello
world
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
k get job/myjob
# output
NAME    STATUS     COMPLETIONS   DURATION   AGE
myjob   Complete   1/1           33s        77s
```

```bash
k describe job/myjob
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
k delete job myjob
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>

```bash
k create job myjob --image=nginx --dry-run=client -oyaml > myjob.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: myjob
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: myjob
          resources: {}
      restartPolicy: Never
status: {}
```

</p>
</details>

### Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: myjob
spec:
  completions: 5
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: myjob
          resources: {}
      restartPolicy: Never
status: {}
```

</p>
</details>

### Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: myjob
spec:
  completions: 5
  parallelism: 5
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: myjob
          resources: {}
      restartPolicy: Never
status: {}
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "_/1 _ \* \* \*" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
k create cj mycj --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
k logs pod/mycj-28940654-87db9
# output
Thu Jan  9 16:14:01 UTC 2025
Hello from the Kubernetes clusters
```

```bash
k delete cj mycj
```

</p>
</details>

### Create the same cron job again, and watch the status. Once it ran, check which job ran by the created cron job. Check the log, and delete the cron job

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

<details><summary>show</summary>
<p>

```bash
k create cj mycj --image=busybox --schedule="* * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

<details><summary>show</summary>
<p>

```bash
k create cj mycj --image=busybox --schedule="* * * * *" --dry-run=client -oyaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > mycj.yaml
vim mycj.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: mycj
spec:
  startingDeadlineSeconds: 12
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: mycj
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
            - command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
              image: busybox
              name: mycj
              resources: {}
          restartPolicy: OnFailure
  schedule: "* * * * *"
status: {}
```

```bash
k create -f mycj.yaml
```

</p>
</details>

### Create a job from cronjob.

<details><summary>show</summary>
<p>

```bash
k create job myjob  --from=cronjob/mycj
```

```bash
k get job
# output
NAME            STATUS     COMPLETIONS   DURATION   AGE
mycj-28940662   Complete   1/1           4s         32s
myjob           Running    0/1           3s         3s
```

```bash
k get po
# output 
NAME                  READY   STATUS      RESTARTS   AGE
mycj-28940662-rlv45   0/1     Completed   0          37s
myjob-4wcbs           0/1     Completed   0          8s
```

```bash
k logs myjob-4wcbs
# output 
Thu Jan  9 16:22:30 UTC 2025
Hello from the Kubernetes cluster
```

</p>
</details>
