## Comparison of Docker and containerD

|   Feature   |                                   Docker                                    |                         ContainerD                          |
| :---------: | :-------------------------------------------------------------------------: | :---------------------------------------------------------: |
|   Purpose   |                     Full container management platform                      |                Lightweight container runtime                |
|  Use Case   |                   Deployment and production environments                    |            Often used in Kubernetes as a runtime            |
|  Features   | Includes tools for building images, managing, containers, and orchestration |         Focuses on running and managing containers          |
| Integration |                Works with containerD as its runtime backend                 |         Directly used by Kubernetes or other tools          |
| Ease of Use |              Developer-friendly with a rich CLI and ecosystem               | More technical; needs higher-level tools for image building |
| Performance |               slightly heavier due to added layers and tools                |          Lightweights and optimized for Kubernetes          |

## CLI - ctr

- ctr comes with containerD
- Not very user friendly
- Only supports limited features

```
ctr images pull docker.io/library/redis:apline

ctr run docker.io/library/redis:apline redis
```

## CLI - nerdctl

- nerdctl provides a Docker-file CLI for containerD
- nerdctl supports docker compose
- nerdctl supports newest features in containerD
  - Encrypted container images
  - Lazy pulling
  - P2P image distribution
  - Image signing and verifying
  - Namespaces in Kubernetes

```
docker $ docker run --name redis redis:apline
nerdctl $ nerdctl run --name redis redis:apline
```

```
docker $ docker run --name webserver -p 80:80 -d nginx
nerdctl $ nerdctl run --name webserver -p 80:80 -d nginx
```

## CLI - crictl

- crictl provides a CLI for CRI compatible container runtime
- installed separately
- used to inspect and debug container runtimes
  - not to create containers ideally
- works across different runtimes

```
crictl pull busybox
```
```
crictl images
```
```
crictl ps -a
```
```
crictl exec -i -t asldkh129uij3e10dy1owh1231ou0d1u012hd189 ls
```
```
crictl logs asldkh129uij
```
```
crictl pods
```

<img src="https://raw.githubusercontent.com/matoanbach/k8s-ckad/assets/docker-vs-crictl.png"/>
<img src="https://raw.githubusercontent.com/matoanbach/k8s-ckad/assets/docker-vs-crictl2.png"/>
