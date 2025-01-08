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
k run nginx1 --image=nginx --restart=Never --labels=app=v1
k run nginx2 --image=nginx --restart=Never --labels=app=v1
k run nginx3 --image=nginx --restart=Never --labels=app=v1
```

or

```bash
for i in `seq 1 3`; do k run nginx$i --image=nginx --restart=Never --labels=app=v1; done
```

</p>
</details>

### Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
k get pod --show-labels
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
k get pod -label-columes=app
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
k label -l "app in (v1,v2)" tier=web
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
k label pod nginx1 nginx2 nginx3 app-
k label pod nginx{1..3} app-
k label pod -l app app-
```

</p>
</details>

### Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>

```bash
k annotate pods nginx1 nginx2 nginx3 description="my description"
k annotate pods nginx{1..3} description="my description"
```

</p>
</details>

### Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
k annotate pod nginx1 --list
k describe pod nginx1 | grep annotations:
k get pod nginx1 -oyaml | grep annotations:
k get po nginx -o custom-column=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

</p>
</details>

### Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
k annotate pod nginx{1..3} description- owner-
```

</p>
</details>

### Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
k delete --force pod nginx1 nginx2 nginx3
```

</p>
</details>

## Pod Placement

### Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

```bash
k label nodes mynode accelerator=nvidia-tesla-p100
k run newpod --image=nginx --restart=Never --dry-run=client -oyaml > newpod.yaml
vim newpod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: newpod
  name: newpod
spec:
  nodeSelector:
    accelerator: nvidia-tesla-p100
  containers:
    - image: nginx
      name: newpod
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
k create -f newpod.yaml
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
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > newpod.yaml
vim newpod.yaml
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
    key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
  containers:
    - image: nginx
      name: nginx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

```bash
k create -f newpod.yaml
k describe pod nginx | grep -i node

Node:             controlplane/192.168.251.229
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
```

</p>
</details>

### Create a pod that will be placed on node `controlplane`. Use nodeSelector and tolerations.

<details><summary>show</summary>
<p>

```bash
k run nginx --image=nginx --restart=Never --dry-run=client -oyaml > newpod.yaml
vim newpod.yaml
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
  nodeSelector:
    beta.kubernetes.io/arch: amd64 # any label works as long as it's part of controlplane's labels
  tolerations:
    key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
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
k get deployment nginx -oyaml
```

</p>
</details>

### View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
k get replicaset <replicaset name> -oyaml
```

</p>
</details>

### Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
k get pod <pod name> -oyaml
```

</p>
</details>

### Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
k rollout status deployment nginx
k rollout history deployment nginx
```

</p>
</details>

### Update the nginx image to nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
k set image deployment nginx nginx=nginx:1.19.8
k edit deployment nginx
```

</p>
</details>

### Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
k rollout history deployment nginx
k get deploy nginx
k get rs
k get po
```

</p>
</details>

### Undo the latest rollout and verify that new pods have the old image (nginx:1.18.0)

<details><summary>show</summary>
<p>

```bash
k rollout undo deployment nginx
k rollout undo deployment nginx --to-revision=<version of rollout>
```

</p>
</details>

### Do an on-purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
k set image deployment nginx nginx=nginx:11000000 # wrong image
```

</p>
</details>

### Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
k rollout status deployment nginx
k get deploy nginx
k get rs
k get po
```

</p>
</details>

### Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

<details><summary>show</summary>
<p>

```bash
k rollout undo deployment nginx --to-revision=2
k rollout status deployment nginx
```

</p>
</details>

### Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
k rollout history deployment nginx --revision=4
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
k get hpa deployment nginx
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
k delete deployment nginx --force
k delete hpa nginx
#or
k delete deployment/nginx hpa/nginx
```

</p>
</details>

### Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio

<details><summary>show</summary>
<p>

```bash
k create deployment old --image=nginx --replicas=3 --dry-run=client -oyaml > old.yaml
vim old.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    version: v2
    app: backend
  name: old
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        version: v1
        app: backend
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

