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

1.  **Le Capteur Binaire (`binary_sensor`)** :
    *   **Nom** : `binary_sensor.lave_linge_en_marche`
    *   **Rôle** : C'est la sentinelle. Il surveille la consommation électrique de la prise.
    *   **Fonctionnement** : Il passe à "ON" quand la puissance dépasse **5W** (début de cycle) et repasse à "OFF" quand elle reste sous 5W pendant 3 minutes (fin de cycle). C'est lui qui réveille l'automatisation.

2.  **Les "Helpers" (La Mémoire)** :
    *   **`input_select.etat_lave_linge`** : Retient l'état du cycle (En marche / Terminé / Éteint) même si HA redémarre.
    *   **`input_datetime.debut_machine`** : Retient l'heure exacte du début.
    *   **`input_datetime.fin_machine`** : Retient l'heure exacte de la fin.

3.  **Le Compteur (`utility_meter`)** :
    *   **Nom** : `sensor.compteur_lave_linge_cycle`
    *   Il isole la consommation électrique (kWh) du cycle en cours.
    *   Il est remis à zéro automatiquement au début de chaque lavage.

4.  **Les Capteurs Intelligents (`template_sensor`)** :
    *   **État** (`sensor.lave_linge_etat`) : Affiche un état lisible ("Lavage", "Terminé") en combinant le capteur binaire et la mémoire.
    *   **Durée** (`sensor.lave_linge_duree_cycle`) : Calcule le temps écoulé en direct.
    *   **Coût** (`sensor.lave_linge_cout_cycle`) : Multiplie les kWh du cycle par votre prix du kWh.

5.  **La Gestion du Coût (`input_number`)** :
    *   **Nom** : `input_number.cout_du_kwh`
    *   ⚠️ **Configuration** : Valeur par défaut : 0 €. Vous devez définir votre prix du kWh (ex: 0.2516) via l'interface (Paramètres > Appareils et services > Entrées > Coût du kWh).
    *   **Abonnement HP/HC** : Pour gérer les heures pleines/creuses, vous devrez créer une automatisation séparée qui met à jour cette valeur automatiquement selon l'heure.

---

## 📂 Contenu du dossier

*   **`lave_linge_package.yaml`** : **Pour la Méthode 1 (Package).** Le fichier tout-en-un recommandé.
    *   **Helpers** (`input_select`, `input_datetime`) pour stocker l'état et les heures.
    *   **Utility Meter** pour calculer la consommation par cycle.
    *   **Template Sensors** pour calculer la durée, le coût et détecter l'état.
    *   **Automation** pour gérer le cycle et envoyer les notifications.
*   **`lave_linge_automation_simple.yaml`** : Automation seule simplifiée (pour Copier-Coller UI).
*   **`templates.yaml`**, **`input_select.yaml`**, **`input_datetime.yaml`**, **`utility_meter.yaml`**, **`input_number.yaml`** : Fichiers découpés pour l'intégration `!include`.
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
3.  **Adaptez la configuration** : Ouvrez le fichier et recherchez les commentaires `# [A_CHANGER]`. Vous devez renseigner vos entités (prise, puissance, énergie, coût).
4.  **Redémarrez Home Assistant**.
5.  **Configuration Finale** : Une fois redémarré, allez dans *Paramètres > Entrées*, trouvez `Coût du kWh` et définissez votre prix.

### Méthode 2 : L'Installation "À la carte" (Manuelle) 🛠️
*Si vous préférez séparer vos fichiers (`!include`) ou utiliser l'interface graphique.*

1.  **Entrées (Helpers) : VIA FICHIERS YAML OU UI**
    *   Copiez le contenu de `input_select.yaml`, `input_datetime.yaml`, `utility_meter.yaml`, `input_number.yaml` dans vos fichiers respectifs.
    *   **OU** créez ces entités via l'interface (UI) :
        *   **`input_select.etat_lave_linge`** : Liste déroulante (`Éteint`, `En marche`, `Terminé`).
        *   **`input_datetime.debut_machine`** : Date et/ou heure (Date + Heure).
        *   **`input_datetime.fin_machine`** : Date et/ou heure (Date + Heure).
        *   **`input_number.cout_du_kwh`** : Nombre (Boîte de saisie).
        *   **`utility_meter.compteur_lave_linge_cycle`** : Compteur (Pas de cycle).
    *   **⚠️ IMPORTANT** : Quelque soit la méthode, n'oubliez pas de définir votre coût du kWh dans `input_number.cout_du_kwh` !

2.  **Sensors : VIA FICHIER YAML**
    *   Copiez le contenu de **`templates.yaml`** dans votre fichier `templates.yaml` (ou votre dossier `templates/`).

3.  **Automation : VIA L'INTERFACE (UI) (Recommandé)**
    *   Créez une nouvelle automatisation vide via l'UI.
    *   Passez en mode YAML.
    *   Copiez-collez le contenu de **`lave_linge_automation_simple.yaml`**.
    *   Enregistrez.
4.  N'oubliez pas d'adapter les entités dans chaque fichier !
