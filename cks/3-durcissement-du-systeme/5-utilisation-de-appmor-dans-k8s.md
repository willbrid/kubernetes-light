# Utilisation d'AppArmor dans les conteneurs k8s
Un profil AppArmor peut être stocké dans un fichier et appliqué à l'aide de la commande *apparmor_parser*.

```
sudo apparmor_parser /path/to/file
```

Par défaut, la commande chargera le profil en mode d'application. Utilisons l'option -C pour le mode réclamation.

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  annotations:
    container.apparmor.security.beta.kubernetes.io/nginx: localhost/k8s-deny-write
spec:
  containers:
  - name: nginx
    image: nginx
```

Utilisons des annotations pour appliquer des profils à des conteneurs spécifiques.<br>

Le nom de l'annotation doit commencer par *container.apparmor.security.beta.kubernetes.io/*, suivi du nom du conteneur.<br>

La valeur d'annotation doit être *localhost/*, suivie du nom du profil AppArmor.

- Créeons et appliquons un profil AppArmor sur le serveur du plan de contrôle<br>

Par exemple nous créeons un profil AppArmor qui empêchera le processus d'écrire des données sur le disque.
```
sudo vi /etc/apparmor.d/deny-write
```

```
#include <tunables/global>
profile deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```

Appliquons le profil.
```
sudo apparmor_parser /etc/apparmor.d/deny-write
```

- Appliquons le profil AppArmor à un conteneur sur le serveur du plan de contrôle.<br>

Créeons un pod qui écrit des données sur le disque.
```
vi apparmor-disk-write-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: apparmor-disk-write
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo "I write to the disk!" > diskwrite.log; sleep 5; done']
```

```
kubectl create -f apparmor-disk-write-pod.yml --save-config
```

Vérifions les journaux du conteneur. Vérifions également le fichier *diskwrite.log* à l'intérieur du conteneur. Vous devriez voir que le conteneur est capable d'écrire sur le disque puisque le profil AppArmor n'est pas appliqué au conteneur.
```
kubectl logs apparmor-disk-write
kubectl exec apparmor-disk-write -- cat diskwrite.log
```

Modifions le manifeste du pod et ajoutons une annotation pour appliquer le profil AppArmor au conteneur.
```
vi apparmor-disk-write-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    container.apparmor.security.beta.kubernetes.io/busybox: localhost/deny-write
  ...
```

Supprimons et recréeons le pod
```
kubectl delete pod apparmor-disk-write --force
kubectl create -f apparmor-disk-write-pod.yml
```

- Créeons et appliquons le profil AppArmor sur les nœuds worker<br>

Sur les deux nœuds worker, créeons et appliquons le profil AppArmor.
```
sudo vi /etc/apparmor.d/deny-write
```

```
#include <tunables/global>
profile deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```

Appliquons le profil.
```
sudo apparmor_parser /etc/apparmor.d/deny-write
```

- Testons notre configuration finale sur le nœud du plan de contrôle

Vérifions à nouveau les journaux du conteneur et le fichier *diskwrite.log*. Le conteneur devrait maintenant être incapable d'écrire sur le disque.
```
kubectl logs apparmor-disk-write
kubectl exec apparmor-disk-write -- cat diskwrite.log
```