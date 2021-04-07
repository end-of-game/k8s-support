## List the resources

```
kubectl get po

No resources found in default namespace
```

```
kubectl get po -n training

No resources found in training namespace
```

```
kubectl get po -n undef

No resources found in undef namespace
```

## First, set the context

```
kubectl config set-context --current --namespace=training
```

```
kubectl get po

No resources found in training namespace
```

## Run a Nginx server with command line

Lancer une instance Tomcat sur le port standard.

```
kubectl run webserver --restart=Never --image=nginx:1.9
```

Verify the pod status

```
kubectl get pods
kubectl get pods -o wide
```

Create a new one with same name

```
kubectl run webserver --restart=Never --image=nginx:1.9

Error from server (AlreadyExists): pods "webserver" already exists
```

So change its name

```
kubectl run webserver2 --restart=Never --image=nginx:1.9
kubectl get pods -o wide

NAME         READY   STATUS    RESTARTS   AGE     IP             NODE                   NOMINATED NODE   READINESS GATES
webserver    1/1     Running   0          6m13s   10.244.0.200   pool-42964rtz2-8qo0h   <none>           <none>
webserver2   1/1     Running   0          8s      10.244.1.9     pool-42964rtz2-8qo0k   <none>           <none>
```

## Sort the pods by creation date

```
kubectl get po -o wide --sort-by=.metadata.creationTimestamp
kubectl get po -o wide --sort-by=.metadata.creationTimestamp | tac
```

You can delete a pod

```
kubectl delete webserver2
error: the server doesn't have a resource type "webserver2"
```

```
kubectl delete po webserver2
pod "webserver2" deleted
```


## Let's use a declarative way 

Create a file with the option **dry-run**.

```
kubectl run webserver3 --restart=Never --image=nginx:1.9 --dry-run=client -o yaml > pod-webserver.yaml
```

Create the instance

```
kubectl apply -f pod-webserver.yaml
kubectl get po
```

Enter into pod/container with exec command

```
kubectl exec -it po webserver3 -- bash
```

```
root@webserver3:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:40 ?        00:00:00 nginx: master process nginx -g daemon off;
nginx        8     1  0 08:40 ?        00:00:00 nginx: worker process
root         9     0  0 08:40 pts/0    00:00:00 bash
root        18     9  0 08:41 pts/0    00:00:00 ps -ef
```

Exit the container and modify the file to add a new container into pod

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webserver3
  name: webserver3
spec:
  containers:
  - image: nginx:1.9
    name: webserver3
    resources: {}
  - name: shell
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Then update the pod

```
kubectl apply -f pod-webserver.yaml
```

You cannot because pod is not mutable

```
kubectl delete -f pod-webserver.yaml
kubectl apply -f pod-webserver.yaml
```

Enter into pod/container with exec command

```
kubectl exec -it po webserver3 -c shell -- bash
```

We are not into nginx. We can verify it with

```
ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:46 ?        00:00:00 sleep 10000
root         8     0  0 08:47 pts/0    00:00:00 bash
root        23     8  0 08:48 pts/0    00:00:00 ps -ef
```

But we can reach the nginx locally. Why ?

```
curl localhost
```

