Ces notes vrac sont d'anciennes notes transférées prises suite à problèmes pour lancer mypy sur mes POCs.

* [Lancement d'un script](#lancement-dun-script)
   * [sys.path et ce qui se retrouve automatiquement dedans](#syspath-et-ce-qui-se-retrouve-automatiquement-dedans)
   * [les deux façons de lancer l'interpréteur sur un script](#les-deux-façons-de-lancer-linterpréteur-sur-un-script)
   * [import relatif vs. import absolu](#import-relatif-vs-import-absolu)
   * [comprendre les erreur d'import relatif](#comprendre-les-erreur-dimport-relatif)
* [Imports python](#imports-python)
* [Imports mypy](#imports-mypy)

# Lancement d'un script

Pour mémoire: un extrait pertinent de mes notes sur le lancement de python :

## sys.path et ce qui se retrouve automatiquement dedans

Le script principal est le script responsable de l'appel de l'interpréteur :

```sh
python mainscript.py
./mainscript.py
```

Le chemin absolu contenant le script principal se retrouve automatiquement dans `sys.path[0]` ; dit autrement, on peut toujours importer directement tout module placé dans le même répertoire que le script principal.

Si l'interpréteur est appelé sans script principal (e.g. `python -c 'print(42)'`), alors une chaîne vide `""` est placée en `sys.path[0]`. Dans ce cas, c'est le répertoire courant (CWD) qui est utilisé comme `sys.path[0]`, **c'est le seul cas où le cwd a une influence sur le sys.path**.

## les deux façons de lancer l'interpréteur sur un script

L'interpréteur lit le fichier passé en paramètre est l'exécute en tant que main :

```sh
python monscript.py
```

L'interpréteur recherche dans les répertoires de `sys.path` un package `pkg.monmodule`, et l'exécute en tant que main :

```sh
python -m pkg.monmodule
```

## import relatif vs. import absolu

Import absolu : python recherche le package `pkg` dans les répertoires du `sys.path` (puis y cherche `subpkg`, puis `mymodule`) :

```python
import pkg.subpkg.mymodule
```

Import relatif : au lieu de le chercher dans `sys.path`, python le recherche par rapport AU MODULE COURANT (celui depuis lequel l'import est fait) :

```python
from ..subpkg import mymodule
```

Ici, il est recherché dans le package parent du module : pour faire un import relatif depuis un module ce module DOIT APPARTENIR À UN PACKAGE.


## comprendre les erreur d'import relatif

Tout découle de cette phrase que j'ai dite plus haut :

> pour faire un import relatif depuis un module ce module DOIT APPARTENIR À UN PACKAGE

Ça dépend de plusieurs choses :
- `__name__` et `__package__`
- le fait de faire un import relatif depuis le module principal, ou un module importé
- la façon dont on appelle le script principal (`python file.py`   vs.   `python -m pkg.module`)

```sh
ValueError: Attempted relative import beyond toplevel package
# on est "remontés" trop loin (le package contenant le module qu'on essaye d'importer n'est pas dans sys.path)

# ValueError: Attempted relative import in non-package
le module depuis lequel on essaye l'import relatif n'est pas dans un package
```

# Imports python

(j'avais fini par en faire [un blogpost](https://phidra.github.io/blog/2021-10-03-from-xxx-import-something/))

https://chrisyeh96.github.io/2017/08/08/definitive-guide-python-imports.html

- import statements search through the list of paths in sys.path
- sys.path always includes the path of the script invoked on the command line and is agnostic to the working directory on the command line.
- importing a package is conceptually the same as importing that package’s __init__.py file
- When a module is imported, Python runs all of the code in the module file.
- When a package is imported, Python runs all of the code in the package’s __init__.py file, if such a file exists.
- All of the objects defined in the module or the package’s __init__.py file are made available to the importer.

Quand on importe un module 'spam' (ou le __init__ d'un package) :
- python regarde si spam existe dans les builtin-modules
- puis, python regarde dans le répertoire courant s'il y a un module (ou package) qui matche
- puis, python regarde dans le sys.path (y compris la stdlib, qui passe donc APRES un module custom du même nom)

NOTE : j'ai fait des POCs sur le sujet = https://github.com/phidra/pocs/tree/4e9dfc553fb02716f023e9b40b67d2e9b4c5103c/python/import

Le `sys.path` a donc spécifiquement pour raison d'exister le fait de répondre à "où dois-je rechercher un module importé" :

- à noter qu'il contient comme premier item le répertoire du script principal (à ne pas confondre avec le cwd du shell ayant lancé le script !)
- le seul cas où le cwd du shell est pertinent, c'est si on n'a pas passé de fichier à exécuter :
    ```
    python3 -c "import sys; print(repr(sys.path[0]))"
    ''
    ```
- preuve que le premier item de `sys.path` est bien le répertoire du script (et pas le cwd) :
    ```sh
    cat > /tmp/pouet.py << EOF
        import sys
        print("sys.path[0] = ", repr(sys.path[0]))
        import os
        print("os.getcwd() = ", os.getcwd())
        EOF
    python3 /tmp/pouet.py
        sys.path[0] =  '/tmp'
        os.getcwd() =  /media/DATA
    ```

----

https://docs.python.org/3/tutorial/modules.html

- Within a module, the module’s name (as a string) is available as the value of the global variable __name__.
- NdM : le `__name__` d'un module est le nom *complet* du module (y compris son package/subpackage), i.e. le fully-qualified module name.
- quand on exécute directement un script (python main.py), __name__ prend la valeur `__main__`

----


[la section dédiée à comment python recherche un module importé](https://docs.python.org/3/tutorial/modules.html#the-module-search-path) ; le sys.path contient entre autres :

- répertoire parent du script principal
- contenu du `PYTHONPATH`
- une liste de répertoires "installation-dependent" (p.ex. sous debian, contient des `site-packages`)

----


https://docs.python.org/3/tutorial/modules.html#standard-modules

- Some modules are built into the interpreter;
- these provide access to operations that are not part of the core of the language but are nevertheless built in, either for efficiency or to provide access to operating system primitives such as system calls.
- The set of such modules is a configuration option which also depends on the underlying platform.
- One particular module deserves some attention: sys, which is built into every Python interpreter. 

----


https://docs.python.org/3/tutorial/modules.html#packages

- For example, the module name `A.B` designates a submodule named `B` in a package named `A`
- Package = regroupement de modules.
- SubPackage = regroupement de modules au sein d'un package.
- When importing the package, Python searches through the directories on sys.path looking for the package subdirectory.
- point important : le métier de sys.path, c'est de fournir des chemins dans lesquels rechercher des package (i.e. des répertoires contenant __init__.py)
- The __init__.py files are required to make Python treat directories containing the file as packages. This prevents directories with a common name, such as string, unintentionally hiding valid modules that occur later on the module search path
- point intéressant : l'objectif de __init__.py est de NE PAS traiter un simple répertoire comme un package par défaut (pour éviter les masquages de noms non-désirés)

----


NdM : ce qui est un peu confusant, c'est que la ligne suivante :

```python
import A.B
```

Peut importer aussi bien un module qu'un package :

- le module B du package A
- le subpackage B du package A (et dans ce cas, c'est le __init__.py de B qui est réellement importé)

De même, la ligne suivante :

```python
from C import D
```

Peut importer le module D, le subpackage D (en exécutant son init) et la fonction "D" du module C.

Beaucoup de confusion facile à faire... Confirmé par la doc :

> Note that when using from package import item, the item can be either a submodule (or subpackage) of the package, or some other name defined in the package, like a function, class or variable.
>
> Contrarily, when using syntax like import item.subitem.subsubitem, each item except for the last must be a package; the last item can be a module or a package but can’t be a class or function or variable defined in the previous item.

# Imports mypy

Contexte = réussir à lancer mes pocs (publiques) et mypy, à la fois depuis leur répertoire, et depuis pre-commit.

Résumé de mon problème mypy :

- mypy a besoin que mes pocs aient des `__init__.py`
- si je rajoute des `__init__.py` :
    - soit mypy échoue quand je le lance depuis la POC (si je garde `from library import machin`)
    - soit l'exécution par python échoue quand je le lance depuis la POC (si j'essaye un import relatif)
- je comprends mieux l'histoire de l'import relatif :
    - l'import relatif ne se fait que dans un package
    - et pour que mon "main" (i.e. le script que j'appelle) soit vu comme un package, il faut l'appeler avec python -m

Sans le __init__.py :

- si je suis dans le répertoire de la poc11, tout fonctionne comme attendu :
    ```sh
    python3 poc.py
    # OK

    mypy .
    # OK
    ```
- si je suis dans le répertoire parent de la poc11, un mypy général échoue, le reste fonctionne :
    ```sh
    python3 poc11_interface6_using_protocols/poc.py
    # OK

    mypy poc11_interface6_using_protocols
    # OK

    mypy .
    # ERROR
    # poc02_using_newtype/poc.py: error: Duplicate module named 'poc' (also at './poc01_when_does_mypy_actually_run/poc.py')
    # Found 1 error in 1 file (errors prevented further checking)
    ```
- [cette issue](https://github.com/python/mypy/issues/4008) est enrichissante :
    - guido suggère que lancer un seul mypy global aux différentes pocs n'est pas la bonne approche (QUESTION = pourquoi)
    - en effet, la commande suivante qui lance plusieurs mypy réussit : `for i in *; do mypy $i; done`
    - (par contre, la commande suivante qui lance un seul mypy sur plusieurs fichiers échoue : `mypy *`)
    - ----------------------------------------
    - le problème rencontré est exactement le mien
    - en fin d'issue, un mec rapporte [une issue spécifiquement sur le sujet](https://github.com/python/mypy/issues/10428), qui correspond exactement à mon besoin (spoiler alert : personne n'a trouvé de solution...)
- cette [autre issue](https://github.com/wemake-services/wemake-django-template/issues/410) m'informe que mypy discover le code en fonction des import
- [cette doc](https://mypy.readthedocs.io/en/latest/running_mypy.html#mapping-file-paths-to-modules) semble expliquer comment mypy fonctionne

Analyse mypy :

- https://mypy.readthedocs.io/en/stable/running_mypy.html
- https://mypy.readthedocs.io/en/stable/running_mypy.html#following-imports
    - par défaut, mypy type-checke les imports (tout le code top-level du module importé, et le body de ses fonctions type-hintées)
- https://mypy.readthedocs.io/en/stable/running_mypy.html#mapping-file-paths-to-modules
    - les fichiers passés explicitement sont checkés, et si on passe un répertoire, on recherche récursivement tous les fichiers xxx.py qu'il contient
    - dans les deux cas, mypy traduit le chemin du fichier (project/a/b/c.py) en un fully-qualified-module-name (a.b.c)
    - le répertoire contenant le package (project) est ajouté au chemin mypy de recherche des module
    - tant que les répertoires parents ont le fichier __init__.py, le fichier.py est considéré comme un sous-module d'un package plus global :
        - `pkg/subpkg/module.py` sera traduit en `pkg.subpkg.module` à condition que `pkg/__init__.py` et `pkg/subpkg/__init__.py` existent
        - l'utilisation de l'option `--namespace-packages` permet de se passer de `pkg/subpkg/__init__.py` :
            - il suffit que `pkg/__init__.py` existe pour tradure `pkg/subpkg/module.py` soit traduit en `pkg.subpkg.module`
            - en gros, on se repose sur le `__init__.py` le plus top-level pour trouver le package "racine" à partir duquel inférer les subpkgs
        - l'utilisation de l'option `--explicit-package-bases` remplace la recherche du top-level `__init__.py` par la recherche du top-level répertoire qui est dans MYPY_PATH
- https://mypy.readthedocs.io/en/stable/running_mypy.html#how-imports-are-found
    - mypy résoud les imports comme python, à quelques cas près, listés dans la page
    > Contrarily, when using syntax like import item.subitem.subsubitem, each item except for the last must be a package; the last item can be a module or a package but can’t be a class or function or variable defined in the previous item.
