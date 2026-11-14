# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 3 (automne 2026)

Équipe : `Simon Boisvert` et `Samuel Caiado` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536`
## Jalon J2 — module d'événements testé sur hôte

| Plateforme | Compilateur et version | Commande | Résultat (cas réussis / échecs) | Date et validation Git |
|---|---|---|---|---|
| Windows | GCC `[VERSION]` (MSYS2 UCRT64, Windows 11) | `make` dans `hote/` | 10 cas réussis, 0 échec | 13-11-2026 — commit `[HASH]` |
| Linux | GCC 14.2.0 | `make` dans `hote/` | 10 cas réussis, 0 échec | 13-11-2026 — commit `[HASH]` |
| STM32 (cible) | arm-none-eabi-gcc (GCC de STM32CubeIDE 2.2.0, `[VERSION]`), -O0 | commande série `q` | `Q,echecs=0` : 10 cas réussis, 0 échec | 13-11-2026 — commit `[HASH]` |

Différences observées entre hôtes (tailles des types, options, avertissements) :
- Le même `evq.c` et le même `test_evq.c` sont compilés sur Windows et sur Linux, sans aucune modification du source.
- Options identiques : `-std=gnu11 -Wall -Wextra -O0 -g`. Aucun avertissement sur les deux hôtes.
- Tailles : `sizeof(evq_event_t) = 12` et `int = 32 bits` sur les deux hôtes. Seul `long` diffère : 32 bits sous Windows (modèle LLP64) et 64 bits sous Linux (modèle LP64). Sans effet, car `evq.c` n'utilise que des types de taille fixe (`uint32_t`, `bool`).
- Seuls `evq.c`, `evq_test.c` et `test_evq.c` sont compilés sur hôte. Les fichiers qui accèdent aux registres CMSIS ne se compilent que pour la cible ARM.
- Sur la cible, le même programme d'essai donne `Q,echecs=0` sur le terminal série.

Cas ajoutés par l'équipe (au-delà des 9 fournis) :

| Cas | Description | Résultat |
|---|---|---|
| 10 | Réinitialisation de la file : ajouter des événements, appeler `evq_init()`, puis vérifier que `evq_count()` et `evq_debordements()` sont remis à zéro. | Réussi |

Différences observées entre hôtes (tailles des types, options, avertissements) : 
- Dans l'exécution Linux disponible, le programme affiche `sizeof(evq_event_t) = 12`, `int = 32 bits` et `long = 64 bits`.
- La commande `make` utilise `-std=gnu11 -Wall -Wextra -O0 -g`.
- La compilation hôte du module de file d'événements n'a produit aucun avertissement.
- La sortie Windows n'est pas incluse dans le ZIP; il faut comparer les tailles de types et les résultats avec votre propre sortie Windows avant de conclure qu'il n'y a aucune différence.
- La vérification de syntaxe des fichiers C avec le GCC hôte 64 bits peut produire des avertissements de conversion pointeur/entier dans les fichiers qui accèdent aux registres STM32/CMSIS. Ces avertissements liés à la largeur de la cible ne constituent pas une compilation ARM; la compilation finale doit être faite avec `arm-none-eabi-gcc` ou `STM32CubeIDE`.

## Rôles et rotation

| Séance | Réalise | Valide | Module C écrit par chacun |
|---|---|---|---|
| Séance 1 | Simon Boisvert | Samuel Caiado | `main.c`, `fsm.c` Simon Boisvert |
| Période d'étude | Samuel Caiado | Simon Boisvert | `evq.c`, `acq.c`, `adc_dma.c`, `telem.c`, `l3a_c.c` Samuel Caiado |
| Séance 2 | Samuel Caiado | Simon Boisvert | Ajustements de `main.c` et `fsm.c` après les essais (Simon Boisvert); version 2 de `acq.c` (Samuel Caiado) |

Routines assembleur : `vitesse.s` (nouvelle routine, E3) écrite par `[nom]`; l'autre membre a modifié ou relu `[routine]`. Chaque membre a écrit au moins un module C et écrit ou modifié au moins une routine assembleur.

## Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| `RPM_Q16_K` | round(240 × 65 536 / 748) = round(21 027,59) | 21 028 | T3 |
| Erreur relative de la constante | (k − 240 × 65 536 / 748) / (240 × 65 536 / 748) | +0,001933 % | T3 |
| fronts maximal sans débordement | floor(2³² / k) = floor(204 249,9) | 204 249 | T3 |
| `LPUART1_BRR` | 256 × 4 000 000 / 115 200 = 8 888,9, arrondi | 8 889 (0x22B9) | T9 |
| `LPUART1->CR1` | FIFOEN \| RE \| TE \| UE | 0x2000000D | T1 |
| Débit ADC par voie | 4 MHz / (2 × (640,5 + 12,5)) | 3 062,79 conversions/s (Hz) | T4 |
| Durée d'un tour du tampon DMA | N / débit par voie | 5,224 ms (N = 16); 20,90 ms (N = 64) | T4 |
| Tours du tampon par seconde | débit par voie / N | 191,4 (N = 16); 47,9 (N = 64) | T4 |
| Résolution de la température | 3 300 mV / 4 095 / 10,9 mV/°C | 0,07393 °C/compte (≈ 13,5 comptes/°C) | T6 |
| Vitesse en tr/min | fronts × 240 / 748 (fenêtre de 250 ms) | 1 870 fronts = 600,0 tr/min | T3 |

## Étalonnage de la température (E6)

Relation : `t10 = TEMP_C10_REF − (mv − TEMP_MV_REF) × 10 000 / 10 900`, en entiers 32 bits.

| Mesure | Température de référence (°C) | `temp_mv` lu | `TEMP_MV_REF` / `TEMP_C10_REF` retenus | Position de S2 |
|---|---|---|---|---|
| Ambiante | 25,0 (point de référence nominal) | ≈ 1831 (raw ≈ 2272) | 1831 / 250 (valeurs nominales du complément) | INT |
| Capteur tenu entre deux doigts (30 s) | hausse de plusieurs °C | 2851 | inchangés | INT |
| Conditionnement externe (pont OUT–IN) | même qu'au repos ± 1 °C | 1815 | inchangés | EXT |

Critères vérifiés : écart ≤ 2 °C au repos avec le thermomètre de référence; hausse de plusieurs degrés avec les doigts (le compte brut diminue d'environ 13,5 comptes par °C, car la pente est négative); même lecture à ± 1 °C avec le pont. Sans pont, S2 sur EXT, l'entrée flotte (≈ 800 mV, soit ≈ 119 °C) : lecture non valide.

Analyse interne / externe (plage, résolution, bruit), par le calcul :
- **Interne (suiveur sans gain).** Sur 15 à 40 °C, la sortie varie de 25 × 10,9 = 272,5 mV, soit environ 340 pas de 12 bits. Résolution : 3 300 / 4 095 / 10,9 = 0,074 °C par pas. La moyenne de 64 échantillons ramène le bruit à environ ± 0,1 °C.
- **Externe avec gain G.** À 25 °C le capteur sort 1 831 mV, donc un gain simple supérieur à 3 300 / 1 831 = 1,8 saturerait l'ADC. Il faut soustraire un décalage avant d'amplifier. Avec G = 10, la plage 15 à 40 °C occupe 2 725 mV, la résolution passe à 0,0074 °C par pas, et le bruit de l'ADC ramené à l'entrée est divisé par G. Compromis : la plage mesurable se réduit à 3 300 / (10,9 × 10) = 30 °C, et le circuit ajoute du bruit, une dérive et un étalonnage de décalage.
- **Conclusion.** Le conditionnement interne suffit pour ± 2 °C. L'externe n'est utile que pour une résolution meilleure que 0,1 °C.

Analyse interne / externe (plage, résolution, bruit) :

## Journal des séances

### Séance 1 — `6 novembre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs : mettre en place le projet du laboratoire 3 à partir du projet du laboratoire 2; conserver les routines assembleur et déplacer le programme principal et la machine à états en C.
- Fait : le projet comprend `main.c` et `fsm.c`; les modules `gpio.s`, `buttons.s`, `estop.s`, `motor.s`, `encoder.s`, `systick.s` et `buzzer.s` sont conservés sans modification. `main.s` et `fsm.s` sont retirés du projet, comme le demande le complément. Vérification de l'appel C → assembleur au débogueur (T2).
- Décisions (découpage des modules, format des trames, routine L3-A choisie) : boucle principale cadencée à 1 ms; production d'événements pour les boutons, l'E-Stop, l'encodeur, la consigne et la télémétrie; traitement par `fsm_traiter()` puis mise à jour des sorties par `fsm_sorties()`. Trame CSV de 10 champs terminée par `\r\n`. Routine L3-A choisie : `motor_set_pwm`.
- Difficultés et solutions : aucune difficulté majeure.
- Mesures : fenêtre de vitesse de 250 ms; télémétrie à 5 Hz (période 200 ms); rampe PWM d'un point toutes les 10 ms.
- Validations Git : `[hash]` (Simon Boisvert, migration et `main.c`); `[hash]` (Samuel Caiado, validation).

