# NAIAD, Scenario 1

Déployer depuis un porteur cinq UUV tier-3 sur des points précis. Un UUV tier-1 les guide en troupeau en se recalant sur des balises larguées au préalable, les fait atterrir un par un, puis contrôle leurs positions.

## Sources

Le modèle s'appuie sur les deux documents du projet, dans `sources/` :
- `Naiad.pptx` : description de la mission ; les numéros de slides cités ci-dessous y renvoient ;
- `descritpionObjectifsVehicules.xlsx` : objectifs de chaque véhicule ; les UID cités (T1 #5, T3 #7…) y renvoient.

## La mission sous forme d'arbre

La tâche racine `mission t1` se décompose en cinq étapes, celles des slides 17 à 21. Chaque étape se décompose à son tour jusqu'aux feuilles : les actions réellement exécutées par les véhicules.

```text
mission
├─ 1. preparer-la-flotte          le T1 et les T3 se dégagent du lanceur et se mettent en circuit d'attente ;
│                                 le T1 rassemble les T3 en troupeau et contrôle qu'ils sont tous là
├─ 2. aller-a-la-zone-de-depot    pour chaque balise de l'itinéraire : la rallier, la rechercher, se recaler ;
│                                 puis mener le troupeau dans la zone de dépôt
├─ 3. deposer-les-t3              le troupeau attend ; un par un (le moins d'énergie d'abord),
│                                 le T1 guide un T3 et le T3 atterrit sur son point final
├─ 4. verifier-les-positions      le T1 visite chaque point final et confirme la position ;
│                                 un T3 mal placé redécolle et se repose
└─ 5. terminer-la-mission-t1      le T1 revient par les balises et se docke sur le porteur,
                                  ou, si le porteur ne peut pas le récupérer, s'océanise
```


## Actions = objectifs du tableur

Chaque action est un objectif de `descritpionObjectifsVehicules.xlsx`, sauf les quatre marquées ✱, qui viennent des slides.

| Action | Objectif (feuille, UID) |
|---|---|
| `se-degager-du-lanceur` | T1 #1, T3 #1 : se dégager du lanceur |
| `circuit-attente-initial` | T1 #2, T3 #2 : se mettre en circuit d'attente initial |
| `rassembler-le-troupeau` | T1 #3 : rassembler les t3 (les T3 se mettent à suivre le T1, T3 #3) |
| `controler-integrite-troupeau` | T1 #4 : contrôler l'intégrité du troupeau |
| `rallier-balise-avec-troupeau`, `rallier-balise-seul` | T1 #5 : rallier une balise |
| `rallier-zone-depot-avec-troupeau` | T1 #6 : rallier la zone de dépôt |
| `rechercher-balise-avec-troupeau` | T1 #7 : rechercher une balise avec troupeau |
| `rechercher-balise-seul` | T1 #8 : rechercher une balise seul (troupeau en attente, ou au retour) |
| `se-localiser-sur-balise` | T1 #9 : se localiser par rapport à une balise |
| `troupeau-en-circuit-attente` | T3 #4 : se mettre en circuit d'attente t3 |
| `troupeau-suit-t1`, `rejoindre-le-troupeau` | T3 #3 : suivre t1 (après une recherche de balise par le T1 seul) |
| `signaler-balise-absente` ✱ | slide 22 : balise introuvable à l'endroit indiqué |
| `appeler-un-t3` ✱ | slide 19 : le T3 suivant (le moins d'énergie d'abord) quitte le circuit d'attente, les autres continuent d'attendre |
| `guider-t3` ✱ | slide 19 : localisation d'un T3 par le T1 avant l'atterrissage |
| `atterrir-sur-point-final` | T3 #7 : atterrir sur le point de dépôt final |
| `rallier-point-final`, `confirmer-position-t3`, `detecter-mauvais-placement` ✱ | T1 #15 : vérifier les localisations finales des t3 |
| `redecoller` | T3 #8 : redécoller |
| `rallier-le-porteur`, `se-docker-sur-le-porteur` | T1 #16, #17 : rallier le porteur, se docker sur le porteur |
| `rallier-zone-oceanisation`, `s-oceaniser-seul` | T1 #13 : s'océaniser seul |

## Ce que décrit un problème

Un problème est le **plan de mission** :
- les véhicules à bord du porteur (`navire`) ;
- l'itinéraire (`zone-attente → balise-1 → balise-2 → balise-3 → zone-depot`) ;
- le point final de chaque T3 ;
- l'ordre d'atterrissage (le moins d'énergie restante d'abord) ;
- les recalages assez précis pour déposer dans la zone de dépôt (`assez-precis balise-3 zone-depot`).

Tout est connu à l'avance, comme en planification HTN classique, y compris ce qui, en mission réelle, ne se découvrirait que sur place (une balise introuvable, un atterrissage raté). Chaque scénario montre donc à quoi ressemble la mission dans ce cas. **Chaque scénario est le problème nominal plus un seul fait** (ou un autre ordre) :

| Fait de scénario | Signification | Effet sur le plan |
|---|---|---|
| `(balise-hs b)` | balise signalée hors service avant le départ | b est ignorée, à l'aller et au retour |
| `(balise-introuvable b)` | balise introuvable à l'endroit prévu | b est recherchée, signalée absente, pas de nouveau recalage |
| `(recherche-seul b)` | le troupeau ne doit pas entrer dans la zone de b | le troupeau attend, le T1 recherche b seul et revient |
| `(atterrissage-rate u)` | le premier atterrissage de u finit hors cible | le contrôle le détecte ; u redécolle et se repose |
| `(porteur-indisponible)` | le porteur ne peut pas récupérer le T1 | le T1 s'océanise au lieu de revenir |

Les faits connus avant le départ (`balise-hs`, `recherche-seul`, `porteur-indisponible`) orientent les choix de la hiérarchie. Ceux qui ne se découvrent qu'en mission (`balise-introuvable`, `atterrissage-rate`) ne sont lus que par l'action qui les découvre, la recherche ou l'atterrissage : la hiérarchie réagit ensuite au résultat observé (`balise-trouvee`, `bien-place`). Le plan se lit donc dans l'ordre réel : on recherche la balise, puis, faute de la trouver, on la signale absente.

**Règle de précision.** Le T1 ne peut mener le troupeau dans la zone de dépôt que s'il a obtenu un recalage assez précis pour cette zone (GPS avant la plongée, ou une balise). C'est ce qui donne de l'importance à une balise manquante.

## Scénarios

| Problème | Changement par rapport au nominal | Ce que fait le plan | Actions |
|---|---|---|---:|
| `p01_nominal` | — | 3 balises, 5 atterrissages, 5 contrôles, retour par les balises, docking | 61 |
| `p02_balise_hs` | balise 2 hors service | la balise 2 n'est jamais visitée (décomposition vide ∅) | 55 |
| `p03_balise_introuvable` | balise 2 introuvable | recherchée, signalée absente ; la balise 3 donne le recalage | 61 |
| `p04_derniere_balise_introuvable` | balise 3 introuvable ; balise 2 assez précise | entrée dans la zone de dépôt avec le recalage de la balise 2 | 61 |
| `p05_derniere_balise_introuvable_sans_plan` | balise 3 introuvable, aucun autre recalage assez précis | **pas de plan** : la règle de précision ne peut pas être respectée | — |
| `p06_recherche_seul` | balise 2 recherchée seul | le troupeau attend à la balise 1 ; le T1 recherche la balise 2 seul et revient ; le troupeau le suit jusqu'à la balise 3 | 64 |
| `p07_autre_ordre_atterrissage` | autres énergies restantes | atterrissages et contrôles dans l'ordre t3-4, t3-2, t3-5, t3-1, t3-3 | 61 |
| `p08_atterrissage_rate` | le T3-2 rate son atterrissage | détecté au contrôle ; il redécolle, est guidé, se repose | 65 |
| `p09_sans_recuperation` | porteur indisponible | après les contrôles, le T1 va à la zone d'océanisation | 52 |
| `p10_sans_balise` | aucune balise ; GPS assez précis | directement du circuit d'attente à la zone de dépôt | 43 |
| `p11_mauvaise_journee` | balise 1 hors service, balise 2 seul, atterrissage raté, pas de récupération | les quatre comportements dans un seul plan | 56 |

## Validation

Les plans de `validation/` ont été produits par le planificateur HTN [SibylSat](https://github.com/gaspard-quenard/sibylsat), avec en entrée le domaine et le problème :

```sh
sibylsat domaine.hddl problemes/p01_nominal.hddl -vp=1
```

Chaque fichier de `validation/` est la sortie complète du planificateur. Le plan est le bloc entre `==>` et `<==` : les actions dans l'ordre d'exécution, puis la décomposition de chaque tâche (`tâche -> méthode sous-tâches`).
- Avec `-vp=1`, SibylSat fait vérifier chaque plan par pandaPIparser (ligne « Plan has been verified by pandaPIparser »).
- Pour p05, SibylSat prouve qu'aucun plan n'existe (« Problem is impossible »).

Pour montrer un scénario, allez sur https://gaspard-quenard.github.io/htn-plan-player/ et déposez-y trois fichiers : le journal de `validation/`, `domaine.hddl` et le problème. Comparer p01 avec une variante montre exactement ce que change le fait ajouté.

## Limites

- Les T3 se déplacent en un seul troupeau : la perte d'un T3 (slide 23, T1 #10–11) n'est pas modélisée, ni la perte du T1.
- Objectifs du tableur non modélisés : localisation sur plusieurs balises (T1 #12), océanisation avec le troupeau (T1 #14), T3 ralliant seul la zone de dépôt ou s'océanisant seul (T3 #5–6).
- Les positions sont symboliques (`balise-2`, `final-3`). Courants, énergie, durées et portées acoustiques ne sont pas représentés. L'ordre d'atterrissage et les faits `assez-precis` sont les endroits où ces calculs entrent, comme données.
