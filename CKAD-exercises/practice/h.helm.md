# Managing Kubernetes with Helm

- Note: Helm is part of the new CKAD syllabus. Here are a few examples of using Helm to manage Kubernetes.

## Helm in K8s

### Creating a basic Helm chart

<details><summary>show</summary>
<p>

```bash
helm create chart
```

</p>
</details>

### Running a Helm chart

<details><summary>show</summary>
<p>

```bash
helm repo add repo-name repo-url

helm install -f myvalues.yaml myredis ./redis
```

</p>
</details>

### Find pending Helm deployments on all namespaces

<details><summary>show</summary>
<p>

```bash
helm list --pending -A
```

</p>
</details>

### Uninstall a Helm release

<details><summary>show</summary>
<p>

```bash
helm uninstall myredis -n namespace
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm repo update

helm upgrade -f myvalues.yaml -f override.yaml myredis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

```bash
helm repo add [name] [URL] [flags]

helm repo list / helm repo ls

helm repo remove [REP01] [flags]

helm repo update / helm repo up

helm repo update / [REP01] [flags]

helm repo index [DIR] [flags]
```

</p>
</details>

### Download a Helm chart from a repository

<details><summary>show</summary>
<p>

```bash
helm pull --untar [char URL | repo/chartname]
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm

<details><summary>show</summary>
<p>

```bash
helm repo add https://charts.bitnami.com/bitnami
helm repo list 
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
helm show values bitnami/node
helm install node-release bitnami/node --set replicaCount=5
```

</p>
</details>
