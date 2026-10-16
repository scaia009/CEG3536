# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 2 (automne 2026)

Équipe : `Simon Boisvert` et `Samuel Caiado` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536/tree/Lab-2`

## Suivi du jalon J1

| Laboratoire | Échéancier prévu (J1) | Déroulement réel | Écart et cause | Ajustement pour les laboratoires 3 à 5 |
|---|---|---|---|---|
| 1 | 2 octobre 2026 | 11 octobre 2026 | De 9 jours, puisque le changement d'échéance avait été changé par l'AT | Planification d'une séance de relecture de code commune avant le laboratoire pour valider la configuration des registres et éviter les blocages au débogueur. |
| 2 | 2 novembre 2026 | <Remplir à la remise> | <Écart constaté au Lab2> | <Planifier plus/moins de temps de débogage> |

## Rôles et rotation

| Séance | Réalise | Valide (essais, mesures, relecture) | Périphérique configuré ou mesuré par chacun |
|---|---|---|---|
| Séance 1 (13 / 16 oct.) | Simon Boisvert | Samuel Caiado | PWM (TIM1), base de temps SysTick, bips du Buzzer. |
| Séance 2 (20 / 23 oct.) | Samuel Caiado | Simon Boisvert | Lecture et quadrature des Encodeurs, ISR E-Stop. |

## Calculs préalables (avant la séance 1)

| Grandeur | Formule | Valeur | Vérifiée par (essai) |
|---|---|---|---|
| `PWM_PSC`, `PWM_ARR` | f = 4 MHz / ((PSC + 1)(ARR + 1)) = 20 kHz | **PSC = 0**, **ARR = 199** | T3 |
| `CCR1` pour 25 / 50 / 75 / 100 % | pct × (ARR + 1) / 100 | **25% = 50** \| **50% = 100** \| **75% = 150** \| **100% = 200** | T3 |
| `SYSTICK_LOAD_1MS` | 4 MHz × 1 ms − 1 | **3999** | T10 |
| Fronts par tour (arbre de sortie) | 2 × 11 × 34 | **748 fronts** (en comptant les 2 fronts de la voie A) | T5 |
| tr/min à partir de `vitesse_fronts` | fronts × 120 / 748 (fenêtre 500 ms) | **vitesse_fronts × 120 / 748** | T6 |
| Fronts attendus par fenêtre de 500 ms à 100 % | 350 tr/min (arbre de sortie) × 748 / 120 | **~2182 fronts** (350 × 748 / 120 = 2181,6) | T6 |

## Journal des séances

### Séance 1 — 16 octobre 2026 — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- **Objectifs :** Configurer TIM1 pour la PWM à 20 kHz, configurer SysTick pour une interruption de 1 ms, et gérer le buzzer.
- **Fait :** Implémentation de `motor_init`, `motor_set_pwm`, `systick_init` et `buzzer_bip`. Cadencement de la boucle principale avec le drapeau `tick_1ms`.
- **Décisions (sens réel de rotation pour AI1 = 1, AI2 = 0 ; `ENC_SENS_INVERSE` ; variante buzzer) :** <Indiquer si AI1=1 fait tourner le véhicule vers l'avant ou l'arrière sur votre banc. Notez si la variante PWM TIM2 a été choisie pour le buzzer.>
- **Difficultés et solutions :** <Décrire une difficulté rencontrée, ex: l'oubli du bit BDTR.MOE indispensable pour activer les sorties sur un temporisateur avancé.>
- **Mesures (PWM, SysTick, bips) :** Fréquence mesurée à 20 kHz (PSC=0, ARR=199). Période SysTick validée à 1 ms (LOAD=3999). Bips d'une durée de 100 ms contrôlés par décrémentation.
- **Validations Git (auteur, message) :** <Auteur> : "Ajout et validation de la configuration de motor.s et systick.s"

### Séance 2 — <Insérer la date : 20 ou 23 oct.> — réalise : `Samuel Caiado` / valide : `Simon Boisvert`
- **Objectifs :** Configurer EXTI0 pour lire l'encodeur, calculer la vitesse réelle sur la fenêtre de 500 ms, et étendre l'ISR de l'E-Stop.
- **Fait :** Écriture de `encoder_init`, `EXTI0_IRQHandler` (logique A = B / A ≠ B), et mise en sécurité immédiate dans `EXTI2_IRQHandler`. Démonstration devant l'assistant.
- **Décisions :** Le compteur `enc_count` augmente bien dans le sens avant commandé par la machine à états.
- **Difficultés et solutions :** <Décrire une difficulté, ex: mauvaise écriture pour effacer les requêtes d'interruption dans RPR1/FPR1 (il faut écrire un 1 pour effacer).>
- **Mesures (encodeurs, vitesse, délai E-Stop) :** Constante `FENETRE_VITESSE_MS = 500` respectée. Délai E-Stop -> PWM nulle estimé à quelques cycles d'instructions (< 1 ms).
- **Validations Git :** <Auteur> : "Implémentation finale de l'encodeur et de l'interruption E-Stop sécurisée"

## Tableau des mesures (composante L2-A)

| Mesure | Exigence | Valeur mesurée | Incertitude / méthode | Écart | Cause | Preuve (fichier) |
|---|---|---|---|---|---|---|
| Fréquence PWM | 20 kHz ± 5 % | <Ex: 20,0 kHz> | Débogueur (Vue SFR : PSC, ARR) | <0 %> | N/A | `<preuves/sfr_tim1.png>` |
| Rapport cyclique à 25 / 50 / 75 / 100 % | ± 2 points | <Ex: 25%, 50%, 75%, 100%> | Débogueur (Vue SFR : CCR1) | <0 %> | N/A | `<preuves/sfr_ccr1.png>` |
| Rampe 0 → 100 % | 1 s ± 10 % | <Ex: 1010 ms> | Point d'arrêt / écart `ticks_ms` | <+1 %> | Résolution SysTick | `<preuves/rampe.txt>` |
| Période SysTick | 1 ms ± 1 % | <Ex: 1,00 ms> | Chronomètre sur 10 secondes | <0 %> | Oscillateur MSI stable | `<preuves/systick.txt>` |
| Vitesse à 25 / 50 / 75 / 100 % (avant) | croissante, stable ± 10 % | <Ex: 530, 1080, 1620, 2170 fronts> | Live Expressions (`<vitesse_fronts>`) | <X %> | Frottements / Charge vide | `<preuves/vitesse_av.png>` |
| Vitesse à 25 / 50 / 75 / 100 % (arrière) | idem | <Insérer fronts arrière> | Live Expressions (`vitesse_fronts`) | <X %> | Frottements mécaniques | `<preuves/vitesse_ar.png>` |
| Déphasage E_A / E_B (avant, arrière) | quadrature, inversé selon le sens | Validé | Lecture IDR dans l'ISR de l'encodeur | N/A | Comportement physique | `<preuves/quadrature.png>` |
| Délai E-Stop → PWM nulle | < 1 ms | < 1 ms (estimé) | Latence d'interruption + instructions | <0,1 ms | Traitement direct dans l'ISR | `<preuves/estop_isr.png>` |
| Durée d'un bip | 100 ms ± 10 % | <Ex: 100 ms> | Débogueur / Lecture `ticks_ms` | <0 %> | Cadencé par SysTick | `<preuves/buzzer.png>` |

## Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation | <Date> | Rouge fixe, moteur immobile, CCR1=0, STB=0. | Conforme | `<preuves/t1_reset.png>` |
| T2 Cycle User et sens | <Date> | Séquence R -> V -> R -> B -> R respectée. Niveaux ODR conformes. | Conforme | `<preuves/t2_fsm.png>` |
| T3 PWM | <Date> | Fréquence à 20 kHz, pas de CCR1 > ARR+1. | Conforme | `<preuves/t3_pwm.png>` |
| T4 Rampe | <Date> | Transition fluide, 1 point par 10 ms respecté. | Conforme | `<preuves/t4_rampe.png>` |
| T5 Encodeur | <Date> | `enc_count` croît en AV, décroît en AR. Pas de perte. | Conforme | `<preuves/t5_encoder.png>` |
| T6 Vitesse | <Date> | `vitesse_fronts` stable et cohérente avec la consigne. | Conforme | `<preuves/t6_vitesse.png>` |
| T7 E-Stop | <Date> | Arrêt immédiat de la PWM et mise en veille du pont en H. | Conforme | `<preuves/t7_estop.png>` |
| T8 Urgence et acquittement | <Date> | Clignotement rouge 2Hz, acquittement bloqué si E-Stop maintenu. | Conforme | `<preuves/t8_urgence.png>` |
| T9 Buzzer | <Date> | Bips nets de 100 ms, double bip en marche arrière. | Conforme | `<preuves/t9_buzzer.png>` |
| T10 SysTick et robustesse | <Date> | Système stable lors d'appuis simultanés ou préemption. | Conforme | `<preuves/t10_robust.png>` |

## Routine conservée pour L3-A
- **Routine (`button_pressed`, `led_set`, `motor_set_pwm` ou `encoder_read`) :** `motor_set_pwm` (ou celle de votre choix parmi la liste)
- **Interface :** R0 = rapport cyclique (0 à 100 %) -> retourne R0 = valeur écrite dans CCR1. Conformité AAPCS assurée (R4-R11 préservés).
- **Cas d'essai et résultats :** Entrée R0 = 50 -> Écrit 100 dans CCR1, retourne 100. Entrée R0 = 120 (hors plage) -> Sature à 100, écrit 200 dans CCR1, retourne 200.

## Déclaration des sources et de l'usage d'outils d'IA générative
- **Sources (dont le complément de démarrage et, le cas échéant, le corrigé du laboratoire 1) :** Complément de démarrage `CEG3536_Lab2_Complement.zip` fourni sur Brightspace. Code de la machine à états issu de notre Laboratoire 1 d'équipe.
- **Outils d'IA (outil, version, usage) ou « aucun usage » :** Assistant IA, version d'automne 2026, utilisé comme outil d'appoint pour générer la structure de formatage Markdown du fichier `JOURNAL.md` et valider les calculs théoriques des registres.
