# Gestion des secrets Kubernetes
Les secrets stockent et gèrent les données sensibles telles que les mots de passe, les jetons d'API et les certificats. Ces données sensibles peuvent être transmises aux conteneurs lors de l'exécution.<br>

- *type* - k8s prend en charge plusieurs types qui spécifient le type de données sensibles stockées. *Opaque* est la valeur par défaut et autorise des données arbitraires définies par l'utilisateur.<br>
- *data* - données de mappage clé-valeur. Les valeurs sont encodées en base64.
- Nous pouvons transmettre des données secrètes au conteneur d'un pod sous la forme de variables d'environnement qui seront visibles par le processus de conteneur lors de l'exécution.
- *env* - fournit des variables d'environnement au conteneur. Le *valueFrom.secretKeyRef* extraira les données du Secret.
- Nous pouvons également transmettre des données secrètes au conteneur d'un pod sous la forme d'un volume monté. Chaque clé de niveau supérieur dans les données secrètes apparaîtra au processus de conteneur sous la forme d'un fichier sur le système de fichiers.
- volumes - Définit un volume k8s qui fait référence au Secret avec *secret.secretName*.
- *containers[].volumeMounts* - Monte le volume à un emplacement spécifique dans le conteneur.

Si nous avons besoin de récupérer des données secrètes à partir de la ligne de commande, vous pouvez le faire :
• Utilisons *kubectl get -o yaml \<secret\>* pour obtenir les données secrètes encodées en base64
• Transmettez les données encodées en base64 à *base64 --decode*.

## Créeons un secret
Obtenons une chaîne encodée en Base64 pour un nom d'utilisateur et un mot de passe.
```
echo admin | base64
echo cantfindme | base64
```

```
YWRtaW4K
Y2FudGZpbmRtZQo=
```

Créeons notre secret
```
vi my-secret.yml
```

```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4K
  password: Y2FudGZpbmRtZQo=
```

```
kubectl create -f my-secret.yml
```

## Transmettre les données secrètes à un conteneur à l'aide de variables d'environnement

Créeons un pod qui charge des données secrètes dans un conteneur à l'aide de variables d'environnement.
```
vi pod-env-secret.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-env-secret
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'echo "The credentials are $USERNAME:$PASSWORD"']
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

```
kubectl create -f pod-env-secret.yml
```

Vérifions la sortie du conteneur.
```
kubectl logs pod-env-secret
```

## Transmettre les données secrètes à un conteneur à l'aide d'un volume
Créeons un pod qui charge des données secrètes dans un conteneur à l'aide d'un volume.

```
vi pod-vol-secret.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-vol-secret
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'cat /etc/credentials/username; echo " "; cat /etc/credentials/password; sleep 3600']
    volumeMounts:
    - name: credentials
      mountPath: "/etc/credentials"
      readOnly: true
  volumes:
  - name: credentials
    secret:
      secretName: my-secret
```

```
kubectl create -f pod-vol-secret.yml
```

Explorons les ﬁchiers à l'intérieur du conteneur.
```
kubectl exec pod-vol-secret -- ls /etc/credentials
kubectl exec pod-vol-secret -- cat /etc/credentials/username
kubectl exec pod-vol-secret -- cat /etc/credentials/password
```

## Récupérer des données à partir d'un secret existant
Pour obtenir des données à partir d'un secret existant, obtenons les données brutes encodées en Base64.
```
kubectl get secret my-secret -o yaml
```

Copions la valeur encodée en Base64 et décodons-la.
```
echo Y2FudGZpbmRtZQo= | base64 --decode
```