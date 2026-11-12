# Comment ça marche

## La chaîne de décision

```
Compteur réseau ─┐
Solaire des kits ─┼─► Macro « consigne idéale » ─► 3 capteurs Consigne idéale ─► Pilotage global ─► outputLimit / inputLimit / acMode
Batteries (SOC)  ─┘         (zendure_multikit.jinja)                              (n'envoie un ordre que si l'écart
Réglages ────────┘                                                                  dépasse « Écart mini avant ordre »)
```

1. **Besoin de la maison** = ce que les kits injectent + ce que le compteur mesure − la recharge réseau en cours − un petit **coussin** (pour viser légèrement l'import plutôt que l'export).
2. **Solaire direct** : chaque kit couvre d'abord le besoin avec son solaire. Le **kit maître** (le plus productif) passe en premier, les autres complètent au prorata de leur production.
3. **Batteries** : s'il manque encore, le manque est réparti entre les batteries disponibles, pondéré par le **carré** de leur réserve utile (au-dessus du Min SOC). Une batterie à 90 % travaille bien plus qu'une à 55 %, ce qui évite qu'une batterie presque vide fasse des micro-cycles.
4. **Pilotage** : applique les consignes, avec une tolérance pour ne pas inonder les boîtiers d'ordres MQTT.

## Les états particuliers d'un kit

| État | Quand | Effet |
|---|---|---|
| **Bypass** | Batterie au-dessus de *Bypass ON* | Le solaire du kit part en direct (moins le *Tampon bypass*). Libéré sous *Bypass OFF*. |
| **Sprint** | Batterie ≥ 98 % et > 150 W de solaire, pas encore en bypass | Sortie à 0 pour finir de remplir la batterie. |
| **Jauge suspecte** | Contrôle de plausibilité | Plus de bypass, de sprint ni de décharge : solaire direct seulement, jusqu'au prochain vrai plein. |
| **Calibrage / charge d'urgence / muet** | — | Le Pilotage ne touche plus à ce kit et redistribue sa part aux autres. |

## Les réglages

| Réglage | Défaut | À quoi il sert | Monter si… / Baisser si… |
|---|---|---|---|
| Coussin | 7 W | Marge visée côté import | Tu exportes un peu en continu / Tu importes trop |
| Coussin charge réseau | 40 W | Même chose pendant une recharge réseau | Idem |
| Seuil gros consommateur | 30 W | En dessous, un appareil de la liste hiver n'est pas déduit | Les veilles sont comptées à tort |
| Écart mini avant ordre | 20 W | Tolérance avant d'envoyer un nouvel ordre | Trop d'ordres MQTT / Régulation trop lente |
| Marge anti-nuage | 100 W | Quand **tous** les kits sont en bypass, marge de sortie au-dessus du besoin | Coupures au passage d'un nuage |
| Tampon bypass | 15 W | Ce qu'un kit en bypass garde sous son solaire | Export en bypass |
| Marge SOC | 2 % | Réserve au-dessus du Min SOC avant de décharger | Micro-cyclage autour du Min SOC |
| Marge SOC jour gris | 10 % | Même chose quand le kit produit entre 10 et 100 W | Tu veux garder plus de réserve les jours gris |
| Bypass ON / OFF | 99 / 95 % | Hystérésis du bypass (bornes : ON 96–100, OFF 85–95) | Claquements de relais → écarter les deux |
| Calibrage charge / décharge | 800 / 250 W | Puissances pendant le calibrage | — |
| Calibrage fin | 13 % | Niveau où s'arrête la décharge de calibrage | — |
| Intervalle calibrage | 35 j | Délai entre deux calibrages | — |

## Le calibrage LFP

La jauge d'une batterie LFP dérive : la tension varie très peu entre 20 et 90 %, le BMS « compte » les ampères et se trompe petit à petit. Seul un **vrai plein** la recale.

1. Le capteur *Heure calibrage* calcule l'heure de départ pour que charge + décharge soient finies au lever du soleil.
2. Le kit charge à la puissance réglée jusqu'au **vrai plein** : une cellule dépasse 3,50 V, ou jauge à 99 % avec un courant faible et des cellules ≥ 3,45 V pendant 5 min. Sans vrai plein en 10 h : abandon propre et notification.
3. Au vrai plein, le drapeau *Jauge suspecte* est levé.
4. Décharge lente jusqu'au seuil de fin, puis le Min SOC d'origine est remis (il est mémorisé, même si Home Assistant redémarre entre-temps).

Armement : manuel (interrupteur *Armement calibrage*) ou automatique avec *Calibrage automatique* quand le décompte arrive à 0.

## Le contrôle de plausibilité des jauges

Une jauge est déclarée **suspecte** si :
- elle saute de **10 points ou plus sans charge**, sauf vers 99-100 % ou vers 0-5 % : ce saut-là est le BMS qui se recale tout seul, il lève au contraire l'alerte ;
- elle affiche **≥ 95 %** alors que les cellules sont **sous 3,34 V** au repos pendant 15 min ;
- elle affiche **≤ 15 %** alors que les cellules sont **≥ 3,28 V** au repos pendant 15 min.

Le kit est alors exclu de la décharge et le motif s'affiche dans le tableau de bord. Retour à la normale automatique au prochain vrai plein (calibrage), vrai vide ou recalage du BMS.

## Les sécurités

| Situation | Réaction |
|---|---|
| Batterie sous 6 % | Recharge réseau à 400 W jusqu'à 10 % |
| Compteur réseau indisponible 1 min | Pilotage coupé, kits bridés à 50 W, relance automatique au retour |
| Kit muet 15 min | Exclu du pilotage + notification, effacée à son retour |
| Home Assistant arrêté | Les kits gardent leur consigne ; le Min SOC est écrit dans le boîtier, qui le respecte seul |
