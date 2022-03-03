# Installation d'un cluster k8s version 1.22 sur ubuntu server 18.04

Dans cette section nous mettrons en place un cluster k8s à 3 noeuds avec containerd.

**Hypothèse**: On supposera que nous avons 3 machines d'adresse IP privée: 192.168.43.251, 192.168.43.252, 192.168.43.253. Sur chacune d'elle est installé le système ubuntu server 18.04. <br>
Notre noeud master ici sera nommé : k8s-control (192.168.43.251) et les deux noeuds worker seront nommés k8s-worker1 (192.168.43.252), k8s-worker2 (192.168.43.253). <br>

**NB**: Vous pouvez remplacer les adresses IP privées par les votres et la procédure d'installation restera toujours la même. <br>

## Configuration du hostname et du fichier hosts
**Sur le nœud master (k8s-control : 192.168.43.251)**
```
sudo hostnamectl set-hostname k8s-control
```

Cette commande permettra de configurer le hostname k8s-control sur le noeud master.

**Sur le nœud worker 1 (k8s-worker1 : 192.168.43.252)**
```
sudo hostnamectl set-hostname k8s-worker1
```

Cette commande permettra de configurer le hostname k8s-worker1 sur le noeud worker 1 .

**Sur le nœud worker 2 (k8s-worker2 : 192.168.43.253)**
```
sudo hostnamectl set-hostname k8s-worker2
```

Cette commande permettra de configurer le hostname k8s-worker2 sur le noeud worker 2 .

NB: Déconnectez-vous des trois serveurs et reconnectez-vous pour voir ces modifications prendre effet.

**Configuration du fichier hosts**<br>
Sur tous les nœuds, configurez le fichier hosts pour permettre à tous les nœuds de se joindre à l'aide de ces noms d'hôte.
```
sudo vi /etc/hosts
```

Sur tous les nœuds, ajoutez ce qui suit à la fin du fichier. Vous devrez fournir l'adresse IP privée pour chaque nœud.
```
192.168.43.251 k8s-control
192.168.43.252 k8s-worker1
192.168.43.253 k8s-worker2
```

NB: Déconnectez-vous des trois serveurs et reconnectez-vous pour voir ces modifications prendre effet.

## Configuration du containerd
**Configuration des modules kernel : overlay et br_netfilter pour containerd**
```
cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

**Activation du routage des paquets et permission de la communication par pont entre conteneurs**
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

**Installation et configuration de containerd**
```
sudo apt-get update && sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

## Désactivation du Swap
Sur tous les noeuds, il faudrait désactiver le swap.
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Installation des packages kubeadm, kubelet et kubectl
Sur tous les noeuds, il faudrait installer les packages kubeadm, kubelet, et kubectl.
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet=1.22.0-00 kubeadm=1.22.0-00 kubectl=1.22.0-00
sudo apt-mark hold kubelet kubeadm kubectl
```

NB: La dernière commande avec **apt-mark** permet de désactiver la mise à jour des packages kubelet kubeadm et kubectl.

## Initialisation du cluster
Sur le nœud master (k8s-control : 192.168.43.251) uniquement, initialisez le cluster et configurez l'accès kubectl.
```
sudo kubeadm init --pod-network-cidr 172.16.0.0/16 --kubernetes-version 1.22.0
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

NB: Ici **172.16.0.0/16** sera la plage du réseau privé de notre cluster. Libre à vous de choisir une plage à votre convenance. <br>

Vous pouvez vérifier si le cluster fonctionne.
```
kubectl get nodes
```

## Installation du module complémentaire réseau Calico.
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

## Ajout des noeuds worker au cluster
**Obtenir la commande de jointure**
```
kubeadm token create --print-join-command
```

NB: cette commande est également affichée à la fin de l'exécution de la commande **kubeadm init**.

**Jointure proprement dit** <br>
Copiez la commande join (résultat de la commande ci-dessus) à partir du nœud master. Exécutez-le sur chaque nœud worker en tant que root (c'est-à-dire avec sudo ).
```
sudo kubeadm join ...
```

Sur le nœud master, vérifiez que tous les nœuds de votre cluster sont prêts. Notez que cela peut prendre quelques instants pour que tous les nœuds passent à l'état **PRÊT** (**READY**).
```
kubectl get nodes
```