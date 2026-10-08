# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 1 (automne 2026)

Équipe : `Simon Boisvert` et `Samuel Caiado` — Section : `A02` — Dépôt Git : `https://github.com/scaia009/CEG3536`

## Jalon J1 (au plus tard le vendredi 25 septembre 2026, validé dans Git)

### Exigences de l'équipe
| Id | Exigence (reformulée par l'équipe) | Critère d'acceptation | Hypothèses |
|---|---|---|---|
| E1 | Au démarrage du système, que ce soit après une mise sous tension ou un appui sur le bouton uC Reset (NRST), le véhicule doit se retrouver dans un état de repos bien défini, sans intervention de l'utilisateur. | Immédiatement après la réinitialisation (dans les 100 ms suivant NRST), seule la DEL rouge est allumée; les DEL verte et bleue sont éteintes. Ce comportement doit être reproductible à chaque réinitialisation. | L'horloge du microcontrôleur reste celle configurée par défaut au démarrage (MSI à 4 MHz), puisque SystemInit est vide dans le code de départ; le délai de 100 ms est donc facilement respecté sans configuration additionnelle du RCC pour l'horloge système. |
| E2 | Le bouton User doit permettre à l'utilisateur de faire défiler manuellement les états de marche du véhicule, dans un ordre fixe qui passe toujours par l'état ARRÊT lors d'un changement de sens. | Chaque appui validé (un seul événement par appui, voir E3) sur User fait avancer l'état selon la séquence ARRÊT → MARCHE_AVANT → ARRÊT → MARCHE_ARRIÈRE → ARRÊT (rouge → verte → rouge → bleue → rouge). Une seule DEL est allumée à la fois, jamais deux simultanément, et le système ne peut jamais passer directement de MARCHE_AVANT à MARCHE_ARRIÈRE sans repasser par ARRÊT. | Le bouton User (PC13) est actif à l'appui selon le niveau logique confirmé à l'essai T4 (lecture du registre IDR); la logique de défilement utilise le résultat déjà normalisé (« appuyé = 1 ») fourni par button_raw, donc elle est indépendante du niveau électrique réel du bouton. |
| E3 | La lecture du bouton User doit filtrer les rebonds mécaniques du contact afin qu'un seul appui physique ne génère jamais plus d'un événement, et qu'un appui maintenu ne génère qu'un seul événement plutôt qu'une répétition continue. | Avec une fenêtre d'anti-rebond de 20 à 50 ms, 20 appuis consécutifs sur User doivent produire exactement 20 transitions d'état, sans détection double ni appui manqué; un appui maintenu sur plusieurs secondes ne doit produire qu'une seule transition. | La boucle principale appelle fsm_step (et donc button_pressed) à une cadence d'environ 1 ms (PERIODE_SCRUTATION_MS), ce qui sert de base de temps à l'anti-rebond; une fenêtre initiale de [20 à 50, valeur à préciser selon vos tests] échantillons consécutifs au même niveau sera utilisée, puis ajustée si des rebonds persistent ou si des appuis courts sont manqués lors des essais T3. |
| E4 | E-Stop (PB2) déclenche une interruption de priorité maximale qui met les sorties en état sûr. | EXTI2, priorité 0. L'ISR éteint verte et bleue, allume la rouge, met estop_flag à 1 et efface la requête; réaction en moins de 10 ms, même pendant delay_ms. | Front choisi selon le niveau actif de PB2 (T4). Sur la L5, EXTICR est dans EXTI. |
| E5 | En urgence, la rouge clignote et User est ignoré. | Rouge à 2 Hz (250 ms / 250 ms, ± 10 %); verte et bleue éteintes; User sans effet. | Demi-période de 250 passages de fsm_step, mesurée avec DWT_CYCCNT. |
| E6 | La sortie d'urgence exige un acquittement volontaire. | Touch En validé avec E-Stop relâché → ARRÊT; refusé si E-Stop maintenu; aucune reprise automatique. | E-Stop lu avec button_raw; estop_flag remis à 0 dès sa consommation (choix A). |
| E7 | Hors urgence, Touch En inverse une autorisation et le signale. | touch_enabled bascule 0/1 à chaque appui validé; DEL active éteinte 100 ms; etat inchangé. | Extinction comptée en 100 passages de fsm_step. |
| E8 | Les durées viennent d'une routine delay_ms calibrée. | Calibration par mesure (delay_ms(100) ≈ 400 000 cycles à 4 MHz); valeur de DELAY_BOUCLES_PAR_MS rapportée. | Boucle calibrée admise au labo 1; SysTick au labo 2. |
| E9 | Le système reste cohérent dans les cas imprévus. | Jamais deux DEL allumées; jamais de sortie d'urgence sans acquittement. | estop_flag consommé avant les autres transitions dans fsm_step. |

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
- Objectifs : gpio_init, led_set, button_raw, button_pressed (anti-rebond), états E1 à E3, niveaux logiques (T4), jalon J1.
- Fait : Samuel : gpio_init et initialisation des boutons (button_raw). Simon : button_pressed (anti-rebond et front) et machine à états pour E1 à E3. Niveaux logiques relevés dans IDR (T4); essais T1 à T4 réalisés. Jalon J1 (exigences, rôles, échéancier) rédigé dans JOURNAL.md.
- Décisions : Anti-rebond de 30 échantillons consécutifs (ANTIREBOND_MS = 30, dans la plage de 20 à 50 ms); un appui n'est signalé que sur le front d'appui validé. Variable prochain_sens pour alterner avant/arrière à partir de ARRÊT. Un seul point d'appel de led_set : fsm_maj_del (critère B3). E-Stop et Touch En en tirage haut interne (PUPDR = 01) : actifs bas, ils flottaient sans tirage. User sans tirage (tirage externe de la Nucleo).
- Difficultés et solutions : Les entrées PB2 et PB5 ne sont pas stables relâchées sans tirage interne : PUPDR mis à tirage haut et BTN_ESTOP_ACTIF_HAUT / BTN_TOUCH_ACTIF_HAUT mis à 0 dans registres.inc.
- Essais et mesures : T4 : User relâché 0 / appuyé 1 (actif haut); E-Stop relâché 1 / appuyé 0 (actif bas); Touch En relâché 1 / appuyé 0 (actif bas). T1 : etat = 0, rouge seule (PA9 = 1, PC7 = 0, PB7 = 0), reproduit 5 fois. T2 : etat 0 → 1 → 0 → 2 → 0 avec ODR conformes. T3 : compteur_transitions 0 → 20 après 20 appuis, puis 20 → 21 après un appui maintenu 2 s.
- Validations Git : Samuel Caiado — gpio_init et button_raw. Simon Boisvert — button_pressed et FSM E1 à E3.

### Séance 2 — `2 octobre 2026` — réalise : `Simon Boisvert` / valide : `Samuel Caiado`
- Objectifs : estop_init et ISR, urgence et acquittement, Touch En, robustesse, mesure du clignotement, démonstration.
- Fait : Samuel : estop_init et EXTI2_IRQHandler (interruption E-Stop, priorité 0). Simon : acquittement par Touch En (fsm_step, fsm_maj_del), clignotement, et suite d'essais T5 à T10. Toutes les exigences E1 à E9 vérifiées; démonstration devant l'assistant [À REMPLIR : date et résultat].
- Décisions : estop_flag remis à 0 par fsm_step dès sa consommation (choix A); l'urgence est portée par etat = 3. L'ISR met les sorties en sécurité (BSRR) avant de lever le drapeau, pour respecter les 10 ms même pendant delay_ms. En urgence, fsm_step appelle button_pressed pour User (ignoré) et pour Touch En à chaque passage. Clignotement mesuré par DWT_CYCCNT (blink_cycles); la première valeur est ignorée.
- Difficultés et solutions : aucun écart constaté aux essais finaux.
- Essais et mesures : T5 : avant l'écriture, OD7 = 1, estop_flag = 0, IABR bit 13 = 1; après, OD7 = 0, OD9 = 1, estop_flag = 1; puis etat = 3. T6 : 10 relevés; min 998 450 cycles (249,61 ms), max 1 001 120 (250,28 ms), moyenne 1 000 080 (250,02 ms), écart 0,008 %; période complète d'environ 500,04 ms (2 Hz). T7 : E-Stop relâché, etat 3 → 0; E-Stop maintenu, etat reste 3; 10 s sans acquittement, etat = 3. T8 : etat = 3 et compteur_transitions = 21 avant et après les appuis sur User. T9 : touch_enabled 0 → 1 → 0, compteur_transitions inchangé, extinction brève visible. T10 : 15 essais par scénario, viol_cnt = 0, etat toujours entre 0 et 3.
- Validations Git : Samuel Caiado — estop_init et ISR. Simon Boisvert — acquittement, clignotement et essais.

## Tableau des essais (T1 à T10)
| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation | 25 septembre 2026 | etat = 0; PA9 = 1, PC7 = 0, PB7 = 0; reproduit 5 fois | Réussi |  |
| T2 Cycle User | 25 septembre 2026 | etat 0 → 1 → 0 → 2 → 0; ODR conformes | Réussi | |
| T3 Anti-rebond | 25 septembre 2026 | 0 → 20 après 20 appuis; 20 → 21 après l'appui maintenu | Réussi | |
| T4 Niveaux logiques | 25 septembre 2026 | User actif haut; E-Stop et Touch En actifs bas, tirage haut | Réussi | |
| T5 E-Stop | 2 octobre 2026 | ISR atteinte; verte/bleue éteintes, rouge allumée, estop_flag = 1, puis etat = 3 | Réussi |  |
| T6 Clignotement | 2 octobre 2026 | moyenne 1 000 080 cycles = 250,02 ms (min 249,61, max 250,28), écart 0,008 % | Réussi | <img width="1896" height="467" alt="t6" src="https://github.com/user-attachments/assets/f2f06878-5600-4ce5-9ad7-c5130e166395" /> |
| T7 Acquittement | 2 octobre 2026 | cas 1 : etat 3 → 0; cas 2 : reste 3; 10 s : etat = 3 | Réussi | <img width="1917" height="1017" alt="t7 1" src="https://github.com/user-attachments/assets/e1e94dff-f45b-468d-8ff3-ac02fe44634e" /> <img width="1917" height="1017" alt="t7 2" src="https://github.com/user-attachments/assets/db06fed1-1b7a-4161-ae01-2fa48ba056dd" /> |
| T8 User ignoré en urgence | 2 octobre 2026 | etat = 3 et compteur_transitions = 21 avant et après | Réussi | <img width="1917" height="502" alt="t81" src="https://github.com/user-attachments/assets/c0fc5f77-a372-4386-ae21-3cdb417b70a5" /> <img width="1917" height="1017" alt="t8 2" src="https://github.com/user-attachments/assets/a58cfaec-502a-4190-b4ff-dd718a736a23" />|
| T9 Touch En hors urgence | 2 octobre 2026 | touch_enabled 0 → 1 → 0; compteur inchangé; extinction visible | Réussi | <img width="1917" height="530" alt="t9 1" src="https://github.com/user-attachments/assets/2769b6bc-1555-4a55-a8aa-67c5a9f780a5" /> <img width="1917" height="472" alt="t9 2" src="https://github.com/user-attachments/assets/d7a67762-2913-4522-b47e-8c3e61bd8dee" /> |
| T10 Robustesse | 2 octobre 2026 | 15 essais par scénario; viol_cnt = 0; etat toujours dans 0 à 3 | Réussi | N/A |

## Routine conservée pour L3-A
- Routine : button_pressed
- Interface : R0 = identifiant (0 User, 1 E-Stop, 2 Touch En) → R0 = 1 si appui validé, 0 sinon. État dans btn_valide et btn_compteur (.bss). Préserve R4–R6, pile alignée sur 8.
- Cas d'essai : relâché → 0; appui net → un seul 1; appui maintenu → un seul 1; rebond court → 0; 20 appuis → 20 fois 1. Résultats : (a) retourne 0 en continu; (b) retourne 1 une seule fois; (c) retourne 1 une seule fois; (d) retourne 0, aucun rebond validé; (e) 20 transitions (T3); (f) retourne 0.

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources : énoncé v1.2 et code de départ (Brightspace); RM0438 et PM0264; manuel Zhu (4e éd.); guide Leafy.
- Outils d'IA (outil, version, usage) : Claude (Anthropic), Claude Sonnet 5.5, interface web claude.ai. Usage : explication de la procédure de preuve des essais T1 à T10 et du débogueur (Live Expressions, SFRs); proposition de code pour button_pressed, estop_init, EXTI2_IRQHandler, fsm_step et fsm_maj_del (y compris la mesure DWT); aide à la structure et à la rédaction du rapport et du journal. Le code proposé a été relu, compilé, testé sur la carte et adapté par l'équipe. Aucun usage pendant la démonstration.
