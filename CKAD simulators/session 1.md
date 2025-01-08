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

## Question 12: Storage, PV, PVC, Pod volume

Solve this question on instance: ssh ckad5601

Create a new PersistentVolume named earth-project-earthflower-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace earth named earth-project-earthflower-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment project-earthflower in Namespace earth which mounts that volume at /tmp/project-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

### Answer:

Step 1: switch context

```bash
ksn earth
```

Step 2: set up Persistent Volume

```bash
vim pv.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: earth-project-earthflower-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Volumes/Data
```

```bash
k create -f pv.yaml
```

Step 3: set up persistent volume claim

```bash
vim pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: earth-project-earthflower-pvc
  namespace: earth
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2i
```

```bash
k create -f pv.yaml
```

Step 4: create a deployment

```bash
k create deploy project-earthflower --image=httpd:2.4.41-alpine -n earth --dry-run=client -oyaml > deploy.yaml
```

```yaml
\apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: project-earthflower
  name: project-earthflower
  namespace: earth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: project-earthflower
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: project-earthflower
    spec:
      containers:
        - image: httpd:2.4.41-alpine
          name: httpd
          resources: {}
          volumeMounts:
          - name: data-volume
            mountPath: /tmp/project-data
        volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: earth-project-earthflower-pvc
```

```bash
k create -f deploy.yaml
```

## Question 13: Storage, StorageClass, PVC

Solve this question on instance: ssh ckad9043

Team Moonpie, which has the Namespace moon, needs more storage. Create a new PersistentVolumeClaim named moon-pvc-126 in that namespace. This claim should use a new StorageClass moon-retain with the provisioner set to moon-retainer and the reclaimPolicy set to Retain. The claim should request storage of 3Gi, an accessMode of ReadWriteOnce and should use the new StorageClass.

The provisioner moon-retainer will be created by another team, so it's expected that the PVC will not boot yet. Confirm this by writing the event message from the PVC into file /opt/course/13/pvc-126-reason on ckad9043.

### Answer:

Step 1: switch context

```bash
ksn moon
```

Step 2: create storage class

```bash
vim storage.yaml
```

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: moon-retain
provisioner: moon-retainer
reclaimPolicy: Retain # default value is Delete
```

Step 3: create persistent volume claim

```bash
vim pvc.yaml
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: moon-pvc-126
  namespace: moon
spec:
  storageClassName: moon-retain
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
k create -f pvc.yaml
```

## Question 14: Secret, Secret-Volume, Secret-Env

Solve this question on instance: ssh ckad9043

You need to make changes on an existing Pod in Namespace moon called secret-handler. Create a new Secret secret1 which contains user=test and pass=pwd. The Secret's content should be available in Pod secret-handler as environment variables SECRET1_USER and SECRET1_PASS. The yaml for Pod secret-handler is available at /opt/course/14/secret-handler.yaml.

There is existing yaml for another Secret at /opt/course/14/secret2.yaml, create this Secret and mount it inside the same Pod at /tmp/secret2. Your changes should be saved under /opt/course/14/secret-handler-new.yaml on ckad9043. Both Secrets should only be available in Namespace moon.

### Answer:

Step 1: switch context

```bash
ksn moon
```

Step 2: create secret1

```bash
k create secret generic secret1 --from-literal=pass=pwd --from-literal=user=test -n moon
```

Step 3: create secret2

```bash
k create -f /opt/course/14/secret2.yaml
```

Step 4: consume those 2 secrets

```bash
cat /opt/course/14/secret-handler.yaml > /opt/course/14/secret-handler-new.yaml
vim /opt/course/14/secret-handler-new.yaml
```

```yaml
# /opt/course/14/secret-handler-new.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    id: secret-handler
    uuid: 1428721e-8d1c-4c09-b5d6-afd79200c56a
    red_ident: 9cf7a7c0-fdb2-4c35-9c13-c2a0bb52b4a9
    type: automatic
  name: secret-handler
  namespace: moon
