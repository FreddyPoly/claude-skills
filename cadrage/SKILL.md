---
name: cadrage
description: Explorer, creuser et approfondir une demande liée à un projet web (site, application web, outil interne, refonte, nouvelle fonctionnalité, ou page HTML existante — souvent créée avec une IA — à transformer en Artifact pour gagner en possibilités) par une interview structurée en français, avant toute planification ou implémentation. Pensé pour des personnes sans background de développeur, dont on ignore le rôle exact (chef de projet, métier, client, direction, communication, soignant, administratif…) : langage courant, aucun jargon, aucune question technique. Couvre le besoin réel, les utilisateurs et leurs parcours, le contenu, les cas limites, les contraintes pratiques, la sécurité et les données personnelles (toujours au moins posées, jamais passées sous silence). Se termine par un Artifact (page claude.ai privée) qui présente la compréhension partagée (« Shared Understanding »), avec une section Sécurité obligatoire, à relire et corriger point par point. Ne génère aucun SPEC.md ni fichier dans le projet. À invoquer explicitement avec /cadrage quand la demande est encore floue ou incomplète (ne se déclenche pas automatiquement, pour ne pas entrer en concurrence avec le skill interview). Ne pas utiliser pour une demande petite et bien spécifiée (coquille, changement d'une ligne, « fais juste X »). Fonctionne uniquement en français : questions, synthèse et Artifact sont toujours rédigés en français.
---

# Cadrage

Avant de concevoir ou d'implémenter quoi que ce soit, mène une interview jusqu'à ce que toi et
l'utilisateur partagiez une compréhension réelle de ce qui doit être construit, pour qui, et
pourquoi. Ne la réduis pas à un tour de questions suivi d'un plan : c'est une conversation qui
continue tant qu'il reste des zones d'ombre, pas un formulaire à remplir une fois.

## Langue

Tout est en français, sans exception : questions, options proposées, synthèse « Shared
Understanding » et Artifact. Si l'utilisateur écrit dans une autre langue, continue en français.
Parle en langage courant : l'interlocuteur n'a pas de background de développeur (voir ci-dessous).

## Public : non-développeurs, rôle inconnu

Ce skill s'adresse à des personnes qui **ne connaissent pas le développement web**. Tu ne sais pas
quel est leur rôle exact (chef de projet, métier, client, direction, communication, soignant,
administratif, quelqu'un qui découvre le sujet…) : ne le présume pas, et ne commence pas par un
interrogatoire sur leur poste. En revanche, pars toujours du principe qu'elles ne maîtrisent ni le
vocabulaire ni les contraintes techniques.

- **Langage courant, zéro jargon.** N'emploie pas de termes comme API, framework, base de
  données, front/back, déploiement, CMS, endpoint, cache, SSL… Si un terme technique est
  indispensable, remplace-le par ce qu'il signifie concrètement (« l'endroit où sont stockées les
  informations » plutôt que « base de données ») ou explique-le en une phrase la première fois.
  Illustre par des exemples tirés de leur quotidien plutôt que par des analogies techniques.
- **Ne pose aucune question technique.** Pas de « quelle stack ? », « quel hébergeur ? », « REST
  ou GraphQL ? ». Ce sont des décisions que tu prends toi-même, ou que tu recommandes (voir
  « Choix techniques » plus bas). Interroge ce que la personne sait et décide vraiment : le besoin,
  les usages, le contenu, les contraintes, les enjeux.
- **Pose la question du rôle sur le projet, pas sur la personne** : « Qui utilisera ça au
  quotidien ? », « Qui décide de ce qui est publié ? », « Qui s'en occupera une fois en ligne ? ».
  Ces réponses éclairent le projet sans mettre la personne en position d'avoir à se justifier.
- **Traduis tout enjeu technique en conséquence concrète** : coût, délai, autonomie (« pourrez-vous
  modifier vous-même ce texte ? »), dépendance à un prestataire, ce que verra l'utilisateur final.
  Les termes techniques ont leur place dans ton raisonnement et, au besoin, entre parenthèses pour
  un lecteur qui reprendrait le projet, jamais comme base de la question.
- **Offre toujours une porte de sortie** : « Je ne sais pas », « À vous de décider » sont des
  réponses valables et attendues. Dans ce cas, propose un défaut raisonnable, explique-le en une
  phrase, et consigne-le comme **hypothèse à confirmer** plutôt que comme décision validée.
- **Ne condescends pas** : l'interlocuteur n'est pas technique, pas peu intelligent. Il connaît son
  métier mieux que toi ; sois précis, concret et respectueux de son expertise.
- **Donne des exemples** quand une question est abstraite (« par exemple, un visiteur qui cherche un
  horaire depuis son téléphone… ») : une personne qui n'a jamais spécifié un outil web répond
  beaucoup mieux à un cas concret qu'à une notion générale.

## Reconnaître la situation

Le skill sert à plusieurs profils et plusieurs types de projet. Avant de poser tes premières
questions, identifie dans quel cas tu es, à partir du message de la personne et de ce que tu vois
dans le dossier de travail (fichiers HTML, site en ligne mentionné…). Ne demande pas ce que tu peux
déduire ; si le doute persiste, pose une seule question simple en langage courant.

- **A. Idée ou projet nouveau** — rien n'existe encore. Suis le déroulé général ci-dessous.
- **B. Site ou outil existant à faire évoluer ou refondre** — il y a de l'existant à comprendre
  avant de questionner (voir « Avant de poser des questions »).
- **C. Page ou projet HTML existant à transformer en Artifact** — typiquement un projet créé avec une
  IA (« vibecodé ») que la personne veut faire passer en Artifact pour disposer de plus de
  possibilités. Applique en plus la section « Cas : passer un HTML existant en Artifact ».

Les cas peuvent se combiner. L'Artifact est parfois la destination aussi pour un projet nouveau
(cas A) : dans ce cas, applique aussi les contraintes décrites dans la section du cas C. Si la
destination est un Artifact, ne pose pas de questions sur le nom de domaine, l'hébergement ou le
référencement sur les moteurs de recherche, qui n'ont pas de sens dans ce cadre.

## Avant de poser des questions

Si l'échange a lieu dans un projet existant, prends quelques minutes pour t'ancrer dans la réalité :
lis le code, la config, la documentation (CLAUDE.md ou équivalent, fichiers concernés, conventions
en place). Des questions fondées sur ce qui existe (« Je vois que X fait déjà Y — la nouvelle
fonctionnalité doit-elle le réutiliser, ou est-ce volontairement séparé ? ») sont bien plus utiles
que des questions génériques. S'il n'y a ni code ni site existant, passe directement aux questions
sans inventer de contexte. Si un site en ligne existe, regarde-le avant de questionner quand tu as
une URL.

## Ce qu'il faut couvrir

Ce sont des angles de vue pour vérifier que rien d'important n'est oublié, pas une liste à réciter
dans l'ordre. Commence toujours par le besoin, pas par la technique.

- **Le besoin réel** — quel problème concret, aujourd'hui, pour qui ? Qu'est-ce qui ne marche pas
  ou fait perdre du temps ? Pourquoi maintenant ? Qu'est-ce qui indiquerait que c'est réussi
  (critère de succès concret) ? Creuse derrière la solution demandée : la personne décrit souvent
  une solution (« il me faut un portail ») alors que le besoin sous-jacent peut être différent.
- **Les utilisateurs et les acteurs** — qui utilise, qui administre, qui alimente le contenu, qui
  valide, qui maintient ? Quels niveaux d'aisance numérique, quels appareils (mobile, poste fixe,
  tablette), quel contexte d'usage (au bureau, en déplacement, sous contrainte de temps) ?
