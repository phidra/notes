# Notes déstructurées sur le packaging python issues de MULTIPLES sources

- **url** = multiples
- **type** = multiple docs
- **auteur** = multiples
- **date de publication** = multiples
- **source** = multiples
- **tags** = language>python ; topic>packaging ; level>beginner

**TL;DR** : agrégats de notes sur le packaging (à la base, dans un contexte où je cherchais à packager un module python codé en C).

* [Notes déstructurées sur le packaging python issues de MULTIPLES sources](#notes-déstructurées-sur-le-packaging-python-issues-de-multiples-sources)
   * [Packaging Python Projects](#packaging-python-projects)
   * [Packaging binary extensions](#packaging-binary-extensions)
   * [Packaging and distributing projects](#packaging-and-distributing-projects)
   * [Building and Distributing Packages with Setuptools](#building-and-distributing-packages-with-setuptools)

## Packaging Python Projects

**url** = https://packaging.python.org/en/latest/tutorials/packaging-projects/

https://packaging.python.org/en/latest/key_projects/#setuptools :

> setuptools is a collection of enhancements to the Python distutils that allow you to more easily build and distribute Python distributions, especially ones that have dependencies on other packages.

NdM : par exemple, une alternative est [flit](https://flit.readthedocs.io/en/latest/) ; [poetry](https://python-poetry.org/) semble également être une alternative à setuptools :

----

https://packaging.python.org/en/latest/key_projects/#distutils :

> Due to the challenges of maintaining a packaging system [...], direct usage of distutils is now actively discouraged, with setuptools being the preferred replacement.

----

https://packaging.python.org/en/latest/tutorials/packaging-projects/ : setup.py et pyproject.toml peuvent utiliser setuptools (recommandé), ou bien un autre outil ; dit autrement, setup.py n'est PAS marié à setuptools.

> build-system.requires gives a list of packages that are needed to build your package. Listing something here will only make it available during the build, not after it is installed.

Ce qui précède peut-être intéressant pour p.ex. utiliser pybind11 pour builder le package, mais ne pas le distribuer pour utiliser le package buildé.

----

https://packaging.python.org/en/latest/tutorials/packaging-projects/#configuring-metadata : c'est très intéressant : setup.cfg et setup.py ne s'opposent PAS, ils se complètent :

> Static metadata (setup.cfg) should be preferred. Dynamic metadata (setup.py) should be used only as an escape hatch when absolutely necessary.
> Any items that are dynamic or determined at install-time, as well as extension modules or extensions to setuptools, need to go into setup.py.
> setup.py used to be required, but can be omitted with newer versions of setuptools and pip.

Même si je peux n'avoir que setup.cfg sans setup.py, le setup.py semble inévitable pour packager une extension C.

----

https://packaging.python.org/en/latest/key_projects/#build

On dirait que [build](https://pypa-build.readthedocs.io/en/stable/index.html) est l'outil qui remplace `python setup.py sdist / bdist_wheel`.

----

https://packaging.python.org/en/latest/tutorials/packaging-projects/#generating-distribution-archives :

> Newer pip versions preferentially install built distributions, but will fall back to source archives if needed.
> You should always upload a source archive and provide built archives for the platforms your project is compatible with.
> In this case, our example package is compatible with Python on any platform so only one built distribution is needed.

Plusieurs axes de variations d'un package buildé :

- axe 1 = source vs. built (et dans le cas particulier de pure-python — i.e. si pas de C-extension — elles contiennent sans doute quasi la même chose)
- axe 2 = platform sur laquelle la built-archive peut être installée (dans le cas pure-python, c'est compatible partout)

## Packaging binary extensions

**url** = https://packaging.python.org/en/latest/guides/packaging-binary-extensions/

Attention : la source est ancienne (dernière revue = 2013)

3 raisons de faire une C extension :

- accélérer du code (idéalement, on écrit le code en C, et également une version équivalente en python, pour servir de fallback)
- wrapper du code C
- accéder à des choses bas-niveau, inaccessible depuis python : _Through platform specific code, extension modules may achieve things that aren’t possible in pure Python code._


Il y a des optimisations possibles vis-à-vis du GIL :

> One particularly notable feature of C extensions is that, when they don’t need to call back into the interpreter runtime, they can release CPython’s global interpreter lock around long-running operations (regardless of whether those operations are CPU or IO bound).

Inconvénient d'utiliser une C extension :

> The main disadvantage of using binary extensions is the fact that it makes subsequent distribution of the software more difficult. [...] One of the advantages of using Python is that it is largely cross platform, and the languages used to write extension modules [...] typically require that custom binaries be created for different platforms.

Un autre = soit l'utilisateur devra compiler l'extension à l'installation, soit quelqu'un doit avoir publié un binaire compatible avec la machine de l'utilisateur :

> binary extensions require that end users be able to either build them from source, or else that someone publish pre-built binaries for common platforms

Un autre pas très grave pour moi pour le moment, mais à garder en tête = marche surtout pour CPython :

> binary extensions often will not work correctly with alternative interpreters such as PyPy, IronPython or Jython

On en revient à l'ABI C :

> The C ABI (Application Binary Interface) is a common standard for sharing functionality between multiple applications.

Liste d'alternatives à utiliser l'API C python directement (i.e. liste des third-party tools wrappant l'API C python) = https://packaging.python.org/en/latest/guides/packaging-binary-extensions/#alternatives-to-handcoded-wrapper-modules

Bonne pratique = si on distribue une extension C, il faut builder des binaires pour toutes les versions de python et plate-formes supportées :

> If you plan to distribute your extension, you should provide wheels for all the platforms you intend to support. For most extensions, this is at least one package per Python version times the number of OS and architectures you support. These are usually built on continuous integration (CI) systems.

`manylinux` semble être une image docker à destination des gens qui package des extensions C, qui leur permet de builder facilement des binaires python à distribuer :

> Linux binaries must use a sufficiently old glibc to be compatible with older distributions.  The manylinux Docker images provide a build environment with a glibc old enough to support most current Linux distributions on common architectures.

## Packaging and distributing projects

Le fichier `setup.py` a deux usages :

- configurer le building du projet
- servir de main pour exécuter des commandes liées au packaging

> setup.py serves two primary functions:
>
> - It’s the file where various aspects of your project are configured. The primary feature of setup.py is that it contains a global setup() function. The keyword arguments to this function are how specific details of your project are defined. The most relevant arguments are explained in the section below.
> - It’s the command line interface for running various commands that relate to packaging tasks.

La commande suivante est utile pour lister les commandes dispos avec `setup.py`, **y compris celles ajoutées programmatiquement dans le script** :

```sh
python setup.py --help-commands
```

Bonne pratique = "dupliquer" le nom du projet :

```
/path/to/myproject/myproject/somecode.py
         │        │
         │        └ sous-répertoire du même nom que mon projet
         └ répertoire du projet
```

> Although it’s not required, the most common practice is to include your Python modules and packages under a single top-level package that has the same name as your project, or something very close.

Quelques mots-clés utiles de la fonction `setup(...)` :

- `packages` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#packages)) permet de configurer les packages du repo à intégrer au package publié
- `py_modules` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#py-modules)) permet de choisir les modules isolés (single-file, et ne faisant pas partie d'un package) à intégrer au package publié
- `install_requires` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#install-requires)) _is a setuptools setup.py keyword that should be used to specify what a project minimally needs to run correctly. When the project is installed by pip, this is the specification that is used to install its dependencies._
- `python_requires` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#python-requires)) permet d'indiquer que le projet ne tourne que sur certaines versions de python.
- `package_data` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#package-data)) permet d'intégrer des fichiers de données au package publié.
- `data_files` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#data-files)) semble correspondre à un sous-cas de package_data, dans lequel les fichiers de données ne sont PAS dans un package python du repo (mais dans un autre répertoire).
- `entry_points > console_scripts` ([source](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#entry-points)) permet de définir des "binaires" (commandes qui seront dans le PATH) qui seront installés quand on installera le package publié

## Building and Distributing Packages with Setuptools

**url** = https://setuptools.pypa.io/en/latest/userguide/index.html

Cette page bien un guide sur SETUPTOOLS (donc pas sur le building/packaging python en général : mais bien sur un outil en particulier = setuptools)

Le métier de setuptools est de builder des packages :

> Setuptools is a collection of enhancements to the Python distutils that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages.

Ça été le cas historiquement, mais maintenant (suite à [la PEP 517](https://www.python.org/dev/peps/pep-0517/#build-requirements)), le build-system n'est PLUS automatiquement setuptools ; le pyproject.toml indique quel build-system utiliser :

> The landscape of Python packaging is shifting and Setuptools has evolved to only provide backend support, no longer being the de-facto packaging tool in the market. Every python package must provide a pyproject.toml and specify the backend (build system) it wants to use. The distribution can then be generated with whatever tool that provides a build sdist-like functionality


Contenu minimal du pyproject.toml pour utiliser setuptools comme build-system :
```toml
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

Le reste du guide indique comment builder un package avec setuptools.