spec:
  volumes:
  - name: cache-volume1
    emptyDir: {}
  - name: cache-volume2
    emptyDir: {}
  - name: cache-volume3
    emptyDir: {}
  - name: secret-volume
    secret:
      secretName: secret2
  containers:
  - name: secret-handler
    image: bash:5.0.11
    args: ['bash', '-c', 'sleep 2d']
    volumeMounts:
    - mountPath: /cache1
      name: cache-volume1
    - mountPath: /cache2
      name: cache-volume2
    - mountPath: /cache3
      name: cache-volume3
    env:
    - name: SECRET_KEY_1
      value: ">8$kH#kj..i8}HImQd{"
    - name: SECRET_KEY_2
      value: "IO=a4L/XkRdvN8jM=Y+"
    - name: SECRET_KEY_3
      value: "-7PA0_Z]>{pwa43r)__"
    - name: SECRET1_USER
      valueFrom:
        secretKeyRef:
          name: user

    - name: SECRET1_PASS
      valueFrom:
        secretKeyRef:
          name: pass
    volumeMounts:
    - name: secret-volume
      mountPath: /tmp/secret2
```

## Question 15: ConfigMap, Configmap-Volume

Solve this question on instance: ssh ckad9043

Team Moonpie has a nginx server Deployment called web-moon in Namespace moon. Someone started configuring it but it was never completed. To complete please create a ConfigMap called configmap-web-moon-html containing the content of file /opt/course/15/web-moon.html under the data key-name index.html.

The Deployment web-moon is already configured to work with this ConfigMap and serve its content. Test the nginx configuration for example using curl from a temporary nginx:alpine Pod.

### Answer:

Step 1: switch context

```bash
ksn moon
```

Step 2: create a configmap

```bash
k create cm configmap-web-moon-html -n moon --from-file=index.html=/opt/course/15/web-moon.html
```

Step 3: check the results

```bash
k -n moon get pod -o wide # get pod cluster IPs
```

```bash
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl 10.44.0.78
```

```bash
k -n moon describe pod web-moon-c77655cc-dc8v4 | grep -A2 Mounts:
```

```bash
k -n moon exec web-moon-c77655cc-dc8v4 find /usr/share/nginx/html
```

## Question 16: Logging sidecar

Solve this question on instance: ssh ckad7326

The Tech Lead of Mercury2D decided it's time for more logging, to finally fight all these missing data incidents. There is an existing container named cleaner-con in Deployment cleaner in Namespace mercury. This container mounts a volume and writes logs into a file called cleaner.log.

The yaml for the existing Deployment is available at /opt/course/16/cleaner.yaml. Persist your changes at /opt/course/16/cleaner-new.yaml on ckad7326 but also make sure the Deployment is running.

Create a sidecar container named logger-con, image busybox:1.31.0 , which mounts the same volume and writes the content of cleaner.log to stdout, you can use the tail -f command for this. This way it can be picked up by kubectl logs.

Check if the logs of the new container reveal something about the missing data incidents.

### Answer:

Step 1: switch context1

```bash
ksn mercury
```

Step 2: add sidecar to the deployment

```bash
cp /opt/course/16/cleaner.yaml /opt/course/16/cleaner-new.yaml
vim /opt/course/16/cleaner-new.yaml
```

```yaml
# /opt/course/16/cleaner-new.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  name: cleaner
  namespace: mercury
spec:
  replicas: 2
  selector:
    matchLabels:
      id: cleaner
  template:
    metadata:
      labels:
        id: cleaner
    spec:
      volumes:
        - name: logs
          emptyDir: {}
      initContainers:
        - name: init
          image: bash:5.0.11
          command: ["bash", "-c", "echo init > /var/log/cleaner/cleaner.log"]
          volumeMounts:
            - name: logs
              mountPath: /var/log/cleaner
        - name: logger-con
          image: busybox:1.31.0
          restartPolicy: Always
          command:
            - /bin/sh
            - -c
            - "tail -f var/log/cleaner/cleaner.log"
          volumeMounts:
            - name: logs
              mountPath: /var/log/cleaner
      containers:
        - name: cleaner-con
          image: bash:5.0.11
          args:
            [
              "bash",
              "-c",
              'while true; do echo `date`: "remove random file" >> /var/log/cleaner/cleaner.log; sleep 1; done',
            ]
          volumeMounts:
            - name: logs
              mountPath: /var/log/cleaner