- **Les parcours et cas d'usage** — le chemin type d'un utilisateur du début à la fin, et les
  scénarios non mentionnés que le projet devra pourtant gérer (première visite, retour, erreur,
  abandon, accès par quelqu'un qui n'a pas de compte…).
- **Contenu et données** — de quoi est faite la matière du projet (textes, documents, images,
  données saisies, imports) ? D'où vient-elle, qui la modifie, à quelle fréquence, dans quel format ?
  Faut-il un espace d'administration ou un outil d'édition ?
- **L'existant et la migration** — site ou outil actuel, données à reprendre, URLs à conserver
  (redirections, référencement), habitudes des utilisateurs à ne pas casser, outils tiers déjà en
  place.
- **Problèmes à anticiper** — pannes, forte affluence, saisies inhabituelles, ce qui se passe quand
  un service externe ne répond plus, quand un utilisateur fait une fausse manipulation, quand du
  contenu est supprimé par erreur. Pose-les sous forme de situations concrètes (« Que doit voir le
  visiteur si le site est indisponible ? »).
- **Choix techniques et outils** — ne les soumets pas à l'utilisateur en termes techniques. Quand
  une vraie décision existe (outil clé en main ou développement sur mesure, hébergement, service
  tiers…), ne la pose que sous l'angle de ses conséquences pour la personne : coût, délai,
  autonomie pour modifier le contenu, dépendance à un prestataire, durée de vie. Fais le choix
  technique toi-même, avec la recommandation que tu défendrais, et consigne-le dans l'Artifact
  comme une recommandation de ta part (pas comme une décision validée par l'utilisateur), avec
  l'alternative écartée et pourquoi. Si l'utilisateur a déjà imposé un outil, une plateforme ou un
  prestataire, note-le comme contrainte.
- **Contraintes pratiques** — budget, délais, échéance imposée, qui sera disponible pour tester et
  valider, règles imposées par l'organisation (service informatique, charte, outils obligatoires),
  nom de domaine ou adresse existants, qui s'occupera du projet une fois en ligne. Pose ces
  questions sans supposer que la personne sait ce qu'est un hébergement ou un environnement de test :
  demande ce qui compte pour elle (« Y a-t-il une date à ne pas dépasser ? », « Votre service
  informatique impose-t-il des outils ? »).
- **Qualité d'usage** — usage sur téléphone ou tablette, accessibilité pour les personnes en
  situation de handicap (et obligation légale éventuelle, par exemple RGAA pour une structure
  publique), rapidité ressentie, langues, visibilité sur les moteurs de recherche si le public doit
  trouver le site, e-mails envoyés automatiquement, suivi de la fréquentation. Formule en
  conséquences : « Vos utilisateurs sont-ils souvent sur téléphone ? », « Faut-il que le site
  apparaisse quand on le cherche sur Google ? ».
- **Sécurité** — obligatoire, jamais facultative : établis toujours au minimum si le projet touche à
  des comptes ou des accès réservés (qui a le droit de voir ou de modifier quoi), à des données
  personnelles ou sensibles (dont données de santé, RGPD), à des mots de passe ou identifiants
  partagés avec d'autres services, à des informations saisies par des inconnus (formulaires,
  envois de documents, services externes), à du paiement, ou s'il a des exigences de conformité. Si la réponse est
  réellement « rien de tout ça » pour un petit projet local ou à faible risque, fais-le énoncer
  explicitement : un « non » documenté est acceptable, une question non posée ne l'est pas. Si
  quelque chose s'applique, creuse comme pour n'importe quelle vraie décision, en langage courant :
  qui doit avoir accès à quoi, quelles informations sont confidentielles et pour qui, combien de
  temps on les garde, ce qui se passerait si quelqu'un d'inattendu les voyait ou les modifiait, et
  quelles règles s'imposent (RGPD, secret professionnel…). Les détails de mise en œuvre (stockage
  des secrets, traitement des entrées malveillantes) sont de ta responsabilité : tu les consignes
  comme exigences dans l'Artifact sans les soumettre à la personne. La profondeur dépend de ce que le projet touche vraiment : pas de
  modélisation de menaces pour une page vitrine statique, mais oui pour tout ce qui gère des comptes,
  des paiements, des données personnelles ou des entrées externes.
