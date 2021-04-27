## Service Accounts and Auditing in Kubernetes

ServiceAccounts et Namespaces vous permettent de limiter les pods et définir les permissions utilisateurs dans K8S.
Les logs d'audit donnent une idée sur les activités des comptes sur les ressources.

### Créer le Namespace

Créer le fichier **api-reader-dev-namespace.yaml**

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Puis créer la ressource

```
kubectl apply -f api-reader-dev-namespace.yaml
```

Puis changer le namespace par défaut sur le contexte courant.

```
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

### Créer un secret

Rappel: un secret est encodé sur base64.

Créer le fichier **api-reader-secret.yaml**

```
---
apiVersion: v1
kind: Secret
metadata:
  name: api-access-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

Puis créer le secret:

```
kubectl apply -f api-reader-secret.yaml
```

Vérifiez que le secret existe bien.

```
kubectl get secrets api-access-secret
```

### Créer le ServiceAccount

Vous pouvez attacher des comptes de service aux pods et les utiliser pour accéder à l'API Kubernetes. 
Si aucun compte de service n'est défini dans la définition du pod, celui-ci utilise le compte de service par défaut pour l'espace-noms.

Les fichiers nommés token, ca.crt et namespace sont automatiquement montés dans le répertoire /var/run/secrets/kubernetes.io/serviceaccount/ de chaque conteneur.
Leur contenu est basé sur le nom du compte de service que vous avez fourni.

Remarque: les secrets affichés dans le répertoire /var/run/secrets/kubernetes.io/serviceaccount/ sont des secrets spécifiques au compte de service montés par le système Kubernetes, et non le secret que vous avez créé. L'accès à ce secret n'indique pas que le pod peut accéder à d'autres secrets avec ce jeton.

Créer le fichier **api-reader-service-accounts.yaml**

```
---
# Service account for preventing API access
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-access-sa
---
# Service account for accessing secrets API
apiVersion: v1
kind: ServiceAccount
metadata:
  name: secret-access-sa
```

Puis créer les :

```
kubectl create -f api-reader-service-accounts.yaml
kubectl get serviceaccounts
```

### Créer les Cluster Roles

Un ClusterRole définit un ensemble d'autorisations utilisé pour accéder aux ressources, telles que les pods et les secrets. Les ClusterRole sont étendus au cluster. Les ClusterRole définis ici sont attachés aux comptes de service via une liaison de rôle dans les étapes suivantes.

L'utilisation d'un RoleBinding au lieu d'un ClusterRoleBinding étend les autorisations à un namespace.

Créer le fichier **api-reader-cluster-roles.yaml**

```
---
# A role with no access
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: no-access-cr
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: [""]
  verbs: [""]
---
# A role for reading/listing secrets
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: secret-access-cr
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```  

Puis créer les ressources

```
kubectl create -f api-reader-cluster-roles.yaml

## Afficher les ClusterRoles triés par ordre de création
kubectl get clusterroles --sort-by='{.metadata.creationTimestamp}'
```

### Créer les liaisons des rôles

Pour appliquer des rôles de cluster aux comptes de service, créez des liaisons de rôle qui les connectent. Lorsque vous liez un rôle de serveur à un compte de service, les autorisations que vous avez définies dans un rôle sont accordées au compte.

Créer le fichier **api-reader-role-bindings.yaml**

```
---
# The role binding to combine the no-access service account and role
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: no-access-rb
subjects:
- kind: ServiceAccount
  name: no-access-sa
roleRef:
  kind: ClusterRole
  name: no-access-cr
  apiGroup: rbac.authorization.k8s.io

---
# The role binding to combine the secret-access service account and role
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: secret-access-rb
subjects:
- kind: ServiceAccount
  name: secret-access-sa
roleRef:
  kind: ClusterRole
  name: secret-access-cr
  apiGroup: rbac.authorization.k8s.io
```

Créer les ressources

```
kubectl create -f api-reader-role-bindings.yaml
kubectl get rolebindings
```

### Créer l'image Docker

Le fichier **Dockerfile**

