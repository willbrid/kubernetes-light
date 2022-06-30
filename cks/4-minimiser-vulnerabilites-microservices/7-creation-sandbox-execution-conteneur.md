# Création d'une sandboxe d'exécution de conteneur
- Installer gVisor Runtime : la première étape consiste à installer l'environnement d'exécution du conteneur qui exécutera les charges de travail en sandboxe.
- Configurer le conteneur : ensuite, nous devons configurer *containerd* pour pouvoir interagir avec *runsc*.
- Créer une *RuntimeClass* : nous devons configurer une *RuntimeClass*, qui nous permettra de désigner les pods qui doivent utiliser le runtime en sandboxe.

## Installer et configurer gVisor
Sur les trois nœuds (plan de contrôle et nœuds worker), installons gVisor.
```
curl -fsSL https://gvisor.dev/archive.key | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64,arm64] https://storage.googleapis.com/gvisor/releases release main"
```

```
sudo apt-get update && sudo apt-get install -y runsc
```

Modifions la configuration du *containerd* et ajoutons la configuration pour *runsc*.
```
sudo vi /etc/containerd/config.toml
```

Si le repertoire */etc/containerd/* n'existe pas, alors nous pouvons faire :
```
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Trouvons la section *disabled_plugins* et ajoutons le plugin de redémarrage.
```
disabled_plugins = ["io.containerd.internal.v1.restart"]
```

Trouvons le bloc *plugins."io.containerd.grpc.v1.cri".containerd.runtimes* . Après le bloc *runc* existant, ajoutons une configuration pour un environnement d'exécution *runsc*. Cela devrait ressembler à ceci une fois terminé :
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v1"
    runtime_engine = ""
    runtime_root = ""
    privileged_without_host_devices = false
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
    runtime_type = "io.containerd.runsc.v1"
```

Localisons le bloc *plugins."io.containerd.runtime.v1.linux"* et définissons *shim_debug* sur *true* .
```
[plugins."io.containerd.runtime.v1.linux"]
  ...
  shim_debug = true
```

Redémarrons *containerd* et vérifiez qu'il fonctionne toujours.
```
sudo systemctl restart containerd
sudo systemctl status containerd
```

## Créer une RuntimeClass
Sur le serveur du plan de contrôle uniquement, créeons une nouvelle *RuntimeClass*.
```
vi runsc-sandbox.yml
```

```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: runsc-sandbox
handler: runsc
```

```
kubectl create -f runsc-sandbox.yml
```

*name* est le nom utilisé pour appliquer la RuntimeClass aux pods.<br>
*handler* fait référence à une section du fichier de configuration *containerd* qui configure *runsc* .

## Tester la sandboxe d'exécution
Créeons un pod qui n'utilise pas la sandboxe.
```
vi non-sandbox-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: non-sandbox-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Running..."; sleep 5; done']
```

```
kubectl create -f non-sandbox-pod.yml
```

Créeons un pod qui utilise la sandboxe.
```
vi sandbox-pod.yml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: sandbox-pod
spec:
  runtimeClassName: runsc-sandbox
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Running..."; sleep 5; done']
```

```
kubectl create -f sandbox-pod.yml
```

Vérifions la sortie *dmesg* des pods.
```
kubectl exec non-sandbox-pod -- dmesg
kubectl exec sandbox-pod -- dmesg
```

Nous verrons une grande différence dans la sortie de ces deux conteneurs, puisque le conteneur en sandbox s'exécute dans un noyau *gVisor* simulé.