**WORK IN PROGRESS** : je suis en train de transférer mes anciennes notes privées vers ce repo public -> ces notes correspondent juste à la portion que j'ai transférée.

* [Worktree](#worktree)
* [Modifier l'historique](#modifier-lhistorique)
   * [Modifier la date d'un commit](#modifier-la-date-dun-commit)
   * [Modifier un commit](#modifier-un-commit)
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

# Misc


Si un git push/pull échoue à cause d'un certificat manquant, le quick-and-dirty contournement est `GIT_SSL_NO_VERIFY` :

```sh
GIT_SSL_NO_VERIFY=1 git push
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