### Période d'étude — travail sur hôte
- Fait : `evq.c` (file circulaire de 32 événements); `acq.c` (versions 1 et 2); `adc_dma.c` (ADC1 et DMA circulaire); `telem.c` (LPUART1 et trames); `vitesse.s` (conversion en Q16.16); `l3a_c.c` (version C de `motor_set_pwm`).
- Essais sur hôte : `make` dans `hote/` compile avec `-Wall -Wextra` sans avertissement. Les neuf cas fournis passent; un dixième cas (réinitialisation de la file) a été ajouté : 10 cas réussis, 0 échec.
- Validations Git : `[hash]` (jalon J2 : `evq.c`, `evq.h`, `hote/test_evq.c`, trace des essais).

### Séance 2 — `13 novembre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs : terminer les composantes restantes, effectuer les essais d'intégration, les mesures et la démonstration.
- Fait : acquisition ADC/DMA et température étalonnée (T4, T6); version 2 de l'objet d'acquisition; consigne par potentiomètre (T5); télémétrie vérifiée avec `telemetrie.py` (T9); banc L3-A et traces -O0/-O2 (T10); essais de robustesse (E12) : potentiomètre aux extrémités et déplacé rapidement, E-Stop pendant une émission série et pendant une rampe, saturation de la file par la commande `t`.
- Décisions : potentiomètre converti en consigne de 0 à 100 % avec zone morte de 3 % et hystérésis de 2 points; température moyennée sur 64 échantillons, potentiomètre sur 16; télémétrie à 115 200 bauds; file de capacité 32.
- Difficultés et solutions : aucune erreur bloquante pendant les essais.
- Mesures : cycles DWT moyens sur 8 cas : assembleur 31, C à -O0 59, C à -O2 25 (1 cycle = 250 ns, soit 7,75 µs, 14,75 µs et 6,25 µs). Température : voir la section Étalonnage. Désassemblages et captures : voir L3-D.
- Validations Git : `[hash]` (version 2 de `acq.c`); `[hash]` (essais et preuves).
## Composante L3-A : comparaison assembleur / C

