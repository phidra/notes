# The [python] import system

- **url** = https://docs.python.org/3.10/reference/import.html
- **type** = doc de référence python
- **auteur** = core-team python
- **date de publication** = ?
- **source** = [The Python Language Reference](https://docs.python.org/3.10/reference/)
- **tags** = language>python ; topic>python>import ; topic>python>modules ; topic>python>packages ; level>intermediate

**TL;DR** : la doc de référence sur les imports. J'y trouve deux points intéressants : la définition formelle des packages, et en quoi ils diffèrent des modules, et les explications détaillées (que j'ai survolées) sur la mécanique qui se mets en branle quand on importe un module.

* [The [python] import system](#the-python-import-system)
   * [idée générale](#idée-générale)
   * [packages](#packages)
   * [namespace packages](#namespace-packages)
   * [résolution de la recherche des modules](#résolution-de-la-recherche-des-modules)
   * [les attributs des modules](#les-attributs-des-modules)
   * [imports relatifs](#imports-relatifs)
   * [le module main](#le-module-main)
   * [import des submodules](#import-des-submodules)

## idée générale

https://docs.python.org/3/reference/import.html#the-import-system

> The import statement combines two operations; it searches for the named module, then it binds the results of that search to a name in the local scope.

La fonction `__import__()` gère la première partie (recherche du module). On peut aussi l'invoquer nous-même, mais normalement, c'est surtout une fonction interne (invoquée notamment par le statement `import xxx`).

En tant qu'utilisateur, on n'a normalement pas besoin d'utiliser `__import__()`, on utilise plutôt `importlib.import_module()`, qui sert à faire des imports de façon dynamique plutôt que déclarative (et qui pourrait tout à fait utiliser elle-même `__import__()`, même si ce n'est pas obligatoire).

## packages

https://docs.python.org/3/reference/import.html#packages

> Python has only one type of module object, and all modules are of this type, regardless of whether the module is implemented in Python, C, or something else.
> You can think of packages as the directories on a file system and modules as files within directories, but don’t take this analogy too literally.
> 
> It’s important to keep in mind that all packages are modules, but not all modules are packages. Or put another way, packages are just a special kind of module.
>
> Specifically, any module that contains a __path__ attribute is considered a package.

Les keypoints :
- **tout package est avant tout un module**, il se trouve que c'est simplement un module qui peut **contenir** d'autres modules.
- Comme un package est avant tout un module, importer un package revient (comme pour tout module) à exécuter son code, puis à lier les top-level objects du package/module à des variables !
- Le code d'un module `mymodule`, c'est le module lui-même `mymodule.py` (je laisse de côté les cas plus complexes).
- Le code d'un package `mypackage`, c'est le fichier `mypackage/__init__.py`.

> A regular package is typically implemented as a directory containing an __init__.py file.\
> When a regular package is imported, this __init__.py file is implicitly executed, and the objects it defines are bound to names in the package’s namespace.\
> The __init__.py file can contain the same Python code that any other module can contain, and Python will add some additional attributes to the module when it is imported.

Un peu plus bas dans la doc, il y a une sous-section sur l'attribut `__path__` ([lien vers la sous-section](https://docs.python.org/3/reference/import.html#module-path)) ; la présence de cet attribut est la **seule** chose qui fait qu'un module est un package : un package est un module ayant un attribut `__path__`.

> By definition, if a module has a __path__ attribute, it is a package.

## namespace packages

Le nom _namespace package_ ([lien glossaire](https://docs.python.org/3/glossary.html#term-namespace-package)) est à interpréter comme "package qui net sert qu'à définir un namespace, i.e. à être un conteneur de module", par opposition aux regular packages ([lien glossaire](https://docs.python.org/3/glossary.html#term-regular-package)) qui sont des modules avant tout, et donc qui ont leur propre code.

En théorie, le terme _package_ ([lien glossaire](https://docs.python.org/3/glossary.html#term-package)) désigne les deux packages. En pratique, les namespace packages sont plus marginaux, et peuvent être ignorés si besoin.

Ils sont définis par [la PEP 420](https://www.python.org/dev/peps/pep-0420/). En gros, un namespace-package est un package virtuel, un groupe de modules (sans pour autant être un module lui-même, à la différence des regular-packages, et en particulier ils n'ont pas de code à exécuter).

https://docs.python.org/3/reference/import.html#namespace-packages

> Namespace packages may or may not correspond directly to objects on the file system; they may be virtual modules that have no concrete representation.

Ça a l'air d'exister depuis python 3.3.

## résolution de la recherche des modules

Il y a de nombreuses sections dédiées à comment python résout les imports des modules, et comment on peut customiser ce comportement.

J'ai essentiellement skimmé.

https://docs.python.org/3/reference/import.html#searching

> To begin the search, Python needs the fully qualified name of the module (or package, but for the purposes of this discussion, the difference is immaterial) being imported.

Notion de fully-qualified name ([lien glossaire](https://docs.python.org/3/glossary.html#term-qualified-name)).
	
----

> This name [may be] the dotted path to a submodule, e.g. foo.bar.baz. In this case, Python first tries to import foo, then foo.bar, and finally foo.bar.baz. If any of the intermediate imports fail, a ModuleNotFoundError is raised.

L'import d'un subpackage/submodule importe tous les packages (donc modules) parents aussi, cf. mes notes juste en dessous et ma POC

----

Au sujet du cache des modules (qui fait qu'on n'importe un module qu'une fois, au premier `import`) : https://docs.python.org/3/reference/import.html#the-module-cache

> This mapping serves as a cache of all modules that have been previously imported.\
> So if foo.bar.baz was previously imported, sys.modules will contain entries for foo, foo.bar, and foo.bar.baz.\
> If the module name is missing, Python will continue searching for the module.

----

https://docs.python.org/3/reference/import.html#finders-and-loaders

La façon exacte dont python importe (le python import protocol) a l'air assez complexe, et customisable. Je ne regarde pas en détail, mais voici quelques mots-clés :

- https://docs.python.org/3/glossary.html#term-finder
- https://docs.python.org/3/glossary.html#term-loader
- https://docs.python.org/3/glossary.html#term-importer
- https://docs.python.org/3/glossary.html#term-import-path
- https://docs.python.org/3/glossary.html#term-path-based-finder

> If the named module is not found in sys.modules, then Python’s import protocol is invoked to find and load the module.\
> This protocol consists of two conceptual objects, finders and loaders.\

Par exemple, python propose un importer par défaut qui sait charger les builtins modules, et un finder qui sait rechercher dans des chemins (comme le sys.path, j'imagine) ce qu'il doit importer.

https://docs.python.org/3/reference/import.html#the-path-based-finder

Pareil, je rentre pas dans les détails, mais en deux mots, le modèle selon lequel on va rechercher les fully-qualified module names depuis une liste de folders (le `sys.path`) est une simplification, python sait fait des choes plus compliquées.

Au sujet du `sys.path`, justement :

> sys.path contains a list of strings providing search locations for modules and packages.\
> It is initialized from the PYTHONPATH environment variable and various other installation- and implementation-specific defaults.

## les attributs des modules

https://docs.python.org/3/reference/import.html#import-related-module-attributes

Cette section (au nom très bien adapté : _Import-related module attributes_) contient des descriptions de plein d'attributs intéressants :

- `__name__`
- `__package__`
- `__path__`
- `__file__`

## imports relatifs

https://docs.python.org/3/reference/import.html#package-relative-imports

Une notion importante, mais une sous-section somme toute assez courte.

## le module main

https://docs.python.org/3/reference/import.html#main-spec

En gros, le main (le script principal, plutôt) est considéré comme un module uniquement si on a utilisé `python -m`.

Dans les autres cas, (notamment si on a exécuté un script à partir d'un fichier source, ce qui est le cas le plus courant, ou encore si on a invoqué l'interpréteur avec l'option `-c`), le main n'est pas un module, et notamment pas importable.

## import des submodules

https://docs.python.org/3/reference/import.html#submodules

En très gros, si on :

```python
import spam.foo
```

Alors :
- le module `spam` est chargé, et notamment : `spam/__init__.py` est exécuté
- le module `spam.foo` est chargé, et notamment : `spam/foo.py` (ou `spam/foo/__init__.py`) est exécuté
- le module `spam` se voit ajouté une nouvelle variable (ou plutôt un nouveau top-level object) `foo`, de sorte que la variable `spam.foo` réference le module `spam.foo`

Le **keypoint**, c'est que quand on importe un sous-module, on exécute tous ses modules parents. J'ai fait [une POC sur le sujet](https://github.com/phidra/pocs/tree/d2e187110a97ae06a74986758b18284bfd9d62df/python/import/import06_importing_packages_executes_its_init).
