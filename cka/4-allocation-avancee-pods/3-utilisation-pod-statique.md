# Utilisation des pods statiques
Dans ce tutoriel, nous présenterons la création d'un pod statique qui est géré directement par le kubelet sur un nœud, et non par le serveur d'API K8s. Ils peuvent fonctionner même s'il n'y a pas de serveur API K8s présent.<br>
Kubelet crée automatiquement des pods statiques à partir de fichiers manifestes YAML situés dans le chemin du manifeste sur le nœud. Il créera un pod miroir pour chaque pod statique. Les pods miroirs vous permettent de voir l'état du pod statique via l'API K8s, mais vous ne pouvez pas les modifier ou les gérer via l'API.<br>

Nous nous connectons sur le noeud *k8s-worker1*, puis nous créeons un pod statique :
```
sudo vi /etc/kubernetes/manifests/test-static-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: test-static-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
```

Après quelques minutes, le pod statique sera créé. Si nous souhaiterons la création immédiate de ce pod statique, nous redemarrons le composant kubelet :
```
sudo systemctl restart kubelet
```

Pour vérifier la création d'un pod statique, nous pouvons lister les pods afin de vérifier si son pod miroir a été créé :
```
kubectl get pods
```

Les pods miroirs respecte une nomenclature précise : *nomPod-nomNoeud*.<br>
Si vous le souhaitez, vous pouvez tenter de supprimer le pod statique à l'aide de l'API k8s. Le Pod sera immédiatement recréé, puisqu'il ne s'agit que d'un Pod miroir créé par le kubelet worker pour représenter le Pod statique.
```
kubectl delete pod test-static-pod-k8s-worker1
```