```

```bash
k create -f /opt/course/16/cleaner-new.yaml
```

```bash
k -n mercury logs cleaner-576967576c-cqtgx -c logger-con
```

## Question 17: InitContainer

Solve this question on instance: ssh ckad5601

Last lunch you told your coworker from department Mars Inc how amazing InitContainers are. Now he would like to see one in action. There is a Deployment yaml at /opt/course/17/test-init-container.yaml. This Deployment spins up a single Pod of image nginx:1.17.3-alpine and serves files from a mounted volume, which is empty right now.

Create an InitContainer named init-con which also mounts that volume and creates a file index.html with content check this out! in the root of the mounted volume. For this test we ignore that it doesn't contain valid html.

The InitContainer should be using image busybox:1.31.0. Test your implementation for example using curl from a temporary nginx:alpine Pod.

### Answer:

Step 1: switch context

```bash
ksn mars
```

Step 2: Add the initContainer

```bash
cat /opt/course/17/test-init-container.yaml > 17.yaml
vim 17.yaml
```

```yaml
# 17_test-init-container.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-init-container
  namespace: mars
spec:
  replicas: 1
  selector:
    matchLabels:
      id: test-init-container
  template:
    metadata:
      labels:
        id: test-init-container
    spec:
      volumes:
        - name: web-content
          emptyDir: {}
      initContainers:
        - name: init-con
          image: busybox:1.31.0
          volumeMounts:
            - name: web-content
              mountPath: /usr/share/nginx/html
          command:
            - sh
            - -c
            - echo "check this out!" > /usr/share/nginx/html/index.html
      containers:
        - image: nginx:1.17.3-alpine
          name: nginx
          volumeMounts:
            - name: web-content
              mountPath: /usr/share/nginx/html
          ports:
            - containerPort: 80
```

```bash
k create -f 17.yaml
```

Step 3: Check the result

```bash
k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl 10.0.0.67
```

## Question 18: Service misconfiguration

Solve this question on instance: ssh ckad5601

There seems to be an issue in Namespace mars where the ClusterIP service manager-api-svc should make the Pods of Deployment manager-api-deployment available inside the cluster.

You can test this with curl manager-api-svc.mars:4444 from a temporary nginx:alpine Pod. Check for the misconfiguration and apply a fix.

### Answer:

Step 1: switch context

```bash
ksn mars
```

Step 2: Everything seems to be running, but we can't seem to get a connection:

```bash
➜ k -n mars run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 manager-api-svc:4444
```

Step 3: Ok, let's try to connect to one pod directly:

```bash
k -n mars get pod -o wide # get cluster IP
```

```bash
k -n mars run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.0.1.14s
```

Step 4: The Pods itself seem to work. Let's investigate the Service a bit:

```bash
k -n mars describe service manager-api-svc
```

## Question 19: Service ClusterIP->NodePort

Solve this question on instance: ssh ckad5601

In Namespace jupiter you'll find an apache Deployment (with one replica) named jupiter-crew-deploy and a ClusterIP Service called jupiter-crew-svc which exposes it. Change this service to a NodePort one to make it available on all nodes on port 30100.

Test the NodePort Service using the internal IP of all available nodes and the port 30100 using curl, you can reach the internal node IPs directly from your main terminal. On which nodes is the Service reachable? On which node is the Pod running?

## Question 20: NetworkPolicy

In Namespace venus you'll find two Deployments named api and frontend. Both Deployments are exposed inside the cluster using Services. Create a NetworkPolicy named np1 which restricts outgoing tcp connections from Deployment frontend and only allows those going to Deployment api. Make sure the NetworkPolicy still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

Test using: wget www.google.com and wget api:2222 from a Pod of Deployment frontend.

## Question 21: Requests and Limits, ServiceAccount

Solve this question on instance: ssh ckad7326

Team Neptune needs 3 Pods of image httpd:2.4-alpine, create a Deployment named neptune-10ab for this. The containers should be named neptune-pod-10ab. Each container should have a memory request of 20Mi and a memory limit of 50Mi.

Team Neptune has it's own ServiceAccount neptune-sa-v2 under which the Pods should run. The Deployment should be in Namespace neptune.

## Question 22: Labels, Annotations

Solve this question on instance: ssh ckad9043

 

Team Sunny needs to identify some of their Pods in namespace sun. They ask you to add a new label protected: true to all Pods with an existing label type: worker or type: runner. Also add an annotation protected: do not delete this pod to all Pods having the new label protected: true.