```bash
k create deployment new --image=nginx  --replicas=1  --dry-run=client -oyaml > new.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: backend
  name: new
spec:
  replicas: 1
  selector:
    matchLabels:
      version: v2
      app: backend
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        version: v2
        app: backend
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

```bash
k expose deployment old --name=backend-service --port=80 --selector=app=backend --dry-run=client -oyaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: backend
  name: backend-service
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: backend
status:
  loadBalancer: {}s
```

</p>
</details>

## Jobs

### Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
k create job pi --image=perl:5.34 --restart=Never --command -- sh -c "perl -Mbignum=bpi -wle 'print bpi(2000)'"
```

</p>
</details>

### Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
k get job pi -owide
k describe job pi -owide
k logs pi
```

</p>
</details>

### Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
k create job busybox --image=busybox -- sh -c 'echo hello;sleep 30;echo world'
```

```bash
k logs busybox-nvjdj
> hello
```

</p>
</details>

### Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
k logs busybox-nvjdj -f
```

</p>
</details>

### See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
k get job
k describe job busybox
k logs jobs/busybox
```

</p>
</details>

### Delete the job

<details><summary>show</summary>
<p>

```bash
k delete job busybox pi
```

</p>
</details>

### Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>

```bash
k create job newjob --image=nginx --dry-run=client -oyaml > newjob.yaml
vim newjob.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: newjob
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: newjob
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
  name: newjob
spec:
  completions: 5
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: newjob
          resources: {}
      restartPolicy: Never
status: {}
```

```bash
k get job newjob -owde
k describe job newjob
k get pod
k delete job newjob
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
  name: newjob
spec:
  completions: 5
  parallelism: 5
  activeDeadlineSeconds: 30
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
        - image: nginx
          name: newjob
          resources: {}
      restartPolicy: Never
status: {}
```

```bash
k get job newjob -owde
k describe job newjob
k get pod
k delete job newjob
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### Create a cron job with image busybox that runs on a schedule of "_/1 _ \* \* \*" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
k create cronjob nginx --image=nginx  --schedule="*/1 * * * *" --dry-run=client -oyaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: nginx
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: nginx
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
            - image: nginx
              name: nginx
              resources: {}
              command:
                - /bin/sh
                - -c
                - "date; echo Hello from the Kubernetes cluster"
          restartPolicy: OnFailure
  schedule: "* * * * *"
status: {}
```

</p>
</details>

### See its logs and delete it

<details><summary>show</summary>
<p>

```bash
k get po
k logs nginx-28939212-z7285
> Wed Jan  8 16:09:04 UTC 2025
> Hello from the Kubernetes cluster
```

```bash
k delete cj busy
```

</p>
</details>

### Create the same cron job again, and watch the status. Once it ran, check which job ran by the created cron job. Check the log, and delete the cron job

<details><summary>show</summary>
<p>

```bash
k get cj
k get job -w
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time).

<details><summary>show</summary>
<p>

```bash
k create cronjob busybox --image=busybox --schedule="*/1 * * * *" --dry-run=client -oyaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > anothercronjob.yaml
```

</p>
</details>

### Create a cron job with image busybox that runs every minute and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it successfully starts but takes more than 12 seconds to complete execution.

<details><summary>show</summary>
<p>

```bash
k create cronjob busybox --image=busybox --schedule="* * * * *" --dry-run=client -oyaml -- /bin/sh -c 'date; echo Hello
from the Kubernetes cluster' > anothercronjob.yaml
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: busybox
spec:
  startingDeadlineSeconds: 17 # add this line
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: busybox
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
              name: busybox
              resources: {}
          restartPolicy: OnFailure
  schedule: "* * * * *"
status: {}
```

</p>
</details>

### Create a job from cronjob.

<details><summary>show</summary>
<p>

```bash
k create job anotherjob --from=cronjob/busyboxs
```

</p>
</details>
