## Se protéger avec les secrets 

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Creer son premier secret

Nous allons écrire notre secret dans un fichier texte.
       
```
echo -n "Radiohead - Karma Police" > ./chanson.txt
```

Ensuite nous allons créer un secret sur base de ce fichier (1 Mo Max pour rappel)
```
kubectl create secret generic chanson --from-file=./chanson.txt
secret "chanson" created
```

Nous pouvons afficher les informations sur le secret

```
kubectl describe secrets/chanson
Name:         chanson
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
chanson.txt:  24 bytes
```

### Utiliser le secret dans un Pod

Nous n'allons pas créer de container mais bien un pod dans l'environnement K8S

Créer le fichier pod.yml avec le contenu suivant :

```
apiVersion: v1
kind: Pod
metadata:
  name: melomane
spec:
  containers:
  - name: shell
    image: centos:7
    command:
      - "bin/bash"
      - "-c"
      - "sleep 10000"
    volumeMounts:
      - name: chansonvol
        mountPath: "/tmp/machansonpreferee"
        readOnly: true
  volumes:
  - name: chansonvol
    secret:
      secretName: chanson
```

Lancer le pod :
```
kubectl create -f pod.yml
```

Puis se connecter au container :
```
kubectl exec melomane -c shell -i -t -- bash
```

Lancer une commande Shell dans le container pour accèder au secret:

```
kubectl exec melomane -c shell -i -t -- bash

# mount | grep chanson
tmpfs on /tmp/machansonpreferee type tmpfs (ro,relatime,seclabel)

cat /tmp/machansonpreferee/chanson.txt
Radiohead - Karma Police

```




