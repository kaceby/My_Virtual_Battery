# My_Virtual_Battery
Gestionnaire de batterie virtuelle avec recalage d'offset pour Home Assistant. /// Virtual battery manager with offset adjustment for Home Assistant.


### 🧠 Logique de calcul (The "Secret Sauce")

L'intégration gère le calcul du solde en utilisant trois composants clés :
1. **Real-time Tracking :** Différentiel entre l'exportation et l'importation réseau.
2. **Threshold Logic :** Respect du seuil de 0.1 kWh (typiquement utilisé par Urban Solar) pour éviter les micro-déclenchements.
3. **The Offset System :** Un service dédié permet d'ajuster le solde à tout moment (ex: réception de facture) sans casser la courbe historique. L'erreur de mesure est absorbée par une variable de correction dynamique.
