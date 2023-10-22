* [Commandes utiles](#commandes-utiles)
* [Installation](#installation)
* [Hook de pre-commit](#hook-de-pre-commit)
* [Détails](#détails)
   * [Version explicite + fichier de style](#version-explicite--fichier-de-style)
   * [Fichiers à formatter](#fichiers-à-formatter)
   * [Vérifier les fichiers sans les modifier vs. les reformatter](#vérifier-les-fichiers-sans-les-modifier-vs-les-reformatter)


# Commandes utiles

**TL;DR** : les commandes complètes :

```sh
# pour formatter le contenu du répertoire 'SOURCES' :

# vérifier les fichiers sans les modifier, sortir en erreur si mauvais formattage :
clang-format-14 --dry-run --Werror --style=file:/path/to/mystyle "$(find SOURCES -name \*.cpp)" "$(find SOURCES -name \*.h)"


# reformatter les fichiers mal formattés :
clang-format-14 -i --style=file:/path/to/mystyle "$(find SOURCES -name \*.cpp)" "$(find SOURCES -name \*.h)"
```

# Installation

Les dépôts apt ont une version de clang-format (la 14 pour debian bookworm et ubuntu 22.04).

Si besoin, on peut aussi installer une version particulière via pipx ; dans ce cas, on peut faire un alias pour avoir la commande avec la version dans le nom :

```sh
pipx install 'clang-format<15'  # installera la version 14
cd ~/.local/bin
ln -s clang-format clang-format-14
```

# Hook de pre-commit

On a le choix entre :

- formatter automatiquement le code lorsqu'on commite (avantage = facilité d'utilisation car on ne s'embête pas avec le formatage ; inconvénient = on perd le contrôle sur ce qu'on committe)
- se contenter de refuser un commit contenant du code mal-formaté (avantages/inconvénients = le contraire)

Une troisième possibilité consiste à définir un hook de pre-commit custom qui 1. refuse un commit mal formatté, mais 2. dans le même temps, reformatte le code.

Ainsi, lorsqu'on essaye de commiter du code mal-formatté, on gagne à la fois :

- le fait que le commit soit refusé
- le fait que le code soit automatiquement reformatté
- le contrôle sur ce qu'on committe puisqu'on peut vérifier le code reformatté avant de le commiter
- la simplicité d'utilisation puisqu'il suffit de relancer aussitôt la commande de commit pour commiter du code (qui ce coup-ci est bien formatté)

Exemple de config pre-commit pour lancer un script custom :

```yml
 - repo: local
   hooks:
       - id: my-clang-format
         name: my-clang-format
         entry: path/to/my-clang-format.sh
         language: system
         types_or: [c++,c]
```

Exemple de hook qui fait le taf :

```sh
#!/bin/bash

set -o nounset
set -o pipefail

prevent_commit_if_wrong_format() {
    # on n'intervient que s'il y a du formattage à faire :
    if ! eval clang-format-14 --dry-run --Werror "$*"
    then
        # dans ce cas, on reformatte les fichiers mal-formattés :
        echo "Formatting in progress..."
        if ! eval clang-format-14 -i "$*"; then
            echo "ERROR"
            return 10
        fi
        echo "Formatting done"

        # et on sort en erreur s'il y a eu du formattage à faire :
        return 11
    fi
}

prevent_commit_if_wrong_format "$*"
```


# Détails

## Version explicite + fichier de style

Pour que le formattage soit déterministe, mieux vaut préciser explicitement la version de `clang-format` à utiliser + passer le fichier de style explicitement :

```sh
# préférer ceci :
clang-format-14 --style=file:/path/to/mystyle ...

# à celà :
clang-format ...
```

À noter que si on ne précise pas le style, celui-ci est lu dans un fichier caché `.clang-format` d'un répertoire parent.

(pour simplifier, dans le reste des notes, je n'indique pas le style ni la version)

## Fichiers à formatter

Il faut préciser explicitement un à un les fichiers à vérifier :


```sh
# ceci ne marche pas :
clang-format FOLDER/

# il faut faire ceci :
clang-format FOLDER/file1.cpp FOLDER/file1.h
```

Du coup, pour analyser tout un répertoire :

```sh
clang-format "$(find FOLDER -name \*.cpp)" "$(find FOLDER -name \*.h)"
```

## Vérifier les fichiers sans les modifier vs. les reformatter

Pour vérifier des fichiers sans les modifier, et sortir en erreur s'ils sont mal-formattés :

```sh
clang-format --dry-run --Werror ...
```

Pour reformatter les fichiers qui le nécessitent (ce qui les modifie) :


```sh
clang-format -i ...
```
