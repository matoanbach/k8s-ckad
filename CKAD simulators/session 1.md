## Question 1: Namespace

Solve this question on instance: ssh ckad5601

The DevOps team would like to get the list of all Namespaces in the cluster. Get the list and save it to `/opt/course/1/namespaces` on ckad5601.

```bash
kubectl get ns > /opt/course/1/namespaces
```

## Question 2: Pods

Solve this question on instance: ssh ckad5601

Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container.

Your manager would like to run a command manually on occasion to output the status of that exact Pod. Please write a command that does this into /opt/course/2/pod1-status-command.sh on ckad5601. The command should use kubectl.

```bash
kubectl run pod1 --image=httpd:2.4.41-alpine -n default --dry-run=client -oyaml > 2.yaml
vim 2.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
  namespace: default
spec:
  containers:
    - image: httpd:2.4.41-alpine
      name: pod1-container #ADD
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create -f 2.yaml

echo "kubectl -n default describe pod pod1 | grep -i status:" > /opt/course/2/pod1-status-command.sh
```

## Question 3: Job

Solve this question on instance: ssh ckad7326

Team Neptune needs a Job template located at /opt/course/3/job.yaml. This Job should run image busybox:1.31.0 and execute sleep 2 && echo done. It should be in namespace neptune, run a total of 3 times and should execute 2 runs in parallel.

Start the Job and check its history. Each pod created by the Job should have the label id: awesome-job. The job should be named neb-new-job and the container neb-new-job-container.

```bash
kubectl create job neb-new-job -n neptune --image=busybox:1.31.0 --dry-run=client -oyaml > /opt/course/3/job.yaml
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: neb-new-job
  namespace: neptune
spec:
  completions: 3
  parallelism: 2
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: awesome-job
    spec:
      containers:
        - image: busybox:1.31.0
          name: neb-new-job-container
          resources: {}
          command:
            - /bin/sh
            - -c
            - sleep 2 && echo done
      restartPolicy: Never
status: {}
```

```bash
k create -f /opt/course/3/job.yaml
k get job -n neptune
```

## Question 4: Helm Management

Solve this question on instance: ssh ckad7326

Team Mercury asked you to perform some operations using Helm, all in Namespace mercury:

Delete release internal-issue-report-apiv1

Upgrade release internal-issue-report-apiv2 to any newer version of chart bitnami/nginx available

Install a new release internal-issue-report-apache of chart bitnami/apache. The Deployment should have two replicas, set these via Helm-values during install

There seems to be a broken release, stuck in pending-install state. Find it and delete it

### Answer

```bash
k config set-context --current --namespace=neptune
```

Delete release internal-issue-report-apiv1

```bash
helm list
helm uninstall internal-issue-report-apiv1
```

Upgrade release internal-issue-report-apiv2 to any newer version of chart bitnami/nginx available

```bash
helm repo list
helm repo update
helm search repo nginx --version
helm upgrade internal-issue-report-apiv2 bitnami/nginx
helm list
```

Install a new release internal-issue-report-apache of chart bitnami/apache. The Deployment should have two replicas, set these via Helm-values during install

```bash
helm show values bitnami/apache

helm show values bitnami/apache | yq e # parse yaml and show with colors

helm install internal-issue-report-apache bitnami/apache --set replicaCount=2
```

Install done, let's verify

```bash
helm ls
k get deploy internal-issue-report-apache
```

There seems to be a broken release, stuck in pending-install state. Find it and delete it

```bash
helm ls -a

helm uninstall internal-issue-report-daniel
```