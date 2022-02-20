# Gérer les RBAC (Role-Based Access Control) k8s
Vous travaillez pour **MiaMiam**, une entreprise qui fournit des livraisons régulières de repas aux clients. La société est en train de construire une infrastructure basée sur Kubernetes pour certains de ses logiciels.

Vos développeurs vous demandent fréquemment de fournir des informations à partir du cluster Kubernetes. Vous souhaitez donc leur donner la possibilité de lire les données du cluster sans y apporter de modifications. À l'aide du contrôle d'accès basé sur les rôles de Kubernetes, assurez-vous que l'utilisateur **dev** peut lire les métadonnées du pod et les journaux de conteneur à partir de n'importe quel pod dans l'espace de nom **miamiam**.

**- Créer un rôle pour l'utilisateur dev**<br>
Nous créeons un rôle appelé **role-reader**. Nous lui fournissons un accès en lecture aux pods et aux journaux de conteneurs dans l'espace de noms **miamiam**.

Nous définissons un fichier role-reader.yml
```
vi role-reader.yml
```

où nous insérons le contenu :
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: miamiam  
  name: role-reader
rules:
- apiGroups: [""]  
  resources: ["pods", "pods/log"]  
  verbs: ["get", "watch", "list"]
```

puis nous exécutons la commande de création de **role** :
```
kubectl create -f role-reader.yml
```

**- Lier le rôle à l'utilisateur dev**<br>
Nous créons un objet **RoleBinding** pour lier le rôle **role-reader** à l'utilisateur de **dev**.<br>

Nous définissons un fichier rolebinding-reader.yml
```
vi rolebinding-reader.yml
```

où nous insérons le contenu :
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:  
  name: rolebinding-reader  
  namespace: miamiam
subjects:
- kind: User  
  name: dev  
  apiGroup: rbac.authorization.k8s.io
roleRef:  		
  kind: Role  
  name: role-reader  
  apiGroup: rbac.authorization.k8s.io
```

puis nous exécutons la commande de création de l'objet **rolebinding** :
```
kubectl create -f rolebinding-reader.yml
```

**- vérification avec le kubeconfig de l'utilisateur dev**<br>
Pour vérifier si le RBAC a été bien configuré, nous testons avec la commande:
```
kubectl get pods -n miamiam --kubeconfig dev-k8s-config
```

où l'option **kubeconfig** permet de renseigner le fichier config de k8s pour l'utilisateur dev. Ici cette commande présente simplement une manière de tester notre RBAC.