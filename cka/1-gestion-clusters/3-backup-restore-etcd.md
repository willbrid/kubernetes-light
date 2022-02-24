# Sauvegarde et restauration des données du cluster Etcd

## Sauvegarde des données du cluster etcd
Si ce n'est pas encore fait alors vous devez installer le package etcd-client.
```
sudo apt install etcd-client
```

**- créer un dossier etcd-certs et copier les certificats de l'etcd**
```
cd $HOME

mkdir etcd-certs

sudo cp /etc/kubernetes/pki/etcd/ca.crt /etc/kubernetes/pki/etcd/server.crt /etc/kubernetes/pki/etcd/server.key etcd-certs/

sudo chown -R $(id -u):$(id -g) etcd-certs/
```

**- sauvegarder les données de l'etcd**
```
ETCDCTL_API=3 etcdctl snapshot save $HOME/etcd_backup.db --endpoints=https://192.168.43.251:2379 --cacert=$HOME/etcd-certs/ca.crt --cert=$HOME/etcd-certs/server.crt --key=$HOME/etcd-certs/server.key
```

## Restauration des données du cluster etcd
```
sudo ETCDCTL_API=3 etcdctl snapshot restore $HOME/etcd_backup.db \
--initial-cluster etcd-restore=https://192.168.43.251:2380 \
--initial-advertise-peer-urls https://192.168.43.251:2380 \
--name etcd-restore \
--data-dir /var/lib/etcd
```

NB: L'adresse **192.168.43.251** est celle du noeud master.