- **Approfondissement sur tout le plan** — ne t'arrête pas à la première couche : chaque réponse
  révèle généralement une autre décision en dessous (c'est normal, continue, ce n'est pas du
  dérapage de périmètre).

## Cas : passer un HTML existant en Artifact

La personne a déjà une page ou un petit projet HTML qui fonctionne, et veut en faire un Artifact
pour avoir plus de libertés. Elle n'a probablement pas écrit le code elle-même et ne sait pas
précisément ce qu'il contient : ne lui demande pas de le décrire, **lis-le toi-même**.

1. **Lis tout le projet** : fichiers HTML, CSS, JavaScript, images, polices, bibliothèques
   chargées depuis Internet, appels à des services externes, données enregistrées dans le
   navigateur, formulaires, mots de passe ou clés d'accès éventuellement écrits dans le code.
2. **Décris-lui ce que fait sa page, en langage courant**, puis fais confirmer : « Votre page
   affiche X, permet de faire Y, garde Z en mémoire sur l'ordinateur de la personne, contacte tel
   service. Est-ce bien ce que vous attendez d'elle ? » Les surprises (fonctions que la personne ne
   connaissait pas, éléments qui ne marchent pas) sont des matières précieuses pour l'interview.
3. **Creuse « plus de libertés »** : c'est la question centrale, la personne ne sait souvent pas
   la formuler. Demande ce qu'elle ne peut pas faire aujourd'hui, avec des exemples concrets à
   choisir : partager la page avec d'autres par un simple lien, que les informations saisies
   soient conservées d'une visite à l'autre ou partagées entre plusieurs personnes, afficher des
   données mises à jour, pouvoir poser une question à Claude depuis la page, conserver des
   fichiers ajoutés par les visiteurs, savoir qui consulte la page, remettre un fichier à
   télécharger. Une réponse vague (« plus de liberté ») ne suffit pas : continue jusqu'à des
   usages précis.
