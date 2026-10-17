# Liste des Entités et Prérequis (Helpers HA)

### 1. Les entrées virtuelles à créer manuellement (Helpers HA)
* **Liste déroulante (`input_select`)** : 
  * `kit_maitre_solaire` (contenant exactement trois options : `kit_1`, `kit_2`, et `kit_3`).
* **Interrupteurs (`input_boolean`)** : 
  * `mode_hiver`
  * `bypass_1`, `bypass_2`, `bypass_3`
  * `charge_urgence_1`, `charge_urgence_2`, `charge_urgence_3`
  * `export_solaire_zendure`
* **Curseurs numériques (`input_number`)** : 
  * `target_min_soc_1`, `target_min_soc_2`, `target_min_soc_3`
  * `prix_kwh_electricite` (pour le calcul financier dynamique).

### 2. Les entités matérielles requises (Capteurs existants à lier)
* **Capteurs Zendure** : 
  * Production solaire de chaque kit (via `solarinputpower`).
  * Niveau de batterie via `electriclevel` (SOC).
  * Température interne du BMS (`hyper_tmp`).
  * Autorisation d'injection réseau (`grid_reverse`).
  * Réglages matériels de limites (`minsoc`, `outputlimit`).
  * Puissances des sous-composants (charge batterie, sortie maison).
* **Capteurs Maison** : 
  * La liste des capteurs de puissance de vos propres gros électroménagers (four, air fryer, radiateurs, etc.).
* **Compteurs Globaux** : 
  * Un capteur de production quotidienne (`daily_solar_production`).
  * Un capteur de production totale (`total_solar_production`).
* **Capteurs Météo** : 
  * Un capteur de température extérieure locale (`sensor.home_temperature`).
  * Un capteur de probabilité de gel (`sensor.freeze_chance`).

### 3. Les services et dépendances logicielles
* **Commandes locales (`rest_command`)** : 
  * Commande API `reveil_solarflow_X` pour réveiller les kits.
* **Notifications** : 
  * Un service de notification mobile classique configuré sur Home Assistant (`notify.mon_smartphone`).
* **Ressources Front-End (via HACS)** : 
  * Installation préalable des modules `custom:html-template-card` et `custom:apexcharts-card`.
