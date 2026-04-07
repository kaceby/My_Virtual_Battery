 ==========================================================

 LOGIQUE DE RECALAGE (OFFSET DYNAMIQUE)

 ==========================================================
 Objectif : Aligner le solde HA sur l'index réel du fournisseur.
 Problème : Le calcul local dérive par rapport au compteur Enedis/Fournisseur.
 Solution : Calculer un "Offset" de correction à une date T précise.

----------------------------------------------------------

 1. PARAMÈTRES REQUIS (Inputs fournis par l'utilisateur)
 
 - Date du relevé (T) : ex. 2026-03-13 
 - Index Réel au moment T : ex. 450 kWh 

----------------------------------------------------------

 2. RÉCUPÉRATION DES DONNÉES HISTORIQUES

 L'intégration doit interroger la base de données (Recorder) pour extraire 
 les valeurs cumulées à la date T précise. 

 Requête logique (SQL actuel) :
 SELECT state FROM states 
 WHERE entity_id = 'sensor.energy_battery_power_in' 
 AND last_updated <= 'Date_du_relevé_T' 
 ORDER BY last_updated DESC LIMIT 1 

 On obtient deux valeurs historiques :
 - In_passé  (Total exporté à la date T) 
 - Out_passé (Total importé de la batterie à la date T) 

 ==========================================================

 PRÉ-REQUIS SYSTÈME : CONFIGURATION DU RECORDER

 ==========================================================

 IMPORTANT : Pour que le recalage historique fonctionne, Home Assistant 
 doit conserver les données en base de données suffisamment longtemps.

 La facture Urban Solar arrivant généralement en milieu de mois suivant, 
 un historique de 15 jours (minimum) est recommandé.

 Configuration requise dans configuration.yaml :
 recorder:
   purge_keep_days: 15
 
 L'intégration devrait idéalement vérifier cette valeur ou avertir 
 l'utilisateur si 'purge_keep_days' est trop court.
 
----------------------------------------------------------
 
 3. CALCUL DU NOUVEL OFFSET
 
 La formule pour que le solde devienne égal à l'index réel à l'instant T :
 Nouveau_Offset = Index_Réel - (In_passé - Out_passé) 

----------------------------------------------------------

 4. APPLICATION DU SOLDE EN TEMPS RÉEL
 
 Le capteur de solde final utilise cet offset en permanence :
 Solde_Actuel = (In_actuel - Out_actuel) + Nouveau_Offset 

----------------------------------------------------------

 5. SÉCURITÉ MÉTIER

 - Le Solde ne peut jamais être négatif : max(Solde, 0) 
 - Si le déstockage est impossible (Batterie < 0.1 kWh), 
   le capteur 'Out' doit cesser de compter. 