4. **Vérifie ce qui est réellement possible** : charge le skill `artifact-capabilities` pour
   connaître les possibilités actuellement disponibles pour cet utilisateur, plutôt que de te fier
   à ta mémoire. Rapproche chaque souhait de ce qui existe, de ce qui demande une adaptation, et de
   ce qui n'est pas possible.
5. **Explique les limites en conséquences, sans jargon.** Un Artifact est une page isolée : pas de
   serveur à soi, pas d'adresse personnalisée, pas de référencement sur les moteurs de recherche,
   taille limitée, bibliothèques externes restreintes à certaines sources. Dis ce que cela
   change pour elle (« la page ne pourra pas être retrouvée sur Google ; elle sera accessible
   par lien »). Ce que la page enregistre dans le navigateur n'est conservé que sur l'appareil de
   chaque visiteur, et ne passe pas d'un visiteur à l'autre.
6. **Sécurité propre à la migration** — toujours à vérifier, même si la personne ne l'a pas
   évoquée : une clé d'accès, un mot de passe ou un jeton écrit dans le code est lisible par toute
   personne qui ouvre la page. Si tu en trouves, ne les affiche pas dans la conversation ni dans
   l'Artifact ; dis simplement qu'il y en a, où, et propose de les retirer ou de passer par une
   autre méthode. Vérifie aussi les services externes contactés (que reçoivent-ils ?), la présence
   de données personnelles ou confidentielles dans la page, et **qui pourra voir la page** (privée
   par défaut, visible par ceux à qui la personne donne le lien).
7. **Fais trier** : pour chaque élément de la page existante, ce qui est conservé tel quel, ce qui
   est adapté, ce qui est abandonné ou impossible, et ce qui est nouveau grâce à l'Artifact.

Une fois la compréhension validée, tu peux proposer de faire la conversion, mais ne l'engage pas
sans l'accord explicite de la personne.

## Comment poser les questions

