# ⚡ Configuration PrisMal & PrisLavVais

Ce dossier contient les tableaux de bord pour le suivi énergétique des prises **PrisMal** et **PrisLavVais**.

## 1. Création des Compteurs (Utility Meter)

Pour que le calcul du coût fonctionne et que le total ne se remette pas à zéro, vous devez créer deux **Compteurs de services publics** via l'interface de Home Assistant.

**Chemin :**
`Paramètres` > `Appareils et services` > `Entrées` > `Créer une entrée` > **`Compteur de services publics (eau, gaz, électricité…)`**

### Configuration pour PrisMal
*   **Nom** : `Compteur PrisMal Total`
    *   *Cela générera l'ID `sensor.compteur_prismal_total` (à vérifier/modifier si nécessaire)*
*   **Capteur d'entrée** : `sensor.prismal_energy`
*   **Cycle de remise à zéro** : `Pas de cycle` (Never)

### Configuration pour PrisLavVais
*   **Nom** : `Compteur PrisLavVais Total`
    *   *Cela générera l'ID `sensor.compteur_prislavvais_total` (à vérifier/modifier si nécessaire)*
*   **Capteur d'entrée** : `sensor.prislavvais_energy`
*   **Cycle de remise à zéro** : `Pas de cycle` (Never)

---

## 2. Tableaux de Bord (Dashboards)

Deux fichiers YAML sont disponibles pour l'affichage :

*   [`dashboard_prismal.yaml`](./dashboard_prismal.yaml) : Carte pour la prise PrisMal.
*   [`dashboard_prislavvais.yaml`](./dashboard_prislavvais.yaml) : Carte pour la prise PrisLavVais.

### Installation
Vous pouvez copier le contenu de ces fichiers dans une carte "Manuel" (YAML) sur votre tableau de bord, ou utiliser `!include` si vous gérez vos dashboards en YAML.

---

## 3. Pré-requis (Coût)

Assurez-vous d'avoir l'entité pour le prix du kWh (déjà présente si vous utilisez le dashboard PrisTVChambX3) :
*   `input_number.cout_du_kwh`
