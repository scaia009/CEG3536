# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 2 (automne 2026)

Équipe : `<NomA>` et `<NomB>` — Section : `A01 / A02` — Dépôt Git : `<adresse>`

## Suivi du jalon J1

| Laboratoire | Échéancier prévu (J1) | Déroulement réel | Écart et cause | Ajustement pour les laboratoires 3 à 5 |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |

## Rôles et rotation

| Séance | Réalise | Valide (essais, mesures, relecture) | Périphérique configuré ou mesuré par chacun |
|---|---|---|---|
| Séance 1 (13 / 16 oct.) | | | |
| Séance 2 (20 / 23 oct.) | | | |

## Calculs préalables (avant la séance 1)

| Grandeur | Formule | Valeur | Vérifiée par (essai) |
|---|---|---|---|
| `PWM_PSC`, `PWM_ARR` | f = 4 MHz / ((PSC + 1)(ARR + 1)) = 20 kHz | | T3 |
| `CCR1` pour 25 / 50 / 75 / 100 % | pct × (ARR + 1) / 100 | | T3 |
| `SYSTICK_LOAD_1MS` | 4 MHz × 1 ms − 1 | | T10 |
| Fronts par tour (arbre de sortie) | 2 × 11 × 34 | | T5 |
| tr/min à partir de `vitesse_fronts` | fronts × 120 / 748 (fenêtre 500 ms) | | T6 |
| Fronts attendus par fenêtre de 500 ms à 100 % | 350 tr/min (arbre de sortie) × 748 / 120 | | T6 |

## Journal des séances

### Séance 1 — `<date>` — réalise : `<nom>` / valide : `<nom>`
- Objectifs :
- Fait :
- Décisions (sens réel de rotation pour AI1 = 1, AI2 = 0 ; `ENC_SENS_INVERSE` ; variante buzzer) :
- Difficultés et solutions :
- Mesures (PWM, SysTick, bips) :
- Validations Git (auteur, message) :

### Séance 2 — `<date>` — réalise : `<nom>` / valide : `<nom>`
- Objectifs :
- Fait :
- Décisions :
- Difficultés et solutions :
- Mesures (encodeurs, vitesse, délai E-Stop) :
- Validations Git :

## Tableau des mesures (composante L2-A)

| Mesure | Exigence | Valeur mesurée | Incertitude / méthode | Écart | Cause | Preuve (fichier) |
|---|---|---|---|---|---|---|
| Fréquence PWM | 20 kHz ± 5 % | | | | | |
| Rapport cyclique à 25 / 50 / 75 / 100 % | ± 2 points | | | | | |
| Rampe 0 → 100 % | 1 s ± 10 % | | | | | |
| Période SysTick | 1 ms ± 1 % | | | | | |
| Vitesse à 25 / 50 / 75 / 100 % (avant) | croissante, stable ± 10 % | | | | | |
| Vitesse à 25 / 50 / 75 / 100 % (arrière) | idem | | | | | |
| Déphasage E_A / E_B (avant, arrière) | quadrature, inversé selon le sens | | | | | |
| Délai E-Stop → PWM nulle | < 1 ms | | | | | |
| Durée d'un bip | 100 ms ± 10 % | | | | | |

## Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation | | | | |
| T2 Cycle User et sens | | | | |
| T3 PWM | | | | |
| T4 Rampe | | | | |
| T5 Encodeur | | | | |
| T6 Vitesse | | | | |
| T7 E-Stop | | | | |
| T8 Urgence et acquittement | | | | |
| T9 Buzzer | | | | |
| T10 SysTick et robustesse | | | | |

## Routine conservée pour L3-A
- Routine (`button_pressed`, `led_set`, `motor_set_pwm` ou `encoder_read`) :
- Interface :
- Cas d'essai et résultats :

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources (dont le complément de démarrage et, le cas échéant, le corrigé du laboratoire 1) :
- Outils d'IA (outil, version, usage) ou « aucun usage » :
