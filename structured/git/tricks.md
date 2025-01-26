* [Utiliser rebase --onto](#utiliser-rebase---onto)
   * [syntaxe contre-intuitive et moyen mnémotechnique](#syntaxe-contre-intuitive-et-moyen-mnémotechnique)
   * [Le problème que rebase --onto résout](#le-problème-que-rebase---onto-résout)
   * [Exemple d'utilisation 1 pour rebaser une sous-branche dérivée d'une branche](#exemple-dutilisation-1-pour-rebaser-une-sous-branche-dérivée-dune-branche)
   * [Exemple d'utilisation 2 si on a fait un hotfix sur la mauvaise branche](#exemple-dutilisation-2-si-on-a-fait-un-hotfix-sur-la-mauvaise-branche)
* [Trouver le commit depuis lequel deux branches ont divergé](#trouver-le-commit-depuis-lequel-deux-branches-ont-divergé)
* [Visualiser facilement les commits avec gitk](#visualiser-facilement-les-commits-avec-gitk)
* [Alias permettant d'explorer les commits en avançant et reculant](#alias-permettant-dexplorer-les-commits-en-avançant-et-reculant)
* [Mettre en cache le mot de passe du serveur git si pas d'authentification par clé](#mettre-en-cache-le-mot-de-passe-du-serveur-git-si-pas-dauthentification-par-clé)
* [Utiliser reflog pour savoir ce qui s'est passé sur une branche LOCALE](#utiliser-reflog-pour-savoir-ce-qui-sest-passé-sur-une-branche-locale)
   * [Formatage](#formatage)
* [Définir des hooks custom](#définir-des-hooks-custom)
* [Ignorer les commits de formattage lorsqu'on git blame](#ignorer-les-commits-de-formattage-lorsquon-git-blame)
* [Configuration git blame](#configuration-git-blame)


# Utiliser rebase --onto

La commande permet de changer le parent d'un set de commits :

```
git rebase --onto  NOUVEAU-PARENT-DU-PREMIER-COMMIT-À-DÉPLACER  ANCIEN-PARENT-DU-PREMIER-COMMIT-À-DÉPLACER
```

Ou bien, si on veut préciser explicitement la fin du set de commits à déplacer (par défaut, le set ira du premier commit jusqu'au head courant, donc si on veut déplacer toute une branche et qu'on est déjà dessus, il est inutile de préciser), la commande complète :

```
git rebase --onto  NOUVEAU-PARENT-DU-PREMIER-COMMIT-À-DÉPLACER  ANCIEN-PARENT-DU-PREMIER-COMMIT-À-DÉPLACER  (DERNIER-COMMIT-QUI-SERA-DÉPLACÉ)
```

## syntaxe contre-intuitive et moyen mnémotechnique

**ATTENTION** : le truc contre-intuitif, c'est que par rapport à une commande de rebase classique, si on veut forger une commande de rebase onto, on ne se contente pas de RAJOUTER un paramètre `--onto` à la commande classique !

Avec une commande de rebase classique :

```
git rebase MASTER
```

Avec une commande de rebase onto :

```
git rebase --onto MASTER COMMIT-X
           ^             ^
           |             |
           |             COMMIT-X a "pris la place de MASTER" comme argument sans option
           |
           MASTER est devenu l'argument de l'option `--onto`
```

La meilleure façon de comprendre la syntaxe est de comprendre que dans la situation classique (`git rebase MASTER`), tout se passe comme si le `--onto` était implicite, et que la vraie commande lancée était :

```
git rebase (--onto) MASTER
                    ^
                    MASTER devient le nouveau parent de la branche courante
```

Du coup, une commande complète avec `--onto` pour faire un rebase partiel se contente d'ajouter un élément = le parent du premier commit à déplacer :

```
git rebase (--onto) MASTER PARENT-DU-COMMIT-À-DÉPLACER
```

Du coup, un alias pour `git rebase` pourrait être `git change-parent`, puisque le fils immédiat de `PARENT` (et toute sa descendance) change de parent pour devenir fils immédiat de `MASTER`.


## Le problème que rebase --onto résout

État du repo :

```
A---B---C---D---E      (MASTER)
     \
      \-W---X---Y---Z  (BRANCH)
```

Que se passe-t-il si je fais un rebase "normal" = `git rebase MASTER (BRANCH)` (préciser `BRANCH` est optionnel si je suis déjà dessus, vu que la valeur par défaut si non-précisé est le HEAD de la branche courante) ? Voici l'état après rebase "normal" :

```
A---B---C---D---E---W---X---Y---Z
                ^               ^
                MASTER          BRANCH
```

Le `rebase --onto` est utile quand je ne veux pas déplacer TOUTE la branche sur master, mais seulement (p.ex.) les deux derniers commits `Y` et `Z`. Ainsi, si la situation souhaitée est celle-ci...

```
A---B---C---D---E---Y---Z
     \          ^       ^
      \         MASTER  BRANCH
       \-W---X
```

...alors la commande `git rebase --onto` permettant de faire ça est :

```
git rebase --onto MASTER COMMIT-X (BRANCH)
```

## Exemple d'utilisation 1 pour rebaser une sous-branche dérivée d'une branche

Supposons que j'ai une branche WORK dérivée de MASTER, et une sous-branche SUBWORK dérivée de WORK :

```
A---B---C---D---E      (MASTER)
     \
      \-W---X          (WORK)
             \
              \-Y---Z  (SUBWORK)
```

Maintenant, je veux rebaser WORK sur MASTER (par exemple, pour prendre en compte de récentes avancées de MASTER). L'état devient :

```
                MASTER
                v
A---B---C---D---E---W---X
     \                  ^
      \                 WORK
       \
        \-W---X---Y---Z  (SUBWORK)
```

Si je veux rebaser SUBWORK sur MASTER à son tour, c'est là que les ennuis commencent : si je me contente d'un rebase classique `git rebase MASTER (SUBWORK)`, on aura quelque chose qui ressemble à :

```
                MASTER
                v
A---B---C---D---E---W---X
                 \      ^
                  \     WORK
                   \
                    \-W---X---Y---Z  (SUBWORK)
```

SUBWORK a avancé pour rejoindre MASTER (elle est passée de B à E), mais ça n'est pas ce qu'on voulait !

C'est problématique, car non seulement on duplique les commits W et X, mais surtout SUBWORK n'est plus fils de WORK (et ne suit donc plus ses évolutions).


C'est là que le `rebase --onto` entre en jeu : dans la situation suivante...
```
A---B---C---D---E---W---X
     \          ^       ^
      \         MASTER  WORK
       \
        \-W---X---Y---Z  (SUBWORK)
```

... il permet de choisir de ne déplacer que Y et Z, avec la commande `git rebase --onto WORK X (SUBWORK)`, ce qui aboutit bien au résultat souhaité :

```
A---B---C---D---E---W---X---Y---Z
                ^        ^      ^
                MASTER   WORK   SUBWORK
```

## Exemple d'utilisation 2 si on a fait un hotfix sur la mauvaise branche

(les exemples concrets d'utilisation de onto dans "git help rebase" présentent ce cas, mais je ne trouve pas que c'est le plus intuitif)

Situation de départ :

```
M---M---M---M---M   (MASTER)
 \
  \-D---D           (DEVELOP)
         \
          \-H---H   (HOTFIX)
```

Oops je me suis trompé, j'ai branché HOTFIX à partir de DEVELOP au lieu de brancher depuis MASTER ! Situation souhaitée :

```
M---M---M---M---M
 \               \
  \-D---D         \-H---H
```

Pour déplacer HOTFIX sur MASTER, en remontant jusqu'au point de divergence entre HOTFIX et DEVELOP, la commande onto :

```
git rebase --onto MASTER DEVELOP HOTFIX
```

# Trouver le commit depuis lequel deux branches ont divergé

```sh
git merge-base mysuperbranch master
# 4d66ff0c242cf11864059048cc8ebc6f87129315
```

# Visualiser facilement les commits avec gitk

```sh
 gitk --all &
```


# Alias permettant d'explorer les commits en avançant et reculant

Ajouter les lignes suivantes dans le `~/.gitconfig` :

```
[alias]
prev = checkout HEAD^1
next = "!sh -c 'git log --reverse --pretty=%H master | awk \"/$(git rev-parse HEAD)/{getline;print}\" | xargs git checkout'"
```

**ATTENTION** : d'après la commande (que j'avais pompée de sametmax), c'est `master` qu'on explore.

# Mettre en cache le mot de passe du serveur git si pas d'authentification par clé

(note que mieux vaut authentifier par clé de toutes façons)

Pour éviter d'avoir à le retaper à chaque `git push`, on peut définir un cache d'une heure pour l'authentification :

```sh
# uniquement pour le repo courant :
git config credential.helper 'cache --timeout=3600'

# pour tous les repos :
git config --global credential.helper 'cache --timeout=3600'
```

# Utiliser reflog pour savoir ce qui s'est passé sur une branche LOCALE

Commande :

```
git reflog BRANCH
```

**ATTENTION** : l'état du reflog est propre au repo git **LOCAL**, il n'est pas poussé sur un serveur externe. Donc on ne peut investiguer que les modifications locales d'une branche.

En gros, ça fournit un historique des différents états du head local de BRANCH : historiquement, le HEAD local de BRANCH a d'abord pointé sur tel commit, puis sur tel commit, puis sur tel autre, etc.

Le message dans le reflog indique quel est la "raison" du changement de HEAD (un commit, un rebase, un merge, etc.) on peut donc retracer ce qui s'est passé (voire annuler un rebase).

## Formatage

À noter que "git reflog" est simplement un alias sur :

```
git log -g --abbrev-commit --pretty=oneline
```

Du coup, on peut formater le log comme on l'entend, avec le formatage classique des logs.

# Définir des hooks custom

Une façon pratique d'activer des hooks custom est de définir un répertoire de hooks custom :

```sh
git config core.hooksPath my_custom_hooks
```

^ à l'intérieur de ce répertoire, ajouter un fichier `pre-commit`

# Ignorer les commits de formattage lorsqu'on git blame

Parfois, on passe un grand coup de formatter sur tout le repo ; inconvénient = tous les git blame ultérieurs finiront par voir ce commit comme parent (i.e. ce commit pollue les `git blame`)

Solution = définir une liste de commits à ignorer dans un fichier + dire à git d'ignorer tous les commits de ce fichier :

# Configuration git blame

Pour ignorer les commits de reformattage de code lors d'un git blame, configurer git avec le fichier ignore suivant :

```sh
git config blame.ignoreRevsFile the_commits_to_ignore_with_git_blame
```

(à l'intérieur, une liste des ids sha ; les commentaires sont ignorés)