```
FROM ubuntu

# Copy files
COPY runtime.sh /
# Modify file permissions
RUN chmod +x runtime.sh


# Install curl
RUN apt-get update
RUN apt-get install curl -y

# Run script on startup
CMD [ "/runtime.sh" ]
```

Ainsi que le fichier pour la commande **runtime.sh**

```
#!/bin/bash

# Read mounted files
KUBE_TOKEN=$(</var/run/secrets/kubernetes.io/serviceaccount/token)
NAMESPACE=$(</var/run/secrets/kubernetes.io/serviceaccount/namespace)

# Resource to get in API [pods/secrets]
RESOURCE="secrets"

# If an argument was set
if [ "$#" -ge 1 ]; then
 NAMESPACE="$1"
fi

#Curl against the resource
echo curl -sSk -H "Authorization: Bearer $KUBE_TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT/api/v1/namespaces/$NAMESPACE/$RESOURCE

curl -sSk -H "Authorization: Bearer $KUBE_TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT/api/v1/namespaces/$NAMESPACE/$RESOURCE
echo ""

# Don't wait on user exec
if [ "$#" -lt 1 ]; then
 # Sleep forever so the pod doesn't crash
 while [ true ]; do
 sleep 10
 done
fi
```

Construisez les images Docker

```
docker build -t nmuller/training-sa .
```



### Créer les pods

Créer le fichier **api-reader-pods.yaml**

```
---
# Create a pod with the no-access service account
kind: Pod
apiVersion: v1
metadata:
 name: no-access-pod
spec:
 serviceAccountName: no-access-sa
 containers:
 - name: no-access-container
   image: nmuller/training-sa
---
# Create a pod with the secret-access service account
kind: Pod
apiVersion: v1
metadata:
 name: secret-access-pod
spec:
 serviceAccountName: secret-access-sa
 containers:
 - name: secret-access-container
   image: nmuller/training-sa
```

Créer les pods:
```
kubectl create -f api-reader-pods.yaml

kubectl get pods
NAME                READY     STATUS    RESTARTS   AGE
no-access-pod       1/1       Running   0          3m
secret-access-pod   1/1       Running   0          3m
```

### Etudier les logs des pods

```
kubectl logs no-access-pod
```

Vous devriez voir un message d'erreur comme quoi l'application cURL n'a pas le droit d'accèder à l'API K8S.
Par contre, si vous faites
```
kubectl logs secret-access-pod
```

Vous pouvez désormais voir le résultat autorisé à l'API K8S.
Vous pouvez même vous cible exactement le secret qui nous intéresse.

Faites un copier/coller de la ligne montrée dans le *echo*. Celle-ci doit ressembler à
```
curl -sSk -H "Authorization: Bearer xxxxxxx" https://10.233.0.1:443/api/v1/namespaces/dev/secrets/
```

Pour commencer, connectez vous dans le container
```
kubectl exec -it secret-access-pod bash
```

Puis ajouter à la ligne précédente la valeur **api-access-secret**

```
curl -sSk -H "Authorization: Bearer xxxxxxx" https://10.233.0.1:443/api/v1/namespaces/dev/secrets/api-access-secret
```

### Générer une erreur

```
kubectl exec -it secret-access-pod /runtime.sh default
```

Parce que le service account est lié au namespace DEV, il n'est pas autorisé à en sortir.

### Logs d'Audit

En fonction des installations, le système d'audit peut ne pas être disponible.
L'API **audit.k8s.io** est disponible depuis la 1.12 en v1beta et seulement en 1.13 en v1.

Les informations se trouvent ici par exemple :
```
/var/log/kubernetes/audit/kube-apiserver-audit.log
```

Pour savoir où se trouve un tel fichier, il faut se connecter dans le pod **kube-apiserver** puis faire un **ps | grep kube** pour avoir le détail du processus et ses options.

## Retour en mode ADMINISTRATEUR pour le TP suivant

```
kubectl config use-context kubernetes-admin@cluster.local
kubectl config set-context $(kubectl config current-context) --namespace=default
kubectl delete ns treeptik-namespace
```
