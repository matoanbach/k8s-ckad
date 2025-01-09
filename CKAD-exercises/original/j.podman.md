# Define, build and modify container images

- Note: The topic is part of the new CKAD syllabus. Here are a few examples of using **podman** to manage the life cycle of container images. The use of **docker** had been the industry standard for many years, but now large companies like [Red Hat](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo) are moving to a new suite of open source tools: podman, skopeo and buildah. Also Kubernetes has moved in this [direction](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). In particular, `podman` is meant to be the replacement of the `docker` command: so it makes sense to get familiar with it, although they are quite interchangeable considering that they use the same syntax.

## Podman basics

### Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

<details><summary>show</summary>
<p>

```bash
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

</p>
</details>

### Build and see how many layers the image consists of

<details><summary>show</summary>
<p>

```bash
sudo podman build -t simpleapp .
# output
STEP 1/2 : FROM docker.io/httpd:2.4
Trying to pull docker.io/library/httpd:2.4...
Getting image source signatures
Copying blob 4f4fb700ef54 done
Copying blob 172b239db5c2 done
Copying blob bbf1f316b6e42 done
Copying blob d7382fd3e491 done
Copying blob 8c3021b233c7 done
Copying blob dc6740587f8f done
Copying config 4ec4c750a done
Writing manifest to image destination
Storing signatures
STEP 2/2 : RUN echo "Hello world" > /usr/local/apache2/htdocs/index.html
COMsudo MIT simpleapp
--> 1097ccdfd41
Successfully tagged localhost/simpleapp:latest
1097ccdfd41cb6decc6ef1796ba31041c4e1e8e0269ec7d19a893cf9cfd293b
candidate@ckad9043:~/myimage$
```

```bash
sudo podman images
# output
REPOSITORY               TAG                 IMAGE ID      CREATED         SIZE
localhost/simpleapp      latest              1097ccdfd41c  2 minutes ago  151 MB
docker.io/library/httpd  2.4                 4ec47c750a58  5 months ago   151 MB
docker.io/library/registry 2                 75e5fb734af4  15 months ago  26 MB
docker.io/library/golang 1.15.15-alpine3.14  1403af3b6d4a  3 years ago    308 MB
```

```bash
sudo podman image tree localhost/simpleapp
# output
Image ID: 1097ccdfd41c
Tags:     [localhost/simpleapp:latest]
Size:     151.5MB

Image Layers
├─ ID: 8b296f486960 Size: 77.89MB
├─ ID: 27719ec707e3 Size: 2.56kB
├─ ID: e09e636c4974b Size: 1.024kB
├─ ID: 65a9cf2fddab Size: 10.64MB
├─ ID: 9f74ca23e843 Size: 62.92MB
├─ ID: 33907ab7ce7c Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└─ ID: 2a2b2344cc33 Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

</p>
</details>

### Run the image locally, inspect its status and logs, finally test that it responds as expected

<details><summary>show</summary>
<p>

```bash
sudo podman run -d 1097ccdfd41c -p 8080:80
```

```bash
sudo podman run -d --name test -p 8080:80 localhost/simpleapp
dee0a083db54604f4c8ae9f12b91156b5e8fec0e785f5271d1c08e437f97c36e
```

```bash
candidate@ckad9043:~/myimage$ sudo podman ps
CONTAINER ID  IMAGE                      COMMAND               CREATED         STATUS            PORTS                   NAMES
43d865990ca2  docker.io/library/registry:2  /etc/docker/regis...  3 months ago    Up 3 months      0.0.0.0:5000->5000/tcp  registry
dee0a083db54  localhost/simpleapp:latest   httpd-foreground      45 seconds ago  Up 45 seconds     0.0.0.0:8080->80/tcp    test

candidate@ckad9043:~/myimage$ curl -m 5 0.0.0.0:8080
Hello world
```

</p>
</details>

### Run a command inside the pod to print out the index.html file

<details><summary>show</summary>
<p>

```bash
sudo podman exec -it test cat /usr/local/apache2/htdocs/index.html
Hello World!
```

</p>
</details>

### Tag the image with ip and port of a private local registry and then push the image to this registry

<details><summary>show</summary>
<p>

> Note: Some small distributions of Kubernetes (such as [microk8s](https://microk8s.io/docs/registry-built-in)) have a built-in registry you can use for this exercise. If this is not your case, you'll have to setup it on your own.

```bash
sudo podman tag localhost/simpleapp $registry_ip:5000/simpleapp
sudo podman push localhost/simpleapp $registry_ip:5000/simpleapp
```

</p>
</details>

Verify that the registry contains the pushed image and that you can pull it

<details><summary>show</summary>
<p>

```bash
curl $registry_ip:5000/simpleapp
sudo rmi $registry_ip:5000/simpleapp
sudo podman pull --untar $registry_ip:5000/simpleapp
```

</p>
</details>

### Create a container without running/starting it

<details><summary>show</summary>
<p>

```bash
sudo podman container create localhost/simpleapp
```

```bash
candidate@ckad9043:~/myimage$ sudo podman container list

CONTAINER ID  IMAGE                      COMMAND               CREATED        STATUS            PORTS                   NAMES
43d865990ca2  docker.io/library/registry:2  /etc/docker/regis...  3 months ago   Up 3 months      0.0.0.0:5000->5000/tcp  registry
dee0a083db54  localhost/simpleapp:latest   httpd-foreground      11 minutes ago Up 11 minutes     0.0.0.0:8080->80/tcp    test
```

```bash
candidate@ckad9043:~/myimage$ sudo podman container list -a

CONTAINER ID  IMAGE                      COMMAND               CREATED        STATUS            PORTS                   NAMES
43d865990ca2  docker.io/library/registry:2  /etc/docker/regis...  3 months ago   Up 3 months      0.0.0.0:5000->5000/tcp  registry
dee0a083db54  localhost/simpleapp:latest   httpd-foreground      11 minutes ago Up 11 minutes     0.0.0.0:8080->80/tcp    test
eb0d0a7913fc  localhost/simpleapp:latest   httpd-foreground      39 seconds ago Created                                  stupefied_mendel

```

</p>
</details>

### Export a container to output.tar file

<details><summary>show</summary>
<p>

```bash
candidate@ckad9043:~/myimage$ sudo podman export dee0a083db54 --output="output.tar"
candidate@ckad9043:~/myimage$ ls
Dockerfile  output.tar
```

</p>
</details>

### Run a pod with the image pushed to the registry

<details><summary>show</summary>
<p>

```bash
sudo podman run newtest $registry_ip:5000/simpleapp --port=80
```

</p>
</details>

### Log into a remote registry server and then read the credentials from the default file

<details><summary>show</summary>
<p>

> Note: The two most used container registry servers with a free plan are [DockerHub](https://hub.docker.com/) and [Quay.io](https://quay.io/).

```bash
sudo podman login --username $YOUR_NAME --password $YOUR_PASSWORD docker.io
```

</p>
</details>

### Create a secret both from existing login credentials and from the CLI

<details><summary>show</summary>
<p>

```bash
:~$ kubectl create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockeconfigjson
secret/mysecret created
:~$ kubectl create secret docker-registry mysecret2 --docker-server=https://index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
secret/mysecret2 created
```

</p>
</details>

### Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

<details><summary>show</summary>
<p>

```bash
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
sudo podman rmi --all
sudo podman rm --all
```

</p>
</details>
