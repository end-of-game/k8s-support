# DaemonSets and NodeSelector

## Node Label

### List your nodes

```
kubectl get nodes --show-labels
```

### Add label to your nodes

```
kubectl label nodes <worker1> ssd=true
```

### Filter nodes based on label

```
kubectl get nodes --selector ssd=true
```

## DaemonSet

Create daemonset from the **daemonset.yml**

```
apiVersion: apps/v1
kind: "DaemonSet"
metadata:
  labels:
    app: nginx
    ssd: "true"
  name: nginx-fast-storage
spec:
  selector:
    matchLabels:
     app: nginx
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      nodeSelector:
        ssd: "true"
      containers:
        - name: nginx
          image: nginx:1.10.0
```          

### Check the nodes where nginx was deployed

```
kubectl get pods -o wide -l ssd=true
```

### Add label ssd=true to another worker node

```
kubectl label nodes <worker2> ssd=true
```

### Check the nodes where nginx was deployed

It should be also another node with **ssd=true** label
nginx should be deployed there automatically

```
kubectl get pods -o wide -l ssd=true
```

Update the label value

```
kubectl label node <worker2> ssd=false --overwrite
```

nginx should be removed there automatically

```
kubectl get pods -o wide -l ssd=true
```

### Update the image

You could update images on the running DS for example with:

```
kubectl set image ds/nginx-fast-storage nginx=nginx:1.11
```

But no way to update it... why ?
Try to understand with the strategy field given by

```
kubectl edit ds/nginx-fast-storage
```

Change it with **RollingUpdate** and let the magic begin !

### Clean up

```
kubectl label node <worker1> ssd-
kubectl label node <worker2> ssd-
kubectl delete ds nginx-fast-storage
```
