### Plain Key Value

```
env:
    - name: APP_COLOR
      value: pink
```

### ConfigMap

```
env:
    - name: APP_COLOR
      valueFrom:
          configMapKeyRef:
```

### Secrets

```
env:
    - name: APP_COLOR
      valueFrom:
        configKeyRef
```
