📋 Spécifications des Entités (Cahier des Charges)
Ce document définit les interfaces de l'intégration MyVirtualBattery.

---------------------------------------------------------------------------------------------------------------------

1. Entrées de Configuration (Config Flow)
Lors de l'installation ou de la configuration, l'utilisateur doit renseigner les paramètres suivants :
  A. Sources de données (Entity Picker) :
       -Capteur de Production (sensor) : Puissance instantanée en Watts (W).
       -Capteur de Consommation (sensor) : Puissance instantanée en Watts (W).

  B. Paramètres de l'Installation
     - Puissance installée (float) : Valeur en kWc (utilisée pour le calcul de l'option batterie).
     - Date du relevé initial (date) : Date de départ pour le calcul du premier offset.

  C. Paramètres Tarifaires (Variables éditables)
    - Tarif Achat Réseau (float) : Prix du kWh acheté au fournisseur (€/kWh).
    - Tarif Déstockage (float) : Prix des taxes/acheminement pour le surplus récupéré (€/kWh).
    - Abonnement Mensuel TTC (float) : Coût fixe de l'abonnement électricité (€).
    - Prix Option Batterie HT (float) : Coût par kWc installé (€/kWc).

---------------------------------------------------------------------------------------------------------------

2. Entités de Sortie (Sorties d'informations)
L'intégration doit générer et maintenir les entités suivantes :

A. Capteurs d'Énergie (kWh)
  - Solde Batterie Virtuelle :
    - state_class: total
    - device_class: energy
    - unit: kWh
    - Logique : (Energy_In - Energy_Out) + Offset
  - Index Export (Le surplus):
    - state_class: total_increasing
    - device_class: energy
    - Logique : Cumul du surplus envoyé au réseau.

B. Capteurs Financiers (€)
  - Coût Mensuel Total : Somme de l'abonnement, des frais fixes et de la consommation réelle (Grid + Battery).
  - Coût Mensuel Batterie : Coût spécifique à l'utilisation du stock virtuel.
  - Prix kWh Actuel : Capteur dynamique affichant soit le prix "Achat" soit le prix "Déstockage" selon l'état de la batterie.

C. Capteurs de Puissance (W) - Optionnels pour Diagnostic
  - Puissance Nette : Flux instantané (Production - Consommation).
  - Flux Batterie (In/Out) : Puissance entrant ou sortant de la batterie virtuelle.

-------------------------------------------------------------------------------------------------------------------

3. Services (Actions)
L'intégration doit exposer un service pour permettre le recalage manuel :
  - Service virtual_battery.recalibrate
    - Argument 1 : date (Date du relevé fournisseur).
    - Argument 2 : value (Index en kWh lu sur la facture).
    - Action : Déclenche la recherche historique dans le recorder et met à jour l'entité offset de manière persistante.
