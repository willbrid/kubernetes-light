# Installation d'un cluster k8s version 1.22 sur rocky linux 8

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

**Configuration du fichier hosts**
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

## Configuration de base sur tous les noeuds
**Désactivation de selinux**
```
sudo setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

**Configuration des paramètres d'activation de la communication au sein du cluster**
```
sudo modprobe br_netfilter
sudo sh -c "echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables"
sudo sh -c "echo '1' > /proc/sys/net/bridge/bridge-nf-call-ip6tables"
sudo sh -c "echo '1' > /proc/sys/net/ipv4/ip_forward"
```

**Désactivation du swap**
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## Installation et configuration de docker sur tous les noeuds
**Suppression de podman et ses dépendances**

```
sudo dnf -y remove podman runc
```

**Installation de yum-utils**

```
sudo dnf install -y yum-utils
```

**Installation du repo docker**

```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

**Vérification de la présence du repo docker**

```
sudo dnf repolist | grep docker
```

**Installation des packages, démarrage et activation du service docker**

```
sudo dnf -y install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

**Configurez le pilote docker cgroup pour systemd**

```
sudo sed -i '/^ExecStart/ s/$/ --exec-opt native.cgroupdriver=systemd/' /usr/lib/systemd/system/docker.service
sudo systemctl daemon-reload
sudo systemctl restart docker
```

L'on peur vérifier que systemd est désormais le cgroupdriver de docker
```
docker info | grep -i cgroup
```

**Ajout de l'utilisateur courant dans le groupe docker**

```
sudo usermod -a -G docker <user>
```

L'utilisateur user aura accès à Docker après sa prochaine connexion.

# Configuration du parefeu firewall-cmd pour autoriser les ports de k8s
**Sur le noeud master**
```
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=10251/tcp
sudo firewall-cmd --permanent --add-port=10252/tcp
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=@ip-worker/32 accept'
sudo firewall-cmd --reload
```

L'on remplacera *@ip-worker* dans la commande d'option *--add-rich-rule* par chacune des adresses ip des workers. Donc cette commande devrait être éxécutée deux fois pour chacun de nos deux workers : *192.168.43.252* et *192.168.43.253* . Elle permet d'autoriser l'accès docker à partir d'un autre nœud.

**Sur les noeuds worker**
```
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=30000-32767/tcp
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --reload
```

Cette configuration de parefeu est basée sur la liste de ports par défaut des composants de k8s dont vous pouvez consulter via ce lien : [k8s-ports-and-protocols](https://kubernetes.io/docs/reference/ports-and-protocols/) .

# Installation du cluster k8s
**Ajout du repo k8s sur tous les noeuds**
```
sudo /etc/yum.repos.d/kubernetes.repo
```

```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
```

La dernière ligne avec le mot *exclude* permet d'empêcher la mise à jour des packages *kubelet*, *kubeadm* et *kubectl*.<br>
L'on met à jour la liste des packages :
```
sudo dnf upgrade -y
```

L'on pourra vérifier les versions disponibles pour *1.22* de chacun des packages *kubelet*, *kubeadm* et *kubectl* :
```
sudo dnf --showduplicates list kubelet --disableexcludes=kubernetes
sudo dnf --showduplicates list kubeadm --disableexcludes=kubernetes
sudo dnf --showduplicates list kubectl --disableexcludes=kubernetes
```

Dans notre cas, à partir des résultats de la commande ci-dessus, nous avons décidé d'installer la version 1.22.0-0 de chaque package.

**Installation des packages *kubelet*, *kubeadm* et *kubectl* sur tous les noeuds**
```
sudo dnf install -y kubelet-1.22.0 kubeadm-1.22.0 kubectl-1.22.0 --disableexcludes=kubernetes
```

**Démarrage et activation du kubelet sur tous les noeuds**
```
sudo systemctl enable kubelet
sudo systemctl start kubelet
```

**Initialisation du cluster sur le noeud master**<br>
Sur le nœud master (k8s-control : 192.168.43.251) uniquement, initialisez le cluster et configurez l'accès kubectl.
```
sudo kubeadm init --pod-network-cidr 172.16.0.0/16 --kubernetes-version 1.22.0
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

NB: Ici 172.16.0.0/16 sera la plage du réseau privé de notre cluster. Libre à vous de choisir une plage à votre convenance.<br>

Vous pouvez vérifier si le cluster fonctionne :
```
kubectl get nodes
```

**Installation du module complémentaire réseau Calico**

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

**Ajout des noeuds worker au cluster**
L'on peut obtenir la commande de jointure avec la commande :
```
kubeadm token create --print-join-command
```

Le résultat de cette commande est également affichée à la fin de l'exécution de la commande *kubeadm init*.<br>

Copiez la commande join (résultat de la commande ci-dessus) à partir du nœud master. Exécutez-le sur chaque nœud worker en tant que root (c'est-à-dire avec sudo ).

```
sudo kubeadm join ...
```

Sur le nœud master, vérifiez que tous les nœuds de votre cluster sont prêts. Notez que cela peut prendre quelques instants pour que tous les nœuds passent à l'état PRÊT (READY).

```
kubectl get nodes
```