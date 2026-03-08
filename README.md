# ⚡ Gestion Énergie & Cycles (Lave-Linge / Lave-Vaisselle)

Ce package contient une solution complète et robuste pour surveiller, calculer le coût et notifier la fin des cycles de vos appareils électroménagers (Lave-Linge et Lave-Vaisselle) dans Home Assistant.

> 📖 **Tous les fichiers YAML** (Packages, Dashboards, Blueprint) sont **intégralement commentés** pour expliquer le rôle de chaque entité, variable et bloc de code. N'hésitez pas à les ouvrir pour comprendre le fonctionnement.

---

## ✨ Fonctionnalités Clés

*   **Détection d'état intelligente** : Ne se base pas simplement sur la puissance instantanée, mais utilise un algorithme (temps + seuil) pour déterminer si la machine est "En marche", "Terminée" ou "Éteinte".
*   **Double compteur d'énergie** : Un compteur **Total** (accumulation globale) et un compteur **Cycle** (remise à zéro automatique à chaque nouveau lavage) pour un suivi précis.
*   **Calcul du coût précis** : Isole la consommation électrique de **chaque cycle** et la multiplie par votre coût du kWh.
*   **Résilience 🛡️** : En cas de redémarrage de Home Assistant *pendant* un lavage, le système reprend exactement là où il en était (temps écoulé, état, coût). Rien n'est perdu.
*   **Notifications Persistantes** : Une fois le cycle terminé, une notification s'affiche dans HA avec le résumé (Coût, Durée, kWh). Elle reste tant que vous n'avez pas éteint la machine/prise.
*   **Aucun Polling** : 100% événementiel. Charge système nulle quand les machines ne tournent pas.

---

## 📂 Modules Disponibles

Chaque appareil possède son propre dossier avec ses fichiers de configuration.

### 🧺 [Gestion du Lave-Linge](./lave_linge/)
*   **Dossier** : [`lave_linge/`](./lave_linge/)
*   **Fonction** : Entités et Dashboards du cycle de lavage (Package & Dashboards).

### 🍽️ [Gestion du Lave-Vaisselle](./lave_vaisselle/)
*   **Dossier** : [`lave_vaisselle/`](./lave_vaisselle/)
*   **Fonction** : Entités et Dashboards du cycle de lavage (Package & Dashboards).

---

## 🎨 Interface Utilisateur (Dashboards)

> ⚠️ **Pré-requis HACS** : Ces interfaces s'appuient sur des cartes personnalisées que vous devez installer via HACS : `mushroom-cards`, `mini-graph-card`, `stack-in-card`, et `card-mod`.

![Aperçu Global du Dashboard](dashboard_toutes_les_cartes.png)

Vous trouverez dans les dossiers respectifs de chaque appareil **trois types de cartes Lovelace** prêtes à l'emploi. Vous pouvez copier leur code YAML directement dans votre dashboard Home Assistant.

### 1. Le Widget "Streamline" (Compact)
*   **Fichiers :** `carte_*_streamline.yaml` (Dans les sous-dossiers `cartes_dashboard/`)
*   **Usage :** Idéal pour une vue d'ensemble sur un écran d'accueil (mobile friendly).
*   **Bonus :** Animation CSS "Aura Verte" autour de l'icône lorsque l'appareil est en marche.
![Aperçu Streamline Lave Linge](lave_linge/cartes_dashboard/carte_lave_linge_streamline.png)

### 2. La Carte "Analyse Détaillée" (Standard)
*   **Fichiers :** `carte_lave_*.yaml` (Dans les sous-dossiers `cartes_dashboard/`)
*   **Usage :** Idéal pour une page dédiée à l'énergie.
*   **Contient :** Historique de la consommation sur les dernières 24h, monitoring de la tension électrique (V), état en direct et bouton d'action sur la prise.
![Aperçu Analyse Détaillée Lave Linge](lave_linge/cartes_dashboard/carte_lave_linge.png)

### 3. Le Dashboard "Suivi Entités" (Admin)
*   **Fichiers :** `carte_*_entites_suivi.yaml` (Dans les sous-dossiers `cartes_dashboard/`)
*   **Usage :** Carte de débogage et de surveillance des variables cachées.
*   **Contient :** L'affichage en temps réel de tous les Helpers (`input_select`, `datetime`), du compteur brut et de l'état des déclencheurs de vos machines.

