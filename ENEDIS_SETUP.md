# ⚡ Intégration Enedis / HA Linky

Guide complet pour connecter vos index Enedis (consommation et production)
à Home Assistant via l'add-on **HA Linky** et les exploiter dans des sensors
utilisables dans des automations.

---

## 1. Prérequis côté Enedis

1. Créer un compte sur [mon-compte.enedis.fr](https://mon-compte.enedis.fr/)
2. Rattacher votre compteur Linky au compte
3. Activer la collecte horaire :
   *Mon compteur → Enregistrement et collecte de mes données*

---

## 2. Générer un token via Conso API

L'API Enedis n'est accessible qu'aux entreprises — **Conso API** fait office
d'intermédiaire gratuit et open-source.

1. Aller sur [conso.boris.sh](https://conso.boris.sh/)
2. Cliquer sur *J'accède à mon espace client Enedis*
3. Accepter le consentement de partage
4. **Noter précieusement le token généré** (valable 3 ans)

> ⚠️ Le partage expire après **3 ans** — à renouveler à cette échéance.

---

## 3. Installer l'add-on HA Linky

Dans Home Assistant :
*Paramètres → Modules complémentaires → Boutique → ⋮ → Dépôts*

Ajouter l'URL : `https://github.com/bokub/ha-linky`

Rechercher *Linky*, installer, activer *Chien de garde* et *Mise à jour automatique*.

### Configuration YAML minimale

Dans l'onglet *Configuration → Modifier en YAML* :

```yaml
meters:
  - prm: "VOTRE_PRM_14_CHIFFRES"
    token: >-
      VOTRE_TOKEN_CONSO_API
    name: Linky consommation
    action: sync
    production: false

  - prm: "VOTRE_PRM_14_CHIFFRES"
    token: >-
      VOTRE_TOKEN_CONSO_API
    name: Linky production
    action: sync
    production: true
costs: []
```

> Le PRM (14 chiffres) se trouve sur votre compteur (touche +)
> ou dans votre espace client Enedis.
>
> Le PRM de production peut être identique ou différent de celui
> de consommation selon votre installation.

### Vérification

Dans l'onglet *Journal / Log*, vous devez voir :

```
Successfully retrieved daily consumption data from ...
Successfully retrieved daily production data from ...
Data import returned XXX data points
```

Si la production affiche `Everything is up-to-date` sans données :
passer `action: reset` sur le compteur production, redémarrer,
puis repasser `action: sync`.

### Synchronisation

HA Linky synchronise automatiquement **deux fois par jour** :
- Entre **6h et 7h** — import des données de la veille
- Entre **9h et 10h** — rattrapage si la première synchro a échoué

Pour les 7 derniers jours : données par **demi-heure**.
Au-delà : données **journalières**.

---

## 4. Récupérer les IDs de statistiques

Les données HA Linky sont stockées comme **statistiques** dans la base HA,
pas comme des entités classiques. Il faut retrouver les IDs numériques
pour les requêtes SQL.

### Installer SQLite Web

*Paramètres → Modules complémentaires → Boutique → SQLite Web*

Activer *Afficher dans la barre latérale*, démarrer l'add-on.

### Requête de recherche

Dans l'interface SQLite Web, onglet *Execute SQL* :

```sql
SELECT id, statistic_id, name, unit_of_measurement
FROM statistics_meta
WHERE statistic_id LIKE '%VOTRE_PRM%';
```

Résultat attendu :

| id | statistic_id | name | unit_of_measurement |
|---|---|---|---|
| 178 | linky:24176845140708 | Linky consommation | Wh |
| 179 | linky_prod:24176845140708 | Linky production | Wh |

> ⚠️ Notez le préfixe différent : `linky:` pour la conso, `linky_prod:` pour la
> production quand le PRM est identique.
>
> **Ces IDs sont propres à votre installation.** Adaptez les `metadata_id`
> dans `sql.yaml` avec vos valeurs.

---

## 5. Vérifier la cohérence Enedis vs capteur local

Avant d'activer la synchro automatique, vérifiez que les deux sources
mesurent bien la même grandeur physique.

### Requête production Enedis (jour par jour, mois en cours)

```sql
SELECT 
  date(datetime(start_ts,'unixepoch','localtime')) AS jour,
  ROUND(SUM(state) / 1000.0, 3) AS kwh_enedis
FROM statistics
WHERE metadata_id = 179
AND strftime('%Y-%m', datetime(start_ts,'unixepoch','localtime')) 
    = strftime('%Y-%m', 'now', 'localtime')
GROUP BY jour
ORDER BY jour;
```

### Requête production capteur local (jour par jour, mois en cours)

```sql
SELECT 
  date(datetime(last_updated_ts,'unixepoch','localtime')) AS jour,
  ROUND(MAX(CAST(state AS FLOAT)) - MIN(CAST(state AS FLOAT)), 3) AS kwh_local
FROM states s
JOIN states_meta m ON s.metadata_id = m.metadata_id
WHERE m.entity_id = 'sensor.energy_battery_power_in'
AND state NOT IN ('unknown', 'unavailable')
AND strftime('%Y-%m', datetime(last_updated_ts,'unixepoch','localtime'))
    = strftime('%Y-%m', 'now', 'localtime')
GROUP BY jour
ORDER BY jour;
```

### Interprétation

| Écart jour | Interprétation |
|---|---|
| < 0.5 kWh | ✅ Normal — biais d'intégration (`method: left`) |
| 0.5 – 5 kWh | ⚠️ À surveiller — vérifier redémarrages HA ce jour |
| > 5 kWh | ❌ Sources incohérentes — ne pas activer la synchro |

Un écart systématique de ~0.1 kWh/jour est normal et représente
exactement ce que corrige la synchro automatique (~3 kWh/mois).

---

## 6. Sensors SQL dans sql.yaml

Une fois les IDs confirmés, voici les sensors à configurer.
**Remplacer `178` et `179` par vos propres IDs.**

```yaml
#====================
# Enedis / Linky
# ===================
- name: "Linky - Conso hier"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 178
    AND date(datetime(start_ts,'unixepoch','localtime')) = date('now','localtime','-1 day');
  column: "kwh"
  unit_of_measurement: "kWh"

- name: "Linky - Production hier"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 179
    AND date(datetime(start_ts,'unixepoch','localtime')) = date('now','localtime','-1 day');
  column: "kwh"
  unit_of_measurement: "kWh"

- name: "Linky - Conso mois en cours"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 178
    AND strftime('%Y-%m', datetime(start_ts,'unixepoch','localtime'))
        = strftime('%Y-%m', 'now', 'localtime');
  column: "kwh"
  unit_of_measurement: "kWh"

- name: "Linky - Production mois en cours"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 179
    AND strftime('%Y-%m', datetime(start_ts,'unixepoch','localtime'))
        = strftime('%Y-%m', 'now', 'localtime');
  column: "kwh"
  unit_of_measurement: "kWh"

- name: "Linky - Production année en cours"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 179
    AND strftime('%Y', datetime(start_ts,'unixepoch','localtime'))
        = strftime('%Y', 'now', 'localtime');
  column: "kwh"
  unit_of_measurement: "kWh"

- name: "Linky - Conso année en cours"
  query: >
    SELECT ROUND(SUM(state) / 1000.0, 2) AS kwh
    FROM statistics
    WHERE metadata_id = 178
    AND strftime('%Y', datetime(start_ts,'unixepoch','localtime'))
        = strftime('%Y', 'now', 'localtime');
  column: "kwh"
  unit_of_measurement: "kWh"

#====================
# Recalage dérive batterie virtuelle
# ===================
- name: "Battery In hier minuit"
  query: >
    SELECT s.state 
    FROM states s
    JOIN states_meta m ON s.metadata_id = m.metadata_id
    WHERE m.entity_id = 'sensor.energy_battery_power_in' 
    AND date(datetime(s.last_updated_ts, 'unixepoch', 'localtime')) = date('now', 'localtime', '-1 day')
    ORDER BY s.last_updated_ts DESC LIMIT 1;
  column: "state"
  unit_of_measurement: "kWh"

- name: "Battery In avant hier minuit"
  query: >
    SELECT s.state 
    FROM states s
    JOIN states_meta m ON s.metadata_id = m.metadata_id
    WHERE m.entity_id = 'sensor.energy_battery_power_in' 
    AND date(datetime(s.last_updated_ts, 'unixepoch', 'localtime')) = date('now', 'localtime', '-2 day')
    ORDER BY s.last_updated_ts DESC LIMIT 1;
  column: "state"
  unit_of_measurement: "kWh"
```

---

## 7. Automation de synchro automatique

```yaml
- id: sync_batterie_enedis_quotidien
  alias: "Batterie Virtuelle : Synchro Enedis Quotidienne"
  mode: single

  trigger:
    - trigger: time
      at: "10:30:00"

  condition:
    # Anti-redémarrage : le sensor SQL ne change pas au boot
    - condition: template
      value_template: >
        {{ (as_timestamp(now()) - as_timestamp(states.sensor.linky_production_hier.last_changed)) > 300 }}

    # Données Enedis disponibles et valides
    - condition: template
      value_template: >
        {{ states('sensor.linky_production_hier') | float(-1) > 0 }}

    # Données SQL locales disponibles
    - condition: template
      value_template: >
        {{ states('sensor.battery_in_hier_minuit') not in ['unknown', 'unavailable'] and
           states('sensor.battery_in_avant_hier_minuit') not in ['unknown', 'unavailable'] }}

    # Limite de sécurité : écart max 5 kWh par jour
    - condition: template
      value_template: >
        {% set enedis = states('sensor.linky_production_hier') | float(0) %}
        {% set ha_hier = states('sensor.battery_in_hier_minuit') | float(0) %}
        {% set ha_avant_hier = states('sensor.battery_in_avant_hier_minuit') | float(0) %}
        {% set ecart = (enedis - (ha_hier - ha_avant_hier)) | abs %}
        {{ ecart < 5 }}

  action:
    - action: homeassistant.update_entity
      target:
        entity_id:
          - sensor.linky_production_hier
          - sensor.battery_in_hier_minuit
          - sensor.battery_in_avant_hier_minuit

    - delay: "00:00:03"

    - action: input_number.set_value
      target:
        entity_id: input_number.offset_batterie_virtuelle
      data:
        value: >
          {% set enedis_prod_hier = states('sensor.linky_production_hier') | float(0) %}
          {% set ha_hier = states('sensor.battery_in_hier_minuit') | float(0) %}
          {% set ha_avant_hier = states('sensor.battery_in_avant_hier_minuit') | float(0) %}
          {% set ha_prod_hier = ha_hier - ha_avant_hier %}
          {% set ecart = enedis_prod_hier - ha_prod_hier %}
          {% set offset_actuel = states('input_number.offset_batterie_virtuelle') | float(0) %}
          {{ (offset_actuel + ecart) | round(2) }}
```

---

## 8. Dépannage

### La table statistics_meta est vide après installation

HA Linky n'a pas encore synchronisé. Vérifier les logs de l'add-on.
La première synchro peut prendre quelques minutes au démarrage.

### `Everything is up-to-date` sans données pour la production

Séquelle d'une ancienne configuration. Solution :
1. Passer `action: reset` sur le compteur production
2. Redémarrer l'add-on
3. Repasser `action: sync`, redémarrer à nouveau

### L'automation ne se déclenche pas à 10h30

Vérifier que `sensor.linky_production_hier` existe et n'est pas en `unknown`.
Forcer une mise à jour manuelle via *Outils de développement → Actions →
`homeassistant.update_entity`*.

### Écart > 5 kWh bloquant la synchro

Vérifier si un redémarrage HA a eu lieu la veille (perturbation de l'intégrale Enphase).
Si ponctuel, attendre le lendemain. Si systématique, revoir la cohérence des sources (§5).
