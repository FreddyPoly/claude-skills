---
name: worktree
description: >
  Crée un worktree git prêt à l'emploi pour une nouvelle évolution, dans un dossier frère du repo
  (../<repo>-<branche avec / → ->), basé par défaut sur origin/staging : nouvelle branche sans
  upstream, copie des .env* ignorés depuis le worktree principal, installation des dépendances
  selon le lockfile, ouverture d'une nouvelle fenêtre VS Code. Gère aussi le nettoyage
  (/worktree remove <branche>) : retrait du worktree, prune, suppression optionnelle de la branche
  locale. Utiliser quand l'utilisateur veut travailler sur une autre branche en parallèle,
  « ouvrir un worktree », « créer une branche dans un nouveau dossier », ou invoque /worktree.
argument-hint: "<branche> [base] | remove <branche>"
---

# Worktree

Deux modes, selon les arguments :

- `/worktree <branche> [base]` → **création**
- `/worktree remove <branche>` → **nettoyage**

Si aucun argument : demander le nom de branche (mode création).

## Règles communes

- Ne jamais committer, ni pousser, ni supprimer une branche distante.
- Toute suppression (worktree, branche) se fait après confirmation explicite de l'utilisateur.
- Toujours mettre les noms de branche et chemins entre guillemets dans les commandes shell.
- Valider le nom de branche avant toute action : `git check-ref-format --branch "$BRANCH"`. S'il est
  invalide, s'arrêter et demander un autre nom.
- **Worktree principal** : `MAIN=$(git worktree list --porcelain | head -1 | sed 's/^worktree //')`.
  C'est la référence pour le nom du repo et la source des `.env`, même si la commande est lancée
  depuis un autre worktree.
- Répondre dans la langue de l'utilisateur.
- **Grille `tour`** (session tmux des worktrees, commande `tour` dans `~/.local/bin`) : le skill
  n'y touche qu'à la création, par `tour add` puis `tour update` lancés dans le worktree (voir
  l'étape 4 bis). Un échec (`tour` ou tmux introuvable, code ≠ 0) est **signalé** dans le compte
  rendu avec la sortie d'erreur, mais n'annule ni n'interrompt la création du worktree.

## Mode création

### 1. Calculer les paramètres

- `REPO=$(basename "$MAIN")`
- `DIR="$(dirname "$MAIN")/$REPO-${BRANCH//\//-}"`, par ex. `feat/standardize-sidebar-titles` →
  `../ASCE-feat-standardize-sidebar-titles`.
- Si `DIR` existe déjà : s'arrêter et le signaler (ne rien écraser).
- `git -C "$MAIN" fetch origin --prune`.

### 2. Déterminer la base

- Base passée en argument : l'utiliser. Si c'est un nom de branche distante qui existe
  (`origin/<base>`), préférer `origin/<base>` à la branche locale, qui peut être en retard. Vérifier
  avec `git rev-parse --verify --quiet`.
- Sinon, si `origin/staging` existe : base = `origin/staging`.
- Sinon : **demander** la base à l'utilisateur (avec `AskUserQuestion`, en proposant la branche
  par défaut du remote et quelques branches récentes).

### 3. Branche déjà existante ?

Vérifier `refs/heads/$BRANCH` et `refs/remotes/origin/$BRANCH` (`git show-ref --verify --quiet`).

