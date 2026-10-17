# Aegis Zendure Ecosystem — Multi-Kit SolarFlow (Home Assistant)

<a href="https://buymeacoffee.com/spyker67600" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Architecture de pilotage local avancée pour réguler **3 kits Zendure SolarFlow** en parallèle sur Home Assistant.

Ce projet résout les limitations d'origine des installations multi-hubs : élimination du soutirage réseau parasite, suppression de l'usure prématurée des relais mécaniques (*anti-flapping*), équilibrage dynamique de charge (*vases communicants*) et supervision industrielle complète.

---

## 🌟 Fonctionnalités clés

* **Moteur de pilotage haute fréquence (Core Engine) :** Régulation temps réel sous 1 seconde avec filtre anti-spam MQTT (seuil d'intervention $\ge 20\text{ W}$) pour préserver les onduleurs.
* **Algorithme des Vases Communicants :** Mutualisation logicielle des 3 batteries physiques en un parc unique. Répartition dynamique selon l'ensoleillement et l'état de charge (SOC) de chaque kit.
* **Coussin dynamique anti-siphon :** Offset mathématique sur la mesure réseau pour garantir une injection zéro sans soutirage involontaire sur le réseau public.
* **Filet de sécurité & Chien de garde (Watchdogs) :** Bascule automatique sur un talon de survie (50 W par kit) en cas de défaillance du capteur principal, réveil local automatique des hubs après redémarrage et protection contre la décharge profonde sous 6 %.
* **Gestion hivernale & Dérogation consommateurs :** Déduction instantanée de la puissance des appareils énergivores ciblés (four, lave-vaisselle, radiateurs) pour éviter de vider les réserves de batterie. Alerte météo anti-gel proactive.
* **SOC minimum différé :** Application de la réserve matérielle uniquement quand le soleil a chargé la batterie, évitant les recharges forcées payantes sur le réseau Enedis.
* **Maintenance LFP automatisée :** Cycles programmables de calibrage à 100 % puis décharge lente à 12 % pour réaligner les BMS, avec déclenchement nocturne en heures creuses.
* **Tableaux de bord dédiés (SCADA & Netrunner HUD) :** 
  * Synoptique animé en SVG avec flux en temps réel, alertes gel et commandes manuelles.
  * HUD financier avec calcul dynamique du retour sur investissement (ROI) et date d'amortissement prévisionnelle.

---

## 📦 Architecture du projet

Le projet est découpé en modules autonomes et progressifs :

| Lot | Désignation | Description |
| :--- | :--- | :--- |
| **Lot 1** | Cœur & Vases Communicants | Automate principal, capteurs virtuels de consigne et arbitrage du kit maître. |
| **Lot 2** | Sécurités & Watchdogs | Repli en cas de coupure de communication, filet sous 6 % et réveil d'urgence. |
| **Lot 3** | Énergie & Hivernage | Dérogation gros consommateurs, SOC différé et alerte gel. |
| **Lot 4** | Maintenance LFP | Automatisation du calibrage complet des batteries et planification nocturne. |
| **Lot 5** | Utilitaires & Matériel | Bascule ZenSDK / Cloud et régulation thermique active des hubs. |
| **Lot 6** | Interfaces (Dashboards) | Cartes Lovelace SCADA temps réel et HUD d'amortissement. |

---

## 🛠️ Prérequis

1. **Home Assistant** (version récente avec prise en charge des templates Jinja2 modernes).
2. **Intégration Zendure** : accès local privilégié via **zenSDK** ou passerelle MQTT locale.
3. **Mesure de puissance générale** : capteur réseau principal (ex. Shelly EM / 3EM ou tore Linky) avec rafraîchissement rapide (< 2 s).
4. **Composants d'interface (via HACS)** :
   * `custom:html-template-card`
   * `custom:apexcharts-card`

---

## 🚀 Mise en œuvre

1. **Création des Helpers :** Créer les `input_boolean`, `input_number` et `counter` requis dans l'interface de Home Assistant (liste détaillée dans `packages/zendure_helpers.yaml`).
2. **Déploiement des Capteurs Virtuels :** Intégrer les templates de calcul de consigne dans votre configuration YAML.
3. **Import des Automatisations :** Importer les scripts de pilotage et les sécurités.
4. **Intégration des Vues :** Créer un tableau de bord en mode panneau (*panel*) et y coller les cartes Lovelace fournies.
