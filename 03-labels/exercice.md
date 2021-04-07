## Travail sur les labels

### Nettoyer l'environnement précédent

`kubectl delete pods --all`

### Let's deploy a webserver

Create a new file **nginx.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: dev
spec:
  containers:
  - name: web
    image: nginx:1.9
```

```
kubectl apply -f nginx.yaml
```

Let's display the labels of the pods:

````
kubectl get pod/nginx --show-labels
nginx   1/1     Running   0          100s   env=dev
````

Add a new label to the pod:

```
kubectl label pod/webserver owner=student

```
kubectl get pod/nginx --show-labels
NAME       READY     STATUS    RESTARTS   AGE       LABELS
nginx   1/1     Running   0          100s   env=dev,owner=student
```

Filter the pods by labels:

```
kubectl get pods -l env=dev
NAME       READY     STATUS    RESTARTS   AGE
nginx   1/1       Running   0          3m
```

```
kubectl get pods -l env=prod
No resources found in training namespace.
```

Remove a label from a pod

```
kubectl label po nginx env-

kubectl get po nginx --show-labels
nginx   1/1     Running   0          4m5s   owner=student
```


## Let's use the selector

Create a new file **apache.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: apache
  labels:
    env: prod
    owner: admin
spec:
  containers:
  - name: web
    image: httpd:2.4
```

Let's run a new container

```
kubectl apply -f apache.yaml
```

````
kubectl get pods --show-labels
NAME         READY     STATUS    RESTARTS   AGE       LABELS
mytomcat     1/1       Running   0          8m        env=development,owner=nicolas
realtomcat   1/1       Running   0          12s       env=production,owner=kevin
````

Filter many pods with *--selector* or *-l*

```
kubectl get pods --selector 'env in (prod, dev)' --show-labels
kubectl get pods -l 'env in (prod, dev)' --show-labels
```

Let's choose only production pod

```
kubectl get pods --selector 'env in (prod)'
kubectl get pods -l 'env in (prod)'
```
