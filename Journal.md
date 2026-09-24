# CEG3536
# Journal d'Équipe - Laboratoire 1 (CEG 3536)

## 1. Informations Générales
* **Numéro d'Équipe :** [Ex: Équipe 42]
* **Membres :**
  * Membre A : [Simon Boisvert, sbois090@uottawa.ca, SimBois]
  * Membre B : [Samuel Dos Santos Caiado, scaia009@uottawa.ca, scaia009]
* **Dépôt Git :** [URL HTTPS de votre dépôt GitHub/GitLab]

---

## 2. Jalon J1 : Exigences, Rôles et Échéancier (Validé le 25 septembre 2026)

### A. Rôles et Rotation pour le Laboratoire 1
Afin de respecter la politique de rotation des rôles, la structure suivante est adoptée :
* **Séance 0 (Prise en main) :** 
  * Réalisation : Membre A & Membre B
* **Séance 1 (Mise en œuvre - E1 à E3) :** 
  * Réalisation (Code) : **[Simon Boisvert]**
  * Validation (Essais & Relecture) : **[Samuel Caiado]**
* **Séance 2 (Achèvement & Démo - E4 à E9) :** 
  * Réalisation (Code) : **[Samuel Caiado]**
  * Validation (Essais & Relecture) : **[Simon Boisvert]**

### B. Échéancier de l'Équipe (Laboratoires 1 à 5)
* **Lab 1 (Panneau de commande) :** 
  * Jalon J1 validé : 25 septembre 2026
  * Démonstration : [29 septembre (A02) OU 2 octobre (A01)] 2026
  * Remise du rapport : 9 octobre 2026, 23 h 59
* **Lab 2 (Commande du moteur & SysTick) :** En attente des directives.
* **Lab 3 (Interface C / Assembleur) :** En attente des directives.
* **Lab 4 (TouchPad & Autorisation) :** En attente des directives.
* **Lab 5 (Projet Final) :** En attente des directives.

---

## 3. Journal des Décisions et de Suivi du Code

### Séance 0 (15 / 18 Septembre 2026)
* **Décisions prises :** Création du dépôt Git privé. Importation du projet `CEG3536_Lab1_Demarrage`.
* **Statut du code :** Compilation initiale réussie sur la Nucleo-L552ZE-Q. La DEL rouge s'allume seule après réinitialisation (Exigence E1 validée).

### Séance 1 (22 / 25 Septembre 2026)
* **Décisions prises :** Lecture du registre IDR pour confirmer la configuration matérielle des broches.
* **Statut du code :** 
  * [ ] Niveaux logiques des boutons confirmés (Essai T4).
  * [ ] Routine `button_pressed` complétée avec gestion anti-rebond (Exigence E3).
  * [ ] Machine à états complétée pour le défilement User (Exigence E2).

---

## 4. Tableau de Suivi des Essais (Séance 1)

| Essai | Procédure | Résultat Attendu | Résultat Observé | Verdict | Preuve / Note |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **T1** | Appuyer sur uC Reset | État ARRÊT : DEL rouge allumée seule dans les 100 ms. | *[À remplir]* | **[Pass/Fail]** | Capture ODR prise |
| **T2** | 4 appuis successifs sur User | Cycle : Rouge → Vert → Rouge → Bleu → Rouge. | *[À remplir]* | **[Pass/Fail]** | Relevé ODR consigné |
| **T3** | 20 appuis rapides + 1 appui maintenu (2s) | Exactement 21 transitions détectées au total. | *[À remplir]* | **[Pass/Fail]** | Variable `compteur_transitions` |
| **T4** | Lecture brute IDR (boutons lâchés/appuyés) | Niveaux actifs et choix PUPDR documentés. | *[À remplir]* | **[Pass/Fail]** | Tableau IDR complété |
