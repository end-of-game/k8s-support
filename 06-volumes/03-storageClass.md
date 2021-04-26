## Create a directory

```
mkdir -p volumes/03-exercises
cd volumes/03-exercises
```

## List the Storage class available

```
kubectl api-resources
kubectl get sc
```

```
NAME                         PROVISIONER                 RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
do-block-storage (default)   dobs.csi.digitalocean.com   Delete          Immediate           true                   17h
```

## Create a PVC

No more need about PV.

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: test-dynamic-volume-claim-$TRIGRAMME
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply the YAML file

```
TRIGRAMME=nmu envsubst < pvc.yaml.tmpl > pvc.yaml
kubectl apply -f pvc.yaml
kubectl get pvc
```

## Resize the PVC

Change the size into the pvc.yaml directly and apply.

```
kubectl describe pvc test-dynamic-volume-claim-<TRIGRAMME>
```

Note that the size is not updated but in the Digital Ocean Dashboard

## Create a pod

Create the file dep.yaml.tmpl

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
        - name: container
          image: ubuntu
          command: ["sleep"]
          args: ["3600"]
          volumeMounts:
          - name: vol
            mountPath: "/data"
      volumes:
      - name: vol
        persistentVolumeClaim: 
          claimName: test-dynamic-volume-claim-$TRIGRAMME
```

Apply the YAML file

```
TRIGRAMME=nmu envsubst < dep.yaml.tmpl > dep.yaml
kubectl apply -f dep.yaml
kubectl get dep
```

```
kubectl exec -it nginx -- bash
mount | grep html
```

Observe the mount point and the size of the pvc

```
kubectl get pvc
```

It is resized.

## Scale the pods

Increase to 10 or more.

```
kubectl edit deploy my-deployment
```

what is happening ?

