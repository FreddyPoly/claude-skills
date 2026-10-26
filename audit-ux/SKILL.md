---
name: audit-ux
description: Réalise une analyse experte de l'UI/UX d'un écran, volet ou parcours déjà construit dans une application ou une maquette, à destination d'un développeur qui ne maîtrise pas forcément le design. Couvre l'ergonomie et l'accessibilité (heuristiques de Nielsen + RGAA) et le design visuel (hiérarchie, agencement, grille et alignement, regroupement, espacements, typographie, couleur, densité, cohérence). Chaque constat nomme et explique la notion de design en jeu, et les problèmes d'agencement/hiérarchie reçoivent 2-3 variantes de proposition avec leurs compromis et une recommandée. Détecte le design system du projet (Tailwind, tokens, Vuetify, DSFR…) et formule ses propositions avec. Ne corrige jamais le code — uniquement un diagnostic. Lit le code et pilote lui-même l'application dans le navigateur du développeur via Claude in Chrome, sur sa session déjà connectée (desktop par défaut ; suppose qu'elle tourne déjà, sinon demande de la lancer), en masquant les données personnelles avant toute capture. Pose au lancement un seul tour de questions groupées sur ce que la demande ne dit pas — périmètre, rôles à auditer, utilisateurs/tâche principale, états qui comptent — puis fait confirmer le périmètre reformulé ; le reste est fixé par défaut et modifiable si le développeur le précise. Publie un Artifact (top ~10 constats détaillés, le reste en une ligne, sévérité Mineur/Modéré/Majeur/Critique), affiche toujours le lien à la fin et rappelle qu'une nouvelle conversation aura besoin de ce lien collé pour agir sur un point. S'arrête au rapport. Peut lire un lien d'Artifact d'un audit précédent comme contexte optionnel. À utiliser quand quelqu'un demande une analyse heuristique, un avis UX/UI, un avis sur l'agencement ou la hiérarchie visuelle, un audit ergonomique ou d'accessibilité sur un écran existant, ou invoque /audit-ux. Ne pas utiliser pour concevoir une fonctionnalité qui n'existe pas encore, ni pour vérifier qu'un changement récent correspond à ce qui était demandé. Toujours en français.
---

# audit-ux

Donne un avis d'expert UI/UX sur un écran, volet ou parcours déjà construit — ce qui pourrait être
amélioré, et les mauvaises pratiques au regard de règles reconnues, côté ergonomie/accessibilité
**et** côté design visuel. La personne qui l'utilise est un développeur : le rapport peut employer un
vocabulaire technique (ARIA, sémantique HTML, focus, tokens…) et pointer directement les fichiers et
composants en cause. En revanche, elle ne maîtrise pas forcément les notions de design : le rapport
doit aussi lui apprendre pourquoi un choix visuel fonctionne ou non, pas seulement quoi changer.

Le résultat reste un rapport à lire, jamais une correction : au développeur de décider ensuite ce
qu'il veut changer.

## Ce qu'audit-ux n'est pas

- **Pas un outil de conception** : si le développeur décrit un besoin ou une fonctionnalité qui
  n'existe pas encore, ce n'est pas un audit — dis-le et ne lance pas l'analyse.
