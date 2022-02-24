# Supervision de la santé d'un pod sur k8s
Dans ce tutoriel, nous présenterons la supervision de la santé d'un pod.<br>

**- supervision de la santé d'un pod en éxécution**<br>
Nous pouvons superviser la santé d'un pod en éxécution via l'option *livenessProbe*. 
- Cette supervision peut se baser sur une commande éxécutée au niveau du conteneur de pod. Nous créeons un pod avec l'option *livenessProbe* basée sur une commande :
```
vi liveness-cmd-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-cmd-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    livenessProbe:
      exec:
        command: ["echo", "Hello, world!"]
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
kubectl create -f liveness-cmd-pod.yml
```

L'option *initialDelaySeconds* de *livenessProbe* permet de définir le délai d'attente initiale avant l'éxécution de la commande pour la première fois.<br>
L'option *periodSeconds* de *livenessProbe* permet de définir l'intervalle de temps régulier d'éxécution de la commande.

- Cette supervision peut se baser sur une requête http éxécutée au niveau du conteneur de pod. Nous créeons un pod avec l'option *livenessProbe* basée sur une requête http :
```
vi liveness-http-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
kubectl create -f liveness-http-pod.yml
```

L'option *httpGet* de *livenessProbe* permet de définir l'uri (option *path*) et le port (option *port*) définissant le endpoint sur lequel sera exécutée une requête http GET.<br>

Par défaut, K8s ne considérera qu'un conteneur comme étant *hors service* si le processus du conteneur s'arrête. C'est pourquoi L'option *livenessProbe* permet de déterminer automatiquement si une application de conteneur est dans un état sain ou non.

**- supervision de la santé d'un pod à sa première éxécution**<br>
Nous créeons un pod avec l'option *startupProbe* basée sur une requête http :
```
vi startup-http-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: startup-http-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
```

```
kubectl create -f startup-http-pod.yml
```

L'option *failureThreshold* de *startupProbe* permet de définir le délai d'attente d'échec d'éxécution du pod. La requête du *startupProbe* arrête de séxécuter une fois que le conteneur a été demarré avec succès. <br>
L'option *startupProbe* est particulièrement utile pour les applications qui peuvent avoir de longs temps de démarrage.

**- supervision de la santé d'un pod après sa première éxécution**<br>
Nous créeons un pod avec l'option *readinessProbe* basée sur une requête http :
```
vi readiness-http-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: readiness-http-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

```
kubectl create -f readiness-http-pod.yml
```

L'on utilise l'option *readinessProbe* principalement pour empêcher le trafic utilisateur d'être envoyé vers des pods qui sont encore en cours de démarrage.