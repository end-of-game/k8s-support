## Create a directory

```
mkdir -p volumes/01-exercises
cd volumes/01-exercises
```

## Create a POD with two containers

Create the file pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: sharedvolume
spec:
  containers:
  - name: container1 
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: mysharedvolume
        mountPath: "/tmp/data1"
  - name: container2
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: mysharedvolume
        mountPath: "/tmp/data2"
  volumes:
  - name: mysharedvolume
    emptyDir: {}
```

Apply the YAML file

```
kubectl apply -f pod.yaml
kubectl get pod
```

```
kubectl exec sharedvolume -c container1 -i -t -- bash
mount | grep data
```

As result 

```
/dev/vda1 on /tmp/data1 type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

Then create file

```
touch /tmp/data1/foo
```

Log into the second one 

```
kubectl exec sharedvolume -c container2 -i -t -- bash
mount | grep data
```

Another mount point
```
/dev/vda1 on /tmp/data2 type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

But similar data
```
ls /tmp/data2
```

## Conclusion

