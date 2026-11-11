# Installation pas à pas

Compter 20 à 30 minutes. Rien n'agit sur tes kits tant que tu n'as pas activé **Zendure Pilotage actif** (étape 7).

---

## 1. Vérifier que tes kits remontent en MQTT local

Dans **Paramètres > Appareils et services > MQTT**, chaque kit doit apparaître comme un appareil avec, entre autres, ces entités :

| Entité (nom d'origine) | Rôle |
|---|---|
| `electricLevel` | niveau de batterie (%) |
| `solarInputPower` | production solaire (W) |
| `outputHomePower` | puissance envoyée à la maison (W) |
| `gridInputPower` | puissance prise au réseau pour charger (W) |
| `outputPackPower` | puissance qui entre dans la batterie (W) |
| `outputLimit` / `inputLimit` | consignes de sortie / de charge (number) |
| `minSoc` | Min SOC du boîtier (number) |
| `acMode` | sélecteur « Input mode / Output mode » |
| `gridReverse` | sélecteur d'export |
| `<n° de pack>_maxVol` / `_minVol` / `_batCur` | tension cellule max / min, courant du pack |

Si ces entités n'existent pas, la régulation ne peut pas fonctionner : il faut d'abord passer tes kits en MQTT local.

## 2. Renommer les appareils

Pour chaque kit : **ouvrir l'appareil > ✏️ > Nom** :

- `Zendure Kit1`, `Zendure Kit2`, `Zendure Kit3`
- Home Assistant propose de **renommer aussi les ID d'entités** : **accepter**.

Résultat attendu : `sensor.zendure_kit1_electriclevel`, `number.zendure_kit1_outputlimit`, `select.zendure_kit1_acmode`, etc.

> Avec 2 kits, utilise `Kit1` et `Kit2`. Le Kit3 absent est ignoré automatiquement.

## 3. Renommer les 3 capteurs de cellules

Les capteurs de cellules gardent le numéro de série du pack dans leur nom (ex. `sensor.zendure_kit1_co4xxxxxxxxxxxx_maxvol`). Renomme leur **ID d'entité** :

| Avant (exemple) | Après |
|---|---|
| `sensor.zendure_kit1_<n° de pack>_maxvol` | `sensor.zendure_kit1_pack_maxvol` |
| `sensor.zendure_kit1_<n° de pack>_minvol` | `sensor.zendure_kit1_pack_minvol` |
| `sensor.zendure_kit1_<n° de pack>_batcur` | `sensor.zendure_kit1_pack_batcur` |

Même chose pour Kit2 et Kit3. Avec plusieurs batteries par kit, prends le **premier pack**.

## 4. Activer les packages (si ce n'est pas déjà fait)

Dans `configuration.yaml` :

```yaml
homeassistant:
  packages: !include_dir_named packages
```

## 5. Copier les 2 fichiers

| Fichier du dépôt | Destination dans ta config HA |
|---|---|
| `packages/zendure_multikit.yaml` | `config/packages/zendure_multikit.yaml` |
| `custom_templates/zendure_multikit.jinja` | `config/custom_templates/zendure_multikit.jinja` |

Crée les dossiers `packages` et `custom_templates` s'ils n'existent pas (avec l'add-on File editor ou Studio Code Server).

## 6. Adapter les 3 lignes ⚠️ À ADAPTER

Ouvre `zendure_multikit.yaml` et cherche `À ADAPTER` :

1. **Compteur réseau** (capteur *Zendure Compteur réseau*), **2 fois la même ligne** :
   ```jinja
   {%- set compteur = 'sensor.MON_COMPTEUR_RESEAU_PUISSANCE' -%}
   ```
   → mets l'entité de ton compteur. **Positif = tu consommes sur le réseau.** Si ton compteur est inversé, écris `{{ - states(compteur) | float(0) }}` à la place de la ligne d'état.

2. **Gros consommateurs** (capteur *Zendure Déduction hiver*) : liste des capteurs de puissance des appareils à garder sur le réseau en mode hiver. Liste vide `[]` = fonction désactivée.

3. **Téléphone** (script *Zendure - Notifier*), facultatif :
   ```jinja
   {{ 'notify.mobile_app_mon_telephone' }}
   ```
   Laisser `{{ '' }}` = notifications dans Home Assistant seulement. Une faute de frappe ici n'empêche jamais la régulation de tourner.

## 7. Redémarrer et démarrer

1. **Outils de développement > YAML > Vérifier la configuration**, puis **Redémarrer**.
2. Au premier démarrage, tous les réglages prennent leur valeur conseillée (automatisation *Valeurs par défaut*).
3. Ajouter la vue `dashboards/zendure_multikit_vue.yaml` à un tableau de bord (*Modifier > Éditeur de configuration brute*, coller sous `views:`).
4. Régler pour chaque kit : **Min SOC** (20 % par défaut) et **Capacité batterie** (1920 Wh = une AB2000).
5. Régler **Limite injection totale** (800 W par défaut).
6. Activer **Zendure Pilotage actif**.

## 8. Vérifier que ça tourne

- *Zendure Compteur réseau* affiche ta consommation en direct.
- Les 3 *Consigne idéale* bougent avec la consommation.
- Après quelques secondes, les `outputLimit` des kits suivent les consignes.
- Le compteur réseau tend vers **0 à quelques dizaines de watts** près.

En cas de souci : **[DEPANNAGE.md](DEPANNAGE.md)**.

---

## Désinstaller

Désactiver *Zendure Pilotage actif*, supprimer les 2 fichiers, redémarrer. Les kits gardent leur dernière consigne : remets-les en mode normal depuis l'appli Zendure si besoin.
