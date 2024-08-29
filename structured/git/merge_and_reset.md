Notes prises en août 2023, issues de la lecture de [cette série d'articles](https://www.freecodecamp.org/news/the-definitive-guide-to-git-merge/).

* [Merge](#merge)
* [Reset](#reset)
   * [reset soft](#reset-soft)
   * [reset mixed](#reset-mixed)
   * [reset hard](#reset-hard)
   * [autres notes](#autres-notes)
* [Concepts de base](#concepts-de-base)

Les articles sont tous assez verbeux et longs, mais en dehors de ça pas mal fichus, et j'en profite pour affûter des trucs que j'avais mal compris jusqu'ici :

- fonctionnement de l'algo de merge
- fonctionnement du `HEAD` et du staging-area
- fonctionnement des 3 resets (et p.ex. conséquences sur les commandes permettant de splitter un commit)
- quelques tricks utiles (`git diff --ours`, son alias `git lol`, etc.)

# Merge

`3 way merge` = appelé comme ça car on compare non pas deux états (notre branche et la branche sur laquelle on merge), mais plutôt TROIS états :

- le **merge base** = le commit à partir duquel les branches on divergé
- le diff entre le merge-base et la branche 1
- le diff entre le merge-base et la branche 2

Il y a plusieurs situations qu'on traite différemment pour produire l'état final mergé :

- trivial = 0 désaccords = si les trois états sont d'accord sur une ligne, on la garde telle quelle dans l'état final
- 1 désaccord = si l'une des branches X est d'accord avec le merge base et pas l'autre branche Y, on garde la ligne de Y ; la justification est que comme le merge base est le plus ancien des 3 états, ça veut dire que la modif sur Y est plus récente que X, et on garde donc la ligne de Y
- 1 désaccord bis = si les deux branches sont d'accord sur une ligne, mais pas d'accord avec le merge base, on garde l'état des branches ; la justification est la même = X et Y sont alors plus récents que merge-base
- 2 désaccords = si sur une ligne, les deux branches diffèrent du merge base, et d'une façon différente sur chaque branche, alors il y a conflit, à résoudre manuellement

NDM 1 = du coup, cette compréhension va m'aider à identifier les provenances des changements dans les résolutions des conflits (alors que jusqu'ici, j'essayais d'inférer les provenances à partir du diff).

Notamment, cette identification dépend du sens dans lequel on a mergé : `git merge A` appelé depuis `B` ou bien le contraire :

- dans les marqueurs de conflit, git indique les deux diffs suivants :
    -  de merge-base vers `A`
    -  de merge-base vers `B`
- `HEAD` identifie la branche sur laquelle on était au moment où on a appelé la commande de merge

NDM 2 = on voit qu'il y a des cas où le merge peut ne pas créer de conflit et pourtant ne pas avoir fait exactement ce que je veux, p.ex. si une branche crée et utilise une fonction `pouet()`, mais qu'une résolution de merge supprime son utilisation : alors il aurait fallu également supprimer la fonction, ce que l'algo de merge ne fera pas.

En cours de résolution de conflit, on peut regarder l'état d'un fichier en conflit sur chacune des versions :

> A nice trick that allows you to see the content quickly without providing the blob's SHA-1 value, is by using git show, like so:

```
git show :<STAGE>:the_conflicted_file.txt
```

Résumé de pourquoi il y a conflit :

> For Git, Paul and John made different changes to the same line, for a few lines. John changed it to one thing, and Paul changed it to another thing. Git cannot decide which one is correct.

Ah intéressant : après avoir modifié manuellement un fichier pour résoudre un conflit, on peut regarder le diff par rapport à notre version initiale :

> To compare the result file to what you had in the branch prior to the merge, you can run:

```
git diff --ours
```

> Similarly, if you wish to see how the result of the merge differs from the branch you merged into our branch, you can run:

```
git diff -–theirs
```

> You can even see how the result is different from both sides using:

```
git diff -–base
```

Son alias pour avoir un historique CLI peut aussi être utile :

https://gist.github.com/outro56/31e588e5622dcf7931de0aee49baa7b2


```
# visualisation de l'historique dans le terminal :
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

# alias lol sur la commande précédente :
git config --global alias.lol "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

# Reset

**À RETENIR** : si on laisse de côté les états branchless, un `git reset OLDER-COMMIT` a donc pour effet de faire pointer la branche courante sur un commit plus ancien (et `--mixed` / `--soft` permettent de conserver les modifs abandonnées dans le working-tree et/ou la staging-area).

----

Comme l'article sur les merge était bien, je digge [un autre article sur reset](https://medium.com/@Omer_Rosenbaum/git-undo-how-to-rewrite-git-history-with-confidence-d4452e2969c2).

> Usually, when we work on our source code we work from a working dir. A working dir(ectrory) (or working tree) is any directory on our file system which has a repository associated with it. It contains the folders and files of our project, and also a directory called .git. I described the contents of the .git folder in more detail in a previous post.
>
> After you make some changes, you may want to record them in your repository. A repository (in short: repo) is a collection of commits, each of which is an archive of what the project’s working tree looked like at a past date, whether on your machine or someone else’s. A repository also includes things other than our code files, such as HEAD, branches etc.
>
> In between, have the index or the staging area, these two terms are interchangeable. When we checkout a branch, Git populates the index with all the file contents that were last checked out into our working directory and what they looked like when they were originally checked out. When we use git commit, the commit is created based on the state of the index.

^ le repo est une collection de snapshots d'un filesystem.

Le staging area est le "vrai" snapshot, actuellement checkouté dans le working-dir.

Le working dir est dérivé du staging area :

- dans un sens (quand on fait un checkout, on remplit le staging area avec le contenu du commit checkouté, puis on remplit le working dir pour qu'il soit une image fidèle de la staging area)
- et dans l'autre (quand on a modifié l'état checkouté et qu'on veut pérenniser nos modifs, on place les modifs dans la staging area, puis on transforme l'état du staging area en un nouveau commit = un nouveau snapshot de filesystem)

> You created a new commit object, which includes a pointer to a tree describing the entire working tree

Un commit n'est PAS un diff par rapport au parent : c'est un snapshot complet d'un tree complet.

> In addition to a pointer to the tree, the commit object includes metadata, such as timestamp and author’s information.

Le sha d'un commit hashe à la fois le tree et les métadonnées (donc si on change p.ex. la date du commit , le hash va changer même si le contenu du tree est absolument le même)

> a branch in Git is just a named reference to a commit. (...) Git has another pointer called HEAD, which points (usually) to a branch, which then points to a commit. By the way, under the hood, HEAD is just a file. It includes the name of the branch with some prefix.

^ explications sur les branches et HEAD

`git commit` fait deux choses en même temps : il crée un commit + il bouge le pointeur de la branche actuelle vers ce nouveau commit.

## reset soft

```
git reset --soft HEAD~1
```

^ se contente de déplacer la branche pointée par `HEAD` (ici, en le reculant d'un commit) sans toucher au staging area ni au working dir. Du coup, les modifs du commit qu'on quitte apparaissent dans le staging area comme "prêtes à être commitées" (avec le working-dir dans le même état → tout se passe comme si on venait de faire un `git add` général).

NDM : j'en déduis que pour un état du staging area donné, ce qui définit à quel point ça apparaît comme "il y a un diff" ou bien "l'état du staging area représente l'état actuel, sans diff" est **la comparaison au HEAD**. Dit autrement : **l'état actuel du repo est défini par HEAD**. Ici, HEAD pointe vers le commit juste avant le staging-area, donc git interprète l'état du staging area comme "on a des modifs prêtes à être commitées".

La suite immédiate de l'article confirme que si je recommite ces modifs une seconde fois, j'ai deux commits ayant le même working tree, mais des metadata différentes ; le HEAD pointe vers le nouveau commit (qui est branchless).

Autre info importante : `git log` travaille à partir de `HEAD`.

**IMPORTANT** = NDM : `HEAD` semble donc très important et représente "l'état actuel du repo" (en pointant sur une branche, donc un commit, donc sur un filesystem tree). De façon contre intuitive, ce ne sont ni le staging area, ni le working dir qui représentent l'état actuel du repo , c'est HEAD. Et le diff du staging area (i.e. ce qui sera commité au prochain commit) est calculé dynamiquement en comparant le staging area à HEAD.

Ma compréhension, c'est que quand on fait `git reset --soft HEAD~1`, chacune des trois zones de git pointe vers un filesystem (= un tree) complet, et les commandes de diff calculent le diff dynamiquement en comparant les tree dynamiquement :

- le tree actuel du repo est le tree pointé par HEAD
- le staging area représente un autre tree, possiblement pas encore commité
- le working dir représente encore un autre tree

Comme les commandes de diff sont résolues dynamiquement (plutôt que, par exemple, un commit contienne l'info de diff, comme avec `hg`), ça explique le comportement d'un `git reset --soft HEAD~1 + git diff --staged + git commit` :

- le tree du repo est positionné sur le commit parent
- le staging area (et le working dir) restent positionnés sur le commit enfant
- conséquence = un `git diff --staged` résoudra dynamiquement le diff, en comparant la staging area et le HEAD, du coup le diff qui apparaîtra sera celui du dernier commit, et le `git commit` aura pour effet de "recommiter" le commit. CQFD

## reset mixed

`git reset --mixed` = idem, mais on ne se contente pas de déplacer la branche pointée par HEAD : on positionne aussi le staging-area sur le même commit que le HEAD, sans toucher au working-dir.

**Important** : `--mixed` est le mode par défaut ; lorsqu'on fait un `git reset TARGET` sans préciser le mode, c'est équivalent à `git reset --mixed TARGET`.

Du coup :

```
git reset --mixed HEAD~1
```

^ cette commande dit "remets le repo et le staging-area dans l'état du commit parent, mais ne touche pas au working-tree".

Dit autrement, le commit actuel (= celui avant exécution de la commande) disparaîtra de la branche (qui pointera alors vers son parent), mais ses modifs resteront visibles sous la forme de modifications locales dans le working-tree, prêtes à être add+commitées.

## reset hard

`git reset --hard` = on déplace les trois états, y compris le working-dir : la commande est **DANGEREUSE** car concrètement, l'état actuel de la branche est définitivement perdu (sauf à jouer avec le `reflog`), alors qu'avec `--soft` et `--mixed`, il reste récupérable depuis le working-tree et/ou le staging-area.

Concrètement, faire `git reset --mixed` fait comme si on n'avait pas encore staged+commit nos modifications → faire `git reset --hard` permet de tout déplacer vers un commit.

## autres notes

> There are two most important tools I want you to take from this post. The second is git reset. The first and by far more important one is to whiteboard the current state versus the state you want to be in.

Un bon conseil = représenter les trois zones git sur une feuille blanche, en deux versions : l'état actuel, et l'état souhaité. Ça permet de réfléchir à ce qu'on veut faire.

Précision : en fait, `git reset` ne déplace pas directement le HEAD, mais déplace en fait la branche pointée par HEAD = la branche actuelle (et HEAD est déplacé par effet de bord, vu qu'il pointe sur cette branche), cf. [la doc](https://git-scm.com/docs/git-reset). C'est logique, sans quoi tous les git reset conduiraient à un état branchless.

Fort de cette connaissance, et si je suppose que marquer un commit comme `edit` lors d'un `git rebase -i` permet d'appliquer tous les commits suivants dans le rebase par dessus le commit édité, alors je comprends mieux les commandes permettant de splitter un commit :

```
git rebase -i <oldsha1>
# mark the desired commit as `edit`

git reset HEAD^  # est un --mixed implicite

git add <myfiles-to-put-in-commit1>
git commit -m "First part"

git add <myfiles-to-put-in-commit2>
git commit -m "Second part"

git rebase --continue
```

La clé est dans `git reset HEAD^` (qui est un `--mixed` implicite) : il positionne la "branche" (en vrai, le rebase en cours) sur le commit parent, ce qui "supprime" le dernier commit de la "branche" et du staging area, mais conserve les modifs dans le working tree.

Un `git status` voit donc les modifs du commit comme "prêtes à être commitées" (vu qu'elles sont dans le working-tree mais pas dans l'index), ça nous permet de les `git add` puis `git commit` en deux fois.

In fine, ça remplace le commit annulé par deux commits splittés (et derrière, `git rebase --continue` applique les autres commits par dessus ce nouvel état).

(Et on peut splitter le dernier commit d'une branche tout pareil, sans avoir à faire de rebase, juste avec un reset : le rebase permet juste de faire la modif au milieu de l'historique, plutôt qu'à la toute fin).

# Concepts de base

https://medium.com/swimm/a-visualized-intro-to-git-internals-objects-and-branches-68df85864037

J'ai lu l'article après, mais c'est en fait le premier post de la série, il définit les concepts de base : bloc, tree, commit, branche, staging area, repository, etc.

> In git, the contents of files are stored in objects called blobs, binary large objects. The difference between blobs and files is that files also contain meta-data

^ un blob est le contenu d'un fichier, sans aucune metadata (de sorte, je suppose, que deux fichiers différents ayant le même contenu puisse utiliser le même blob pour stocker leur contenu)

> In git, the equivalent of a directory is a tree. A tree is basically a directory listing, referring to blobs as well as other trees. Trees are identified by their SHA-1 hashes as well.

^ un tree est l'équivalent d'un répertoire. Un tree peut donc représenter tout un filesystem.

> Now it’s time to take a snapshot of that file system — and store all the files that existed at that time, along with their contents. In git, a snapshot is a commit. A commit object includes a pointer to the main tree (the root directory), as well as other meta-data such as the committer, a commit message and the commit time.

^ un commit est un snapshot de l'état du filesystem associé à des métadonnées, et éventuellement à un (voire des) parents.

> Every commit holds the entire snapshot, not just diffs from the previous commit(s).

^ un commit est autoporteur !

Tout ce beau monde : blob, tree, commit, etc. est référençable par son hash sha.

> So this is the trick — as long as an object doesn’t change, we don’t store it again.

^ ceci permet de stocker efficacement des tétrachiées de fichiers : on ne stocke qu'une fois chaque objet donc on ne restocke que ce qui change.

> A branch is just a named reference to a commit.

^ une branche est un pointeur vers un commit, nommé du nom de la branche.

> How does git know what branch we’re currently on? It keeps a special pointer called HEAD. Usually, HEAD points to a branch, which in turns points to a commit

**IMPORTANT** : `HEAD` ne référence pas un commit, mais référence une branche = celle sur laquelle on est actuellement, donc l'état actuel du repository.

Chaque nouveau commit créé par `git commit` est ajouté comme fils de la branche actuel (donc du dernier commit de la branche actuelle). En plus de créer un objet commit, `git commit` mets à jour la branche (donc indirectement HEAD) pour pointer sur ce nouveau commit.

> A working dir(ectrory) (or working tree) is any directory on our file system which has a repository associated with it.

^ working dir = un répertoire associé à un repo.

> A repository (in short: repo) is a collection of commits, each of which is an archive of what the project’s working tree looked like at a past date, whether on our machine or someone else’s. A repository also includes things other than our code files, such as HEAD, branches etc.

^ un repo est la collection des snapshots passés (=l'historique des commits) + des metadata.

> Unlike other, similar tools you may have used, git does not commit changes from the working tree directly into the repository. Instead, changes are first registered in something called the index, or the staging area

^ staging area = l'une des trois zones git, intermédiaire entre le working dir et le repo.

> When we checkout a branch, git populates the index with all the file contents that were last checked out into our working directory and what they looked like when they were originally checked out

^ important pour comprendre reset (et comment git fonctionne en général) : se placer sur une branche revient à placer les trois zones sur cette branche : HEAD, l'index, et le working dir.

> When we use git commit, the commit is created based on the state of the index.

Autre point à comprendre, un peu contre-intuitif : un commit est créé à partir du staging area, pas du working dir.

> Tracked files are files that git knows about. They either were in the last snapshot (commit), or they are staged now (that is, they are in the staging area). Untracked files are everything else

^ un fichier traqué est soit staged (s'il est nouveau), soit connu lors d'un précédent commit.

> At this point, it would be helpful to make a distinction between two types of git commands: plumbing and porcelain. The application of the terms oddly comes from toilets (yeah, these — 🚽), traditionally made of porcelain, and the infrastructure of plumbing (pipes and drains). We can say that the porcelain layer provides a user-friendly interface to the plumbing. (...) the low-level commands that users don’t usually need to use directly (“plumbing” commands) from the more user-friendly high level commands (“porcelain” commands).

Origines des termes plumbing et porcelain !

Le reste du post est très intéressant mais inutile à annoter : on crée des objets git (blob, tree, commit, branche) à la mano , ou du moins le plus à la mano possible, avec éventuellement des commandes plumbing, mais sans commandes porcelain.
