# A most vexing parse, but for Python packaging

- **url** = https://blog.yossarian.net/2022/05/09/A-most-vexing-parse-but-for-Python-packaging
- **type** = post
- **auteur** = [William WOODRUFF aka yossarian](https://yossarian.net/about/) = dev open-source
- **date de publication** = 2022-05-09
- **source** = [son blog](https://blog.yossarian.net/)
- **tags** = language>python ; topic>packaging ; level>beginner


* [A most vexing parse, but for Python packaging](#a-most-vexing-parse-but-for-python-packaging)
   * [sujet de l'article (mais pas de ces notes) = le vexing parse](#sujet-de-larticle-mais-pas-de-ces-notes--le-vexing-parse)
   * [organisation du packaging python, sdist vs. bdist](#organisation-du-packaging-python-sdist-vs-bdist)
   * [critères de choix d'une distribution plutôt qu'une autre](#critères-de-choix-dune-distribution-plutôt-quune-autre)
   * [Simple Repository API](#simple-repository-api)
   * [Noms des fichiers wheels](#noms-des-fichiers-wheels)
      * [PEP 427 pour les noms des wheels](#pep-427-pour-les-noms-des-wheels)
      * [PEP 425 pour les compatibility-tags](#pep-425-pour-les-compatibility-tags)
      * [PEP 625 pour les noms des fichiers sdist](#pep-625-pour-les-noms-des-fichiers-sdist)
      * [manylinux ?](#manylinux-)

**TL;DR** : le sujet du post expose un problème de parsing des noms des package python, mais je l'annote surtout car il donne un bon résumé de sujets liés au packaging python + il est une bonne occasion pour moi de regarder de plus près certains sujets, comme les _compatibility tags_ de la [PEP 425](https://peps.python.org/pep-0425/)

## sujet de l'article (mais pas de ces notes) = le vexing parse

En deux mots, concernant le sujet de l'article = le vexing parse :
- il commence par rappeler le problème du most-vexing parse en C++ =
    ```cpp
    WidgetFrobulator frob(Widget());
    ```
- le code ci-dessus peut soit déclarer une fonction `frob` qui prend un argument = une fonction anonyme sans argument renvoyant un widget, soit instancier un `frob` de type `WidgetFrobulator` en lui passant un objet default-constructed de type `Widget`
- côté packaging python, on a un problème de parsing un peu similaire :
    - en gros, quand le package installer ne trouve pas de bdist compatible, il fallback sur une sdist
    - mais problème : certains (anciens) pacakges ont un nom ambigü, avec un tiret en fin de version = `1.0.2-2`
    - du coup, le parsing échoue...

## organisation du packaging python, sdist vs. bdist

Mais ce qui m'intéresse, c'est surtout ce qu'il dit sur le packaging python, qui commence à **Python packaging: a primer** :

> a project can have zero or more releases, each of which can, in turn, have zero or more distributions.

Organisation :

- **Projet** (e.g. `requests`) = _Projects correspond to human-readable identifiers in the global Python packaging namespace: requests, cryptography, ..._
- **Release** (e.g. `1.2.0`) = _Releases correspond to the human notion of “versions”_
- **Distribution** (e.g. `numpy-1.22.4-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl`) = _Finally, distributions correspond to the installable assets for a release. These contain the actual code for the release, and come in a surprisingly large number of formats._

Le point qui m'intéresse ici : comment distribuer le code d'une release d'un projet ? La réponse n'est en fait pas triviale, il y a plein de façons de faire ça (p.ex. on peut même fabriquer une distribution au format RPM !). Mais dans le cadre du post, on se limite aux **sdist** et **bdist** :

**sdist** = une archive qui contient les sources + un `setup.py` (ou un équivalent plus récent) qui encodera comment installer le package à partir des sources :

- inconvénient = pour installer le package, on exécute du code arbitraire du `setup.py`
- inconvénient = du coup, certaines dépendances peuvent très bien être exprimées dynamiquement par `setup.py` (et il est donc difficile de les connaître à l'avance)
- inconvénient = par ailleurs, on peut avoir besoin d'un compilateur sur la machine de l'utilisateur final si la distribution utilise des extensions compilées

**bdist** = c'est une archive qui contient uniquement des fichiers prêts à l'emploi (il suffit de les copier+coller sur la machine de l'utilisateur final) + des fichiers déclaratifs pour exprimer les dépendances :

- avantage = pas besoin de `setup.py` ni d'exécution de code arbitraire (du coup, on connaît directement toutes les dépendances vu qu'elles sont déclarées dans les metadata de la bdist)
- avantage = pas besoin de compilo
- avantage = plus rapide à installer
- inconvénient = en cas d'extension C/C++, le mainteneur doit créer une bdist pour toutes les combinaisons finales possibles : machine, libc, ABI de python, etc.
- inconvénient = si l'utilisateur final a une combinaison (machine + libc + version de ptyhon) pour laquelle aucune bdist n'existe, pip doit se rabattre sur une sdist
- les bdist sont préférées aux sdist pour ces raisons

À l'installation d'une bdist, pip se débrouille pour trouver la "bonne" bdist à utiliser au vu de la machine de l'utilisateur final. Dans la suite du post et de mes notes, on se concentre sur un type particulier de bdist = les **wheels**.

## critères de choix d'une distribution plutôt qu'une autre

> However, to actually choose a wheel over its corresponding source distribution for a release, a package installer must go through a search process.

C'est le coeur du post = comment pip choisit la bonne bdist à utiliser ? Le truc intéressant (et qui est la raison pour laquelle l'article m'intéresse, vu que c'est une question que je me pose) , c'est qu'il résume ce qui joue sur la compatibilité d'une bdist avec l'ordinateur d'un utilisateur final :

- **platform** = windows vs. linux
- (**python version** = certaines wheels sont conçues pour ne fonctionner qu'avec une version spécifique de python, e.g. 3.8 ; NdM = mais je pense que c'est pas lié aux questions de compatibilité binaires qui m'intéressent ici)
- **python implementation** = certaines wheels sont conçues pour ne fonctionner qu'avec une implémentation spécifique de python e.g. cpython ou pypy (si une wheel est compatible avec toutes les versions de python, elle peut indiquer le wildcard `py` pour dire qu'elle est implementation-agnostic, i.e. elle n'utilise pas de features qui sont propres à une implémentation particulière)
- **python ABI** = la wheel cible non seulement une implémentation spécifique de python, mais également une ABI spécifique de cette implémentation, e.g. `cp37` (ici aussi il y a un wildcard `abi3` pour dire _"compatible with the CPython 3 universal ABI"_)

## Simple Repository API

Pour une release d'un projet, pour lister facilement les wheels disponibles sur un serveur sans avoir à télécharger chaque fichier, on a la [PEP 503 = Simple Repository API](https://peps.python.org/pep-0503/) :

- la racine `/` d'un serveur implémentant la **Simple Repository API** doit être une liste de liens (un lien par projet), dont le href pointe sur l'URL du projet
    - exemple = https://pypi.org/simple/
    - (attention, la page est lourde à charger, elle fait 26 Mio et contient 377k liens : `curl https://pypi.org/simple/ | grep -c href`)
- chaque sous-page de projet (`/myproject`) doit être une liste de liens (un lien par fichier), dont le href pointe sur l'URL permettant de télécharger ce fichier, et dont le nom est le nom du fichier à télécharger
    - exemple 1 = https://pypi.org/simple/numpy/
    ```sh
    curl -s https://pypi.org/simple/numpy/ | pandoc -f html -t plain | grep numpy-1.22.4
    # numpy-1.22.4-cp310-cp310-macosx_10_14_x86_64.whl
    # numpy-1.22.4-cp310-cp310-macosx_10_15_x86_64.whl
    # numpy-1.22.4-cp310-cp310-macosx_11_0_arm64.whl
    # numpy-1.22.4-cp310-cp310-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
    # numpy-1.22.4-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
    # numpy-1.22.4-cp310-cp310-win32.whl
    # numpy-1.22.4-cp310-cp310-win_amd64.whl
    # numpy-1.22.4-cp38-cp38-macosx_10_15_x86_64.whl
    # numpy-1.22.4-cp38-cp38-macosx_11_0_arm64.whl
    # numpy-1.22.4-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
    # numpy-1.22.4-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
    # numpy-1.22.4-cp38-cp38-win32.whl
    # numpy-1.22.4-cp38-cp38-win_amd64.whl
    # numpy-1.22.4-cp39-cp39-macosx_10_14_x86_64.whl
    # numpy-1.22.4-cp39-cp39-macosx_10_15_x86_64.whl
    # numpy-1.22.4-cp39-cp39-macosx_11_0_arm64.whl
    # numpy-1.22.4-cp39-cp39-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
    # numpy-1.22.4-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
    # numpy-1.22.4-cp39-cp39-win32.whl
    # numpy-1.22.4-cp39-cp39-win_amd64.whl
    # numpy-1.22.4-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
    # numpy-1.22.4.zip
    ```
    - exemple 2 = https://pypi.org/simple/pip-audit/
    ```sh
    curl -s https://pypi.org/simple/pip-audit/ | pandoc -f html -t plain | grep pip_audit-2.3.1
    # pip_audit-2.3.1-py3-none-any.whl
    # pip_audit-2.3.1.tar.gz
    ```


Ici, numpy a plein de wheels, et une unique sdist `numpy-1.22.4.zip` (dont l'archive contient plein de fichiers codés en C, alors qu'une wheel décompressée n'en a presque pas)

## Noms des fichiers wheels

### PEP 427 pour les noms des wheels

C'est le nom de la wheel qui encode les informations permettant à un package-installer comme pip de savoir si la wheel est compatible avec le système de l'utilisateur final. Ce nom est normalisé par un bout de la [PEP 427](https://peps.python.org/pep-0427/#file-format) normalisant les wheels :

```
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl
```
Prenons quelques exemples issus de `numpy` et `pip-audit` :

```sh
numpy-1.22.4-pp38-pypy38_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
# DISTRIBUTION = numpy
# VERSION      = 1.22.4
# BUILD        = None
# PYTHON       = pp38
# ABI          = pypy38_pp73
# PLATFORM     = manylinux_2_17_x86_64.manylinux2014_x86_64

numpy-1.22.4-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
# DISTRIBUTION = numpy
# VERSION      = 1.22.4
# BUILD        = None
# PYTHON       = cp38
# ABI          = cp38
# PLATFORM     = manylinux_2_17_x86_64.manylinux2014_x86_64

numpy-1.22.4-cp38-cp38-macosx_10_15_x86_64.whl
# DISTRIBUTION = numpy
# VERSION      = 1.22.4
# BUILD        = None
# PYTHON       = cp38
# ABI          = cp38
# PLATFORM     = macosx_10_15_x86_64

pip_audit-2.3.1-py3-none-any.whl
# DISTRIBUTION = pip_audit
# VERSION = 2.3.1
# BUILD        = None
# PYTHON = py3 (la wheel est donc utilisable avec n'importe quel python3, ce qui est assez logique vu que c'est un package pure-python)
# ABI = none  (c'est donc un package pure-python)
# PLATFORM = any (la wheel est donc installable sur n'importe quelle architecture, ce qui est assez logique vu que c'est un package pure-python)
```

Et côté sdist ? Il existe bien la [PEP 625](https://peps.python.org/pep-0625/) pour normaliser le nom des fichiers sdist, mais elle est encore en draft depuis sa création en juillet 2020

### PEP 425 pour les compatibility-tags

La [PEP 425](https://peps.python.org/pep-0425/) donne des détails sur les 3 tags de compatibilité :

- python
- abi
- platform

À quoi ces tags servent-ils ?

> The tags are used by installers to decide which built distribution (if any) to download from a list of potential built distributions
>
> This example list is for an installer running under CPython 3.3 on a linux_x86_64 system. It is in order from most-preferred to least-preferred :
>
> - cp33-cp33m-linux_x86_64
> - cp33-abi3-linux_x86_64
> - cp3-abi3-linux_x86_64
> - cp33-none-linux_x86_64*
> - cp3-none-linux_x86_64*
> - py33-none-linux_x86_64*
> - py3-none-linux_x86_64*
> - cp33-none-any
> - cp3-none-any
> - py33-none-any
> - py3-none-any
> - py32-none-any
> - py31-none-any
> - py30-none-any

**python** :

- indique l'implémentation nécessaire (cpython ? pypy ?) et sa version
- si on s'en fiche (e.g. dans le cas d'une distribution pure-python qui ne fait pas appel à des features implementation specific), on peut se contenter de `py3`
- NdM : quelle différence avec le tag d'ABI ? Je suppose que même un projet pure-python pourrait faire appel à du code cpython-only ? Par exemple, si le projet veut utiliser `dis` ou `sys.gettrace` qui utilisent des détails d'implémentations de cpython

**abi** :
- _The ABI tag indicates which Python ABI is required by any included extension modules._
- la différence est ici qu'une EXTENSION C/C++ a besoin d'une version de python (pour linker avec)
- ça va un peu plus loin que la version de python, p.ex. on peut dépendre de la lib python avec symboles de debug ?
- le joker est `abi3` (qui indique qu'on ne dépend que de l'ABI cpython stable) : _The CPython stable ABI is abi3 as in the shared library suffix._

**platform** :

C'est ce qui est renvoyé par :

```
>>> import distutils.util
>>> distutils.util.get_platform()
'linux-x86_64'
```

Je vois à mes exemples ci-dessus que ça peut être un tag assez complexe.

un complément d'info intéressant (même si je suppose que c'est une très mauvaise pratique) :

> Built distributions may be platform specific for reasons other than C extensions, such as by including a native executable invoked as a subprocess.

### PEP 625 pour les noms des fichiers sdist

En gros, pas encore de standard, mais [la PEP 625](https://peps.python.org/pep-0625/    ) vise à standardiser + il existe un standard de facto :

https://packaging.python.org/en/latest/specifications/source-distribution-format/#source-distribution-file-name

> The file name of a sdist is not currently standardised, although the de facto form is {name}-{version}.tar.gz, where {name} is the canonicalized form of the project name.

(à noter que ça ne semble pas être suivi par numpy, qui utilise un `.zip` plutôt qu'un `.tar.gz`, donc le standard n'est peut-être pas si standard que ça...)

### manylinux ?

Certaines wheels ont `manylinux` comme sous-partie du platform tag.

En gros, il s'agit [d'une image docker](https://github.com/pypa/manylinux), qui, si on l'utilise pour fabriquer notre wheel, produira des wheels compatibles avec plein de linux des utilisateurs. L'objectif est pour le mainteneur d'un package d'éviter d'avoir à fabriquer plein de wheels linux différentes (une pour unbuntu, une pour redhat, etc.).

En revanche, pour un manylinux donné, il faudra tout de même builder pour plusieurs archis différentes (x86_64, i686, aarch64, etc.), mais une seule wheel manylinux x86_64 pourra être utilisée sur tous les linux x86_64.

En résumé, ça résout une partie du problème de "j'ai trop de wheels à builder" = celle de l'OS spécifique (dans le cas linux, du moins).

Par ailleurs, j'ai l'impression dans mes exemples ci-dessus que le platform-tag peut en spécifier plusieurs : `manylinux_2_17_x86_64.manylinux2014_x86_64` semble être l'agréation de `manylinux_2_17_x86_64` et de `manylinux2014_x86_64`.


Le [rationale de la PEP 600](https://peps.python.org/pep-0600/#rationale) a des infos intéressantes :

> Python users appreciate it when PyPI has pre-compiled packages for their platform, because it makes installation fast and simple. But distributing pre-compiled binaries on Linux is challenging because of the diversity of Linux-based platforms. For example, Debian, Android, and Alpine all use the Linux kernel, but with radically different userspace libraries, which makes it difficult or impossible to create a single wheel that works on all three. This complexity has caused many previous discussions of Linux wheels to stall out.
>
> The “manylinux” project succeeded by adopting a strategy of ruthless pragmatism. We chose a large but tractable set of Linux platforms – specifically, mainstream glibc-based distributions like Debian, OpenSuSE, Ubuntu, RHEL, etc. – and then we did whatever it takes to make wheels that work across all these platforms.
>
> This approach requires many compromises. Manylinux wheels can only rely on external libraries that maintain a consistent ABI and are universally available across all these distributions, which in practice restricts them to a small set of core libraries like glibc and a few others. Wheels have to be built on carefully-chosen platforms of the oldest possible vintage, using a Python that is itself built in a carefully-chosen configuration. Other shared library dependencies have to be bundled into the wheel, which requires a complex process to avoid collisions between unrelated wheels.

Par ailleurs, on peut voir (p.ex. dans [la PEP 513](https://peps.python.org/pep-0513/) qui est la PEP d'un manylinux particulier) ce qu'il y a dans un manylinux = un jeu de librairies et de symboles très restreints, qu'on suppose présents sur tous les linux qui seront concernés :

> To be eligible for the manylinux1 platform tag, a Python wheel must therefore both (a) contain binary executables and compiled code that links only to libraries with SONAMEs included in the following list:
>
> - [...]
> - libncursesw.so.5
> - libgcc_s.so.1
> - libstdc++.so.6
> - libm.so.6
> - libdl.so.2
> - librt.so.1
> - libc.so.6
> - [...]
>
> On Debian-based systems, these libraries are provided by the packages
> - libncurses5 libgcc1 libstdc++6 libc6 libx11-6 libxext6
> - libxrender1 libice6 libsm6 libgl1-mesa-glx libglib2.0-0

L'idée est que les binaires embarqués dans la wheel ne doivent dépendre que de ces librairies.
