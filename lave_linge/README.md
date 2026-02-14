# 🧺 Gestion du Lave-Linge (Package)

Ce dossier contient la configuration complète pour la gestion intelligente de votre lave-linge dans Home Assistant.

## ⚙️ Comment ça marche ? (Logique & Performance)

Cette automatisation a été conçue pour être **légère, robuste et silencieuse**.

### 1. Déclenchement Intelligent (Pas de boucle infinie !)
Contrairement à d'anciennes méthodes qui vérifiaient l'état toutes les minutes ("polling"), cette automatisation est **événementielle** :
*   Elle dort 99% du temps. 😴
*   Elle ne se réveille que lors d'un changement d'état précis (via le `binary_sensor` basé sur la puissance > 5W).
*   **Résultat** : Charge processeur nulle pour Home Assistant tant que la machine ne tourne pas.

### 2. Le Cycle de Lavage
1.  **Démarrage** : Quand le sensor detecte que la puissance dépasse **5W** pendant une minute il s'active, la machine passe en état "En marche".
2.  **Lavage** : L'automatisation retourne en veille. Elle ne surveille rien activement.
3.  **Fin** : Quand le sensor detecte que la puissance repasse sous **5W** pendant 3 minutes, la machine passe en état "Terminé".

### 3. La Notification (Une seule !)
Une fois le cycle terminé, une **notification persistante** apparaît dans Home Assistant.
*   Elle contient le résumé : Durée, Coût, Consommation.
*   Elle se met à jour toutes les 5 minutes (tant que la prise reste allumée) pour actualiser le coût/durée si besoin, mais **sans spammer** (c'est la même notification qui est mise à jour).
*   Dès que vous éteignez la prise, la boucle s'arrête définitivement.

### 4. Robustesse / Reprise après coupure
Si Home Assistant redémarre **PENDANT** un lavage ou **APRÈS** la fin (mais avant que vous n'ayez éteint la prise) :
*   Aucun problème ! Grâce aux "Helpers" (mémoire), il sait exactement où il en était.
*   Il reprendra le calcul de la durée et du coût là où il s'était arrêté, sans perdre les données du début de cycle.

## 🧠 Architecture & Rôle des Entités

Ce système repose sur plusieurs briques qui travaillent ensemble :

1.  **Le Capteur Binaire (`binary_sensor`)** : C'est la sentinelle.
    *   Il surveille la puissance électrique.
    *   Il décide si la machine est "ON" (>5W pendant 1 min) ou "OFF" (<5W pendant 3 min).
    *   C'est lui qui déclenche l'automatisation.

2.  **Les "Helpers" (La Mémoire)** :
    *   `input_select` : Retient l'état du cycle (En marche / Terminé / Éteint) même si HA redémarre.
    *   `input_datetime` : Retient l'heure exacte du début et de la fin pour le calcul de durée.

3.  **Le Compteur (`utility_meter`)** :
    *   Il isole la consommation électrique (kWh) du cycle en cours.
    *   Il est remis à zéro automatiquement au début de chaque lavage.

4.  **Les Capteurs Intelligents (`template_sensor`)** :
    *   **État** : Affiche un état lisible ("Lavage", "Terminé") en combinant le capteur binaire et la mémoire.
    *   **Durée** : Calcule le temps écoulé en direct.
    *   **Coût** : Multiplie les kWh du cycle par votre prix du kWh.

---

## 📂 Contenu du dossier

*   **`lave_linge_package.yaml`** : **Pour la Méthode 1 (Package).** Le fichier tout-en-un recommandé.
    *   **Helpers** (`input_select`, `input_datetime`) pour stocker l'état et les heures.
    *   **Utility Meter** pour calculer la consommation par cycle.
    *   **Template Sensors** pour calculer la durée, le coût et détecter l'état.
    *   **Automation** pour gérer le cycle et envoyer les notifications.
*   **`lave_linge_automation.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient l'automatisation seule.
*   **`lave_linge_templates.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient les capteurs seuls.
*   **`dashboard_prismal.yaml`** : Code YAML de la carte Lovelace (Dashboard) associée.

## � Interface (Dashboard)

Voici à quoi ressemble la carte une fois installée :

![Aperçu de la carte](carte%20lave%20linge.png)

Le code complet de cette carte se trouve dans le fichier **`dashboard_prismal.yaml`**.
*   **Compatible avec les deux méthodes** : Que vous utilisiez le Package ou l'installation Manuelle, les entités ont les mêmes noms.
*   **Fonctionnalités** :
    *   État en temps réel + animation (icône qui tourne si en marche).
    *   Temps écoulé / Durée totale.
    *   Coût du cycle.
    *   Graphique de consommation (Puissance, Tension, Ampérage).

## �🚀 Installation & Utilisation

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
    *   `utility_meter.compteur_prismal_cycle` (Source : votre capteur énergie, Cycle : pas de remise à zéro automatique, adaptez le nom selon votre appareil)
2.  **Sensors** : Copiez le contenu de **`lave_linge_templates.yaml`** dans votre fichier `templates.yaml` (ou dossier `templates/`).
3.  **Automation** : Copiez le contenu de **`lave_linge_automation.yaml`** dans votre fichier `automations.yaml` (ou créez une nouvelle automatisation via l'UI en mode YAML).
4.  N'oubliez pas d'adapter les entités dans chaque fichier !
