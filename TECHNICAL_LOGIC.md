🏗 Logique Technique Complète (Backend)

📊 Gestion de l'historique et Recalage
L'intégration doit être capable de retrouver les index Energy_In et Energy_Out à une date passée spécifiée par l'utilisateur (Date du relevé fournisseur).
  1) Source : Entités sensor.energy_battery_power_in et sensor.energy_battery_power_out.
  2) Méthode : Accès au recorder (regler a 15j ou plus dans configuration.yaml) de Home Assistant pour extraire l'état à T = date_releve

💸 Moteur de calcul de coût
Le calcul du coût mensuel suit cette hiérarchie :
  1) Frais fixes : (Abonnement_Mensuel_TTC) + (Puissance_kWc * Option_Batterie_HT * 1.20)
  2) Consommation Variable :
       Si solde batterie $> 0.1$ kWh : Utilisation du Tarif_Destockage
       Si solde batterie $\le 0.1$ kWh : Utilisation du Tarif_Achat_Reseau

⚙️ Variables de Configuration (Config Flow)
L'interface de configuration doit demander les champs suivants (issus des input_numbers) :
    Hardware : Puissance installée (kWc)
    Tarification : Prix Achat, Prix Déstockage, Abonnement TTC, Prix Option HT.
    État Initial : Date et valeur du dernier relevé fournisseur pour initialiser l'Offset.

🔌 Configuration des Sources (Setup)
L'intégration doit demander à l'utilisateur de pointer vers deux entités existantes :
  1. **Production Sensor (W)** : La puissance instantanée générée par les panneaux.
  2. **Consumption Sensor (W)** : La puissance totale appelée par la maison.

Traitement Python
- Calcul du flux Net (Production - Consommation).
- Intégration de type Riemann (méthode: left) pour convertir ces Watts en kWh cumulés (Energy In / Energy Out). Ces index cumulés serviront de base au calcul du solde et au recalage SQL.
