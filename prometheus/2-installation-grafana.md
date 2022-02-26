# Installation de grafana 8.4.2 sur Rocky linux 8
Dans ce tutoriel, nous allons installer grafana et ajouter la source de données de prometheus.

## Installation de grafana
- Nous téléchargeons et installons le binaire de grafana
```
wget https://dl.grafana.com/oss/release/grafana-8.4.2-1.x86_64.rpm
sudo yum install grafana-8.4.2-1.x86_64.rpm
```

- Nous activons et demarrons grafana
```
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

- Nous autorisons le port 3000 au niveau du parefeu
```
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

- Nous accédons à l'interface ui via http://grafana_IP:3000

## Ajout de la source Prometheus à Grafana
- Nous cliquons sur l'option "ajouter une source" sur la page d'accueil de Grafana.
- Nous ajoutons le nom de la source, les détails du point de terminaison Prometheus et enregistrons-le.
- Nous sélectionnons l'option de création de tableau de bord.
- Nous sélectionnons le type de graphique. Nous pourrons sélectionner le type en fonction du type de visualisation et de tableau de bord dont nous avons besoin.
- Nous sélectionnons l'option d'édition en haut du panneau.
- Nous sélectionnons la source de données Prometheus et entrons l'expression Prometheus qui doit être représentée graphiquement sous l'onglet des métriques. Nous pouvons prévisualiser le graphique à l'aide du bouton de prévisualisation. Sous l'onglet général, nous pouvons attribuer un nom au tableau de bord. Nous enregistrons le tableau de bord après l'aperçu.

## Importation de modèles de tableau de bord Grafana prédéfinis
Nous pouvons trouver tous les tableaux de bord communautaires partagés à partir des [modèles partagés Grafana](https://grafana.com/dashboards?dataSource=prometheus)

- Nous sélectionnons l'option d'importation.
- Voici les options d'importation prises en charge. Nous pouvons ajouter l'ID de tableau de bord que nous obtenons sur le site Web de grafana, télécharger le json ou coller le json dans la zone de texte.
- Nous ajoutons un nom de modèle, une source Prometheus, un dossier de tableau de bord de destination et cliquez sur *importer*.