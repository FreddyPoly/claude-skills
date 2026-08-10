---
name: brainstorming
description: Interroger longuement l'utilisateur sur une nouvelle idée, fonctionnalité ou problème avant de planifier ou d'implémenter quoi que ce soit — points de friction, cas limites, choix techniques, spécifications et cas d'usage manquants, sécurité (auth, données sensibles, secrets, entrées non fiables — toujours au moins soulevée). Se termine en consignant les décisions validées, y compris une section Sécurité obligatoire, dans SPEC.md pour qu'un futur agent reprenne le projet sans repasser par l'interview. À utiliser quand l'utilisateur décrit quelque chose à construire/résoudre et veut être interrogé au préalable, ou invoque /brainstorming explicitement. Ne pas utiliser pour des demandes petites déjà bien spécifiées (typo, changement d'une ligne) — réservé aux idées/problèmes réellement nouveaux dont les exigences ne sont pas encore définies.
---

# Brainstorming

Avant de concevoir ou d'implémenter quoi que ce soit, interroge l'utilisateur jusqu'à ce que vous
partagiez tous les deux une compréhension réelle de ce qui est construit et pourquoi. Ne laisse pas
cela se réduire à une seule série de questions suivie d'un plan — traite-le comme une conversation
qui se poursuit jusqu'à épuisement des questions ouvertes, et non comme un formulaire à remplir une
fois pour toutes.

## Avant de poser la moindre question

Si cela se passe dans un projet existant, prends d'abord quelques minutes pour t'ancrer dans la
réalité : lis le code, la configuration et la documentation pertinents (CLAUDE.md ou équivalent,
fichiers liés, patterns existants). Des questions construites sur ce qui existe réellement ("Je vois
que X fait déjà Y — la nouvelle fonctionnalité doit-elle en tirer parti, ou est-ce volontairement
séparé ?") sont bien plus utiles que des questions génériques, et elles montrent que tu as vraiment
regardé. S'il n'y a pas de projet ou de base de code existante à examiner, saute cette étape et pose
directement tes questions — n'invente pas de contexte qui n'existe pas.

## Ce qu'il faut couvrir

Explore les angles suivants — non pas comme une liste rigide à réciter, mais comme un ensemble de
prismes garantissant que rien d'important n'est laissé de côté :

- **Points de friction** — ce qui est concrètement cassé ou pénible aujourd'hui, pas seulement dans
  l'abstrait
- **Problèmes à anticiper** — modes de défaillance, situations de concurrence, cas limites/d'échelle,
  ce qui se passe quand les choses tournent mal ou que les entrées sont inhabituelles
- **Choix techniques/outils** — quand il existe une vraie décision à prendre (nouvelle dépendance,
  nouveau pattern, plusieurs approches valables), expose explicitement le compromis plutôt que d'en
  choisir un silencieusement
- **Spécifications manquantes** — comportements que l'utilisateur n'a pas encore fixés et qui
  admettent plusieurs interprétations raisonnables
- **Cas d'usage manquants** — scénarios que l'utilisateur n'a pas mentionnés mais que la
  fonctionnalité/l'idée devra quand même gérer
- **Sécurité** — obligatoire, pas optionnel : établis toujours au moins si le projet touche à
  l'authentification/aux autorisations, à des données sensibles ou personnelles, à des
  secrets/identifiants, à des entrées non fiables (données soumises par l'utilisateur, API tierces,
  fichiers téléversés), ou à des exigences de conformité. Si la réponse est vraiment "rien de tout
  ça" pour un petit projet local à faible risque, fais-le dire explicitement plutôt que de laisser
  le sujet passer sous silence — un "non" documenté est acceptable, une question jamais posée ne
  l'est pas. Si l'un de ces points s'applique, va aussi loin que pour n'importe quelle autre vraie
  décision : qui a confiance en quoi, comment les secrets sont stockés, ce qui se passe avec une
  entrée malformée ou malveillante, quel est l'impact si cette partie précise tombe en panne. La
  profondeur doit être proportionnée à ce que le projet touche réellement — ce n'est pas un exercice
  complet de modélisation des menaces pour un script sans utilisateurs ni exposition réseau, mais ça
  l'est pour tout ce qui touche à l'authentification, aux paiements, aux données personnelles ou aux
  entrées externes
- **Approfondissement sur l'ensemble du plan** — ne t'arrête pas à la première couche ; chaque réponse
  révèle généralement une autre décision sous-jacente (c'est normal — continue, ne le considère pas
  comme une dérive du périmètre)

## Comment poser les questions

- Privilégie `AskUserQuestion` pour les points de décision concrets — c'est plus rapide pour
  l'utilisateur que de la prose, et cela t'oblige à vraiment nommer les options plutôt que de
  demander vaguement. Regroupe les questions liées entre elles (jusqu'à 4 par appel) plutôt que de
  les poser une par une.
- Quand tu as une véritable recommandation, place-la en première option et marque-la
  "(Recommandé)" — ne présente pas de menus faussement neutres quand tu connais en réalité la
  meilleure option par défaut.
- Utilise des questions en texte libre/prose pour ce qui ne se réduit pas à un menu (chiffres,
  descriptions, "qu'est-ce qui se passe d'autre ici").
- Procède par rounds. Les réponses du premier round ouvrent régulièrement un deuxième round — c'est
  le signe que l'interview fonctionne, pas que tu as posé les mauvaises questions au départ. Continue
  jusqu'à ce qu'il n'y ait vraiment plus de questions ouvertes, pas jusqu'à en avoir posé "assez".
- Ne pose pas de questions sur ce que tu peux simplement aller vérifier toi-même (code existant,
  contenu de fichiers, valeurs par défaut évidentes) — réserve les questions à ce que seul
  l'utilisateur peut réellement décider.

## Quand s'arrêter

Arrête-toi quand tu pourrais reformuler fidèlement la demande à l'utilisateur — objectif,
contraintes, cas limites, et forme d'une solution — sans qu'il ait besoin de te corriger. À ce
moment-là, résume la compréhension partagée en prose (pas une nouvelle série de questions) pour
qu'elle soit visible et confirmable.

## Consigner la décision

Une fois que l'utilisateur confirme le résumé, consigne-le dans un document à la racine du projet
avant de passer à la suite — c'est ce qui permettra à un futur agent (ou à toi-même dans une future
session) de reprendre le projet à froid et de commencer à implémenter sans refaire l'interview.

- Réutilise `SPEC.md` si le projet en a déjà un (voir la convention de la skill `documentation`) ;
  crée-le sinon. Pour un projet trop petit pour justifier un fichier dédié, une section clairement
  identifiée dans `CLAUDE.md` fait aussi l'affaire.
- N'écris que les décisions réellement validées dans la conversation — objectif, contraintes, cas
  limites couverts, approche retenue, et compromis explicitement tranchés (y compris les
  alternatives rejetées et pourquoi, quand ce n'est pas évident). Ce n'est pas une transcription des
  questions/réponses — condense-la sous la forme des décisions prises, pas des échanges qui y ont
  mené.
- Si `SPEC.md` contient déjà du contenu, réconcilie tes ajouts avec l'existant plutôt que d'ajouter
  un bloc déconnecté — mets à jour ce que l'interview vient de changer ou de clarifier.
- Inclus toujours une section `## Sécurité`, même quand la réponse est "non applicable" — indique
  le niveau de risque et pourquoi (par ex. "pas d'authentification, pas de données utilisateur
  persistées, pas d'exposition réseau — risque faible"). Quand des sujets de sécurité s'appliquent
  vraiment, consigne les décisions concrètes : ce qui est fiable ou non, comment les secrets sont
  gérés, comment les entrées non fiables sont traitées, quelles données sont sensibles et comment
  elles sont protégées. C'est cette section qui permet à `doc-to-issues` de repérer en aval quels
  issues sont sensibles côté sécurité — ne la laisse pas trop vague pour ça.
- Laisse les questions encore ouvertes marquées comme telles si certaines restent réellement non
  résolues (par exemple reportées par l'utilisateur à plus tard) — ne fais pas comme si elles
  étaient tranchées.

Si le résultat s'oriente vers des changements de code, ce document est ton signal pour passer à la
planification (par exemple le mode plan) ensuite — mais ne commence pas à rédiger un plan ni à
toucher aux fichiers d'implémentation tant que l'interview est encore ouverte.
