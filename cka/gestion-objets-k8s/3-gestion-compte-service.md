# Gérer les comptes de services

Dans cette tutoriel, nous allons créer un compte de service et utiliser un rolebinding pour relier ce compte de service à un role.

**- créer un simple compte de service**<br>
Nous définissons un fichier payment-serviceaccount.yml
```
vi payment-serviceaccount.yml
```
où nous insérons le contenu :
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payment-serviceaccount
```

puis nous exécutons la commande de création de ce compte de service :
```
kubectl create -f payment-serviceaccount.yml
```

L'on peut aussi créer un compte de service via la méthode impérative :
```
kubectl create sa payment-serviceaccount -n default
```

L'on peut afficher le détail de ce compte de service :
```
kubectl describe sa payment-serviceaccount
```

L'on peut lister les comptes de service dans un espace de nom :
```
kubectl get sa -n default
```

NB: **sa** est le dimunitif de l'objet **serviceAccount**.<br>

**- attacher un role à un compte de service**<br>
Nous utiliserons le role **role-reader** définit dans le tutoriel précédent.
Nous définissons un fichier sa-rolebinding-reader.yml
```
vi sa-rolebinding-reader.yml
```
où nous insérons le contenu :
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-rolebinding-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: payment-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: role-reader
  apiGroup: rbac.authorization.k8s.io
```

puis nous exécutons la commande de création de cet objet **rolebinding** :
```
kubectl create -f sa-rolebinding-reader.yml
```