- **N'existe nulle part** : cas normal, passer à l'étape 4.
- **Existe déjà** : le signaler (et ignorer la base demandée), puis proposer d'ouvrir un worktree
  sur cette branche existante plutôt que d'en créer une nouvelle. Si l'utilisateur refuse, s'arrêter.
  - Si la branche locale est déjà extraite dans un worktree (visible dans
    `git worktree list --porcelain`), ne rien créer : donner le chemin et proposer
    `code -n "<chemin>"`.
  - Branche locale : `git worktree add "$DIR" "$BRANCH"`.
  - Branche seulement sur origin : `git worktree add --track -b "$BRANCH" "$DIR" "origin/$BRANCH"`
    (ici on garde le suivi, c'est la bonne branche distante).

### 4. Créer le worktree (nouvelle branche)

```bash
git -C "$MAIN" worktree add "$DIR" -b "$BRANCH" "$BASE"
git -C "$DIR" branch --unset-upstream 2>/dev/null || true
```

Retirer l'upstream évite qu'un futur `git push` vise la branche de base. Le premier push se fera
avec `git push -u origin "$BRANCH"`.

### 4 bis. Ajouter le worktree à la grille `tour`

Une fois le worktree créé (nouvelle branche ou branche existante de l'étape 3), l'ajouter à la
grille (voir « Règles communes ») : `tour add` inscrit le repo dans `~/.claude/tour.conf` s'il n'y
est pas (sans doublon), puis `tour update` crée la session en arrière-plan ou y ajoute les
worktrees apparus, sans s'y rattacher. Le chemin est celui que donne
`git worktree list --porcelain` pour la branche :

```bash
WT=$(git -C "$MAIN" worktree list --porcelain | awk -v ref="branch refs/heads/$BRANCH" \
  '/^worktree /{p=substr($0,10)} $0==ref{print p}')
if [ -z "$WT" ]; then
  echo "worktree introuvable dans git worktree list : grille non mise à jour" >&2
elif ! command -v tour >/dev/null 2>&1; then
  echo "tour introuvable dans le PATH : grille non mise à jour" >&2
else
  (cd "$WT" && tour add && tour update)
fi
```

Continuer quel que soit le résultat ; le reporter dans le compte rendu (sortie de `tour update`,
« grille à jour, N worktree(s) ajouté(s) », ou son message d'erreur).

### 5. Copier les .env

Copier depuis `MAIN` les fichiers `.env*` **non suivis et ignorés** par git, en gardant le même
chemin relatif :

```bash
cd "$MAIN"
find . \( -name node_modules -o -name .git -o -name .next \) -prune -o -name '.env*' -type f -print |
while read -r f; do
  git ls-files --error-unmatch "$f" >/dev/null 2>&1 && continue   # suivi (ex. .env.example) → ignorer
  git check-ignore -q "$f" || continue                             # pas ignoré → ignorer
  [ -e "$DIR/$f" ] && continue                                     # ne jamais écraser
  mkdir -p "$DIR/$(dirname "$f")" && cp -p "$f" "$DIR/$f" && echo "$f"
done
```

Lister les fichiers copiés dans le compte rendu (chemins seulement, jamais leur contenu).

### 6. Installer les dépendances

Selon le lockfile à la racine de `DIR` :

| Lockfile | Commande |
|---|---|
| `pnpm-lock.yaml` | `pnpm install` |
| `package-lock.json` | `npm install` |
| `yarn.lock` | `yarn install` |
| `bun.lock` / `bun.lockb` | `bun install` |
| aucun | ne rien faire |

Lancer dans `DIR` avec un timeout long (10 min). En cas d'échec : le signaler avec la fin de la
sortie, **sans** supprimer le worktree.

### 7. Ouvrir VS Code

`code -n "$DIR"`. Si la commande `code` n'est pas disponible, donner le chemin à ouvrir à la main.

### 8. Compte rendu

Court : dossier, branche, base (avec son commit court), fichiers `.env` copiés, résultat de
l'installation, ajout à la grille `tour` (ou son échec), commande du premier push. Si
`git worktree list` montre des worktrees « prunable », le signaler et suggérer `/worktree remove`
ou `git worktree prune`.

## Mode nettoyage

### 1. Trouver le worktree

Dans `git worktree list --porcelain`, trouver l'entrée dont `branch refs/heads/$BRANCH`.

- Introuvable : le signaler. S'il y a des entrées « prunable », proposer `git worktree prune`.
  Si la branche locale existe sans worktree, passer directement à l'étape 4.
- Refuser si c'est le worktree principal, ou le dossier de travail de la session en cours.

### 2. Vérifications

- `git -C "$WT" status --porcelain` non vide → **refuser** et lister les modifications.
  Les `.env` copiés ne gênent pas : ils sont ignorés par git.
- Commits non poussés (`git -C "$WT" log --oneline "origin/$BRANCH..$BRANCH"`, ou tous les commits
  depuis la base si la branche n'a jamais été poussée) → les signaler avant de demander
  confirmation.

### 3. Retirer le worktree

Après confirmation :

```bash
git -C "$MAIN" worktree remove "$WT"   # sans --force
git -C "$MAIN" worktree prune
```

Ne pas toucher à la grille `tour` : le volet du worktree retiré y reste et se ferme à la main.

### 4. Branche locale

Vérifier si elle est mergée dans `origin/staging` ou dans la branche par défaut du remote
(`git merge-base --is-ancestor "$BRANCH" <cible>`).

- **Mergée** : proposer de la supprimer. Utiliser `git branch -D "$BRANCH"`, car `-d` compare
  avec la branche courante du worktree principal et refuse à tort quand celle-ci ne contient pas
  la cible. La vérification `merge-base` ci-dessus sert de garde-fou.
- **Non mergée** : le dire clairement (en précisant qu'un squash-merge apparaît comme « non
  mergé »), puis demander. Supprimer avec `git branch -D` seulement après un oui explicite.
- Ne jamais supprimer la branche distante.
