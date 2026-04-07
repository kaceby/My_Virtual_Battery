![Dashboard Preview](dashboard.png)

# My_Virtual_Battery
Gestionnaire de batterie virtuelle avec recalage d'offset pour Home Assistant. /// Virtual battery manager with offset adjustment for Home Assistant.


### 🧠 Logique de calcul (The "Secret Sauce")

L'intégration gère le calcul du solde en utilisant trois composants clés :
1. **Real-time Tracking :** Différentiel entre l'exportation et l'importation réseau.
2. **Threshold Logic :** Respect du seuil de 0.1 kWh (typiquement utilisé par Urban Solar) pour éviter les micro-déclenchements.
3. **The Offset System :** Un service dédié permet d'ajuster le solde à tout moment (ex: réception de facture) sans casser la courbe historique. L'erreur de mesure est absorbée par une variable de correction dynamique.

## 📊 Visualisation recommandée
Pour un rendu optimal comme sur la capture d'écran, il est recommandé d'utiliser la carte Lovelace [power-flow-card-plus](https://github.com/flixlix/power-flow-card-plus) disponible sur HACS. 

L'intégration `MyVirtualBattery` fournit les entités nécessaires pour alimenter les champs "Individual" ou "Battery" de cette carte.

📂 Ressources Techniques 

Vous trouverez dans le [dossier](https://github.com/kaceby/My_Virtual_Battery/tree/main/logic_reference) les fichiers YAML originaux. Ils servent de spécification fonctionnelle :
    sensors_logic.yaml : La gestion des flux de puissance.
    battery_logic.yaml : L'algorithme de gestion du stock et des tarifs.
    recalibration_logic.yaml : La méthode SQL pour le recalage historique (point crucial du projet).
