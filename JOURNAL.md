# JOURNAL.md — Journal d'équipe, CEG 3536, projet final (automne 2026)

Équipe : `Simon Boisvert`, `Samuel Caiado`, `Tristan Larabie` et `Hassan Hashish` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536/tree/Projet`
— Variante L4-A attribuée : `0` — Journal du laboratoire 3 : `JOURNAL_Lab3.md`

# Partie I

## Partie I — Rôles et rotation

| Séance | Réalise | Valide | Pilote I²C écrit par chacun | Pilote SPI ou routine assembleur de chacun |
|---|---|---|---|---|
| Séance 1 | `Simon Boisvert` / `Tristan Larabie` | `Samuel Caiado` / `Hassan Hashish` | • **Simon** : Bus I²C (`i2c_scan`, init) & Extenseurs (DEL/SW)<br>• **Samuel** : Écran LCD & Rétroéclairage RGB | • **Simon** : Routine ASM de délai/timing ou `trame_xor8`<br>• **Samuel** : Routine ASM d'écriture/conversion LCD |
| Entre les séances | `Samuel Caiado` / `Hassan Hashish` | `Simon Boisvert` / `Tristan Larabie` | • **Tristan** : TouchPad (MPR121, seuils, électrodes)<br>• **Hassan** : Capteur environnemental (I²C/SPI) | • **Tristan** : Gestionnaire d'interruption THI / garde Touch En<br>• **Hassan** : Pilote SPI du capteur environnemental |
| Séance 2 | `Samuel Caiado` / `Hassan Hashish` | `Simon Boisvert` / `Tristan Larabie` | Intégration et débogage croisé des pilotes I²C (L4-A) | Intégration PIR, télémétrie et tests croisés T1–T10 |

## Partie I — Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| Pas de l'horloge I²C | (PRESC + 1) / 4 MHz |  | T1 |
| SCL bas / SCL haut | (SCLL + 1) × pas / (SCLH + 1) × pas |  | T1 |
| Fréquence SCL (avec les délais de synchronisation) |  |  | T1 |
| `TIMINGR` retenu |  |  | T1 |
| Durée d'une écriture de deux octets de données | 27 bits / fréquence SCL | 290 µs | T1 (DWT) |
| Débit SPI | PCLK2 / 2^(BR + 1) |  | T7 |
| Pire cas de réécriture du LCD | 32 caractères × (position + caractère) × 1 tick |  | T3 |
| Transactions I²C par seconde au repos |  |  | T10 |

## Partie I — Relevés des modules

| Module | Relevé | Valeur | Preuve (fichier) |
|---|---|---|---|
| Bus I²C | Adresses trouvées par `i2c_scan` |  |  |
| Bus I²C | `GPIOF->IDR` bits 0 et 1 au repos ; `I2C2->ISR` après une transaction |  |  |
| Extenseur des DEL | Octet écrit pour allumer Y1 seule ; R4 seule |  |  |
| Extenseur des commutateurs | Octet lu au repos ; SW1 enfoncé ; SW8 enfoncé |  |  |
| LCD | Octets de commande confirmés (fonction, allumage, effacement, mode) |  |  |
| Rétroéclairage | Registres rouge / vert / bleu confirmés |  |  |
| TouchPad | Front actif de THI ; `0x5E` relu après `tp_init` ; deux seuils relus |  |  |
| Capteur environnemental | Identifiant lu ; page utilisée pour chaque registre |  |  |
| Capteur environnemental | Température / humidité / pression au repos ; écart avec le capteur analogique |  |  |
| PIR | Durée du niveau haut ; `pir_t_ms` de deux fronts successifs |  |  |

Table des touches (électrode → touche), relevée au débogueur :

| Électrode | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Touche |  |  |  |  |  |  |  |  |  |  |  |  |

## Partie I — Essais hôte (E11)

| Plateforme | Compilateur et version | `test_evq` (réussis / échecs) | `test_tableau` (réussis / échecs) | Date et validation Git |
|---|---|---|---|---|
| Windows |  |  |  |  |
| Linux |  |  |  |  |
| STM32 (cible) | arm-none-eabi-gcc | commande `q` : | commande `f` : |  |

Cas ajoutés à `tableau_test.c` (au-delà des 6 fournis) :

| Cas | Description | Résultat |
|---|---|---|
| 7 |  |   |

## Partie I — Banc de `trame_xor8` (T9)

| Cas | Attendu | Obtenu | Verdict |
|---|---|---|---|
| Chaîne vide | 0x00 |  |  |
| « A » | 0x41 |  |  |
| « T,0 » | 0x48 |  |  |
| « 123456789 » | 0x31 |  |  |
| Octets 0x00 à 0xFF | 0x00 |  |  |

## Partie I — Journal des séances

### Séance 1 — `20 novembre 2026` — réalise : `Simon Boisvert` et `Tristan Larabie` / valide : `Samuel Caiado` et `Hassan Hashish`
- Objectifs :
- Fait :
- Décisions (découpage, répartition du bus, format des lignes) :
- Difficultés et solutions :
- Mesures :
- Validations Git :

### Entre les séances — travail sur hôte
- Fait (tableau.c, test_tableau, fsm.c, lecture de l4a_defaut.c) :
- Validations Git :

### Séance 2 — `27 novembre 2026` — réalise : `Samuel Caiado` et `Hassan Hashish` / valide : `Simon Boisvert` et `Tristan Larabie`
- Objectifs :
- Fait :
- Décisions :
- Difficultés et solutions :
- Mesures (capteur, PIR, télémétrie, L4-A) :
- Validations Git :

## Partie I — Composante L4-A : diagnostic d'un défaut TouchPad/I²C

Variante : `<n>` — validation Git de la version défectueuse : — de la version corrigée :

1. Symptôme (attendu, observé, conditions de reproduction) :

2. Hypothèses, classées avant toute mesure :

| Rang | Hypothèse | Niveau (lignes et broches / adressage et protocole / configuration du composant / interruption et garde) | Ce qu'on observerait si elle était vraie |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |

3. Mesures :

| Hypothèse visée | Mesure (registre, balayage, relecture, compteur) | Résultat | Preuve (fichier) |
|---|---|---|---|
|  |  |  |  |

4. Essai discriminant — prédiction écrite **avant** l'essai (date et heure) :

   Résultat de l'essai :

5. Interprétation (hypothèses écartées, hypothèse retenue, pourquoi) :

6. Correction et non-régression (les 12 touches, la garde Touch En) :

7. Conclusion et limites (ce qui est démontré, ce qui est supposé, ce qui reste à vérifier) :

## Partie I — Table des états et des priorités (en vue du jalon J3)

| État | Événement (par priorité décroissante) | Condition | État suivant | Sorties (moteur, DEL, avertisseur, LCD) |
|---|---|---|---|---|
| ARRÊT | | | | |
| MARCHE_AVANT | | | | |
| MARCHE_ARRIÈRE | | | | |
| ARRÊT_URGENCE | | | | |
| ARRÊT_ÉVÉNEMENT | | | | |

Priorités des interruptions (valeur, justification) : E-Stop — encodeur — SysTick — PIR — THI —

## Partie I — Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Bus I²C et balayage |  |  |  |  |
| T2 Extenseurs |  |  |  |  |
| T3 LCD et rétroéclairage |  |  |  |  |
| T4 TouchPad et garde |  |  |  |  |
| T5 Commandes du TouchPad |  |  |  |  |
| T6 PIR |  |  |  |  |
| T7 Capteur environnemental |  |  |  |  |
| T8 Événements et priorités |  |  |  |  |
| T9 Télémétrie contrôlée |  |  |  |  |
| T10 Robustesse et non-régression |  |  |  |  |

# Partie II

## Partie II — Rôles et rotation

| Période | Réalise | Valide | Module sans matériel de chacun | Routine assembleur ou intégration de chacun |
|---|---|---|---|---|
| Entre les séances 1 et 2 (prise en main de l'outil) | `Simon Boisvert` / `Tristan Larabie` | `Samuel Caiado` / `Hassan Hashish` | • **Simon** : File d'événements (`evq.c`) & sections critiques<br>• **Samuel** : Table de transitions & automate (`tableau.c`, `fsm.c`) | • **Simon** : Primitives ASM de masquage d'interruptions (`PRIMASK`)<br>• **Samuel** : Interface d'adaptation automate (`automate.py` / cible) |
| Séance 2 (24 / 27 nov.) | `Samuel Caiado` / `Hassan Hashish` | `Simon Boisvert` / `Tristan Larabie` | • **Tristan** : Régulateur PI et calcul vitesse (`regul.c`)<br>• **Hassan** : Gestion du trajet et surveillance (`trajet.c`, `surv.c`) | • **Tristan** : Routine ASM de calcul Q16.16 / Q8 pour le PI<br>• **Hassan** : Intégration des capteurs de sécurité (thermique, encodeur) |
| Jalon J3 (27 nov.) | `Simon Boisvert` / `Samuel Caiado` | `Tristan Larabie` / `Hassan Hashish` | Validation croisée des modules hôte (`E11`, tests unitaires) | Démonstration et validation de la table des états/priorités |
| Entre les séances 2 et 3 | `Tristan Larabie` / `Hassan Hashish` | `Simon Boisvert` / `Samuel Caiado` | • **Simon & Samuel** : Portage multiplateforme (`L5-C`, Windows/Linux)<br>• **Tristan & Hassan** : Bancs de test hors-cible (`test_trajet`, `test_regul`, `test_surv`) | • **Simon** : Couche d'abstraction matérielle (HAL/mock)<br>• **Tristan** : Calibration des gains Kp/Ki |
| Séance 3 (1er / 4 déc.) | `Simon Boisvert` / `Tristan Larabie` | `Samuel Caiado` / `Hassan Hashish` | Validation des traces d'exécution (`L5-B`) et robustesse (`L5-A`) | Intégration complète sur le véhicule et essais T11–T18 |
| Démonstration finale (4 / 8 déc.) | `Samuel Caiado` / `Hassan Hashish` | `Simon Boisvert` / `Tristan Larabie` | Finalisation du journal (`JOURNAL.md`) et montage vidéo | Présentation des scénarios S1 à S5 et défense technique |

## Partie II — Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| Fronts par fenêtre à 100 tr/min | 100 × 748 / 600 |  | T14 |
| Résolution de la vitesse à 100 tr/min | 1 front / fronts par fenêtre |  | T14 |
| Constante Q16.16 de la fenêtre de 100 ms | 600 × 65 536 / 748, arrondie |  | banc du lab. 3 |
| Fronts du trajet de référence | 105 × 748 |  | T13 |
| Durée prévue du trajet de référence | somme des tours / (tr/min / 60) |  | T13 |
| Commande initiale du régulateur à 150 tr/min | 150 × 100 / 340 |  | T14 |
| Durée de masquage admise | 200 cycles / 4 MHz |  | T11 |
| Occupation de la file au pire cas prévu |  |  | T11 |

## Partie II — Automate (L5-B)

| Date | Version de la table (somme) | Résultat du vérificateur | Erreurs et contre-exemples | Correction |
|---|---|---|---|---|
|  |  |  |  |  |

Invariants imposés et ajoutés par l'équipe :

| Invariant | Type | Pourquoi | Résultat |
|---|---|---|---|
|  |  |  |  |

Version fautive conservée : fichier `…`, validation Git `…`, contre-exemple :

```
(copier ici le contre-exemple produit par l'outil)
```

Correspondance entre les événements de la file et les entrées du modèle
(traduction faite dans `fsm.c`) :

| Événement, garde | Entrée du modèle |
|---|---|
|  |  |

Comparaison des traces (au moins trois séquences) :

| Séquence | Pas | Hôte Windows | Hôte Linux | Cible (commande `g`) |
|---|---|---|---|---|
|  |  |  |  |  |

## Partie II — File d'événements et sections critiques (L5-A)

| Question | Réponse de l'équipe | Preuve |
|---|---|---|
| Producteurs (gestionnaires) et consommateur |  |  |
| Instructions placées dans la section critique |  |  |
| Politique de débordement retenue, et pourquoi |  |  |
| Mécanisme de garantie pour E-Stop et défaut |  |  |
| `crit_autotest` (commande `c`) : cas réussis |  |  |
| `crit_max_cycles` relevé (repos, scénario S3, rafale) |  |  |
| `evq_max` relevé (repos, scénario S3, rafale) |  |  |
| Rafale `e` : événements acceptés, refusés, consommés |  |  |
| Délai de E-Stop pendant une rafale |  |  |

## Partie II — Réglage du régulateur

Consigner chaque essai, y compris ceux qui échouent : c'est la justification
des gains retenus.

| Essai | Kp (Q8) | Ki (Q8) | Consigne (tr/min) | Vitesse à 2 s | Erreur (%) | Dépassement | Observation |
|---|---|---|---|---|---|---|---|
| 1 | 26 | 26 |  |  |  |  |  |
|  |  |  |  |  |  |  |  |

Gains retenus : Kp = `…`, Ki = `…` — raison :

## Partie II — Relevés de la partie II

| Essai | Relevé | Valeur | Preuve (fichier) |
|---|---|---|---|
| T13 | Distance aux changements de segment (20, 60, 75) et à la fin (105) |  |  |
| T13 | Reprise après un arrêt au feu rouge : distance avant / après |  |  |
| T14 | Erreur 2 s après chaque changement de consigne (quatre segments) |  |  |
| T14 | Plafond abaissé pendant S2 : commande et vitesse |  |  |
| T15 | Délai entre la commande `s` et l'état DÉFAUT |  |  |
| T15 | Délai entre la commande `n` et l'état DÉFAUT |  |  |
| T16 | Limitation à 42,0 °C ; arrêt à 52,0 °C ; sortie à 25,0 °C |  |  |
| T18 | Initialisation : durée de INIT, ressources vérifiées |  |  |
| T18 | Démarrage avec un module absent (commande `d`) |  |  |

## Partie II — Portage (L5-C)

| | Windows x86-64 | Linux x86-64 | STM32 |
|---|---|---|---|
| Compilateur et version |  |  |  |
| Options |  |  |  |
| Tailles : `int`, `long`, pointeur, `size_t` |  |  |  |
| Fichiers remplacés (couche d'adaptation) |  |  |  |
| `test_evq` / `test_tableau` |  |  |  |
| `test_trajet` / `test_regul` / `test_surv` |  |  |  |
| `test_automate` (traces) |  |  |  |
| Rejeu de la trace du scénario `…` |  |  |  |
| Rejeu de la trace du scénario `…` |  |  |  |

Différences observées et explication :

## Partie II — Décision collective et désaccord (L5-D)

Décision technique prise en équipe :

| Élément | Contenu |
|---|---|
| Question à trancher |  |
| Options envisagées |  |
| Critères |  |
| Choix et raison |  |
| Résultat observé après coup |  |

Désaccord (réel, ou scénario de `dossier/scenario_desaccord.md`) :

| Élément | Contenu |
|---|---|
| Objet du désaccord |  |
| Position de chacun |  |
| Démarche suivie |  |
| Issue |  |

## Partie II — Séances

### Séance 2 — date : `27 novembre 2026` — réalise : `Samuel Caiado` et `Hassan Hashish` / valide : `Simon Boisvert` et `Tristan Larabie`

- Objectifs :
- Réalisé :
- Problèmes rencontrés, hypothèses, essais :
- Validations Git (auteur, identifiant, objet) :
- À faire avant la séance 3 :

### Entre les séances 2 et 3

- Réalisé :
- Validations Git :

### Séance 3 — date : `4 decembre 2026` — réalise : `Simon Boisvert` et `Tristan Larabie` / valide : `Samuel Caiado` et `Hassan Hashish`

- Objectifs :
- Réalisé :
- Problèmes rencontrés, hypothèses, essais :
- Validations Git (auteur, identifiant, objet) :
- À faire avant la démonstration finale :

### Démonstration finale — date : `4 decembre 2026` — réalise : `Samuel Caiado` et `Hassan Hashish` / valide : `Simon Boisvert` et `Tristan Larabie`

- Scénarios S1 à S5 : résultat de chacun
- Questions posées et réponses à approfondir dans le rapport :

## Vidéo de présentation (énoncé, section 7.2)

Durée visée : 7 à 10 minutes. Filmer les scénarios dès qu'ils fonctionnent, sans attendre la veille de la remise.

| Séquence | Durée prévue | Qui parle | Ce qui est montré (véhicule, afficheur, télémétrie, code) | Tournée le |
|---|---|---|---|---|
| Présentation de l'équipe, du véhicule et de l'architecture | 1 min |  |  |  |
| Démonstration : conduite manuelle, trajet autonome, arrêt et reprise, E-Stop, défaut injecté | 4 min |  |  |  |
| Explication technique du membre A (`Simon Boisvert`) : `…` | 45 s |  |  |  |
| Explication technique du membre B (`Samuel Caiado`) : `…` | 45 s |  |  |  |
| Explication technique du membre C (`Tristan Larabie`) : `…` | 45 s |  |  |  |
| Explication technique du membre D (`Hassan Hashish`) : `…` | 45 s |  |  |  |
| Bilan : limites, ce qui serait fait autrement | 1 min |  |  |  |

Durée finale : `…` — fichier ou lien : `…` — outils de montage : `…`

## Sources et outils

| Source ou outil | Usage |
|---|---|
| Compléments du laboratoire 4 et du projet |  |
| Outil `automate.py` |  |
| Corrigé du laboratoire 3 (le cas échéant) |  |
| Fiches techniques, manuel |  |
| Outils d'IA générative (outil, version, usage) ou « aucun usage » |  |
