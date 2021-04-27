## CertificateSigningRequest

### Nettoyer l'environnement précédent

`kubectl delete daemonsets,replicasets,services,deployments,pods,rc --all`

### Générer une clé et une demande de certificat

Deux possibilités :
- Cfssl
- Openssl

#### CFSSl

##### Installer cfssl

```
mkdir ~/bin
curl -s -L -o ~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
curl -s -L -o ~/bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x ~/bin/{cfssl,cfssljson}
export PATH=$PATH:~/bin
```

##### Créer une clé et une demande de certificat

```
echo '{"CN":"treeptik.student","hosts":[""],"key":{"algo":"rsa","size":2048}}' | cfssl genkey  - | cfssljson -bare treeptik.student
```

#### Openssl

##### Générez la clé privée du nouvel utilisateur

```
openssl genrsa -out treeptik.student-key.pem 2048
```

##### Créez une demande de certificat

```
openssl req -new -key treeptik.student-key.pem -out treeptik.student.csr \
-subj "/CN=treeptik.student/O=treeptik"
```

### Créer la demande de signature du certificat

Créer un fichier certsingnrequest.yaml
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: user-request-treeptik-student
spec:
  groups:
  - system:authenticated
  request: <key>
  usages:
  - digital signature
  - key encipherment
  - client auth
```

Remplacer la valeur du champs request par le résultat de la commande :
```
cat treeptik.student.csr | base64 | tr -d '\n'
```

Créer la ressource :
```
kubectl create -f certsingnrequest.yaml
```

### Valider le certificat

```
kubectl certificate approve user-request-treeptik-student
```

### Créer un namespace

```
kubectl create namespace treeptik-namespace
```

### Créer un role

Créer un fichier role.yaml :
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: treeptik-namespace
  name: pod-reader
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Créer la ressource :
```
kubectl create -f role.yaml
```

### Créer un rolebinding

Créer un fichier rolebinding.yaml :
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pod
  namespace: treeptik-namespace # Nom du namespace
subjects:
- kind: User
  name: treeptik.student
  apiGroup: ""
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: ""
```

Créer la ressource :
```
kubectl create -f rolebinding.yaml
```

### Récupérer le certificat du nouvel utilisateur

Sous Linux
```
kubectl get csr user-request-treeptik-student -o jsonpath='{.status.certificate}' | base64 -d > treeptik.student.pem
```

Sous MacOS
```
kubectl get csr user-request-treeptik-student -o jsonpath='{.status.certificate}' | base64 -D > treeptik.student.pem
```


### Créer l'utilisateur

```
kubectl config set-credentials treeptik.student --embed-certs=true --client-certificate=treeptik.student.pem --client-key=treeptik.student-key.pem
```

### Créer le contexte de travail

Récupérer le nom du cluster :
```
kubectl config get-clusters
```

Créer le contexte :
```
kubectl config set-context treeptik-context \
                --cluster=<nom du cluster> \
                --user=treeptik.student --namespace=treeptik-namespace
```

### Tester le nouvel utilisateur

Changer de contexte :
```
kubectl config use-context treeptik-context
```

Tester si l'utilisateur peut travailler dans le namespace default ( la réponse devrait être non) :
```
kubectl auth can-i get deployments --namespace default
```

Tester si l'utilisateur peut travailler dans le namespace treeptik-namespace ( la réponse devrait être oui) :
```
kubectl auth can-i get deployments --namespace treeptik-namespace
```

### Pour créer un fichier kubeconfig personalisé

```
kubectl config set-credentials treeptik.student --kubeconfig treeptik-student-config \
                    --embed-certs=true \
                    --client-certificate=treeptik.student.pem \
                    --client-key=treeptik.student-key.pem
```

```
kubectl config set-context  --kubeconfig treeptik-student-config treeptik-context \
                            --cluster=<nom du cluster> \
                            --user=treeptik.student \
                            --namespace=treeptik-namespace
```

```
kubectl config use-context --kubeconfig treeptik-student-config treeptik-context
```

```
kubectl config set-cluster  --kubeconfig treeptik-student-config <nom du cluster> \
                            --insecure-skip-tls-verify=true \
                            --server=<adresse du serveur>
```

Le fichier kubeconfig est : "treeptik-student-config" pour l'utiliser:
```
export KUBECONFIG=<chemin du fichier>/treeptik-student-config
```

## AKS

Dans le cas de AKS, il faut aller chercher la valeur de l'adresse dans le fichier `~/.kube/config`
Il faut garder **https://** ainsi que le port **443**

*Exemple*
```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://xxx-b3fa9656.hcp.westeurope.azmk8s.io:443
  name: xxx
...
...
```

## Retour en mode ADMINISTRATEUR pour le TP suivant

```
kubectl config use-context kubernetes-admin@cluster.local
kubectl delete ns treeptik-namespace
```
