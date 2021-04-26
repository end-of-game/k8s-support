## Create a directory

```
mkdir -p volumes/02-exercises
cd volumes/02-exercises
```

## Créer des volumes et les monter dans des pods

### Creer un Persistent Volume

Fichier pv.yaml.tmpl

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv-$TRIGRAMME
  labels:
    type: local
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow-$TRIGRAMME
  hostPath:
    path: "/home/$TRIGRAMME/tmp"
```

Lancer les commandes:
```
TRIGRAMME=nmu envsubst < pv.yaml.tmpl > pv.yaml
kubectl apply -f pv.yaml
kubectl get pv
```

### Créer un Persistent Volume Claim

Fichier pvc.yaml.tmpl

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: sLow-$TRIGRAMME
```


```
TRIGRAMME=nmu envsubst < pvc.yaml.tmpl > pvc.yaml
kubectl apply -f pvc.yaml
kubectl get pvc
```

Corriger l'erreur.
Comment la trouver ?

### Créer un second Persistent Volume Claim

Fichier pvc2.yaml.tmpl

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-pvc2
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: sLow-$TRIGRAMME
```


```
TRIGRAMME=nmu envsubst < pvc2.yaml.tmpl > pvc2.yaml
kubectl apply -f pvc2.yaml
kubectl get pvc
```

Quelle est l'erreur ?

### Créer un Déploiment

Créer le fichier deployment.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
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
          claimName: my-pvc
```

`kubectl apply -f deployment.yml`

Listez les pods:

`kubectl get pods -o wide`

```
NAME                          READY     STATUS    RESTARTS   AGE
deployment-59ccfff984-4h4pd   1/1       Running   0          10m
```

Lancez un shell dans un des pods précédemment créé:

`kubectl exec -it deployment-59ccfff984-4h4pd -- sh`

Créez un fichier dans le volume monter:

`touch /data/test`

Question : qui a-t-il de choquant ?

`dd if=/dev/zero of=/data/6G.img bs=1 count=0 seek=6G`

Question : qui a-t-il de choquant ?

Lancez un shell dans un autre pod:

`kubectl exec -it deployment-59ccfff984-5zjg7 -- sh`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

### Créer un deuxième Déploiement

Créer le fichier deployment2.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment2
spec:
  replicas: 5
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
          image: alpine
          command: ["watch"]
          args: ["echo", "hello"]
          volumeMounts:
          - name: vol
            mountPath: "/data"
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: my-pvc
```

`kubectl apply -f deployment2.yml`

Listez les pods:

`kubectl get pods -o wide`

Lancez un shell dans un des pods du deuxième déploiement du noeud initial
Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

Lancez un shell dans un des pods du deuxième déploiement **d'un autre noeud** 
que le noeud sur lequel le pod du premier déploiement se trouve.

`kubectl exec -it deployment.apps/deployment2 -- bash`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

Créez un fichier dans le volume monté:

`touch /data/test2`

Lancez un shell dans un autre pod du deuxième déploiement:

`kubectl exec -it deployment2-59ccfff984-x6vvv -- sh`

Affichez le contenu du volume partagé entre les pods:

`ls /data/`

Que voyez vous ? Pourquoi ?

## Solution

https://kubernetes.io/fr/docs/concepts/storage/persistent-volumes/