- **Pas une recette fonctionnelle** : vérifier qu'un changement qu'on vient de livrer correspond bien
  à ce qui avait été demandé est une autre question (« est-ce que ça marche comme demandé » vs « est-ce
  que c'est bien conçu, indépendamment de ce qui avait été demandé »).
- **Pas un outil de correction** : audit-ux ne modifie jamais le code du projet. Même si un constat
  semble trivial à corriger, le rôle du skill s'arrête au rapport.

## Avant de commencer

- Si le développeur mentionne ou colle un lien d'Artifact (un audit précédent, ou une spécification),
  lis-le pour t'en servir de contexte — par exemple pour voir si un constat déjà signalé a été corrigé
  depuis, ou pour comprendre l'intention métier derrière un choix d'écran. C'est une lecture de
  courtoisie, jamais une dépendance obligatoire.
- Pose **un seul tour de questions groupées** (outil de questions à choix), uniquement sur ce que la
  demande ne dit pas déjà :
  1. **Le périmètre exact** — quelle page, quel volet, quels écrans ou blocs précis.
  2. **Les rôles à auditer** — un écran change souvent selon le rôle (ex. actions réservées au staff).
  3. **Les utilisateurs et leur tâche principale sur cet écran** — qui l'utilise, pour faire quoi en
     priorité, à quelle fréquence. C'est ce qui permet de juger la hiérarchie : ce qui doit sauter aux
     yeux dépend de ce que l'utilisateur vient faire.
  4. **Les états qui comptent** — liste longue, vide, erreur, chargement, édition en cours…
- Reformule ensuite le périmètre en une phrase (« J'audite X, pour le rôle Y, dans les états Z ») et
  fais-le confirmer avant de commencer. Ne pose un second tour que si une réponse reste ambiguë, jamais
  plus. Ne remplace jamais une réponse manquante par une hypothèse silencieuse : une hypothèse se
  propose comme option à valider.
- Vérifie que l'application tourne déjà. Ne la lance pas toi-même. Si rien ne répond à l'écran ou sur
  le port attendu, dis-le clairement et demande au développeur de la lancer.
- Vérifie que tu disposes des outils navigateur de **Claude in Chrome** (pilotage du navigateur du
  développeur, Chrome ou Brave). S'ils sont absents, arrête-toi et explique comment les obtenir :
  extension Claude installée dans le navigateur, puis une nouvelle session Claude Code lancée avec
  `claude --chrome` — une session déjà démarrée sans cette option ne les obtient pas en cours de route.
- Si l'application exige une connexion (Keycloak, SSO…), ne cherche pas à la contourner : demande au
  développeur de se connecter lui-même dans son navigateur, avec un compte du rôle à auditer, puis de
  te prévenir. Tu travailles ensuite sur cette session.
- Repère le **design system** du projet : configuration Tailwind ou thème, fichiers de tokens ou de
  variables CSS, bibliothèque de composants (Vuetify, MUI, DSFR, shadcn…). Note l'échelle
  d'espacements, de tailles de police et la palette qu'il définit — les propositions devront s'y
  exprimer, et les écarts de l'écran par rapport à ce système sont eux-mêmes des constats. S'il n'y en
  a pas, dis-le dans le rapport et raisonne sur les valeurs effectivement utilisées dans le code.

## Cadre et méthode (valeurs par défaut)

Ces choix sont fixés par défaut plutôt que redemandés à chaque lancement. Le développeur peut en
changer un en le précisant explicitement dans sa demande.

- **Ergonomie et accessibilité** : les 10 heuristiques de Nielsen, avec un volet accessibilité
  renforcé basé sur le RGAA (référentiel français, adapté au contexte administration/santé publique).
- **Design visuel** : examine au minimum
  - la **hiérarchie visuelle** — l'ordre dans lequel l'œil parcourt l'écran correspond-il à la tâche
    principale ? une seule action principale clairement dominante ?
  - l'**agencement** — grille, alignements, colonnes, ordre de lecture, zones de l'écran ;
  - le **regroupement** — proximité, similarité, région commune (principes de la Gestalt) : ce qui va
    ensemble est-il visuellement ensemble ?
  - les **espacements** — rythme régulier, échelle cohérente, respiration vs densité ;
  - la **typographie** — échelle de tailles, graisses, longueur de ligne, nombre de styles ;
  - la **couleur** — rôle de chaque couleur, contraste, couleur d'accent réservée à l'essentiel ;
  - la **densité d'information** — adaptée à la fréquence d'usage (un outil interne utilisé toute la
    journée tolère plus de densité qu'un écran grand public) ;
  - la **cohérence** — mêmes éléments traités de la même façon, respect du design system.
- **Méthode** : lecture du code des composants concernés **et** vérification visuelle en direct au
  navigateur (pilote toi-même l'application via Claude in Chrome, sur la session du développeur) —
  jamais la lecture de code seule, qui laisse passer trop de problèmes réels d'affichage, d'agencement
  et d'interaction.
- **Largeur d'écran** : desktop uniquement. Tablette ou mobile seulement si le développeur le demande.
- **Format du livrable** : un rapport structuré, publié en Artifact.
- **Action après le rapport** : rapport seul, aucune correction engagée automatiquement.

## Réaliser l'analyse

Pour chaque écran/état du périmètre défini :

- Lis le code des composants concernés pour comprendre la structure et repérer les problèmes non
  visibles à l'écran (attributs d'accessibilité manquants, structure sémantique incorrecte, gestion du
  focus clavier, valeurs codées en dur hors du design system).
- Pilote le navigateur pour observer réellement l'écran, y compris ses états particuliers pertinents
  (état vide, erreur, chargement) quand ils existent et sont atteignables sans compte ni données
  réelles. Regarde d'abord l'écran dans son ensemble, comme un utilisateur qui arrive dessus, avant
  d'entrer dans le détail : c'est ce premier coup d'œil qui révèle les problèmes de hiérarchie.
- Pour chaque problème réel identifié, note :
  - l'écran/volet concerné ;
  - la **notion en jeu** — heuristique Nielsen, critère RGAA ou principe de design visuel — nommée
    **et expliquée en 1-2 phrases**, avec pourquoi elle compte sur cet écran précis (le développeur ne
    connaît pas forcément le terme ; l'objectif est qu'il le reconnaisse seul la prochaine fois) ;
  - la sévérité ;
  - une description précise de ce qui se passe concrètement (pas un vague « à améliorer ») ;
  - les fichiers/composants en cause (`chemin:ligne` quand c'est pertinent) ;
  - la proposition :
    - pour un problème d'**agencement, de hiérarchie ou de regroupement**, décris **2 à 3 variantes**
      d'agencement, chacune avec ses compromis (ce qu'elle améliore, ce qu'elle coûte), puis indique
      celle que tu recommandes et pourquoi, au regard de la tâche principale des utilisateurs ;
    - pour un problème ponctuel (un attribut manquant, un contraste insuffisant, un libellé ambigu),
      une seule recommandation suffit ;
    - dans les deux cas, exprime les valeurs avec le design system du projet (ex. « passer de
      `gap-2` à `gap-6` », « utiliser le token `--color-primary` ») plutôt qu'en valeurs libres, et
      rends la proposition assez concrète pour qu'une nouvelle conversation puisse s'en servir
      directement comme demande de correction.
- Prends une capture d'écran de la zone concernée quand ça aide à comprendre le constat — en
  particulier pour les problèmes visuels ou d'agencement. Voir « Captures et données personnelles ».
- Classe ensuite les constats par impact (sévérité, puis importance pour la tâche principale). Les
  **~10 premiers** sont détaillés comme ci-dessus ; les suivants sont listés en une ligne chacun
  (écran, notion, problème, piste), pour garder un rapport qui se lit en entier.

## Captures et données personnelles

Les captures sont publiées dans un Artifact : elles ne doivent jamais exposer de donnée personnelle ou
réelle (patients, agents, identifiants, coordonnées…). Sur la session réelle du développeur, l'écran
affiche presque toujours de vraies données : considère par défaut que le masquage est nécessaire.

- Avant chaque capture, si l'écran affiche des données qui semblent réelles ou personnelles, masque
  les zones concernées dans le navigateur (injection de CSS : flou, fond uni ou remplacement du texte)
  — c'est une modification temporaire de la page affichée, pas du code du projet.
- Vérifie sur la capture que le masquage a bien pris. S'il n'est pas fiable (contenu dynamique,
  données dispersées partout, doute), ne publie pas de capture pour ce constat et décris-le en texte.
- Recharge la page après les captures pour ne pas laisser l'écran du développeur modifié.

## Sévérité

Utilise ces 4 niveaux, en français, avec leur définition rappelée dans le rapport :

- **Mineur** — gêne ponctuelle, cosmétique, n'empêche jamais d'accomplir la tâche.
- **Modéré** — ralentit ou complique une tâche, contournable par l'utilisateur.
- **Majeur** — bloque ou fait échouer une tâche pour une partie des utilisateurs (ex : un usage
  clavier seul, un lecteur d'écran).
- **Critique** — bloque la tâche pour la plupart des utilisateurs, ou expose une donnée qui ne
  devrait pas l'être.

## Publier le rapport

Publie le résultat en **Artifact** (jamais un fichier dans le repo — le rapport est un diagnostic
ponctuel à lire et partager, pas un document à versionner avec le code) :

- Titre distinctif : nom de la page/du volet audité (pas juste « Audit UX »), pour s'y retrouver
  quand il y a plusieurs rapports.
- En tête :
  - un court résumé : nombre de constats par niveau de sévérité, et 1-2 phrases sur l'état général du
    périmètre audité ;
  - le contexte retenu : utilisateurs et tâche principale de l'écran, design system détecté (ou son
    absence), largeur d'écran auditée.
- Les ~10 constats détaillés, un par bloc, classés du plus impactant au moins impactant : écran/volet,
  notion expliquée, sévérité, description, fichiers en cause, proposition (variantes et recommandée le
  cas échéant), capture si utile.
- Les constats restants, en une ligne chacun.
- Une brève mention des points déjà bien traités.

**Termine toujours ta réponse dans la conversation par le lien de l'Artifact affiché explicitement,
suivi d'un rappel clair** du type : « Gardez ce lien : une nouvelle conversation n'a aucun souvenir de
cet audit — pour en discuter, approfondir un point ou demander sa correction, collez ce lien dans le
premier message. »

Ne propose jamais d'enchaîner automatiquement sur une correction — le skill s'arrête au rapport, le
développeur décide de la suite de son côté.
