# Gouverner les configurations des pods avec les politiques de sécurité des pods
Les politiques de sécurité des pods permettent aux administrateurs de cluster de contrôler les configurations liées à la sécurité avec lesquelles les pods sont autorisés à s'exécuter.<br>
Utilisez-les pour appliquer automatiquement les configurations de sécurité souhaitées au sein du cluster.<br>
Les politiques de sécurité des pods peuvent également modifier les pods en fournissant des valeurs par défaut pour certaines valeurs.
Cela vous permet d'appliquer certaines configurations de manière transparente pour vos utilisateurs.<br>
Les politiques de sécurité des pods peuvent contrôler des choses comme :

- Mode privilégié : Autoriser ou interdire l'exécution
conteneurs en mode privilégié.
- Espaces de noms d'hôte : Bloquez les paramètres d'espace de noms d'hôte tels que *hostNetwork=true*.
- runAsUser/runAsGroup : exécuter en tant qu'utilisateur ou groupe spécifique, ou simplement empêcher l'exécution en tant que root.
- Volumes : contrôlez les types de volume autorisés pour les volumes de stockage de pod.
- allowHostPaths : limite les volumes hostPath à des chemins spécifiques uniquement.<br>

Les politiques de sécurité des pods sont obsolètes et seront remplacées par différentes fonctionnalités k8s à l'avenir ! <br>

Utilisons les politiques de sécurité des pods pour appliquer les configurations de sécurité souhaitées pour les nouveaux pods.<br>

Les politiques de sécurité des pods peuvent rejeter les pods qui ne répondent pas à la norme souhaitée ou modifier les pods en appliquant les paramètres par défaut.