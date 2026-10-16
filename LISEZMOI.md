# CEG 3536 — Laboratoire 2 — Complément de démarrage

Ce complément s'ajoute au projet du laboratoire 1 de l'équipe (ou au corrigé
du laboratoire 1 publié après le 9 octobre, à citer comme source). Il ne
constitue pas un projet complet : il fournit les constantes, l'en-tête C et
quatre squelettes commentés. Voir l'énoncé, sections 4.2, 4.4 et 4.5.

| Fichier | Rôle | État |
|---|---|---|
| `Core/Src/registres_lab2.inc` | Constantes TIM1/TIM2, SysTick, SHPR3, EXTI0, brochage moteur / encodeur / buzzer, temporisations, paliers de consigne | Fourni ; `ENC_PUPDR`, `ENC_SENS_INVERSE` et les valeurs calculées (`PWM_ARR`, `SYSTICK_LOAD_1MS`) à vérifier par mesure (T3, T5, T10) |
| `Core/Inc/lab2.h` | Prototypes C de toutes les routines et variables des laboratoires 1 et 2 (inclut `lab1.h`) | Fourni ; à tenir à jour si l'interface est étendue |
| `Core/Src/motor.s` | `motor_init`, `motor_set_pwm`, `motor_set_dir`, `motor_stop_now` | GPIO, `motor_set_pwm` et `motor_stop_now` fournis ; **TIM1 (E2) et règle de sens (E3) à écrire** |
| `Core/Src/encoder.s` | `encoder_init`, `EXTI0_IRQHandler`, `encoder_read` ; `enc_count`, `vitesse_fronts`, `sens_mesure` | `encoder_init` fournie ; **comptage dans l'ISR (E5) et `encoder_read` (E6) à écrire** |
| `Core/Src/systick.s` | `systick_init`, `SysTick_Handler`, `delay_ms` ; `ticks_ms`, `tick_1ms` | `SysTick_Handler` fourni ; **`systick_init` et `delay_ms` à écrire (E10)** |
| `Core/Src/buzzer.s` | `buzzer_init`, `buzzer_set`, `buzzer_bip`, `buzzer_step` ; `buzzer_continu` | `buzzer_init` et `buzzer_set` fournis ; **`buzzer_bip` et `buzzer_step` à écrire (E9)** |
| `Makefile`, `build.sh` | Construction en ligne de commande (facultatif) : `make` (make de STM32CubeIDE ou de STM32CubeCLT) ou `bash build.sh [flash]` sans make ; cible `CEG3536_Lab2` | Fournis |
| `JOURNAL.md` | Modèle de journal d'équipe du laboratoire 2 (mesures, essais T1 à T10, suivi J1) | Fourni ; à tenir à chaque séance |

## Mise en route

1. Copier le projet du laboratoire 1 : dans STM32CubeIDE, clic droit sur le
   projet > *Copy*, puis *Paste* avec le nom `CEG3536_Lab2_<équipe>`
   (ou dupliquer le dossier et renommer le projet dans `.project`).
2. Dézipper le complément et copier son contenu **par-dessus** le projet
   copié : `Core/Src/*.s`, `Core/Src/registres_lab2.inc`, `Core/Inc/lab2.h`,
   `Makefile`, `JOURNAL.md`. Aucun fichier du laboratoire 1 n'est écrasé.
3. Dans `Core/Src/buttons.s`, **supprimer la routine `delay_ms`** (boucle
   calibrée) : elle est maintenant définie dans `systick.s`. Sinon l'édition de
   liens échoue (`multiple definition of 'delay_ms'`).
4. Remplacer la boucle principale de `Core/Src/main.s` (voir ci-dessous).
5. *Project > Build Project* : le projet doit compiler sans avertissement.
   *Run > Debug As > STM32 C/C++ Application* : après *Resume*, la DEL rouge
   s'allume (E1) ; rien d'autre ne réagit tant que `systick_init` n'est pas
   écrite (la boucle principale attend `tick_1ms`).

## Modifications à faire dans les fichiers du laboratoire 1

### `main.s` — initialisations et boucle cadencée par SysTick (E10)

```asm
#include "registres.inc"
#include "registres_lab2.inc"
    ...
main:
    bl      gpio_init               /* DEL en sortie, boutons en entrée      */
    bl      buzzer_init             /* PA0 en sortie, silence                */
    bl      motor_init              /* TIM1 PWM 20 kHz à 0 %, STB = 0        */
    bl      encoder_init            /* EXTI0 sur PB0, priorité 1             */
    bl      estop_init              /* EXTI2 sur PB2, priorité 0             */
    ldr     r0, =SYSTICK_PERIODE_US
    bl      systick_init            /* 1 ms, priorité 2                      */
    bl      fsm_init                /* ARRÊT, DEL rouge seule (E1)           */
boucle_principale:
    ldr     r0, =tick_1ms
    ldr     r1, [r0]
    cmp     r1, #0
    beq     boucle_principale       /* attente du prochain tick              */
    movs    r1, #0
    str     r1, [r0]                /* tick consommé                         */
    bl      fsm_step
    b       boucle_principale
```

Les sorties (DEL, buzzer, moteur) sont mises en état sûr avant d'activer les
interruptions. `delay_ms` n'est plus appelée dans la boucle.

### `estop.s` — gestionnaire E-Stop étendu à la propulsion (E7)

Ajouter `#include "registres_lab2.inc"`. Dans `EXTI2_IRQHandler`, **avant** les
DEL et le drapeau :

