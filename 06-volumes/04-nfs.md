## Create a directory

```
helm repo update
helm repo add stable https://charts.helm.sh/stable
```

##

```
---
# Nfs-provisioner params for Helm install
#
# For more details and possible options please see the table at:
#  - <https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner>

persistence:
  # Enables persistence of config values
  # Including the provisioner ID
  #  -> Crucial so that the provisioner recognise itself after restarting
  enabled: true
  # Note that if storageClass is not defined,
  # Then this parameters defaults to the default storage class in the cluster,
  # which, as of today, in GCP, uses kubernetes.io/gce-pd provisioner
  # Careful: Don't use EmptyDir, as it will not survive the pod's death
  #  -> Config will therefore be lost anyway
  storageClass: do-block-storage
  size: "20Gi"

storageClass:
  # Name of the storage class that will be managed by the provisioner
  name: nfs
```

```
helm install nfs-provisioner stable/nfs-server-provisioner -f values.yaml
```

## Storage Class

Observe a new one

```
kubectl get sc
```

## Create a deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        resources: {}
        volumeMounts:
        - mountPath: /data
          name: data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: nfs-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-data
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs
```

As output

```
deployment.apps/web created
persistentvolumeclaim/nfs-data created
```

## Write into

```
kubectl exec -it web-74dff7fbcc-lfb58 -- touch /data/foo
```

## Scale

```
kubectl scale deployment web --replicas=10
```

```
kubectl get po -o wide

NAME                                       READY   STATUS    RESTARTS   AGE
nfs-provisioner-nfs-server-provisioner-0   1/1     Running   0          9m28s
web-74dff7fbcc-428ms                       1/1     Running   0          24s
web-74dff7fbcc-6mnnr                       1/1     Running   0          24s
web-74dff7fbcc-6mwjk                       1/1     Running   0          46s
web-74dff7fbcc-87f7c                       1/1     Running   0          24s
web-74dff7fbcc-9l4hg                       1/1     Running   0          46s
web-74dff7fbcc-lfb58                       1/1     Running   0          3m48s
web-74dff7fbcc-lv4gz                       1/1     Running   0          24s
web-74dff7fbcc-p8dw5                       1/1     Running   0          24s
web-74dff7fbcc-vnx5q                       1/1     Running   0          24s
web-74dff7fbcc-zjckb                       1/1     Running   0          24s
```

# Validate the share directory

Choose pods on different ndoes
