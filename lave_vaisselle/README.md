# 🍽️ Gestion du Lave-Vaisselle (Package)

Ce dossier contient la configuration complète pour la gestion intelligente de votre lave-vaisselle dans Home Assistant.

## ⚙️ Comment ça marche ? (Logique & Performance)

Cette automatisation est **légère, robuste et silencieuse** (identique à celle du Lave-Linge).

### 1. Déclenchement Intelligent
*   Elle est **événementielle** : ne se réveille que si la puissance change significativement (via le `binary_sensor`).
*   Charge processeur nulle 99% du temps.

### 2. Le Cycle de Lavage
1.  **Démarrage** : Puissance > **5W** pendant 1 minute -> État "En marche".
2.  **Fin** : Puissance < **5W** pendant 3 minutes -> État "Terminé".

### 3. La Notification
*   Une fois terminé, une **notification persistante** affiche le résumé (Durée, Coût, Conso).
*   Elle se met à jour toutes les 5 minutes tant que la prise est allumée.

### 4. Robustesse
*   Reprise automatique du cycle après un redémarrage de Home Assistant grâce aux Helpers.

## 🧠 Architecture & Rôle des Entités

25.  *Note : Toutes les entités "Helpers" ci-dessous (`input_*`, `utility_meter`) peuvent être créées via l'interface de Home Assistant (Paramètres > Appareils et services > Entrées) si vous ne souhaitez pas utiliser le YAML.*

1.  **Binary Sensor** (`binary_sensor`) :
    *   **Nom** : `binary_sensor.lave_vaisselle_en_marche`
    *   **Rôle** : C'est la sentinelle. Il surveille la consommation électrique de la prise.
    *   **Fonctionnement** : Il passe à "ON" quand la puissance dépasse **5W** (début de cycle) et repasse à "OFF" quand elle reste sous 5W pendant 3 minutes (fin de cycle). C'est lui qui réveille l'automatisation.

2.  **Helpers** :
    *   **`input_select.etat_lave_vaisselle`** : Mémorise l'état du cycle (Éteint / En marche / Terminé). Essentiel pour la reprise après redémarrage.
    *   **`input_datetime.debut_lave_vaisselle`** : Mémorise l'heure de début pour le calcul de durée.
    *   **`input_datetime.fin_lave_vaisselle`** : Mémorise l'heure de fin.

3.  **Utility Meter** :
    *   **Nom** : `sensor.compteur_prislavvais_cycle`
    *   **Rôle** : Compte les kWh consommés uniquement pendant le cycle en cours. Il est remis à zéro automatiquement au début de chaque lavage.

4.  **Template Sensors** :
    *   **État** (`sensor.lave_vaisselle_etat`) : Affiche un état lisible ("Lavage", "Terminé") en combinant le capteur binaire et l'input_select.
    *   **Durée** (`sensor.lave_vaisselle_duree_cycle`) : Calcule le temps écoulé en temps réel.
    *   **Coût** (`sensor.lave_vaisselle_cout_cycle`) : Estime le coût du cycle en cours (kWh * Prix).

5.  **La Gestion du Coût (`input_number`)** :
    *   **Nom** : `input_number.cout_du_kwh`
    *   **Rôle** : Stocke votre prix du kWh pour le calcul du coût.
    *   ⚠️ **Configuration** : Valeur par défaut : 0 €. Vous devez définir votre prix via l'interface.

---

## 📂 Contenu du dossier

*   **`lave_vaisselle_package.yaml`** : **Pour la Méthode 1 (Package).** Le fichier tout-en-un recommandé.
*   **`lave_vaisselle_automation.yaml`** : Automation seule (pour Copier-Coller UI).
*   **`templates.yaml`**, **`input_select.yaml`**, **`input_datetime.yaml`**, **`utility_meter.yaml`**, **`input_number.yaml`** : Fichiers découpés pour l'intégration `!include`.
*   **`dashboard_prislavvais.yaml`** : Code YAML de la carte Lovelace (Dashboard) associée.

## 🚀 Installation & Utilisation

### Méthode 1 : Le "Package" (Recommandé) ✨
1.  Copiez **`lave_vaisselle_package.yaml`** dans `packages/`.
2.  **Adaptez la configuration** : Ouvrez le fichier et recherchez les commentaires `# [A_CHANGER]` (notamment pour la prise connectée).
3.  **Redémarrez Home Assistant**.
4.  **Configuration Finale** : Une fois redémarré, allez dans *Paramètres > Entrées*, trouvez `Coût du kWh` et définissez votre prix.

### Méthode 2 : L'Installation "À la carte" (Manuelle) 🛠️
*Pour ceux qui utilisent des fichiers séparés (`!include`).*

1.  **Entrées (Helpers) : VIA FICHIERS YAML OU UI**
    *   Copiez le contenu de `input_select.yaml`, `input_datetime.yaml`, `utility_meter.yaml`, `input_number.yaml` dans vos fichiers respectifs.
    *   **OU** créez ces entités via l'interface (UI) :
        *   **`input_select.etat_lave_vaisselle`** : Liste déroulante (`Éteint`, `En marche`, `Terminé`).
        *   **`input_datetime.debut_lave_vaisselle`** : Date et/ou heure (Date + Heure).
        *   **`input_datetime.fin_lave_vaisselle`** : Date et/ou heure (Date + Heure).
        *   **`input_number.cout_du_kwh`** : Nombre (Boîte de saisie).
        *   **`utility_meter.compteur_prislavvais_cycle`** : Compteur (Pas de cycle).
    *   **⚠️ IMPORTANT** : Quelque soit la méthode, n'oubliez pas de définir votre coût du kWh dans `input_number.cout_du_kwh` !

2.  **Sensors : VIA FICHIER YAML**
    *   Copiez le contenu de **`templates.yaml`** dans votre fichier `templates.yaml` (ou votre dossier `templates/`).

3.  **Automation : VIA L'INTERFACE (UI) (Recommandé)**
    *   Créez une nouvelle automatisation vide via l'UI.
    *   Passez en mode YAML.
    *   Copiez-collez le contenu de **`lave_vaisselle_automation.yaml`**.
    *   Enregistrez.

## 📱 Interface (Dashboard)

Le code de la carte se trouve dans **`dashboard_prislavvais.yaml`**.
Il affiche :
*   Consommation temps réel (W, V, A).
*   État du cycle, Durée, Coût, kWh totaux.
*   Contrôle de la prise et du verrouillage enfant.
