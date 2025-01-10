# Managing Kubernetes with Helm

- Note: Helm is part of the new CKAD syllabus. Here are a few examples of using Helm to manage Kubernetes.

## Helm in K8s

### Creating a basic Helm chart

<details><summary>show</summary>
<p>

```bash
helm create newchart
```

</p>
</details>

### Running a Helm chart

<details><summary>show</summary>
<p>

```bash
helm install -f values.yaml redis ./redis
```

</p>
</details>

### Find pending Helm deployments on all namespaces

<details><summary>show</summary>
<p>

```bash
helm list --pending -A
helm list --pending --all-namespaces
```

</p>
</details>

### Uninstall a Helm release

<details><summary>show</summary>
<p>

```bash
helm uninstall releasename -n namespace
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm repo update

helm upgrade -f values.yaml -f override.yaml redis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

```bash
helm repo add       # add a chart repository
helm repo index     # generate an index file given a directory containing packaged charts
helm repo list      # list chart repositories
helm repo remove    # remove one or more chart repositories
helm repo update    # update information of available chart locally from chart repositories
```

</p>
</details>

### Download a Helm chart from a repository

<details><summary>show</summary>
<p>

```bash
helm pull --untar chart_url
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm

<details><summary>show</summary>
<p>

```bash
helm repo add [NAME] [URL]
helm repo add bitnami https://charts.bitnami.com/bitnami
```

</p>
</details>

### Write the contents of the values.yaml file of the `bitnami/node` chart to standard output

<details><summary>show</summary>
<p>

```bash
helm show values bitnami/node
```

</p>
</details>

### Install the `bitnami/node` chart setting the number of replicas to 5

<details><summary>show</summary>
<p>

```bash
helm show values bitnami/node  | grep -i replica
# Output:

## @param replicaCount Specify the number of replicas for the application
replicaCount: 1
```

```bash
helm install simplenode bitnami/node --set replicaCount=5
```

```bash
k get po
# output:
NAME                                  READY   STATUS     RESTARTS   AGE
simplenode-855fc95848-58pgj           0/1     Init:0/2   0          3s
simplenode-855fc95848-5pfpb           0/1     Init:0/2   0          3s
simplenode-855fc95848-8kdvh           0/1     Init:0/2   0          3s
simplenode-855fc95848-nf4rz           0/1     Init:0/2   0          3s
simplenode-855fc95848-v7x8m           0/1     Init:0/2   0          3s
simplenode-mongodb-79f6b54b67-4glw4   0/1     Pending    0          3ss
```

</p>
</details>
