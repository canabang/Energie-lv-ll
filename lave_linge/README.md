# 🧺 Gestion du Lave-Linge (Package)

Ce dossier contient la configuration complète pour la gestion intelligente de votre lave-linge dans Home Assistant.

## 📂 Contenu du dossier

*   **`lave_linge_package.yaml`** : **Pour la Méthode 1 (Package).** Le fichier tout-en-un recommandé.
    *   **Helpers** (`input_select`, `input_datetime`) pour stocker l'état et les heures.
    *   **Utility Meter** pour calculer la consommation par cycle.
    *   **Template Sensors** pour calculer la durée, le coût et détecter l'état.
    *   **Automation** pour gérer le cycle et envoyer les notifications.
*   **`lave_linge_automation.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient l'automatisation seule.
*   **`lave_linge_templates.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient les capteurs seuls.
*   **`dashboard_prismal.yaml`** : Code YAML de la carte Lovelace (Dashboard) associée.

## 🚀 Installation & Utilisation

Vous avez deux méthodes pour installer cette configuration.

### Méthode 1 : Le "Package" (Recommandé) ✨
*Tout est regroupé dans un seul fichier. C'est le plus simple à installer et à gérer.*

1.  Assurez-vous que votre fichier `configuration.yaml` inclut les packages :
    ```yaml
    homeassistant:
      packages: !include_dir_named packages
    ```
2.  Copiez le fichier **`lave_linge_package.yaml`** dans votre dossier `packages/`.
3.  **Adaptez la configuration** : Ouvrez le fichier et recherchez les commentaires `# [CONFIG]`. Vous devez renseigner vos entités (prise, puissance, énergie, coût).
4.  Redémarrez Home Assistant.

### Méthode 2 : L'Installation "À la carte" (Manuelle) 🛠️
*Si vous préférez séparer vos fichiers ou utiliser l'interface graphique.*

1.  **Entrées (Helpers)** : Créez manuellement via l'interface (Paramètres > Appareils et services > Entrées) :
    *   `input_select.etat_lave_linge` (Options : Éteint, En marche, Terminé)
    *   `input_datetime.debut_machine` (Date et heure)
    *   `input_datetime.fin_machine` (Date et heure)
    *   `utility_meter.compteur_prismal_cycle` (Source : votre capteur énergie, Cycle : pas de remise à zéro automatique, prismal à adapter suivant le nom de votre device) 
2.  **Sensors** : Copiez le contenu de **`lave_linge_templates.yaml`** dans votre fichier `templates.yaml` (ou dossier `templates/`).
3.  **Automation** : Copiez le contenu de **`lave_linge_automation.yaml`** dans votre fichier `automations.yaml` (ou créez une nouvelle automatisation via l'UI en mode YAML).
4.  n'oubliez pas d'adapter les entités dans chaque fichier !

## ⚙️ Fonctionnement

1.  **Détection** : Quand la puissance dépasse **5W** pendant **1 min**, la machine est considérée "En marche".
2.  **Cycle** : La date de début est enregistrée, le compteur de cycle est remis à zéro.
3.  **Fin** : Quand la puissance repasse sous **5W** pendant **3 min**, le cycle est "Terminé".
4.  **Notification** : Une notification persistante est envoyée sur HA avec le résumé (Durée, Coût, Conso). Elle se met à jour toutes les 5 minutes tant que la prise n'est pas éteinte.
5.  **Robustesse** : Si HA redémarre pendant un cycle, il reprend l'état correct grâce aux helpers.
