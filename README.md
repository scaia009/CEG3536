# Laboratoire 3 — fichiers complétés

Ce zip = le complément CEG3536_Lab3 dont tous les « À COMPLÉTER » sont écrits :
`vitesse.s` (E3), `evq.c` (E7), `acq.c` (E4-E6, versions 1 et 2), `adc_dma.c` (E4),
`telem.c` (E10), `fsm.c` (E5, E8, E9), `main.c` (E3, E5), `l3a_c.c` (L3-A).

## Contenu de Src/ (vos fichiers des laboratoires 1 et 2, tels quels)
gpio.s, buttons.s, estop.s, motor.s, encoder.s, systick.s, buzzer.s, registres.inc,
registres_lab2.inc, syscalls.c, sysmem.c (+ lab2.h dans Inc/).
main.s et fsm.s ne sont volontairement PAS inclus : main.c et fsm.c les remplacent.
Inc/lab1.h (le vôtre) et STM32L552ZETXQ_FLASH.ld sont inclus.
Startup/startup_stm32l552zetxq.s est inclus : le zip est complet.

## Vérifié ici
- hote/ : `make` -> 9 cas sur 9 réussis (gcc, -Wall -Wextra, sans avertissement).
- Constante k = 21028, BRR = 8889, cas T3 sans débordement 32 bits.
- Tous les .c passent `gcc -fsyntax-only -Wall -Wextra` avec vos lab1.h / lab2.h / registres (pas de compilation ARM, assemblage des .s ni essai sur carte).

## À vérifier sur votre matériel
- `l3a_c.c` : aligné sur votre motor.s (saturation à 100, CCR1 = pct x 200 / 100, retour = CCR1) ; vérifier `B,ecarts=0`.
- Noms/types de lab2.h utilisés : button_raw, encoder_read(&sens_mesure), sens_mesure, motor_stop_now.
- Tampon DMA porté à 64 échantillons/voie (ADC_N_TAMPON) pour permettre la moyenne sur 64 de la
  température (version 2) ; un tour dure ~21 ms.
- telem_put est « tout ou rien » : une trame qui ne tient pas dans la file est refusée, jamais tronquée.
- Étalonnage température : relever TEMP_MV_REF / TEMP_C10_REF (commande a) et les mettre dans lab3.h.

## Disposition du projet (d'après votre _cproject)
Votre projet utilise Src/, Inc/ et Startup/ à la racine (pas Core/) : ce zip suit cette disposition.
Dans STM32CubeIDE (Debug ET Release), ajouter :
- Include paths (C) : `../Drivers/CMSIS/Include` et `../Drivers/CMSIS/Device/ST/STM32L5xx/Include`
- Symbole C : `STM32L552xx` (votre projet n'a que STM32L552ZETxQ, insuffisant pour CMSIS)
- Avertissements C : `-Wextra`
