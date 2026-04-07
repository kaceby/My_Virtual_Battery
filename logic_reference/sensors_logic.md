 ==========================================================

 LOGIQUE DES FLUX D'ÉNERGIE (PUISSANCE -> ÉNERGIE)

 ==========================================================

 Cette étape est le moteur qui alimente la batterie virtuelle.

 ----------------------------------------------------------

 1. SOURCES DE DONNÉES (Input)
 L'utilisateur doit fournir deux entités de puissance (W) :
 - Production Solaire (P) : ex. sensor.envoy_production 
 - Consommation Maison (C) : ex. sensor.envoy_consumption 

 ----------------------------------------------------------

 2. CALCUL DU FLUX NET (Net Power)

 Formule : Net_Power = Production - Consommation 
 - Si résultat > 0 : Surplus envoyé vers la batterie (In) 
 - Si résultat < 0 : Besoin puisé dans la batterie (Out) 

 ----------------------------------------------------------

 3. INTÉGRATION DE RIEMANN (Conversion W en kWh)

 Pour obtenir des index cumulés, on utilise la méthode d'intégration "Left".
 - Energy_Battery_In : Intégrale de Net_Power quand il est positif 
 - Energy_Battery_Out : Intégrale de Net_Power (abs) quand il est négatif 
   CONDITION : Le comptage 'Out' ne se fait QUE si le solde batterie > 0.

 ----------------------------------------------------------

 4. EXPORT POUR LE DASHBOARD ÉNERGIE

 Création d'un index 'Total Increasing' pour la compatibilité HA :
 Formule : (Production - Consommation) / 1000

Les fichier origineaux YAML sont situer [ICI](https://github.com/kaceby/My_Virtual_Battery/tree/main/logic_reference/legacy_YAML) 
