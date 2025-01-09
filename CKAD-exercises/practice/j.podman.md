# Define, build and modify container images

- Note: The topic is part of the new CKAD syllabus. Here are a few examples of using **podman** to manage the life cycle of container images. The use of **docker** had been the industry standard for many years, but now large companies like [Red Hat](https://www.redhat.com/en/blog/say-hello-buildah-podman-and-skopeo) are moving to a new suite of open source tools: podman, skopeo and buildah. Also Kubernetes has moved in this [direction](https://kubernetes.io/blog/2022/02/17/dockershim-faq/). In particular, `podman` is meant to be the replacement of the `docker` command: so it makes sense to get familiar with it, although they are quite interchangeable considering that they use the same syntax.

## Podman basics

### Create a Dockerfile to deploy an Apache HTTP Server which hosts a custom main page

<details><summary>show</summary>
<p>

```bash
vim Dockerfile
```

```yaml
FROM docker.io/httpd:2.4
RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
```

</p>
</details>

### Build and see how many layers the image consists of

<details><summary>show</summary>
<p>

```bash
sudo podman build -t myapache .
# output
STEP 1/2 : FROM docker.io/httpd:2.4
Trying to pull docker.io/library/httpd:2.4...
Getting image source signatures
Copying blob bbf1f316b6e42 done
Copying blob 4f4b7090ef54 done
Copying blob dc6740587f8f done
Copying blob 8c3021b233c7 done
Copying blob 173229bd5c2c done
Copying blob d73be74d9e41 done
Copying config 4ec4c750a done
Writing manifest to image destination
Storing signatures
STEP 2/2 : RUN echo "Hello, World!" > /usr/local/apache2/htdocs/index.html
COMMIT simpleapp
---> a07ed3575b9a
Successfully tagged localhost/simpleapp:latest
```

```bash
candidate@ckad9043:~/myapaches$ sudo podman images
REPOSITORY                        TAG         IMAGE ID      CREATED           SIZE
localhost/simpleapp               latest      a07ed3575b9a  44 seconds ago    151 MB
registry.killer.sh:5000/sun-cipher v1-podman   7853733e4705  26 hours ago      8.04 MB
docker.io/library/httpd           2.4         4ec4c750a585  5 months ago      151 MB
docker.io/library/registry        2           75e5fb73afaf  15 months ago     26 MB
docker.io/library/golang          1.15.15-alpine3.14 140a37b6d4da  3 years ago   308 MB
docker.io/library/alpine          3.12.4      88dd2752d2ea  3 years ago       5.85 MB
```

```bash
candidate@ckad9043:~/myapaches$ sudo podman image tree localhost/simpleapp
Image ID: a07ed3575b9a
Tags:     [localhost/simpleapp:latest]
Size:     151.5MB

Image Layers
├─ ID: 8b296f468960 Size: 77.89MB
├─ ID: 27719ce47073 Size: 2.56kB
├─ ID: e09e36c4974b Size: 1.024kB
├─ ID: 65a9cf2fddab Size: 10.64MB
├─ ID: 9f4fac23e834 Size: 62.92MB
├─ ID: 33907ab7ce7c Size: 3.584kB Top Layer of: [docker.io/library/httpd:2.4]
└─ ID: f6447479d604 Size: 6.144kB Top Layer of: [localhost/simpleapp:latest]
```

</p>
</details>

### Run the image locally, inspect its status and logs, finally test that it responds as expected

<details><summary>show</summary>
<p>

```bash
sudo podman run -d --name=test -p 8080:80 localhost/simpleapp
# output
19a4eab02da2b4f9c0b05f13043d4cad59fdbf5b96b8e5ef56b749ab3de5e6a2
```

```bash
sudo podman ps
# output
CONTAINER ID  IMAGE                                         CREATED        STATUS            PORTS                    COMMAND         NAMES
43d865990ca2  docker.io/library/registry:2                 3 months ago   Up 3 months       0.0.0.0:5000->5000/tcp   /etc/docker/    registry
b0b013b25119  registry.killer.sh:5000/sun-cipher:v1-podman 26 hours ago   Up 26 hours       -                        ./app           sun-cipher
19a4eab02da2  localhost/simpleapp:latest                   2 minutes ago  Up 2 minutes      0.0.0.0:8080->80/tcp     httpd-foreground test

```

</p>
</details>

### Run a command inside the pod to print out the index.html file

<details><summary>show</summary>
<p>

```bash
sudo podman exec -it test cat /usr/local/apache2/htdocs/index.html
Hello, World!
```

</p>
</details>

### Tag the image with ip and port of a private local registry and then push the image to this registry

<details><summary>show</summary>
<p>

> Note: Some small distributions of Kubernetes (such as [microk8s](https://microk8s.io/docs/registry-built-in)) have a built-in registry you can use for this exercise. If this is not your case, you'll have to setup it on your own.

```bash
sudo podman tag locahost/simpleapp $registy_ip:5000/simpleapp

sudo podman push $registry_ip:5000/simpleapp
```

</p>
</details>

Verify that the registry contains the pushed image and that you can pull it

<details><summary>show</summary>
<p>

```bash
curl http://$registry_ip:5000/simpleapp
```

```bash
# remove the image already present
sudo podman rmi $registry_ip:5000/simpleapp
```

```bash
sudo podman pull  $registry_ip:5000/simpleapp

# output
Trying to pull 10.152.183.13:5000/simpleapp:latest...
Getting image source signatures
Copying blob 643ea8c2c185 skipped: already exists
Copying blob 972107ece720 skipped: already exists
Copying blob 9857eeea6120 skipped: already exists
Copying blob 93859aa62dbd skipped: already exists
Copying blob 8e47efbf2b7e skipped: already exists
Copying blob 42e0f5a91e40 skipped: already exists
Copying config ef4b14a72d done
Writing manifest to image destination
Storing signatures
ef4b14a72d02ae0577eb0632d084c057777725c279e12ccf5b0c6e4ff5fd598b
```

</p>
</details>

### Create a container without running/starting it

<details><summary>show</summary>
<p>

```bash
sudo podman create busybox
# output
Resolved "busybox" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob sha256:213a27df5921cd9ae24732504c590bb6408911c20fb50a597f2a40896d554a8f
Copying config sha256:3fba0c87fcc8ba126bf99e4ee205b43c91ffc6b15bb052315312e71bc6296551
Writing manifest to image destination
51b613406e8889213c176523e1c430e4bd00047965b0c22cff5b1c9badfbc452
```

```bash
sudo podman container ls -a
# output
CONTAINER ID  IMAGE                             COMMAND     CREATED        STATUS      PORTS       NAMES
51b613406e88  docker.io/library/busybox:latest  sh          2 minutes ago  Created
```

</p>
</details>

### Export a container to output.tar file

<details><summary>show</summary>
<p>

```bash
sudo podman export --output="busybox.tar" 51b613406e88
ls -al busybox.tar
-rw-r--r--@ 1 root root  4272640 28 Aug 13:48 busybox.tar
```

</p>
</details>

### Run a pod with the image pushed to the registry

<details><summary>show</summary>
<p>

```bash
k run simpleapp --image=$registry_ip:5000/simpleapp --port=80 --expose
k get po -owide
curl -m 5 <simpleappIP:PORT>
```

</p>
</details>

### Log into a remote registry server and then read the credentials from the default file

<details><summary>show</summary>
<p>

> Note: The two most used container registry servers with a free plan are [DockerHub](https://hub.docker.com/) and [Quay.io](https://quay.io/).

```bash
sudo podman login --username $YOUR_USER --password $YOUR_PWD docker.io
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
kubectl create secret generic mysecret --from-file=.dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type=kubernetes.io/dockerconfigjson

kubectl create secret docker-registry mysecret2 --docker-server=https:/index.docker.io/v1/ --docker-username=$YOUR_USR --docker-password=$YOUR_PWD
```

</p>
</details>

### Create the manifest for a Pod that uses one of the two secrets just created to pull an image hosted on the relative private remote registry

<details><summary>show</summary>
<p>

```YAML
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
sudo podman rm --all --force
sudo podman rmi --all --all
```
</p>
</details>
```