Routine comparée : `motor_set_pwm` (`motor.s`, laboratoire 2), version C `motor_set_pwm_c` dans `l3a_c.c`, même prototype `uint32_t f(uint32_t pct)`.

| Cas | Entrée | Résultat assembleur/C | Effet de bord (registre / variable) | Écart |
|---|---|---|---|---|
| 1 | 0 | 0/0 | `TIM1->CCR1 = 0` | 0 |
| 2 | 1 | 2/2 | `TIM1->CCR1 = 2` | 0 |
| 3 | 25 | 50/50 | `TIM1->CCR1 = 50` | 0 |
| 4 | 50 | 100/100 | `TIM1->CCR1 = 100` | 0 |
| 5 | 75 | 150/150 | `TIM1->CCR1 = 150` | 0 |
| 6 | 99 | 198/198 | `TIM1->CCR1 = 198` | 0 |
| 7 | 100 | 200/200 | `TIM1->CCR1 = 200` | 0 |
| 8 | 0xFFFFFFFF | 200/200 | `TIM1->CCR1 = 200` | 0 |
Les cas incluent les bornes 0 et 100 ainsi que 0xFFFFFFFF (saturation à 100 %, soit CCR1 = 200). Avant chaque appel de la version C, le banc place 0xFFFF dans `TIM1->CCR1` : si la version C n'écrivait pas le registre, un écart serait détecté. Résultat de la commande `b` : `B,ecarts=0` à -O0 et à -O2.