![Aperçu Suivi Entités](lave_vaisselle/cartes_dashboard/carte_lave_vaisselle_entites_suivi.png)

---

## 🚀 Installation & Utilisation (V4.2 - Blueprint)

L'architecture repose désormais sur un système hybride extrêmement puissant et simple à configurer : **Les Packages (pour les entités) + Un Blueprint (pour l'intelligence).**

### 1. Préparation des Entités (Le Package)
Déposez le fichier de configuration de l'appareil souhaité dans votre dossier `packages/` :

> 💡 **Si vous n'avez pas de dossier `packages`** : Créez simplement un dossier nommé `packages` à l'endroit où se trouve votre `configuration.yaml` dans Home Assistant. Ensuite, ajoutez ces 2 lignes dans votre fichier `configuration.yaml` :
> ```yaml
> homeassistant:
>   packages: !include_dir_named packages
> ```
> *(N'oubliez pas de redémarrer Home Assistant ensuite)*

*   [`lave_vaisselle_package.yaml`](./lave_vaisselle/package/lave_vaisselle_package.yaml)
*   [`lave_linge_package.yaml`](./lave_linge/package/lave_linge_package.yaml)

> ⚠️ **ÉTAPE CRUCIALE AVANT REDÉMARRAGE** :
> Les fichiers ont été rendus génériques pour être partagés. Vous **DEVEZ** ouvrir le fichier `package` choisi et rechercher les occurrences taguées **`[A_CHANGER]`**.
> 
> 🍽️ **Lignes à modifier pour le Lave-Vaisselle (`lave_vaisselle_package.yaml`) :**
> *   **Lignes 64 & 73** : `source: sensor.A_CHANGER_ENERGIE` ➡️ Votre capteur de consommation d'énergie en kWh (ex: `sensor.prise_energie`)
> *   **Ligne 94** : `{{ states('sensor.A_CHANGER_PUISSANCE')|float(0) > ... }}` ➡️ Votre capteur de puissance en Watts (ex: `sensor.prise_puissance`)
>
> 🧺 **Lignes à modifier pour le Lave-Linge (`lave_linge_package.yaml`) :**
> *   **Lignes 71 & 80** : `source: sensor.A_CHANGER_ENERGIE` ➡️ Votre capteur de consommation d'énergie en kWh (ex: `sensor.prise_energie`)
> *   **Ligne 100** : `{{ states('sensor.A_CHANGER_PUISSANCE')|float(0) > ... }}` ➡️ Votre capteur de puissance en Watts (ex: `sensor.prise_puissance`)
>
> Sans remplacer ces variables clés, les capteurs ne remonteront aucune mesure et le blueprint de notifications ne s'activera pas !

> *Ce fichier `package` va générer automatiquement toutes les entités, les compteurs de consommation et les variables (Helpers) nécessaires au bon fonctionnement.*

> 💡 **Note** : Les deux packages déclarent le même Helper `input_number.cout_du_kwh` (prix du kWh). C'est **un seul et unique** paramètre partagé par les deux appareils. Il suffit qu'**un seul** des deux packages soit installé pour que le système fonctionne. Si vous installez les deux, commentez l'un des deux dans le fichier package.

### 2. Importation du Blueprint (Le Cerveau)
Téléchargez ou copiez le fichier [`blueprint_gestion_cycle_appareil.yaml`](./blueprint_gestion_cycle_appareil.yaml) dans le dossier `blueprints/automation/` de votre Home Assistant.
> *Ce fichier Blueprint universel contient toute la logique complexe de suivi (reset des compteurs, timers, détection intelligente).*

### 3. Création de l'Automatisation via l'Interface (L'Action)
Depuis l'interface visuelle de Home Assistant, créez une nouvelle automatisation à partir du Blueprint `🔄 Gestion Énergie - Cycle Appareil`.
**C'est à vous de jouer ! 🎮**

Vous n'aurez qu'à configurer **2 entités** :
1. Le capteur de fonctionnement de l'appareil (ex: `binary_sensor.lave_vaisselle_en_marche`).
2. La prise connectée physique (ex: `switch.votre_prise`).

![Blueprint Interface 1](blueprint_gestion_cycle_appareil_01.png)
![Blueprint Interface 2](blueprint_gestion_cycle_appareil_02.png)

### 🔌 Options & Actions Libres
Le Blueprint déduira automatiquement le reste de vos entités. Vous avez le contrôle total sur la manière d'être notifié :
*   **Notification Persistante HA :** Vous pouvez activer/désactiver l'apparition de la notification locale de Home Assistant.
*   **Message de Fin Personnalisable :** Le texte de la notification est entièrement personnalisable depuis l'interface du Blueprint.
*   **Actions Libres (Démarrage & Fin) :** Deux blocs d'actions libres vous permettent de construire vos propres notifications (Discord, Telegram, awtrix, Alexa...).

> 💡 **Le petit plus magique :** Vous remarquerez que le texte par défaut contient des balises comme `{{ states(cout_cycle) }}`. Vous pouvez modifier la phrase comme bon vous semble, tant que vous gardez ces balises, le Blueprint s'occupera d'aller chercher et calculer automatiquement le bon capteur de coût, de durée ou d'énergie pour l'appareil concerné ! Vous pourrez ensuite réutiliser ce texte final dans vos Actions via la variable globale `{{ message_fin }}`.

### 4. Configuration Initiale de l'Énergie (À faire une seule fois)
Après le redémarrage de Home Assistant, toutes vos nouvelles entités (Helpers) ont été créées. Il reste **une étape indispensable** pour que le système puisse calculer le coût d'un lavage en euros :

1. Vous devez initialiser le prix de votre électricité dans l'entité `input_number.cout_du_kwh`.
2. Pour ce faire, le moyen le plus simple est de **copier le code de la carte "Suivi Entités"** (ex: `carte_lave_linge_entites_suivi.yaml`) sur l'un de vos Dashboards.
3. Depuis cette carte visuelle sur votre téléphone ou PC, vous verrez une ligne **"Coût du kWh"**. Entrez simplement votre tarif (ex: `0.25` pour 25 centimes). 
4. La valeur sera sauvegardée définitivement et partagée pour les deux appareils !

---

## 📋 Pré-requis Généraux

*   Une **prise connectée** avec mesure de consommation (Puissance W & Energie kWh) pour chaque appareil.
*   Avoir configuré le `packages: !include_dir_named packages` dans votre `configuration.yaml`.
*   Avoir redémarré Home Assistant (ou rechargé les automatisations+blueprints) après l'ajout des fichiers.

---

## 🎓 FAQ & Pièges à éviter (Pour les débutants)
Pour ceux qui se lancent, voici les "pièges" classiques dans lesquels il est facile de tomber lors de la configuration initiale :

1. ⚠️ **Ne pas confondre Puissance (Watts) et Énergie (kWh) :**
   C'est l'erreur numéro un ! Pour savoir si la machine tourne, l'automatisation regarde la **Puissance** (la consommation instantanée en `W` ou `kW`). Mais pour que le compteur `utility_meter` fonctionne et calcule le coût mensuel, il **doit** recevoir l'**Énergie** (le volume total accumulé en `kWh`). Vérifiez minutieusement vos types d'entité lors de la personnalisation du Package !
2. 🔄 **Blueprints locaux = Recharger YAML :**
   Quand vous copiez manuellement le fichier Blueprint dans votre dossier, Home Assistant ne le "voit" pas directement. Pensez bien à aller dans *Outils de développement* > *YAML* et à cliquer sur **Recharger les Automatisations** (ou a minima redémarrer le système) avant d'essayer de créer votre automatisation.
3. 🔁 **L'initialisation post-reboot :**
   Comme expliqué à l'Étape 4, une fois toutes vos entités générées, le capteur de coût ne calculera rien si votre prix du kWh est à `0.00€`. Entrez votre tarif via la carte Lovelace.
4. ⚙️ **Conflits de Helpers (Si vous installez les 2 packages) :**
   Le Lave-Linge et le Lave-Vaisselle partagent le *même* Helper pour le prix du kWh (`input_number.cout_du_kwh`). Si vous activez les *deux* fichiers YAML package, il y aura un doublon de création d'entité, et Home Assistant affichera une erreur dans les logs. Pensez simplement à "commenter" (avec des `#`) tout le bloc définissant `cout_du_kwh` dans le deuxième fichier package que vous installez.

