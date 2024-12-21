```
apiVersion: v1
kind: pod
metadata:
    name: simple-webapp-color
spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: blue
        - name: APP_MODE
          value: prod
```

### Use of ConfigMaps

#### Imperative ConfigMaps

```
kubectl create configmap
    <config-name> --from-literal=<key>=<value>
```

- For example,

```
kubectl create configmap \
    app-config --from-literal=APP_COLOR=blue
```

```
# ConfigMap
APP_COLOR: blue
APP_MODE: prod
```

```
apiVersion: v1
kind: pod
metadata:
    name: simple-webapp-color
spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
          name: app-secret
```

#### Create ConfigMap using files

```
kubetcl create configmap
    <config-name> --from-file=<path-to-file>
```

- For example:

```
kubectl create configmap \
    app-config --from-file=app_config.properties
```

#### Declarative ConfigMap

```
kubectl create -f
```

```
# Config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config

data:
    APP_COLOR: blue
    APP_MODE: prod
```

### View ConfigMaps

```
kubectl get configmaps
```

### ConfigMap in Pods

```
apiVersion: v1
kind: pod
metadata:
    name: simple-webapp-color
spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```
