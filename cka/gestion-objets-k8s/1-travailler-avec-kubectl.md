# Travailler avec kubectl sur les objets k8s
Cette section présentera quelques exemples avec les pods

**- créer un pod**
Supposons la définition yaml d'un pod nginx dans le fichier **pod.yml**
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
```

Pour créer ce pod, on exécute la commande :
```
kubectl apply -f pod.yml
```
ou
```
kubectl create -f pod.yml
```

L'option **create** permet de créer un pod (en général un objet k8s) mais si l'objet avait été déjà créé alors une sortie d'erreur sera affichée.
L'option **apply** permet aussi de créer un pod (en général un objet k8s) mais si l'objet avait été déjà créé alors il sera tout simplement mis à jour.

**- avoir la liste des pods**
```
kubectl get pods
```

Cette commande va lister l'ensemble des pods dans le namespace par défaut.

**- expérimenter le listing des pods avec différents formats de sortie**<br>
**--- afficher le listing avec plus de colonnes d'informations**
```
kubectl get pods -o wide
```

**--- afficher le listing sous format json**
```
kubectl get pods -o json
```

**--- afficher le listing sous format yaml**
```
kubectl get pods -o yaml
```

**--- trier le résultat du listing avec le format wide**
```
kubectl get pods -o wide --sort-by .spec.nodeName
```

Au préalable, pour mieux avoir le champ requis du tri, l'on peut afficher un pod sous format yaml.
```
kubectl get pods nginx-pod -o yaml

```

**--- filtrer le résultat par le label des pods**
```
kubectl get pods -n kube-system --selector k8s-app=calico-node
```

L'option **-n** permet de préciser le namespace. L'option **--selector** permet de préciser les labels.

**- afficher le détails d'un pod**
```
kubectl describe pod nginx-pod
```

**- exécuter une commande dans un conteneur de pod**
```
kubectl exec nginx-pod -c nginx-container -- echo "Hello, world!"
```
Après l'option **exec**, l'on précise le nom du pod.<br>
L'option **-c** permet de préciser un conteneur dans un pod.<br>
L'option **--** permet de préciser la commande à exécuter dans le conteneur. Si un pod contient un seul conteneur alors il serait facultatif de préciser ce conteneur dans la commande sinon il faudrait absolument préciser le conteneur où la commande sera exécutée.
 
**- supprimer un pod** 
```
kubectl delete pod nginx-pod
```