# Define, build and modify container images

- Note: The topic is part of the new CKAD syllabus. Here are a few examples of using **podman** to manage the life cycle of container images. The use of **docker** had been the industry standard for many years, but now large companies like [Red Hat](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo) are moving to a new suite of open source tools: podman, skopeo and buildah. Also Kubernetes has moved in this [direction](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). In particular, `podman` is meant to be the replacement of the `docker` command: so it makes sense to get familiar with it, although they are quite interchangeable considering that they use the same syntax.

## Podman basics

### Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

<details><summary>show</summary>
<p>

```Dockerfile
FROM docker.io/httpd:2.4
RUN echo "Hello it's me" > /usr/local/apache2/htdocs/html
```

</p>
</details>

### Build and see how many layers the image consists of

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman build -t mysimpleapp .
STEP 1/2 : FROM docker.io/httpd:2.4
STEP 2/2 : RUN echo "Hello it's me" > /usr/local/apache2/htdocs/index.html
COMMIT mysimpleapp
--> 3a9f62dac26
Successfully tagged localhost/mysimpleapp:latest
3a9f62dac26c84108a54b49cd0c67583046fb6f281de4445ff957760423ef65
```

```bash
candidate@ckad9043:~/myimage$ sudo podman images
REPOSITORY                TAG                 IMAGE ID      CREATED          SIZE
localhost/mysimpleapp     latest              3a9f62dac26c  2 minutes ago    151 MB
localhost/simpleapp       latest              1097ccdfd41c  19 hours ago     151 MB
docker.io/library/httpd   2.4                 4ce47c750a58  5 months ago     151 MB
docker.io/library/registry 2                  75ef5b734af4  15 months ago    26 MB
docker.io/library/golang  1.15.15-alpine3.14  1403af3b6d4a  3 years ago      308 MB
```

```bash
candidate@ckad9043:~/myimage$ sudo podman image tree localhost/mysimpleapp
Image ID: 3a9f62dac26c
Tags:      [localhost/mysimpleapp:latest]
Size:      151.5MB
Image Layers
├─ ID: 8b296f486960 Size: 77.89MB
├─ ID: 27719ec707e3 Size: 2.56kB
├─ ID: e9e636c4974b Size: 1.024kB
├─ ID: 65a9cfc2fdab Size: 10.64MB
├─ ID: 9f74ca23e843 Size: 62.92MB
├─ ID: 33907ab7ce7c Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└─ ID: 42012a369e31 Size: 6.144kB Top Layer of: [localhost/mysimpleapp:latest]
```

</p>
</details>

### Run the image locally, inspect its status and logs, finally test that it responds as expected

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman run -d -p 8080:80 localhost/mysimpleapp
748e796cae1d02d0b345f435bfcd4e31915da14e75090c85be516658b2ac417a
```

```bash
candidate@ckad9043:~/myimage$ sudo podman ps -a
CONTAINER ID  IMAGE                            COMMAND          CREATED         STATUS             PORTS                   NAMES
748e796cae1d  localhost/mysimpleapp:latest     httpd-foreground 2 minutes ago   Up 2 minutes ago   0.0.0.0:8080->80/tcp    sleepy_tesla
```

```bash
candidate@ckad9043:~/myimage$ curl -m 5 0.0.0.0:8080
Hello it's me
```

</p>
</details>

### Run a command inside the pod to print out the index.html file

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman exec -it 748e796cae1d cat /usr/local/apache2/htdocs/index.html
Hello it's me
```

</p>
</details>

### Tag the image with ip and port of a private local registry and then push the image to this registry

<details><summary>show</summary>
<p>

> Note: Some small distributions of Kubernetes (such as [microk8s](https://microk8s.io/docs/registry-built-in)) have a built-in registry you can use for this exercise. If this is not your case, you'll have to setup it on your own.

```bash
sudo podman tag localhost/mysimpleapp $registry_ip:5000/mysimpleapp
```

```bash
sudo podman push $registry_ip:5000/mysimpleapp
```

</p>
</details>

Verify that the registry contains the pushed image and that you can pull it

<details><summary>show</summary>
<p>

```bash
curl $registry_ip:5000/mysimpleapp
sudo podman rmi $registry_ip:5000/mysimpleapp
sudo podman pull $registry_ip:5000/mysimpleapp
```

</p>
</details>

### Create a container without running/starting it

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman container create localhost/mysimpleapp
692488a0489567e8c961e1c8c7c9e053b274a2df3cf371afb3361ff61f1b3b72
```

```bash
candidate@ckad9043:~/myimage$ sudo podman container list
CONTAINER ID  IMAGE                         COMMAND              CREATED        STATUS            PORTS                 NAMES
748e796cae1d  localhost/mysimpleapp:latest  httpd-foreground     11 minutes ago Up 11 minutes ago 0.0.0.0:8080->80/tcp  sleepy_tesla
```

```bash
candidate@ckad9043:~/myimage$ sudo podman container list -a
CONTAINER ID  IMAGE                         COMMAND              CREATED        STATUS            PORTS                 NAMES
748e796cae1d  localhost/mysimpleapp:latest  httpd-foreground     11 minutes ago Up 11 minutes ago 0.0.0.0:8080->80/tcp  sleepy_tesla
692488a04895  localhost/mysimpleapp:latest  httpd-foreground     9 seconds ago  Created                                elastic_bhaskara
```

</p>
</details>

### Export a container to output.tar file

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman export --output="output.tar" elastic_bhaskara
```

```bash
candidate@ckad9043:~/myimage$ ls
Dockerfile  output.tar
```

```bash
candidate@ckad9043:~/myimage$ ls -al
total 146732
drwxrwxr-x 2 candidate candidate      4096 Jan 10 15:36 .
drwxr-x--- 8 candidate candidate      4096 Jan 10 15:12 ..
-rw-rw-r-- 1 candidate candidate        93 Jan 10 15:12 Dockerfile
-rw-r--r-- 1 root      root      150236160 Jan 10 15:36 output.tar
```

</p>
</details>

### Run a pod with the image pushed to the registry

<details><summary>show</summary>
<p>

```bash
sudo k run pod --image=$registry_ip:5000/mysimpleapp --port=80 --restart=Never
```

</p>
</details>

### Log into a remote registry server and then read the credentials from the default file

<details><summary>show</summary>
<p>

> Note: The two most used container registry servers with a free plan are [DockerHub](https://hub.docker.com/) and [Quay.io](https://quay.io/).

```bash
sudo podman login --username $YOUR_UNAME --password $YOUR_PWD docker.io
```

```bash
:~$ cat ${XDG_RUNTIME_DIR}/containers/auth.json
{
        "auths": {
                "docker.io": {
                        "auth": "Z2l1bGl0JLSGtvbkxCcX1xb617251xh0x3zaUd4QW45Q3JuV3RDOTc="
                }
        }
}
```

</p>
</details>

### Create a secret both from existing login credentials and from the CLI

<details><summary>show</summary>
<p>

```bash
kubectl create secreate docker_registry mysecret --docker-username=$YOUR_UNAME --docker-password=$YOUR_PWD --docker-server="https://index.docker.io/v1/"
```

</p>
</details>

### Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

<details><summary>show</summary>
<p>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
    - name: private-reg-container
      image: $YOUR_PRIVATE_IMAGE
  imagePullSecrets:
    - name: mysecret
```

</p>
</details>

### Clean up all the images and containers

<details><summary>show</summary>
<p>

```bash
sudo podman rmi --all --force
sudo podman rm --all --force
```

</p>
</details>
