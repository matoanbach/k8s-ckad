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

## Question 5: ServiceAccount, Secret

Solve this question on instance: ssh ckad7326

Team Neptune has its own ServiceAccount named neptune-sa-v2 in Namespace neptune. A coworker needs the token from the Secret that belongs to that ServiceAccount. Write the base64 decoded token to file /opt/course/5/token on ckad7326.

### Answer:

Step 1:

```bash
kubectl config set-context --current --namespace=neptune
```

Step 2:

```bash
kubectl get sa neptune-sa-v2 -oyaml
echo "token" | base64 -d
```

or

```bash
kubectl describe sa neptune-sa-v2
```

Step 3:

```bash
echo token > /opt/course/5/token
```

## Question 6: ReadinessProbe

Solve this question on instance: ssh ckad5601

Create a single Pod named pod6 in Namespace default of image busybox:1.31.0. The Pod should have a readiness-probe executing cat /tmp/ready. It should initially wait 5 and periodically wait 10 seconds. This will set the container ready only if the file /tmp/ready exists.

The Pod should run the command touch /tmp/ready && sleep 1d, which will create the necessary file to be ready and then idles. Create the Pod and confirm it starts.

### Answer:

```bash
kubectl run pod6 -n default --image=busybox:1.31.0 --dry-run=client -oyaml --command -- sh -c "touch /tmp/ready && sleep 1d" > 6.yaml
vim 6.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod6
  name: pod6
  namespace: default
spec:
  containers:
    - image: busybox:1.31.0
      name: pod6
      resources: {}
      command:
        - /bin/sh
        - -c
        - "touch /tmp/ready && sleep 1d"
      readinessProbe:
        exec:
          command:
            - cat
            - /tmp/ready
        initialDelaySeconds: 5
        periodSeconds: 10
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create -f 6.yaml
k get pod pod6
```

## Question 7: Pods, Namespaces

Solve this question on instance: ssh ckad7326

The board of Team Neptune decided to take over control of one e-commerce webserver from Team Saturn. The administrator who once setup this webserver is not part of the organisation any longer. All information you could get was that the e-commerce system is called my-happy-shop.

Search for the correct Pod in Namespace saturn and move it to Namespace neptune. It doesn't matter if you shut it down and spin it up again, it probably hasn't any customers anyways.

### Answer:

```bash
kubectl config set-context --current --namespace=saturn
```

```bash
kubectl describe pod # look for the pod with keyword of "my-happy-shop"
kubectl get pod webserver-sat-003 -oyaml > 7.yaml
kubectl delete --force webserver-sat-003
vim 7.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod7
  name: webserver-sat-003
  namespace: neptune # CHANGED
```

```bash
k create -f 7.yaml
```

## Question 8: Deployment, Rollouts

Solve this question on instance: ssh ckad7326

There is an existing Deployment named api-new-c32 in Namespace neptune. A developer did make an update to the Deployment but the updated version never came online. Check the Deployment history and find a revision that works, then rollback to it. Could you tell Team Neptune what the error was so it doesn't happen again?

### Answer:

```bash
ksn neptune
```

```bash
kubectl get deploy
kubectl rollout history
```

```bash
k get deploy,pod | grep api-new-c32
k describe pod api-new-c32-7d64747c87 | grep -i error
k describe pod api-new-c32-7d64747c87 | grep -i image
```

Let's revert to the previous version:

```bash
k rollout undo deploy api-new-c32
```

or

```bash
k rollout undo deploy api-new-c32 --to-revision=4
```

Verify it works:

```bash
k get deploy pod
```

## Question 9: Pod -> Deployment

Solve this question on instance: ssh ckad9043

In Namespace pluto there is single Pod named holy-api. It has been working okay for a while now but Team Pluto needs it to be more reliable.

Convert the Pod into a Deployment named holy-api with 3 replicas and delete the single Pod once done. The raw Pod template file is available at /opt/course/9/holy-api-pod.yaml.

In addition, the new Deployment should set allowPrivilegeEscalation: false and privileged: false for the security context on container level.

Please create the Deployment and save its yaml under /opt/course/9/holy-api-deployment.yaml on ckad9043.

### Answer:

```bash
ksn pluto
```

Step 1: make a back up yaml

```bash
cat /opt/course/9/holy-api-pod.yaml > /opt/course/9/holy-api-deployment.yaml
vim /opt/course/9/holy-api-deployment.yaml
```

Step 2: convert pod to deployment