1. `motor_stop_now` (CCR1 = 0, STB = 0, AI1 = AI2 = 0) ;
2. `buzzer_set(1)` et `buzzer_continu = 1` (son continu) ;
3. puis la suite du laboratoire 1 (DEL, `estop_flag`, effacement RPR1/FPR1, `dsb`).

Ces deux routines n'utilisent que r0-r1, mais un `bl` dans une ISR exige de
sauvegarder LR : `push {r4, lr}` au début et `pop {r4, pc}` à la fin
(8 octets : pile alignée ; `pc` reçoit EXC_RETURN).
Dans `estop_init`, la priorité peut s'écrire `PRIO_ESTOP << PRIO_POS` (= 0).

### `fsm.s` — propulsion, consigne, rampe, fenêtre de vitesse, buzzer (E3, E4, E6, E8, E9)

Ajouter `#include "registres_lab2.inc"` et les variables `consigne_pct`
(globale, `CONSIGNE_INITIALE` dans `fsm_init`), `pwm_pct` (globale),
`rampe_compteur` et `fenetre_compteur`. La bascule `touch_enabled` et
l'extinction brève du laboratoire 1 (E7) sont retirées (`touch_enabled`
reste définie, à 0, pour `lab1.h`). Puis :

- **Entrée dans un état hors urgence** (ARRÊT, AVANT, ARRIÈRE) : une seule
  routine locale (par exemple `fsm_entree_etat`) qui écrit `etat`, incrémente
  `compteur_transitions`, remet `pwm_pct` et `rampe_compteur` à 0, appelle
  `motor_set_dir` avec la direction de l'état (table `etat_vers_dir` :
  ARRÊT → `MOTOR_ARRET`, AVANT → `MOTOR_AVANT`, ARRIÈRE → `MOTOR_ARRIERE`) et
  `buzzer_bip` avec le nombre de bips de l'état (1, 1, 2). `motor_set_dir`
  accepte toujours ici : vers ARRÊT elle force CCR1 = 0 ; depuis ARRÊT, CCR1
  est déjà nul.
- **Partie A (E-Stop consommé)** : `motor_stop_now` (redondant avec l'ISR,
  garantit l'état sûr), `pwm_pct = 0`, `buzzer_continu = 1`, puis la
  transition vers ARRÊT_URGENCE comme au laboratoire 1.
- **Acquittement (E8)** : Touch En validé et `button_raw(BTN_ESTOP) == 0` →
  `buzzer_continu = 0`, entrée dans ARRÊT (bip court).
- **Touch En hors urgence (E4)** : `consigne_pct += CONSIGNE_PAS` ; si
  `> CONSIGNE_MAX` → `CONSIGNE_MIN` ; `buzzer_bip(1)`.
- **Fenêtre de vitesse (E6)** : `fenetre_compteur++` ; à
  `FENETRE_VITESSE_MS`, remise à 0 et `encoder_read(&sens_mesure)`.
- **Sorties** (ancienne `fsm_maj_del`, renommée par exemple
  `fsm_maj_sorties`) : (1) rampe — en AVANT/ARRIÈRE, `rampe_compteur++` ; à
  `RAMPE_PAS_MS`, `pwm_pct` se rapproche de `consigne_pct` d'un point puis
  `motor_set_pwm(pwm_pct)` ; (2) DEL comme au laboratoire 1 (clignotement en
  urgence) ; (3) `buzzer_step`. Ce point unique de mise à jour des sorties est
  le critère B4.

## Vérifications au débogueur (fenêtre SFRs) avant l'oscilloscope

| Registre | Valeur attendue (MSI 4 MHz) |
|---|---|
| `TIM1_CR1` / `CCMR1` / `CCER` / `ARR` / `BDTR` | `0x81` / `0x68` / `0x1` / `0xC7` / `0x8000` |
| `SysTick CTRL` / `LOAD` ; `SCB_SHPR3` | `0x7` (+ bit 16) / `0xF9F` ; `0x40000000` |
| `NVIC_ISER0` ; `NVIC_IPR` octets 11 et 13 | bits 11 et 13 ; `0x20` et `0x00` |
| `EXTI_EXTICR1` ; `RTSR1` ; `FTSR1` ; `IMR1` | `0x00010001` ; bits 0 et 2 ; bit 0 ; bits 0 et 2 |
| `GPIOE_MODER` bits 19:18 et 31:30 ; `GPIOE_AFRH` champ 1 | `10` (AF) et `01` (sortie) ; `0x1` |

Variables à surveiller (*Expressions*) : `etat`, `estop_flag`, `consigne_pct`,
`pwm_pct`, `enc_count`, `vitesse_fronts`, `sens_mesure`, `ticks_ms`,
`buzzer_continu`, `compteur_transitions`.

## Vitesse attendue de l'encodeur

Le guide (p. 39) laisse entendre que les 350 tr/min sont réduits par le
réducteur 34:1 ; en réalité 350 tr/min est la vitesse de l'arbre de sortie
(mesurée : 348 tr/min à 100 % sans charge). Attendre donc environ 2 200
fronts de la voie A par fenêtre de 500 ms à pleine vitesse, et non 64 ; l'ISR
EXTI0 est appelée environ 4 400 fois par seconde.

## Sécurité

Le moteur tourne dès que `motor_set_dir` accepte une marche et que la rampe
monte : lire l'encadré « Sécurité moteur » de l'énoncé avant le premier essai
avec la plateforme alimentée. Le squelette de `motor_set_dir` refuse tout
(retour 1) : le moteur ne peut pas démarrer avant que la règle de sens soit
écrite, et il faut que `motor_stop_now` et l'ISR E-Stop fonctionnent avant
tout essai à pleine vitesse.
