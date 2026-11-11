# Journal des versions

## 2.0.0 — réécriture complète

Basée sur la régulation qui tourne en production chez l'auteur (V8.6), rendue installable par n'importe qui.

**Installation**
- Un seul package YAML + un fichier Jinja, plus de helpers à créer à la main.
- Noms d'entités réels de l'intégration MQTT locale Zendure : il suffit de renommer les appareils en `Zendure Kit1/2/3`.
- Seulement 3 lignes à adapter (compteur, gros consommateurs, téléphone).
- Valeurs conseillées appliquées automatiquement au premier démarrage.
- Vue de tableau de bord en cartes natives (rien à installer).

**Régulation**
- Calcul des consignes dans une macro commune aux 3 kits (au lieu de 3 copies).
- 13 réglages modifiables depuis le tableau de bord, avec bornes de sécurité (le bypass OFF ne peut plus dépasser le ON).
- Marge « jour gris » cohérente entre les kits.

**Calibrage LFP**
- Fin de charge détectée sur la **tension des cellules** (vrai plein), plus sur la jauge.
- Abandon propre et notification si le vrai plein n'est pas atteint en 10 h.
- Min SOC mémorisé : restauré même après un redémarrage pendant le calibrage.
- Heure de départ calculée selon la capacité réelle du kit.
- Armement automatique en option.

**Nouveautés**
- Contrôle de plausibilité des jauges BMS, avec exclusion du kit suspect de la décharge et retour automatique.
- Alerte « kit muet » (MQTT).
- Notification centralisée : une erreur dans le service téléphone ne bloque jamais la régulation.

**Retiré de la base** (trop spécifique à une installation) : ventilation des hubs, réveil HTTP au démarrage, bascule cloud/zenSDK, briefing météo, HUD financier.

## 1.x — version initiale (archivée dans la branche `v1-archive`)
