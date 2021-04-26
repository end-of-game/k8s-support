## Bien configurer ses livrables

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Se placer dans un répertoire

```
mkdir config-demo
cd config-demo
```

### Telecharger deux fichiers

```
wget https://k8s.io/examples/configmap/game.properties
wget https://k8s.io/examples/configmap/ui.properties
```

Puis créer la configuration
```
kubectl create configmap game-config --from-file=.
```

### Lister les configurations

```
kubectl describe configmaps game-config

Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
```

### Afficher toutes les informations

```
kubectl get configmaps game-config -o yaml
apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: 2018-06-18T19:59:18Z
  name: game-config
  namespace: default
  resourceVersion: "770631"
  selfLink: /api/v1/namespaces/default/configmaps/game-config
  uid: 158b2f24-7332-11e8-9d8f-7a2fde66199e
```


