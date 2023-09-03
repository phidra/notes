**WORK IN PROGRESS** : je suis en train de transférer mes anciennes notes privées vers ce repo public -> ces notes correspondent juste à la portion que j'ai transférée.

* [Misc](#misc)
   * [contourner un certificat manquant](#contourner-un-certificat-manquant)
   * [commit parent de deux branches](#commit-parent-de-deux-branches)
   * [formatter l'affichage d'un commit pour afficher des champs en particulier](#formatter-laffichage-dun-commit-pour-afficher-des-champs-en-particulier)
   * [lister des branches distantes ou locales](#lister-des-branches-distantes-ou-locales)
   * [lister les infos des branches distantes](#lister-les-infos-des-branches-distantes)
* [Worktree](#worktree)
* [Revert](#revert)
   * [Revert un commit classique](#revert-un-commit-classique)
   * [Revert un commit de merge](#revert-un-commit-de-merge)
* [Modifier l'historique](#modifier-lhistorique)
   * [Modifier la date d'un commit](#modifier-la-date-dun-commit)
   * [Modifier l'auteur d'un commit](#modifier-lauteur-dun-commit)
   * [Modifier un commit](#modifier-un-commit)
   * [Rebaser le tout premier commit](#rebaser-le-tout-premier-commit)
   * [Splitter un commit](#splitter-un-commit)
* [Tagging](#tagging)
   * [Créer un tag](#créer-un-tag)
   * [Afficher les tags existants](#afficher-les-tags-existants)
   * [Supprimer un tag](#supprimer-un-tag)
   * [Renommer un tag](#renommer-un-tag)
* [Stashing](#stashing)
   * [Stasher le working dir actuel](#stasher-le-working-dir-actuel)
   * [Consulter les stashs](#consulter-les-stashs)
   * [Appliquer et/ou supprimer un stash](#appliquer-etou-supprimer-un-stash)
* [Remotes](#remotes)
* [LFS](#lfs)



# Misc

## contourner un certificat manquant

Si un git push/pull échoue à cause d'un certificat manquant, le quick-and-dirty contournement est `GIT_SSL_NO_VERIFY` :

```sh
GIT_SSL_NO_VERIFY=1 git push
```

## commit parent de deux branches

Afficher le hash du commit parent le plus récent de deux branches (i.e. le dernier commit avant que les branches ne divergent) :

```sh
git merge-base BRANCH1 BRANCH2
```

## formatter l'affichage d'un commit pour afficher des champs en particulier

```sh
git show b6296d75b2865eeb5c18dd7627fb5f3a8399d89f --pretty='format:%ai'
2021-09-24 08:51:39 +0000
```

Grepper `placeholders` dans `git help show` pour savoir comment afficher tel ou tel champ (ou `git log --help`).

Exemple :

```sh
git log --format="%H  %h  %an  %ae  %cn  %ce   %s"
# H  = hash
# h  = hash abrégé
# an = author name
# ae = author mail
# cn = committer name
# ce = committer mail
# s  = message de commit
```


## lister des branches distantes ou locales

```sh
git for-each-ref PATTERN
# BRANCHES LOCALES = git for-each-ref refs/heads
# BRANCHES REMOTE QUE J'AI PULL LOCALEMENT = git for-each-ref refs/remotes/origin
# branche distantes... ?
```

## lister les infos des branches distantes

Comment lister les infos des branches distantes (date, auteur, nom de branche) :

```sh
git ls-remote --heads origin|cut -f1 > INPUT
while read line
    do
    git log -1 --format='format:%ai  %d === %an%n' $line
    done < INPUT | sort
```

# Worktree

TL;DR : permet de checkout une autre branche que l'actuelle, dans un répertoire quelconque.

https://git-scm.com/docs/git-worktree

Exemple d'usage : "jeter un coup d'oeil" à une branche sans avoir besoin de stasher mes modifs :

```sh
# Création du worktree :
git worktree add

# Lister les worktrees :
git worktree list  # le main worktree est toujours listé en premier

# Utilisation :
cd /tmp/glimpse
git checkout another-branch  # p.ex.
git status  # p.ex.

# Suppression, attention, il faut DEUX commandes :
git worktree remove
git worktree prune
```

Différence entre MAIN working tree (1 seul, voire 0 si repo bare) et LINKED working tree (entre 0 et N) :

- `git init` (ou `git clone`) = préparer un **main** working tree
- `git worktree add` = préparer un **linked** working tree

NOTE : le répertoire qui acceuille le worktree "secondaire" peut-être n'importe où :

- dans le repo (dans ce cas, attention à l'ajouter au `.gitignore` ; par exemple pour le répertoire de publication de [mon blog](https://phidra.github.io/blog/))
- ailleurs, par exemple dans `/tmp` !

# Revert

Permet d'annuler un commit A, en créant un nouveau commit B comportant un diff opposé à A (l'annulation du commit est donc inscrite dans l'historique, ce qui peut ou non être ce qu'on veut faire).

## Revert un commit classique

```sh
# reverter un commit arbitraire :
git revert <commit-id>

# E.g. annuler le dernier commit :
git revert HEAD
```

## Revert un commit de merge

**TL;DR** :

- identifier dans le commit de merge quels sont les numéros des deux branches parentes :
    ```sh
    git log -1 <commit-de-merge>
    # commit bb5ee6580d99f081e4f941109764951a47ae4291
    # Merge: d213e1968 c2bd4d214        <------  le commit n°1 est d213e1968 (l'autre est n°2)
    # Author: My Self <my@self.fr>
    # Date:   Sat Sep 17 16:17:05 2022 +0200
    ```
- faire un revert du commit de merge, en passant `-m X` où X est le numéro de parent **qu'on veut garder** (i.e. on ne veut PAS reverter X, on veut reverter l'autre parent)
    ```sh
    git revert -m 1 <commit-de-merge>
    ```
- éventuellement, vérifier que le nouveau commit de revert créé par la commande précédente annule bien les modifs souhaitées :
    ```sh
    git show HEAD
    ```

Plus en détail : conceptuellement, reverter un commit de merge est plus compliqué qu'un commit classique, puisqu'on n'a pas un diff dont on pourrait construire l'opposé : un commit de merge est un commit "vide" qui n'a d'intérêt que d'avoir deux parents, i.e. de "réunir" deux branches.

En pratique, si on veut reverter un commit de merge, c'est souvent qu'on veut annuler le merge d'une branche dans une autre. Exemple concret : on a une branche `master`, une feature-branch `mywork`, et on a fait :

```sh
git checkout master  # on suppose qu'on est sur master
git merge --no-ff mywork
```

Le merge de `mywork` en `--no-ff` a créé un commit de merge `C`.

Maintenant, on veut annuler le merge de `mywork`, et retrouver un état de `master` comme si on n'avait pas mergé `mywork`. La commande suivante va créer un commit de revert `R` qui annulera tous les diffs de `mywork` :

```sh
git checkout master  # on suppose qu'on est sur master
git revert -m 1 <hash-du-commit-de-merge-C>  # un commit de revert R est créé
```

Explications :

- le principe va être d'annuler complètement tous les diffs introduits par l'une des deux branches du merge
- la partie importante est `-m 1` : c'est ce qui permet d'indiquer qu'on veut **garder** les commits du premier parent du merge
- pour connaître les "numéros" des deux parents du merge :
    ```sh
    git log -1 <commit-de-merge>
    # commit bb5ee6580d99f081e4f941109764951a47ae4291
    # Merge: d213e1968 c2bd4d214        <------  le commit n°1 est d213e1968 (l'autre est n°2)
    ```
- (apparemment, en général, quand on merge une feature-branch sur un tronc, le tronc se trouve n°1 et la feature-branch est n°2)
- si besoin, on peut consulter les commits parents pour être sûr de bien identifier les branches :
    ```sh
    git show --name-status d213e1968
    git show --name-status c2bd4d214
    ```


Plus de détails avec `git revert --help` :

> -m parent-number, --mainline parent-number \
>    Usually you cannot revert a merge because you do not know which side of the merge should be considered the mainline. This option specifies the parent number (starting from 1) of the mainline and allows revert to reverse the change relative to the specified parent.
>
>    Reverting a merge commit declares that you will never want the tree changes brought in by the merge. As a result, later merges will only bring in tree changes introduced by commits that are not ancestors of the previously reverted merge. This may or may not be what you want.

La dernière phrase m'interpelle (possiblement, elle empêchera de "remerger" la branche après éventuelles corrections) mais je n'ai pas encore creusé.


Le man pointe aussi vers un howto, qui se trouve être [revert-a-faulty-merge.txt](https://github.com/git/git/blob/d420dda0576340909c3faff364cfbd1485f70376/Documentation/howto/revert-a-faulty-merge.txt) (idem, j'ai pas creusé)

# Modifier l'historique

## Modifier la date d'un commit

Attention, chaque commit est associé à **deux** dates ([lien StackOverflow](https://stackoverflow.com/questions/11856983/why-is-git-authordate-different-from-commitdate/11857467#11857467)), `AuthorDate` qui est la date originale et `CommitDate` qui est la date de dernière modif du commit, e.g. du dernier rebase. Sur un commit simple, les deux dates sont identiques :

```sh
git log -1 --pretty=fuller
# commit e91bbb290a940ed96bc6868cca154db8fe2d6299 (HEAD -> master, origin/master, origin/HEAD)
# Author:     MonPrénom MonNom <moi@mondomaine.fr>
# AuthorDate: Sat Jul 30 10:26:44 2022 +0200
# Commit:     MonPrénom MonNom <moi@mondomaine.fr>
# CommitDate: Sat Jul 30 10:26:44 2022 +0200
```

Pour modifier les deux dates, il faut utiliser à la fois l'option `--date` et l'envvar `GIT_COMMITTER_DATE` :

```sh
NEWDATE="2022-02-03T04:05:06" ; GIT_COMMITTER_DATE="$NEWDATE" git commit --amend --no-edit --date "$NEWDATE"
```

Les formats de date acceptés par git sont [documentés ici](https://git-scm.com/docs/git-commit#_date_formats).

## Modifier l'auteur d'un commit

Même problématique ici, il y a **deux** auteurs `Author` (auteur original) et `Committer` (auteur de la modification la plus récente).

L'idée va être de réécrire le commit avec rebase (ce qui settera le committer à l'user actuel) et d'utiliser l'option `--reset-author` pour setter également l'author à l'user actuel.

Il y a un hic : la `CommitDate` sera également resettée à la date actuelle -> si on veut plutôt la laisser inchangée, il faut carabistouiller :

```sh
# marquer le commit à modifier en EDIT :
git rebase -i PARENT

# recommiter à l'identique en ne changeant que l'auteur (pour l'user actuel) :
git commit --no-verify --amend --reset-author --no-edit --date="$(git log -n 1 --format=%aD)" && git rebase --continue
```

## Modifier un commit

Pour modifier le dernier commit : simplement faire les modifs souhaitées, puis :

```sh
git add .
git commit --amend  # éventuellement : --no-edit
git push --force
```

Pour modifier un commit plus ancien, c'est le même principe, mais au milieu d'un rebase interactif :

```sh
git rebase -i PARENT
# sur le commit souhaité, choisir "edit"

# faire mes modifs
git add .
git commit --amend  # éventuellement : --no-edit

git rebase --continue
```

## Rebaser le tout premier commit

Le premier commit n'ayant pas de parent, on utilise l'option `--root` :

```
git rebase -i --root
```

## Splitter un commit

Le principe est de faire un rebase interactif, et de recommiter le commit en deux fois ([post qui donne ces commandes](https://emmanuelbernard.com/blog/2014/04/14/split-a-commit-in-two-with-git/)) :


```sh
git rebase -i <oldsha1>
# mark the desired commit as `edit`

git reset HEAD^

git add <myfiles-to-put-in-commit1>
git commit -m "First part"

git add <myfiles-to-put-in-commit2>
git commit -m "Second part"

git rebase --continue
```

# Tagging

NOTE : les tags annotés sont plus complets et pérennes que les tags légers...

## Créer un tag

```sh
# tag annoté sur la version courante :
git tag -a TAGNAME

# tag léger sur la version courante :
git tag TAGNAME

# tag annoté sur une commit antérieur :
git tag -a TAGNAME COMMIT
```

ATTENTION : par défaut, un simple `git push` n'envoie pas les tags sur le serveur, il faut :

```sh
# pusher un tag sur le serveur :
git push origin TAGNAME

# pusher tous les tags sur le serveur :
git push origin --tags
```

## Afficher les tags existants

```sh
# afficher tous les tags :
git tag

# filtrer avec un pattern :
git tag -l "VERSION__*"

# afficher le commit d'un tag :
git show TAGNAME
```

## Supprimer un tag

```sh
git tag -d MYSUPERTAG
git push origin :refs/tags/MYSUPERTAG
```

## Renommer un tag

Créer un tag du nouveau nom, puis supprimer l'ancien tag.

# Stashing

Concept :

- Lorsqu'on a des modifications du working dir, qu'on souhaite revenir temporairement à un état propre du working dir, mais sans commiter les modification.
- (exemple : on a un dev en cours, mais on a soudainement besoin de revenir à un état antérieur dans l'historique)
- En d'autres termes, on souhaite annuler temporairement la modification du working dir, mais sans la commiter.
- `git stash` permet de stocker les modifications pour les réappliquer plus tard.

À noter qu'en alternatives, on peut utiliser les worktrees ou consulter l'IHM github/gitlab.

Références :

- https://git-scm.com/docs/git-stash
- `git stash --help`

## Stasher le working dir actuel

```sh
# Version longue (permettant de décrire stash, recommandé pour s'y retrouver)
git stash save "Description de mon stash"

# Version courte = le nom est automatique
git stash
```

## Consulter les stashs

```sh
# Voir tous les stashs enregistrés :
git stash list

# Détails (fichiers modifiés) du dernier stash enregistré
git stash show

# patch (i.e. le diff) du dernier stash enregistré.
git stash show -p

# Donne les détails du stash identifié par "stash@{2}"
git stash show stash@{2}
```

## Appliquer et/ou supprimer un stash

```sh
# Cas le plus courant = appliquer puis supprimer le stash
# Ça remet le working_dir + les stashs dans l'état avant le stashing.

# le dernier stash enregistré :
git stash pop
# un stash en particulier :
git stash pop stash@{2}


# Dropper un stash = le supprimer SANS L'APPLIQUER au working_dir :

# le dernier stash enregistré :
git stash drop
# un stash en particulier :
git stash drop stash@{2}


# Edge-case = appliquer le patch d'un stash sans le supprimer :
# (e.g. pour l'appliquer à plusieurs branches)

# le dernier stash enregistré :
git stash apply
# un stash en particulier :
git stash apply stash@{2}
```

# Remotes

http://git-scm.com/book/fr/Les-bases-de-Git-Travailler-avec-des-d%C3%A9p%C3%B4ts-distants

Concept :

- Les remote sont des versions distantes de mon repo.
- On n'est pas limité au seul remote "origin" : pour un même repo local, on peut avoir autant de remote qu'on veut.
- Ils permettent une gestion basique des droits : lecture ou lecture+écriture.

HOWTO :

```sh
# Lister les remotes :
git remote
git remote -v

# Créer un remote (en l'identifiant localement comme REMOTENAME) :
git remote add REMOTENAME REMOTEURL
# Exemple :
git remote add mysuperproduction ssh://root@webprod.fr:/opt/gitbare/dev_site_copro

# Afficher les infos d'un remote :
git remote show REMOTENAME

# Renommer un remote :
git remote rename OLDNAME NEWNAME

# Supprimer un remote :
git remote rm REMOTENAME
```

# LFS


Setup d'un repo gitlab LFS :

- vérifier dans la config du projet gitlab que LFS est activé
    ```
    Settings > Visibility, project features, permissions > Repository > Git Large File Storage (LFS)
    ```
- sur ma machine locale, installer un client :
    ```sh
    sudo apt install git-lfs
    ```
- cloner le repo, et indiquer qu'on veut traquer certains fichiers (ici, PDF) avec LFS :
    ```sh
    git clone git@mysupergitlab.server:mysuperuser/test-lfs.git
    git lfs install
    git lfs track "*.pdf"
    git add .gitattributes
    git commit -m "tracking pdfs"
    ```

Notes à l'utilisation :

- les large-files sont pushés en HTTP (utiliser `GIT_SSL_NO_VERIFY=1` si besoin)
- tout fonctionne de façon intuitive :
    - modifier un fichier PDF est bien vu comme "fichier modifié" (en vrai, le diff montre que c'est le pointeur qui a changé)
    - quand on revient à un état antérieur dans l'historique, on retrouve bien son fichier passé
    - on peut supprimer un fichier, tout en le retrouvant dans l'historique (on se contente sans doute de supprimer le pointeur)
    - un clone récupère bien automatiquement les fichiers PDF
- C'est pas encore clair où les fichiers sont stockés, et comment faire du ménage ([cette page](https://github.com/git-lfs/git-lfs/blob/main/docs/man/git-lfs-prune.1.ronn) indique comment pruner les fichiers devenus inutiles)
- c'est le fichier `.gitattributes` qui stocke la config de quels fichiers on doit gérer avec LFS (il doit donc être traqué avec le reste du repo)
- Commandes utiles :
    ```
    git lfs help
    git lfs env
    git lfs prune
    ```
- Quelques références sur les avantages/inconvénients :
    - https://gregoryszorc.com/blog/2021/05/12/why-you-shouldn%27t-use-git-lfs/ :
    > une fois que tu as LFS sur un repo, tu ne l'enlèves plus (ou pas facilement)
    - (donc adapté pour un répo de pérennisation, mais pas à des repos qui mélangent du code et des largefiles)
    - [discussion HackerNews](https://news.ycombinator.com/item?id=27134972) spawnée par le précédent lien :
    > LFS is not a distributed version control system; once you use it, a clone is no longer “as good” as the original, because it refers to a LFS server that is independent of your clone.
