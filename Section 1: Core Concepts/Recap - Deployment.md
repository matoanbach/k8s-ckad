## Kubernetes Deployment

- deployment-definition.yml

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp-deployment
    labels:
        app: myapp
        type: front-end
spec:
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
                type: front-end
        spec:
            containers:
            -   name: nginx-container
                image: nginx
replicas: 3
selector:
    matchLabels:
        type: front-end
```

```
kubectl create -f deployment-defintion.yml
```

```
kubectl get deployments
```

## Commands

```
kubectl get all
```

## Certification tip: Formatting output with kubetcl

- The default output format for all kubectl commands is the human-readable plain-text format

- The -o flag allows us to output the details in several different formats

```
kubectl [command] [TYPE][NAME] -o <output_format>
```

- Here are some of the commonly used formats:

  1. <code>-o json</code> output a JSON formatted API object.
  2. <code>-o name</code> print only the resource name and nothing else.
  3. <code>-o wide</code> output in the plain-text format with any additional information.

<img src="https://github.com/matoanbach/k8s-ckad/blob/main/assets/sec%201/4.png"/>

- For more details, refer:

  - https://kubernetes.io/docs/reference/kubectl/overview/

  - https://kubernetes.io/docs/reference/kubectl/cheatsheet/
