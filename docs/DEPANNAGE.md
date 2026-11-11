# Dépannage

**Les consignes restent à 0**
- *Zendure Pilotage actif* est-il activé ?
- *Zendure Compteur réseau* affiche-t-il une valeur ? Sinon, revoir la ligne ⚠️ du compteur.
- Les batteries sont-elles au-dessus de Min SOC + Marge SOC ? En dessous, c'est normal : seul le solaire direct est utilisé.
- Une *Jauge suspecte* est-elle activée ? Le kit ne fait alors que du solaire direct.

**`TemplateNotFound: zendure_multikit.jinja`**
Le fichier doit être dans `config/custom_templates/` (même niveau que `configuration.yaml`), pas dans un sous-dossier. Puis *Outils de développement > YAML > Recharger les modèles personnalisés*.

**Entités `unknown` / avertissements « unknown entity sensor.zendure_kitN_… » au démarrage**
Les appareils ou les capteurs de cellules n'ont pas été renommés (étapes 2 et 3 de l'installation). Avec seulement 2 kits, les avertissements sur le Kit3 sont normaux.

**J'exporte un peu en continu**
Monter le *Coussin* de 5 en 5 W.

**Les relais claquent souvent** (compteurs de commutations élevés)
Écarter *Bypass ON* et *Bypass OFF*, monter *Écart mini avant ordre*.

**Le compteur réseau est à l'envers** (négatif quand je consomme)
Dans le capteur *Zendure Compteur réseau*, remplacer `{{ states(compteur) | float(0) }}` par `{{ - states(compteur) | float(0) }}`.

**Les notifications n'arrivent pas sur le téléphone**
Vérifier le nom du service dans le script *Zendure - Notifier* (*Outils de développement > Actions*, chercher `notify.`). Les notifications restent visibles dans Home Assistant dans tous les cas.

**Ouvrir une issue**
Joindre : version de HA, nombre et modèle de kits, une capture de la vue Zendure Multi-Kit, et les lignes du journal contenant `zendure`.
