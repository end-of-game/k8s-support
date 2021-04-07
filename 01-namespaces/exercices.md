## Lister les Namespaces disponibles

```
kubectl get ns

NAME          STATUS    AGE
default           Active   34h
kube-node-lease   Active   34h
kube-public       Active   34h
kube-system       Active   34h
```

Comme toutes ressources il est possible de la labeliser mais bien plus encore.

```
kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```

## Create a new one

Add the file 'training-ns.yaml'
```
apiVersion: v1
kind: Namespace
metadata:
  name: training
``` 

```
kubectl apply -f training-ns.yaml
```

List the namespaces

```
kubectl get ns --show-labels
```

Add labels as metadata

```
apiVersion: v1
kind: Namespace
metadata:
  name: training
  labels:
    key1: val1
    key2: val2
```

List the namespaces

```
kubectl get ns --show-labels
```

Remove the first label

```
kubectl label ns training key1-
```

Add a new one

```
kubectl label ns training key3=val3
```

List the namespaces

```
kubectl get ns --show-labels
```

