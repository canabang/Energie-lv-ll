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

1.  **Binary Sensor** (`binary_sensor.lave_vaisselle_en_marche`) : Surveille la puissance (>5W).
2.  **Helpers** (`input_select`, `input_datetime`) : Mémorisent l'état et les heures.
3.  **Utility Meter** (`compteur_prislavvais_cycle`) : Compte les kWh du cycle courant.
4.  **Template Sensors** : Calculent la durée, le coût et l'état composite.

### 5. La Gestion du Coût (`input_number`)
⚠️ **Point Important** : Le calcul du coût repose sur l'entité `input_number.cout_du_kwh`.
*   **Vous devez créer cette entité** (dans Paramètres > Appareils et services > Entrées > Créer une entrée > Nombre).
*   **Adaptation selon votre abonnement** :
    *   **Tarif Base** : Mettez simplement votre prix fixe (ex: 0.2516) dans la valeur.
    *   **Tarif Heures Pleines / Heures Creuses (HP/HC)** : Vous devez automatiser la mise à jour de ce nombre. Par exemple, une automatisation qui change la valeur à 0.27 (HP) ou 0.20 (HC) selon l'heure ou l'état de votre compteur Linky.

---

## 📂 Contenu du dossier

*   **`lave_vaisselle_package.yaml`** : **Pour la Méthode 1 (Package).** Le fichier tout-en-un recommandé.
*   **`lave_vaisselle_automation.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient l'automatisation seule.
*   **`lave_vaisselle_templates.yaml`** : **Pour la Méthode 2 (Manuelle).** Contient les capteurs seuls.
*   **`dashboard_prislavvais.yaml`** : Code YAML de la carte Lovelace (Dashboard) associée.

## 🚀 Installation & Utilisation

### Méthode 1 : Le "Package" (Recommandé) ✨
1.  Copiez **`lave_vaisselle_package.yaml`** dans `packages/`.
2.  Adaptez les entités (`# [CONFIG]`) si besoin (par défaut pré-configuré pour `prislavvais`).
3.  Redémarrez HA.

### Méthode 2 : L'Installation "À la carte" (Manuelle) 🛠️
1.  **Entrées (Helpers)** : Créez `input_select.etat_lave_vaisselle`, `input_datetime` (debut/fin), `utility_meter` via l'UI.
2.  **Sensors** : Copiez le contenu de `lave_vaisselle_templates.yaml` dans votre config.
3.  **Automation** : Copiez le contenu de `lave_vaisselle_automation.yaml`.

## 📱 Interface (Dashboard)

Le code de la carte se trouve dans **`dashboard_prislavvais.yaml`**.
Il affiche :
*   Consommation temps réel (W, V, A).
*   État du cycle, Durée, Coût, kWh totaux.
*   Contrôle de la prise et du verrouillage enfant.
