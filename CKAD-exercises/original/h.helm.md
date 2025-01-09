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
helm install -f myvalues.yaml myrelease ./mychart
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
helm uninstall myrelease -n mynamespace
```

</p>
</details>

### Upgrading a Helm chart

<details><summary>show</summary>
<p>

```bash
helm repo update
helm upgrade -f myvalues.yaml -f overrides.yaml myrelease redis ./redis
```

</p>
</details>

### Using Helm repo

<details><summary>show</summary>
<p>

```bash
helm repo add
helm repo index
helm repo list
helm repo remove
helm repo update
```

</p>
</details>

### Download a Helm chart from a repository

<details><summary>show</summary>
<p>

```bash
helm pull [char URL | repo/chartname] [...] [flags]
helm pull --untar mychar.registry.com
```

</p>
</details>

### Add the Bitnami repo at https://charts.bitnami.com/bitnami to Helm

<details><summary>show</summary>
<p>

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

</p>
</details>

### Write the contents of the values.yaml file of the `bitnami/node` chart to standard output

<details><summary>show</summary>
<p>

```bash
helm search repo bitnami # show all charts within a repo
helm show values bitnamo/node
```

</p>
</details>

### Install the `bitnami/node` chart setting the number of replicas to 5

<details><summary>show</summary>
<p>

```bash
helm show values bitnami/node | grep -i replica
```

```bash
helm install newrelease bitnami/node --set replicaCount=5
```

```bash
k get po
# output
NAME                                  READY   STATUS     RESTARTS   AGE
newrelease-mongodb-58f99bbc9f-k26rl   0/1     Pending    0          3s
newrelease-node-65877d99d-948d5       0/1     Init:0/2   0          3s
newrelease-node-65877d99d-dwfp4       0/1     Init:0/2   0          3s
newrelease-node-65877d99d-gdz6z       0/1     Init:0/2   0          3s
newrelease-node-65877d99d-h6npb       0/1     Init:0/2   0          3s
newrelease-node-65877d99d-rbnwz       0/1     Init:0/2   0          3s
```

</p>
</details>
