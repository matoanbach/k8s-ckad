## What is a Replica and Why do we need a replication controller?

To prevent users from losing access to our application, we want more than one instance or pod running simultaneously. This way, if one pod fails, the other continues to run, ensuring high avalability. The <code>replication controller</code> helps us achieve this by running multiple instances of a single pod in the k8s cluster.

If planning to have only one pod, the replication controller will make sure it always running.

## Load balancing & Scaling

Another important role of the replication controler is to distribute load across multiple pods. For example, if a single pod is serving users and the number of users increases, deploying additional pods can balance the load. If demand further increases and resources on one node are exhausted, additional pods can be deployed on other nodes in the cluster. The replication controller spans across multiple nodes, balancing the load and scaling the application as demand increases.

## Replication vs Replica Set

They are two similar terms, serving the same purpose, however, they're not the same. The replication controller is the older technology, gradually being replaced by the replica set. Replica set is the recommended approach moving forward. However, everything we discuss about replication controllers applies to replica sets as well, with minor differences.

### Replcation controller

- rc-definition.yml

```
apiVersion: v1
kind: ReplicationController
metadata:
    name: myapp-rc
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
                    - name: nginx-container
                      image: nginx
    replicas: 3
```

```
kubectl create -f rc-definition.yml
```

```
kubectl get replicationcontroller
```

### replicaSet

- replicaset-definition.yml

```
apiVersion: v1
kind: ReplicaSet
metabdata:
    name: myapp-replicaset
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
                    - name: nginx-container
                      image: nginx
    replicas: 3
    selector:
        matchLabels:
            type: front-end
```

```
kubectl create -f replicaset-definition.yml
```

```
kubectl get replicaset
```

## Scale

Ways to increase replicas

1. edit the definition file and save it

```
kubectl replace -f replicaset-definition.yml
```

2. using cli

```
kubectl scale --replicas=6 replicaset myapp-replicaset
kubectl scale --replicas=6 -f replicaset-definition.yml
```
