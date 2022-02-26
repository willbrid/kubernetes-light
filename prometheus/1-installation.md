# Installation de prometheus 2.33.4 sur Rocky linux 8
Nous supposons que nous avons une machine virtuelle virtual-box de 32Go de disque, 2Go de Ram, où est installé le système Rocky linux 8.
<br>

## Installation de prometheus 2.33.4
- Nous faisons la mise à jour des packages de notre système
```
sudo yum update -y
```

- Nous téléchargeons et faisons l'extraction de prometheus v2.33.4
```
curl -LO url -LO https://github.com/prometheus/prometheus/releases/download/v2.33.4/prometheus-2.33.4.linux-amd64.tar.gz

tar -xvf prometheus-2.33.4.linux-amd64.tar.gz
```

- Nous renommons le dossier prometheus-2.33.4.linux-amd64
```
mv prometheus-2.33.4.linux-amd64 prometheus-files
```

- Nous créeons un utilisateur Prometheus, les répertoires requis et faisons de Prometheus l'utilisateur en tant que propriétaire de ces répertoires
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

- Nous copieons le binaire prometheus et promtool du dossier prometheus-files vers /usr/local/bin et changeons la propriété en utilisateur prometheus
```
sudo cp prometheus-files/prometheus /usr/local/bin/
sudo cp prometheus-files/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

- Nous déplaçons les répertoires consoles et console_libraries de prometheus-files vers le dossier /etc/prometheus et changeons la propriété en utilisateur prometheus
```
sudo cp -r prometheus-files/consoles /etc/prometheus
sudo cp -r prometheus-files/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

- Nous créons le fichier de configuration de prometheus et changeons la propriété en utilisateur prometheus
```
sudo vi /etc/prometheus/prometheus.yml
```

```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

*targets* indique l'adresse et le port de prometheus.

```
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

- Nous configurons un service Prometheus
```
sudo vi /etc/systemd/system/prometheus.service
```

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --storage.tsdb.retention.time 2d \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

*storage.tsdb.path* indique le répertoire de stockage des données de prometheus. *config.file* indique son fichier de configuration. *storage.tsdb.retention.time* indique la durée maximale de sauvegarde des données.

- Nous vérifions l'état du service prometheus à l'aide de la commande suivante
```
sudo systemctl status prometheus
```

Pour accéder à l'interface web, nous devons d'abord autoriser le port 9090 au niveau du pare-feu
```
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload
```

Vous pourrez maintenant accéder à l'interface utilisateur prometheus sur le port 9090 du serveur prometheus.

## Installation de node-exporter sur le noeud de prometheus
-  Nous téléchargeons et faisons l'extraction de node-exporter v1.3.1
```
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz

tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
```

- Nous copieons le binaire node_exporter du dossier node_exporter-1.3.1.linux-amd64 vers /usr/local/bin
```
sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
```

- Nous créeons un utilisateur node_exporter pour exécuter le service d'exportation de nœud
```
sudo useradd -rs /bin/false node_exporter
```

- Nous créeons un fichier de service node_exporter sous systemd
```
sudo vi /etc/systemd/system/node_exporter.service
```

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/bin/sh -c '/usr/local/bin/node_exporter'

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

- Nous vérifions le status du service node_exporter
```
sudo systemctl status node_exporter
```

- Nous configurons le service node_exporter en tant que cible sur le service Prometheus
```
sudo vi /etc/prometheus/prometheus.yml
```

```
global:
  scrape_interval: 10s

scrape_configs:
  ...
  - job_name: 'node_exporter_metrics'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']
```

```
sudo systemctl restart prometheus
```

Maintenant, si vous vérifions la cible dans l'interface utilisateur Web prometheus (http://prometheus-ip:9090/targets), nous pourrons voir le statut du node_exporter_metrics à *UP*.