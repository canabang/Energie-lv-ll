# ⚡ Gestion Énergie & Cycles (Lave-Linge / Lave-Vaisselle)

Ce package contient une solution complète et robuste pour surveiller, calculer le coût et notifier la fin des cycles de vos appareils électroménagers (Lave-Linge et Lave-Vaisselle) dans Home Assistant.

---

## ✨ Fonctionnalités Clés

*   **Détection d'état intelligente** : Ne se base pas simplement sur la puissance instantanée, mais utilise un algorithme (temps + seuil) pour déterminer si la machine est "En marche", "Terminée" ou "Éteinte".
*   **Calcul du coût précis** : Isole la consommation électrique de **chaque cycle** (pas le total cumulé à vie) et la multiplie par votre coût du kWh.
*   **Résilience 🛡️** : En cas de redémarrage de Home Assistant *pendant* un lavage, le système reprend exactement là où il en était (temps écoulé, état, coût). Rien n'est perdu.
*   **Notifications Persistantes** : Une fois le cycle terminé, une notification s'affiche dans HA avec le résumé (Coût, Durée, kWh). Elle reste tant que vous n'avez pas éteint la machine/prise.
*   **Aucun Polling** : 100% événementiel. Charge système nulle quand les machines ne tournent pas.

---

## 📂 Modules Disponibles

Chaque appareil possède son propre dossier avec sa documentation détaillée et ses fichiers de configuration.

### 🧺 [Gestion du Lave-Linge](./lave_linge/)
*   **Dossier** : [`lave_linge/`](./lave_linge/)
*   **Fonction** : Suivi du cycle de lavage.
*   **Entités** : `lave_linge_*`

### 🍽️ [Gestion du Lave-Vaisselle](./lave_vaisselle/)
*   **Dossier** : [`lave_vaisselle/`](./lave_vaisselle/)
*   **Fonction** : Suivi du cycle de lavage.
*   **Entités** : `lave_vaisselle_*`

---

## 🚀 Installation : Deux Philosophies

Pour chaque module, nous proposons deux méthodes d'installation selon votre niveau et vos préférences :

1.  **Le Package (Recommandé) ✨** :
    *   Un seul fichier YAML (`*_package.yaml`) à déposer dans votre dossier `packages/`.
    *   Tout est inclus (Helpers, Sensors, Automation).
    *   C'est la méthode la plus simple et la plus portable.

2.  **L'Installation Manuelle (À la carte) 🛠️** :
    *   Pour ceux qui préfèrent séparer leurs fichiers (`sensors/`, `automations/`, etc.).
    *   Possibilité de créer les Helpers (Entrées) via l'interface graphique (UI) de Home Assistant.
    *   Automatisation simplifiée disponible (`*_automation_simple.yaml`) sans dépendances externes.

---

## 📋 Pré-requis Généraux

*   Une **prise connectée** avec mesure de consommation (Puissance W & Energie kWh) pour chaque appareil.
*   Avoir configuré le `packages: !include_dir_named packages` dans votre `configuration.yaml` (si méthode Package).
*   Définir votre **Coût du kWh** dans l'entité commune `input_number.cout_du_kwh` (incluse dans les packages).

