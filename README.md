- Create a deployment manifest file called my-webapp using the imperative command:-

```bash
kubectl create deployment my-webapp --image=nginx --replicas=2 --dry-run=client -oyaml > my-webapp.yaml
```

- To expose the deployment as a node port service using the imperative command:-

```bash
kubectl expose deployment my-webapp --name front-end-service --type NodePort --port 80 --dry-run=client -oyaml > front-end-service.yaml
```

- now, add the nodePort field under the ports section as follows:-
```bash
ports:
 - port: 80
   protocol: TCP
   targetPort: 80
   nodePort: 30083
```