- Privilégie `AskUserQuestion` pour les points de décision concrets : c'est plus rapide pour
  l'utilisateur qu'un paragraphe, et ça t'oblige à nommer les options. Regroupe les questions
  liées (jusqu'à 4 par appel) au lieu d'en poser une à la fois.
- Quand tu as une vraie recommandation, mets-la en première option avec « (Recommandé) » : ne
  présente pas de menu faussement neutre si tu connais le meilleur défaut. Rédige chaque option
  avec sa conséquence concrète, pas seulement son nom.
- Pour ce qui ne se réduit pas à un menu (chiffres, descriptions, « que se passe-t-il d'autre »),
  pose une question ouverte en prose.
- Avance par rounds. Les réponses du premier tour ouvrent presque toujours un deuxième tour :
  c'est l'interview qui fonctionne, pas le signe que tu as mal posé les premières questions.
  Continue jusqu'à épuisement réel des questions ouvertes, pas jusqu'à en avoir posé « assez ».
- Si la demande initiale est très vague, commence par un tour large et ouvert sur le besoin et les
  utilisateurs avant toute question de détail ou de technique.
- Reformule de temps en temps ce que tu as compris en une ou deux phrases avant de passer au sujet
  suivant : c'est le moyen le plus fiable de repérer un malentendu tôt.
- Ne demande pas ce que tu peux vérifier toi-même (code existant, contenu des fichiers, site en
  ligne, valeurs par défaut évidentes) : réserve les questions à ce que seul l'utilisateur peut
  trancher.

## Quand s'arrêter

Arrête-toi quand tu pourrais expliquer la demande à l'utilisateur, sans qu'il ait à te corriger :
l'objectif, les utilisateurs, les contraintes, les cas limites et la forme d'une solution. À ce
moment, ne lance pas un nouveau tour de questions : construis l'Artifact de compréhension
partagée (voir ci-dessous) et annonce-le dans le chat.

Après la publication, **STOP**. Ne commence ni planification ni implémentation, n'écris aucun
fichier dans le projet. Demande explicitement à l'utilisateur de relire l'Artifact et de confirmer
ou d'indiquer ce qu'il faut corriger. C'est un garde-fou strict, pas une formalité : attends son
feu vert explicite (ou ses corrections, que tu intègres en republiant le même Artifact) avant
toute suite. La suite (plan, implémentation, conversion) est à son initiative : tu peux la proposer, jamais l'engager sans accord explicite.

## L'Artifact « Shared Understanding »

Le livrable de l'interview est un Artifact HTML en français, privé par défaut, qui contient la
compréhension partagée. Il ne remplace pas la conversation : il rend la synthèse lisible, relisible
et facile à confirmer ou corriger point par point, y compris par quelqu'un qui n'était pas dans
l'échange. Ne crée ni `SPEC.md`, ni `CLAUDE.md`, ni aucun autre fichier dans le projet.

**Publication.** Utilise l'outil `Artifact` : fais d'abord un appel `action: "quickstart"` avec
`intent: "other"` pour obtenir les consignes de design, charge le skill `artifact-design` avant
d'écrire la page, écris le fichier HTML dans le répertoire scratchpad de la session (pas dans le
projet), puis publie-le avec un `title` court en français, de deux à quatre mots (par exemple
« Cadrage — nom du projet »), une `description` d'une phrase et une `icon` générique. Après
publication, donne le lien à l'utilisateur.

**Contenu.** Écris uniquement des éléments réellement discutés dans la conversation, condensés en
décisions plutôt qu'en questions-réponses. Numérote chaque point pour que l'utilisateur puisse
écrire « le 4 est faux ». Sections, dans cet ordre, en omettant celles qui ne concernent
vraiment pas le projet :

1. **Résumé** — le besoin et l'objectif en quelques phrases, compréhensibles sans contexte.
2. **Utilisateurs et acteurs** — qui utilise, administre, alimente, valide, maintient.
3. **Parcours et cas d'usage clés** — y compris les cas non mentionnés au départ.
4. **Contenu et données** — nature, origine, qui les modifie, fréquence.
5. **Existant et migration** — ce qui est repris, conservé ou remplacé.
6. **Contraintes** — budget, délais, hébergement, équipe, environnements, qualité d'usage
   (mobile, accessibilité, navigateurs, langues, SEO).
7. **Cas limites et risques anticipés** — ce qui peut mal tourner et comment on le traite.
8. **Approche retenue** — les choix fonctionnels validés, les alternatives écartées et pourquoi
   quand ce n'est pas évident. Y ajouter une sous-partie « Recommandation technique (de Claude) »
   pour les choix que l'utilisateur n'a pas tranchés lui-même, clairement distinguée des décisions
   validées, avec une phrase en langage courant sur ce que cela change pour lui.
9. **Sécurité** — section toujours présente, même quand la réponse est « non applicable » :
   indique le niveau de risque et pourquoi (par exemple « pas d'authentification, pas de données
   personnelles, site statique — risque faible »). Quand des sujets de sécurité s'appliquent,
   consigne les décisions réelles : ce qui est de confiance ou non, gestion des secrets,
   traitement des entrées non fiables, données sensibles et leur protection, conformité (RGPD,
   etc.).
10. **Hypothèses à confirmer** — ce que la personne ne savait pas ou a délégué, et le défaut que
    tu as proposé. Sépare-les visuellement des décisions validées pour qu'on ne les prenne pas pour
    acquises.
11. **Questions ouvertes ou reportées** — ce qui reste réellement sans réponse. Ne les maquille
    pas en décisions.

**Cas de la migration.** Si tu es dans le cas C, adapte la liste : remplace « Contenu et données »
et « Existant et migration » par trois sections lisibles en clair : **Ce que fait la page
aujourd'hui** (ce que tu as lu dans le code, décrit simplement), **Ce que vous voulez pouvoir faire
de plus** (les usages précis, un par ligne, avec la possibilité de l'Artifact qui y répond ou la
mention « impossible »), et **Conservé, adapté, abandonné** (le tri validé élément par élément).
Garde « Sécurité » en incluant les clés d'accès trouvées (sans les recopier) et la visibilité
voulue de la page, et ajoute les limites d'un Artifact qui concernent cette personne.

**Langage.** L'Artifact est lu par la personne interrogée, qui n'est pas développeuse : même règle
que dans la conversation, langage courant et pas de jargon. Les termes techniques indispensables
(pour un futur développeur qui reprendrait le projet) vont dans la sous-partie « Recommandation
technique », avec leur explication en clair à côté.

**Design.** Page sobre et lisible, mise en page qui tient sur mobile. Distingue clairement trois
statuts : décision validée, hypothèse à confirmer, question ouverte (badge ou couleur, pas
seulement le texte). N'ajoute ni schéma ni élément décoratif qui n'apporte pas d'information.

**Corrections.** Quand l'utilisateur corrige ou complète, mets à jour le même fichier et republie
avec le même chemin pour conserver l'URL. Mentionne dans le chat ce qui a changé. Si l'échange
ouvre de nouvelles questions de fond, reprends l'interview avant de republier.