```yaml
# /opt/course/9/holy-api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: holy-api # name stays the same
  namespace: pluto # important
spec:
  replicas: 3 # 3 replicas
  selector:
    matchLabels:
      id: holy-api # set the correct selector
  template:
    # => from here down its the same as the pods metadata: and spec: sections
    metadata:
      labels:
        id: holy-api
      name: holy-api
    spec:
      containers:
        - env:
            - name: CACHE_KEY_1
              value: b&MTCi0=[T66RXm!jO@
            - name: CACHE_KEY_2
              value: PCAILGej5Ld@Q%{Q1=#
            - name: CACHE_KEY_3
              value: 2qz-]2OJlWDSTn_;RFQ
          image: nginx:1.17.3-alpine
          name: holy-api-container
          securityContext: # add
            allowPrivilegeEscalation: false # add
            privileged: false # add
          volumeMounts:
            - mountPath: /cache1
              name: cache-volume1
            - mountPath: /cache2
              name: cache-volume2
            - mountPath: /cache3
              name: cache-volume3
      volumes:
        - emptyDir: {}
          name: cache-volume1
        - emptyDir: {}
          name: cache-volume2
        - emptyDir: {}
          name: cache-volume3
```

To indent multiple lines using `vim` you should set the shiftwidth using `:set shiftwidth=2`. Then mark multiple lines using `Shift V` and the up/down keys

To then indent the marked lines press `>` or `<` and to repeat the action press `.`

Step 3:

```bash
k create -f /opt/course/9/holy-api-deployment.yaml
```

Step 4: delete the old pod running

```bash
k -n pluto delete pod holy-api --force --grace-period=0
```

## Question 10: Service, Logs

Solve this question on instance: ssh ckad9043

Team Pluto needs a new cluster internal Service. Create a ClusterIP Service named project-plt-6cc-svc in Namespace pluto. This Service should expose a single Pod named project-plt-6cc-api of image nginx:1.17.3-alpine, create that Pod as well. The Pod should be identified by label project: plt-6cc-api. The Service should use tcp port redirection of 3333:80.

Finally use for example curl from a temporary nginx:alpine Pod to get the response from the Service. Write the response into /opt/course/10/service_test.html on ckad9043. Also check if the logs of Pod project-plt-6cc-api show the request and write those into /opt/course/10/service_test.log on ckad9043.

### Answer:

Step 1: switch context

```bash
ksn pluto
```

Step 2: create the pod to expose

```bash
k run -n pluto project-plt-6cc-api --image=nginx:1.17.3-alpine -l project=plt-6cc-api
```

Step 3: create a service for the pod

```bash
k expose pod project-plt-6cc-api --name=project-plt-6cc-svc -n pluto --port=3333 --target-port=80
```

Step 4: Check the connectivity

```bash
k get ep
k run tmp --restart=Never --image=nginx:alpine --rm -it -- curl -m 5 project-plt-6cc-svc.pluto:3333
```

## Question 11: Working with Containers

Solve this question on instance: ssh ckad9043

There are files to build a container image located at /opt/course/11/image on ckad9043. The container will run a Golang application which outputs information to stdout. You're asked to perform the following tasks:

1. Change the Dockerfile: set ENV variable SUN_CIPHER_ID to hardcoded value 5b9c1065-e39d-4a43-a04a-e59bcea3e03f

2. Build the image using sudo docker, tag it registry.killer.sh:5000/sun-cipher:v1-docker and push it to the registry

3. Build the image using sudo podman, tag it registry.killer.sh:5000/sun-cipher:v1-podman and push it to the registry

4. Run a container using sudo podman, which keeps running detached in the background, named sun-cipher using image registry.killer.sh:5000/sun-cipher:v1-podman

5. Write the logs your container sun-cipher produces into /opt/course/11/logs on ckad9043


### Answer:

2. 

```bash
sudo docker build -t registry.killer.sh:5000/sun-cipher:v1-docker /opt/course/11/image
sudo docker image ls
```

```bash
sudo docker push registry.killer.sh:5000/sun-cipher:v1-docker 
```

3. 

```bash
sudo podman build -t registry.killer.sh:5000/sun-cipher:v1-podman 
sudo podman image ls
```

```bash
sudo podman push registry.killer.sh:5000/sun-cipher:v1-podman
```

4. 

```bash
sudo podman run -d --name=sun-cipher registry.killer.sh:5000/sun-cipher:v1-podman 
```

5. 

```bash
sudo podman logs sun-cipher > /opt/course/11/logs 
```