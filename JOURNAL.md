# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 1 (automne 2026)

Équipe : `Simon Boisvert` et `Samuel Caiado` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536`

## Jalon J1 (au plus tard le vendredi 25 septembre 2026, validé dans Git)

### Exigences de l'équipe
| Id | Exigence (reformulée par l'équipe) | Critère d'acceptation | Hypothèses |
|---|---|---|---|
| E1 | | | |
| E2 | | | |
| E3 | | | |
| E4 | | | |
| E5 | | | |
| E6 | | | |
| E7 | | | |
| E8 | | | |
| E9 | | | |

### Rôles et rotation
| Séance | Réalise | Valide (essais, mesures, relecture) |
|---|---|---|
| Séance 0 | Simon Boisvert | Samuel Caiado |
| Séance 1 | Samuel Caiado | Simon Boisvert |
| Séance 2 | Simon Boisvert | Samuel Caiado |

### Échéancier des laboratoires 1 à 5
| Laboratoire | Séances | Démonstration | Remise | Responsable du suivi |
|---|---|---|---|---|
| 1 | #0, #1 et #2 | 2 octobre 2026 | 9 octobre 2026 | Simon Boisvert |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

## Journal des séances

### Séance 0 — `18 septembre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs :
  Prendre en main le kit Leafy + Nucleo-L552ZE-Q.
  Créer et configurer le projet STM32CubeIDE en assembleur, avec TrustZone désactivé.
  Écrire, compiler et charger le premier programme pour allumer la DEL rouge LD3 sur PA9.
  Vérifier le fonctionnement au débogueur et observer les registres RCC/GPIO.
  Initialiser le dépôt Git de l’équipe et commencer JOURNAL.md.
- Fait :
  Projet STM32CubeIDE créé et compilé.
  Programme assembleur main.s écrit et chargé sur la carte.
  Horloge de GPIOA activée, PA9 configurée en sortie et LD3 allumée.
  Vérifications effectuées au débogueur sur RCC_AHB2ENR, GPIOA_MODER, GPIOA_ODR et GPIOA_IDR.
  Dépôt Git initialisé et journal de séance commencé.
- Décisions :
  Utiliser un projet non sécurisé avec TZEN = 0.
  Conserver les options de compilation du cours : Cortex-M33, Thumb, FPU fpv5-sp-d16, ABI hard.
  Utiliser BSRR pour contrôler la DEL sans modifier les autres broches du port.
  Conserver une boucle stop à la fin de main.
  Documenter les constantes et adresses des registres avec les références au RM0438.
- Difficultés et solutions :
  Vérification du branchement USB sur CN1 plutôt que CN15 pour le ST-LINK.
- Essais et mesures :
  Lecture de RCC_AHB2ENR : bit 0 à 1.
  GPIOA_MODER configuré à 0xABF7FFFF, avec les bits 19:18 configurés à 01.
  GPIOA_ODR : bit 9 à 1.
  GPIOA_IDR : bit 9 à 1.
  LD3 rouge allumée sur PA9.
- Validations Git (auteur, message) :
  Simon Boisvert — réalisation de la séance 0 et du premier programme assembleur.
  Samuel Caiado — validation de la séance 0 et vérification du dépôt/journal.

### Séance 1 — `25 septembre 2026` — réalise : `Samuel Caiado` / valide : `Simon Boisvert`
- Objectifs :
- Fait :
- Décisions :
- Difficultés et solutions :
- Essais et mesures :
- Validations Git :

### Séance 2 — `2 octobre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs :
- Fait :
- Décisions :
- Difficultés et solutions :
- Essais et mesures :
- Validations Git :

## Tableau des essais (T1 à T10)
| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation | | | | |
| T2 Cycle User | | | | |
| T3 Anti-rebond | | | | |
| T4 Niveaux logiques | | | | |
| T5 E-Stop | | | | |
| T6 Clignotement | | | | |
| T7 Acquittement | | | | |
| T8 User ignoré en urgence | | | | |
| T9 Touch En hors urgence | | | | |
| T10 Robustesse | | | | |

## Routine conservée pour L3-A
- Routine : `button_pressed` ou `led_set`
- Interface :
- Cas d'essai :

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources :
- Outils d'IA (outil, version, usage) ou « aucun usage » :
