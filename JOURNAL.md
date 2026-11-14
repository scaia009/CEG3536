# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 3 (automne 2026)

Équipe : `Simon Boisvert` et `Samuel Caiado` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536`
## Jalon J2 — module d'événements testé sur hôte

| Plateforme | Compilateur et version | Commande | Résultat (cas réussis / échecs) | Date et validation Git |
|---|---|---|---|---|
| Windows | Windows 11 26H2 | `make` dans `hote/` | 10 cas réussis, 0 échec | 13-11-2026 |
| Linux | GCC 14.2.0 | `make` dans `hote/` | 10 cas réussis, 0 échec | 13-11-2026 |
| STM32 (cible) | arm-none-eabi-gcc | commande série `q` | 10 cas réussis, 0 échec | 13-11-2026 |

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
| Séance 2 | Samuel Caiado | Simon Boisvert | Simon Boisvert |

## Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| `RPM_Q16_K` | round(240 × 65 536 / 748) | 21 028 | T3 |
| Erreur relative de la constante | (k − 240 × 65 536 / 748) / (240 × 65 536 / 748) | 0,001933 % | T3 |
| fronts maximal sans débordement | 2³² / k | 204 250 | T3 |
| `LPUART1_BRR` | 256 × 4 000 000 / 115 200 | 8 889 | T9 |
| Débit ADC par voie | 4 MHz / (2 × (640,5 + 12,5)) | 3 062,79 | T4 |
| Durée d'un tour du tampon DMA | 16 / débit par voie | 5,224 | T4 |
| Résolution de la température | 3 300 mV / 4 095 / 10,9 mV/°C | 0,07393 °C/compte | T6 |

## Étalonnage de la température (E6)

| Mesure | Température de référence (°C) | `temp_mv` lu | `TEMP_MV_REF` / `TEMP_C10_REF` retenus | Position de S2 |
|---|---|---|---|---|
| Ambiante | | | Valeurs nominales dans le ZIP : 1831 mV / 250 (25,0 °C); à remplacer par vos valeurs relevées | INT |
| Capteur tenu entre deux doigts (30 s) | | | | EXT |
| Conditionnement externe (pont OUT–IN ou circuit) | | | | EXT |

Analyse interne / externe (plage, résolution, bruit) :

## Journal des séances

### Séance 1 — `6 novembre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs : Mettre en place le projet du laboratoire 3 à partir du projet du laboratoire 2; conserver les routines assembleur existantes nécessaires et déplacer le programme principal ainsi que la machine à états vers les modules C.
- Fait : Le projet comprend `main.c` et `fsm.c` les modules `gpio.s`, `buttons.s`, `estop.s`, `motor.s`, `encoder.s`, `systick.s` et `buzzer.s` sont conservés. `main.s` et `fsm.s` ne sont pas présents dans l'archive, conformément aux instructions du complément.
- Décisions (découpage des modules, format des trames, routine L3-A choisie) : Boucle principale cadencée à 1 ms; production d'événements pour boutons, E-Stop, encodeur, consigne et télémétrie; traitement centralisé par `fsm_traiter()` puis mise à jour des sorties.
- Difficultés et solutions : Aucune difficulté majeure rencontrée.
- Mesures : Fenêtre de vitesse de 250 ms; télémétrie prévue à 5 Hz (période 200 ms); rampe PWM d'un point toutes les 10 ms.
- Validations Git :

### Période d'étude — Travail sur hôte
- Fait : `evq.c` implémente la file circulaire de 32 événements; `acq.c` contient les versions 1 et 2 de l'objet d'acquisition; `adc_dma.c` configure ADC1 et DMA circulaire; `telem.c` configure LPUART1 et les trames; `vitesse.s` réalise la conversion de vitesse en Q16.16; l3a_c.c contient la version C de `motor_set_pwm`.
- Validation hôte présente dans l'archive : `make` dans `hote/` compile avec `-Wall -Wextra` et exécute les neuf cas; résultat vérifié ici : 9 réussis, 0 échec.
- Validations Git :

### Séance 2 — `13 novmebre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs : Terminer les composantes restantes du laboratoire 3, effectuer les essais d'intégration et vérifier le comportement du système complet.
- Fait : Selon votre déclaration, l'équipe a terminé le laboratoire au complet et n'a pas rencontré d'erreur pendant les essais. Les modules présents dans le ZIP couvrent l'acquisition ADC/DMA, la file d'événements, la télémétrie, la machine à états, la conversion de vitesse et la comparaison C/assembleur.
- Décisions : Potentiomètre converti en consigne de 0 à 100 %, avec zone morte de 3 % et hystérésis de 2 points; température moyennée sur 64 échantillons; télémétrie à 115 200 bauds; file d'événements de capacité 32.
- Difficultés et solutions : Aucune erreur n'a été observée pendant la réalisation et la validation du laboratoire, selon l'équipe.
- Mesures (température, cycles DWT, désassemblage) : Les mesures numériques et captures ne sont pas contenues dans le ZIP. Ajouter les valeurs de température, les cycles `bench_cycles_asm` / `bench_cycles_c` et les preuves de désassemblage obtenues sur la carte.
- Validations Git :

## Composante L3-A : comparaison assembleur / C

Routine comparée : `<motor_set_pwm | button_pressed | led_set | encoder_read>`

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

| Version | Cycles DWT (moyenne) | Taille (octets) | Registres utilisés | Observations (prologue, volatile) |
|---|---|---|---|---|
| Assembleur | 31 |  | r0 entrée/retour; r1 calcul; r2 adresse de TIM1 | Routine feuille; pas de prologue/épilogue de pile; écrit directement `TIM1->CCR1`#2. |
| C, -O0 | 59 |  |  | Variable d'accès au registre matériel via `TIM1->CCR1`; code généré dépendant du compilateur et de l'optimisation. |
| C, -O2 | 25 |  |  | Comparer les optimisations, les registres et les éventuels prologue/épilogue avec `-O0`. |

## Composante L3-C : objet d'acquisition

- Version 1 (validation Git) : `acq_init()` configure la source, le pas entre les voies entrelacées, le nombre d'échantillons à moyenner et les paramètres de conversion linéaire. `acq_brut()` calcule la moyenne arrondie des échantillons; `acq_mv()` convertit le résultat en millivolts; `acq_valeur()` applique `mv × gain_num / gain_den + decalage`.
- Version 2 (validation Git) et ce qui a changé : `acq_config_filtre()` configure une zone morte, une hystérésis et les limites min/max sans changer l'interface d'utilisation. Pour le potentiomètre, `main.c` configure une zone morte de 3 % et une hystérésis de 2 points. La température utilise une moyenne de 64 échantillons par voie; le potentiomètre utilise 16 échantillons.
- Essai d'indépendance des deux instances : `main.c` alloue deux objets distincts, `acq_pot` et `acq_temp`, avec des sources, gains et filtres indépendants. Le code montre deux configurations séparées. Le ZIP ne contient pas de résultat d'essai documenté où les deux voies sont modifiées simultanément; ajouter votre observation expérimentale.
- Comparaison avec la version impérative (annexe) : La version impérative de référence ou son résultat comparatif n'est pas inclus dans le ZIP; ajouter la comparaison demandée à partir du document/annexe du cours si elle a été réalisée.
- Limites : L'objet suppose que la source pointe vers un tampon valide et que n_moy ne dépasse pas le nombre d'échantillons disponibles dans le tampon par voie. La moyenne est calculée en entiers; la résolution dépend du convertisseur 12 bits et de la référence 3,3 V. Les résultats de bruit et de stabilité doivent être confirmés expérimentalement.

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
