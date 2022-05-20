# Analyse des images pour les vulnérabilités connues
Une vulnérabilité logicielle est une faille ou une faiblesse dans un logiciel qui peut être utilisée par un attaquant.<br>
Les chercheurs en sécurité travaillent constamment pour découvrir et documenter les vulnérabilités.<br>

L'analyse des vulnérabilités consiste à utiliser des outils pour détecter les vulnérabilités connues de votre logiciel.<br>
Dans Kubernetes, l'analyse d'images signifie analyser les images de conteneurs pour voir si elles incluent des logiciels vulnérables.

## Trivy
*Trivy* est un outil en ligne de commande qui nous permet d'analyser les images de conteneurs à la recherche de vulnérabilités. *Trivy* est facile à utiliser. Par example:
```
trivy image nginx:1.14.1
trivy nginx:1.14.1
```

Trivy génère un rapport qui répertorie les vulnérabilités connues
trouvé dans l'image. Il présente son rapport sous forme de tableau dont certaines colonnes sont les suivantes :<br>
- *LIBRARY* - quelle bibliothèque de logiciels est affectée.
- *VULNERABILITY ID* - un identifiant unique pour la vulnérabilité.
- *SEVERITY* - indique le niveau de risque de sécurité créé par la vulnérabilité.<br>

Installons *Trivy* sur votre nœud de plan de contrôle :
```
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
```

```
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
```

```
sudo apt-get update && sudo apt-get install -y trivy
```

Si nous avons besoin d'obtenir le nom et le tag de l'image d'un conteneur existant, nous pouvons le faire en utilisant *kubectl describe* :
```
kubectl get pods
kubectl describe pod $pod_name
```

Scannons une image avec Trivy :
```
trivy image nginx:1.14.1
```