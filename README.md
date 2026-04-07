![Dashboard Preview](dashboard.png)

# My_Virtual_Battery
Ce projet vise à transformer un prototype complexe basé sur du YAML en une intégration officielle Home Assistant (Custom Component).

Le but est de simuler une Batterie Virtuelle (type Urban Solar Design ou JPME) pour les propriétaires de panneaux solaires. Contrairement aux batteries physiques, ce système stocke numériquement votre surplus sur le réseau et vous permet de le "déstocker" plus tard, avec une gestion précise des taxes d'acheminement et des recalages sur factures réelles.

### 🧠 Logique de calcul

L'intégration gère le calcul du solde en utilisant trois composants clés :
1. **Real-time Tracking :** Différentiel entre l'exportation et l'importation réseau.
2. **Threshold Logic :** Respect du seuil de 0.1 kWh (typiquement utilisé par Urban Solar) pour éviter les micro-déclenchements.
3. **The Offset System :** Un service dédié permet d'ajuster le solde à tout moment (ex: réception de facture) sans casser la courbe historique. L'erreur de mesure est absorbée par une variable de correction dynamique.

## 📊 Visualisation recommandée
Pour un rendu optimal comme sur la capture d'écran, il est recommandé d'utiliser la carte Lovelace [power-flow-card-plus](https://github.com/flixlix/power-flow-card-plus) disponible sur HACS. 

L'intégration `MyVirtualBattery` fournit les entités nécessaires pour alimenter les champs "Individual" ou "Battery" de cette carte.

## ✨ Fonctionnalités clés

- Suivi en temps réel : Calcul du solde de la batterie (kWh) basé sur la production et la consommation.
- Logique de déchargement : Gestion du seuil de 0.1 kWh pour basculer intelligemment entre l'énergie stockée et l'achat sur le réseau.
- Gestion financière : Calcul du coût mensuel réel (Abonnement, Option Batterie par kWc, Taxes de déstockage, et Achat réseau).
- Recalage Historique (Unique) : Un système permettant de corriger le solde local à partir d'un relevé fournisseur à une date passée, en utilisant les données historiques du Recorder.

## 📂 Ressources Techniques 

Si vous souhaitez contribuer à transformer ce YAML en Python, veuillez consulter les dossiers suivants :

Vous trouverez dans le [dossier](https://github.com/kaceby/My_Virtual_Battery/tree/main/logic_reference) les spécifications théoriques et mathématiques (Sensors, Battery, Recalibration).

Vous trouverez dans le [dossier](https://github.com/kaceby/My_Virtual_Battery/tree/main/logic_reference/legacy_YAML) le code source YAML original qui sert de "Proof of Concept" fonctionnel.

Objectif final : Une installation via HACS, une configuration via Config Flow (UI), et une compatibilité totale avec le Dashboard Énergie officiel.

# 🤝 Comment aider ?

Le projet cherche actuellement un développeur Python pour :
    - Créer le config_flow.py pour une configuration simplifiée.
    - Importer la logique de calcul dans un coordinator.py.
    - Implémenter le service de recalage via l'API recorder.