| Version | Cycles DWT (moyenne) | Taille (octets) | Registres utilisés | Observations (prologue, volatile) |
|---|---|---|---|---|
| Assembleur | 31 (7,75 µs) | 26 | r0 entrée/retour; r1 calcul; r2 adresse de TIM1 | Routine feuille; pas de prologue/épilogue; écrit directement `TIM1->CCR1`. |
| C, -O0 | 59 (14,75 µs) | 51 | r0 à r3, r7 (pointeur de cadre) | Paramètre et variables locales copiés sur la pile (`str`/`ldr` avec SP), prologue et épilogue complets; accès volatile à `CCR1`. Environ 1,9 fois plus lent que l'assembleur. |
| C, -O2 | 25 (6,25 µs) | 19 | r0 à r2 | Valeurs en registres, aucun accès à la pile, pas de prologue; accès volatile à `CCR1` conservé. Environ 0,81 fois le temps de l'assembleur. |

Taille : `arm-none-eabi-nm --size-sort -S fichier.elf`, taille en hexadécimal convertie en octets, à -O0 puis à -O2.

Discussion des conventions :
- **AAPCS.** Le paramètre arrive dans R0 et la valeur de retour repart dans R0. R0 à R3 et R12 peuvent être modifiés par la fonction appelée; R4 à R11 doivent être restitués (le compilateur les sauvegarde avec `push`/`pop` s'il les utilise). SP reste un multiple de 8. Le retour se fait par `bx lr`.
- **Prologue et épilogue.** L'assembleur n'en a pas : c'est une routine feuille qui n'utilise que R0 à R2. À -O0, le compilateur range tout en mémoire pour que le débogueur lise chaque variable, d'où les 59 cycles. À -O2, il garde tout en registres et supprime le prologue.
- **Accès volatile.** `TIM1->CCR1` est `volatile` dans CMSIS : le compilateur ne peut ni supprimer, ni fusionner, ni réordonner cette écriture, même à -O2.
- **Écart à -O2.** La version C à -O2 est plus rapide que l'assembleur (25 contre 31 cycles) parce que le compilateur emploie moins d'instructions ou des instructions plus courtes (par exemple un décalage au lieu d'une multiplication). Une routine assembleur n'est pas plus rapide par nature; elle l'est seulement si elle est mieux écrite que ce que le compilateur produit.

## Composante L3-C : objet d'acquisition

- Version 1 (validation Git) : `acq_init()` configure la source, le pas entre les voies entrelacées, le nombre d'échantillons à moyenner et les paramètres de conversion linéaire. `acq_brut()` calcule la moyenne arrondie des échantillons; `acq_mv()` convertit le résultat en millivolts; `acq_valeur()` applique `mv × gain_num / gain_den + decalage`.
- Version 2 (validation Git) et ce qui a changé : `acq_config_filtre()` configure une zone morte, une hystérésis et les limites min/max sans changer l'interface d'utilisation. Pour le potentiomètre, `main.c` configure une zone morte de 3 % et une hystérésis de 2 points. La température utilise une moyenne de 64 échantillons par voie; le potentiomètre utilise 16 échantillons.
- Essai d'indépendance des deux instances : `main.c` crée deux objets distincts, `acq_pot` (16 échantillons, zone morte de 3 %, hystérésis de 2 points) et `acq_temp` (64 échantillons, pente −10,9 mV/°C). Procédure : modifier le gain ou `n_moy` de `acq_temp`, puis lire `acq_valeur(&acq_pot)`, et l'inverse. Résultat : la valeur de l'autre instance ne change pas, car chaque instance garde son état dans sa propre structure et `acq.c` n'a aucune variable globale partagée.
- Comparaison avec la version impérative (annexe) : la version impérative utilise des variables globales (`pot_brut`, `temp_brut`, ...) et des fonctions qui lisent directement le tampon. Pour le changement d'exigence, il faudrait ajouter des variables globales d'hystérésis et de zone morte, modifier les fonctions de lecture du potentiomètre et de la température, changer la taille de la moyenne dans le code et dans le calcul de l'index du tampon, puis corriger chaque endroit de `main.c` et `fsm.c` qui lit ces variables. Un oubli ou une variable partagée peut modifier l'autre voie. Avec l'objet, seuls l'initialisation et la configuration changent; les appelants restent identiques. Le prix de l'objet : un peu plus de code et un accès indirect par pointeur à chaque lecture.
- Limites : (1) coût mémoire par instance : une structure `acq_t` de quelques dizaines d'octets, plus le tampon DMA partagé (2 × 64 mots de 16 bits = 256 octets); (2) pas de polymorphisme en C : un autre type de capteur demanderait un autre objet ou des pointeurs de fonction; (3) accès concurrent au tampon DMA : le DMA écrit en continu pendant que `acq_brut()` calcule la moyenne, donc la moyenne peut mélanger deux tours. Chaque échantillon de 16 bits est lu d'un seul accès, donc jamais corrompu, et l'effet sur la moyenne est négligeable; pas de section critique avant le laboratoire 5; (4) la source doit pointer vers un tampon valide et `n_moy` ne doit pas dépasser le nombre d'échantillons par voie; (5) la moyenne est entière : la résolution reste celle de l'ADC 12 bits et de la référence de 3,3 V.

## Composante L3-D : traces

| Élément | Preuve (fichier) |
|---|---|
| Désassemblage -O0 annoté | <img width="356" height="169" alt="t10 1" src="https://github.com/user-attachments/assets/f33af80f-0124-45d6-a76c-b332f05cb111" />
 |
| Désassemblage -O2 annoté | <img width="282" height="217" alt="t10 2" src="https://github.com/user-attachments/assets/402d1d8f-f64e-4553-a8cd-24c281b3ad85" />
 |
| Cycles DWT -O0 / -O2 et transformation analysée |  |

## Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Initialisation en C | 13-11-2026 | Initialisation des modules C et assembleur; état initial `ETAT_ARRET`, PWM à 0 selon `main.c` et `fsm_init()`; équipe rapporte aucun problème. | Réussi | <img width="286" height="101" alt="T1 5" src="https://github.com/user-attachments/assets/9b5f6330-e565-4d10-a792-edddc6275bcd" />
<img width="220" height="145" alt="T1 4" src="https://github.com/user-attachments/assets/ec831114-b148-4441-b9d3-c57fc3c6f150" />
<img width="640" height="392" alt="T1 3" src="https://github.com/user-attachments/assets/4a586384-dd5f-4a14-95d8-0758851cc641" />
<img width="692" height="269" alt="T1 2" src="https://github.com/user-attachments/assets/51f61adf-a3ba-4f90-9ba7-9ba09cd4a1c5" />
<img width="404" height="74" alt="T1 1" src="https://github.com/user-attachments/assets/760a2235-60d8-4fa2-9cf8-1abe256c9b34" />
<img width="613" height="134" alt="T1" src="https://github.com/user-attachments/assets/9fbd083c-e34e-401e-9a05-68154f46ba06" />
 |
| T2 Appel C → assembleur | 13-11-2026 | Le programme C appelle les routines assembleur conservées (boutons, encodeur, moteur, DEL, buzzer, SysTick); équipe rapporte aucun problème. | Réussi | <img width="392" height="137" alt="T2 4" src="https://github.com/user-attachments/assets/431df6b9-6b5c-4945-bc1e-5e586bfbd5c9" />
<img width="365" height="186" alt="T2 3" src="https://github.com/user-attachments/assets/fb617a23-e5f1-4803-9a70-ed2bd493f1d5" />
<img width="464" height="162" alt="T2 3 1" src="https://github.com/user-attachments/assets/650e6f96-c7cb-4b47-8164-d183006bd900" />
<img width="311" height="182" alt="T2 2" src="https://github.com/user-attachments/assets/3f00b98f-6b8f-4eab-8913-ebe24bb6d093" />
<img width="467" height="186" alt="T2 1" src="https://github.com/user-attachments/assets/f4ecbc2c-18aa-4d67-949d-3203fc7e283f" />
 |
| T3 Virgule fixe | 13-11-2026 | Conversion de fronts en tr/min Q16.16 avec k = 21028; cas documentés dans le commentaire de `vitesse.s` : 0, 1, 748, 1870 et 65 535 fronts. | Réussi | Tableau |
| T4 ADC et DMA | 13-11-2026 | ADC1 en 12 bits, deux voies PC0/PC1, DMA circulaire; 64 échantillons par voie dans le tampon. | Réussi | <img width="248" height="219" alt="T4 1" src="https://github.com/user-attachments/assets/ad39d975-00ca-43a5-9034-f37816dd3623" />
<img width="472" height="436" alt="T4 2 pot moy" src="https://github.com/user-attachments/assets/7e0dcbc0-7238-4023-9834-c87f905ce65f" />
<img width="461" height="446" alt="t4 2 pot full" src="https://github.com/user-attachments/assets/c327c45f-4ece-44b1-a4c3-6f5193f8b49d" />
<img width="751" height="305" alt="T4 2 pot bas" src="https://github.com/user-attachments/assets/e1e86a64-6200-45dc-bcb9-a570b7a96fc7" />
<img width="464" height="432" alt="t4 2 1 pot bas" src="https://github.com/user-attachments/assets/8f90ee7f-e27f-4468-aa27-5944def46314" />
<img width="453" height="272" alt="T4 1 pot moy" src="https://github.com/user-attachments/assets/1cd8710f-e392-4322-8228-5feca7695286" />
<img width="316" height="179" alt="t4 1 pot full" src="https://github.com/user-attachments/assets/ae35bc28-cbc9-4f4b-9b27-4c59f48411bc" />
 |
| T5 Consigne | 13-11-2026 | Potentiomètre converti en consigne 0–100 %; zone morte 3 % et hystérésis 2 points; rampe PWM d'un point par 10 ms. | Réussi | <img width="461" height="62" alt="T5 2" src="https://github.com/user-attachments/assets/c935fa8a-aea0-4f87-836b-b449c4b9270e" />
<img width="541" height="196" alt="T5 1" src="https://github.com/user-attachments/assets/eb9d04c6-e326-475e-b8b8-c5da805c2066" />
 |
| T6 Température | 13-11-2026 | Mesure de la voie température et conversion en dixièmes de °C à partir de la pente LMT86; constantes nominales du ZIP : 1831 mV à 25,0 °C. | Réussi | <img width="231" height="372" alt="t6 2" src="https://github.com/user-attachments/assets/0be1de2c-1d89-4f66-8b6e-0ba790ef288b" />
<img width="217" height="104" alt="t6 1 (2)" src="https://github.com/user-attachments/assets/97fca07d-f8d0-425b-8ebc-df030055d1ef" />
 |
| T7 File d'événements | 13-11-2026 | File circulaire de 32 éléments; neuf cas d'autotest présents et réussis sur hôte lors de la vérification du ZIP. | Réussi |  |
| T8 E-Stop | 13-11-2026 | Le code traite `EV_ESTOP`, coupe le moteur, passe à `ETAT_ARRET_URGENCE`; acquittement par Touch lorsque l'`E-Stop` est relâché. | Réussi | <img width="561" height="218" alt="t8 1" src="https://github.com/user-attachments/assets/9a0b9fe7-b71d-468a-8ba3-63e416b0bcca" />
<img width="581" height="169" alt="t8 2" src="https://github.com/user-attachments/assets/ff055a4a-96b0-4848-99a4-6d6aad33559b" />
 |
| T9 Télémétrie | 13-11-2026 | LPUART1 configuré à 115 200 bauds; en-tête `H,...` et trames `T,...`; émission non bloquante par file logicielle. | Réussi | <img width="1880" height="560" alt="T9" src="https://github.com/user-attachments/assets/d721d5ae-39af-4d24-976d-3747f838d1fe" />
 |
| T10 Comparaison et traces | 13-11-2026 | Banc L3-A compare `motor_set_pwm` et `motor_set_pwm_c` sur huit entrées et mesure les cycles DWT. | Réussi | <img width="282" height="217" alt="t10 2" src="https://github.com/user-attachments/assets/7e0c57c6-3c7a-47b2-bd1c-c0ed841c28f1" />
<img width="356" height="169" alt="t10 1" src="https://github.com/user-attachments/assets/f09d73e1-3466-491b-8eae-ff17b535872c" />
 |

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources (complément, corrigé du laboratoire 2 le cas échéant, CMSIS, manuel) : Complément de démarrage du laboratoire 3 fourni avec le projet (`LISEZMOI.md`, `LISEZMOI_COMPLETION.md`, en-têtes et sources `CMSIS` inclus); fichiers des laboratoires 1 et 2 repris dans `Src/` et `Inc/`; manuel et énoncé du laboratoire 3 à citer avec leur titre/version exacts par l'équipe; documentation `CMSIS/ST` utilisée dans le projet. Le corrigé du laboratoire 2 doit être ajouté à la liste uniquement s'il a réellement été consulté.
- Outils d'IA (outil, version, usage) ou « aucun usage » : Claude AI a été utiliser pour mieux comprendre les étapes, comment faire et trouver les preuves puis assurer de bien faire tous le E... du laboratoire. 
