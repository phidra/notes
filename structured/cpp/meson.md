Quelques exemples valant mieux qu'un long fichier de notes : ne pas hésiter à aller voir mes POCs sur meson.

- [Notes à l'utilisation](#notes-à-lutilisation)
   * [Utilisation basique](#utilisation-basique)
   * [Connaître les différentes options de paramétrage du build](#connaître-les-différentes-options-de-paramétrage-du-build)
   * [compile_commands.json](#compile_commandsjson)
   * [HOWTO débugger](#howto-débugger)
- [Gestion des dépendances externes avec meson wrap](#gestion-des-dépendances-externes-avec-meson-wrap)
   * [exemple concret](#exemple-concret)
   * [gitignore](#gitignore)
- [Installation](#installation)
   * [Sample project](#sample-project)
- [Pourquoi meson plutôt que cmake ou make](#pourquoi-meson-plutôt-que-cmake-ou-make)
- [Notes vrac à la lecture de la doc](#notes-vrac-à-la-lecture-de-la-doc)


# Notes à l'utilisation

## Utilisation basique

Une fois qu'on a écrit un script `meson.build` :

```
meson setup builddir/
meson compile -C builddir/
meson test -C builddir/
```

## Connaître les différentes options de paramétrage du build

Pour visualiser les différentes options de configuration pour le projet, leur valeur actuelle, leurs valeurs possibles, et leur description :

```
meson configure builddir/
```

## compile_commands.json

À la différence de cmake, la génération d'un `compile_commands.json` est active par défaut (il faut éventuellement en faire un lien symbolique à la racine du projet pour les IDE)

## HOWTO débugger

- pour débugger la commande de compilation d'un fichier, aller regarder `compile_commands.json`
- pour débugger la commande de build, on peut lancer un `ninja -v` (précédé éventuellement d'un `ninja -t clean`)

# Gestion des dépendances externes avec meson wrap

**TL;DR** : meson intègre un gestionnaire de packages = `meson wrap`.

## exemple concret

```sh
# lister les packages installables avec wrap :
meson wrap list

# installer le package sdl2 :
mkdir subprojects
meson wrap install sdl2
```

^ installe `sdl2` dans le répertoire `subprojects` par défaut (i.e. pas eu besoin de préciser la localisation)

Le contenu du fichier `subprojects/sdl2.wrap` décrit simplement où trouver les sources :

```
[wrap-file]
directory = SDL2-2.28.1
source_url = https://github.com/libsdl-org/SDL/releases/download/release-2.28.1/SDL2-2.28.1.tar.gz
source_filename = SDL2-2.28.1.tar.gz
source_hash = 4977ceba5c0054dbe6c2f114641aced43ce3bf2b41ea64b6a372d6ba129cb15d
patch_filename = sdl2_2.28.1-2_patch.zip
patch_url = https://wrapdb.mesonbuild.com/v2/sdl2_2.28.1-2/get_patch
patch_hash = 2dd332226ba2a4373c6d4eb29fa915e9d5414cf7bb9fa2e4a5ef3b16a06e2736
source_fallback_url = https://github.com/mesonbuild/wrapdb/releases/download/sdl2_2.28.1-2/SDL2-2.28.1.tar.gz
wrapdb_version = 2.28.1-2
[provide]
sdl2 = sdl2_dep
sdl2main = sdl2main_dep
sdl2_test = sdl2_test_dep
```

Les sources sont downloadées au build-time, dans un sous-répertoire de `subprojects`.

## gitignore

Typiquement, on voudra ignorer les packages installés dans `subprojects`, mais on voudra commiter leur spec wrap :

```sh
# on ignore subprojects :
subprojects/*

# mais on commit les spec wrap :
!subprojects/*.wrap
```

# Installation

Versions dispos dans les repos APT :

- ubuntu 22.04 = `0.61.2-1`
- ubuntu 20.04 = `0.53.2-2ubuntu2`
- debian bookworm = `1.0.1-5`

La version seuil à partir de laquelle la commande `meson compile` est dispo [semble être](https://mesonbuild.com/Commands.html#compile) la `0.54`.

Le tool est aussi pip(x) installable :

```sh
pipx install meson
# installed package meson 1.2.2, installed using Python 3.8.10
```

## Sample project

La commande `meson init --name pouet --build` créé un sample de projet dans le répertoire courant. Out-of-the-box, sur ce projet sample, on a :

- un meson.build = le script qui paramètre le build :
    ```
    project('pouet', 'c', version : '0.1', default_options : ['warning_level=3'])
    exe = executable('pouet', 'pouet.c', install : true)
    test('basic', exe)
    ```
- un `compile_commands.json`
- des tests (lançables avec `meson test`, ils se contentent de lancer l'exécutable)
- des métadonnées au format json

# Pourquoi meson plutôt que cmake ou make

**TL;DR** : les killers features sont un outil moderne, simple, et bien documenté.

- plus simple que make et cmake
- (plus rapide aussi, mais c'est marginal, et pas limitant)
- on trouve des témoignages positifs de transition de cmake à meson
- make = syntaxe plus difficile + plus compliqué à maintenir
- semble moins utilisé que cmake, mais [massivement utilisé tout de même](https://mesonbuild.com/Users.html) (notamment par des gros projets comme Gnome  (Gtk, Nautilus, etc.), QEMU, PostgreSQL, ...) :
- syntaxe proche de python -> meilleure que celle de cmake / make
- DSL volontairement non turing-complet, pour limiter la complexité
- gestion correcte des espaces
- meilleures typing des données : [lien](https://en.wikipedia.org/wiki/Meson_(software)#Language)
- le projet a 10 ans, mais la première release stable date d'août 2023
- modernité :
    > A stated goal of Meson is to facilitate modern development practices. As such, Meson knows how to do unity builds, build with test coverage, link time optimization etc without the programmer having to write support for this.
- killer feature de meson = excellente documentation

Ce qu'en dit [la doc de meson](https://mesonbuild.com/Comparisons.html) :
- +++ The fastest build system see measurements
- +++ user friendly, designed to be as invisible to the developer as possible
- +++ native support for modern tools (precompiled headers, coverage, Valgrind etc).
- +++ Not Turing complete so build definition files are easy to read and understand.
- --- Relatively new so it does not have a large user base yet, and may thus contain some unknown bugs.

Ce qu'en dit [https://gms.tf/the-rise-of-meson.html](cet article) : certes moins utilisé que cmake, mais tout de même très utilisé, et surtout de plus en plus ; c'est la version "moderne" de cmake.

[Cet autre témoignage](https://niekbouman.medium.com/c-build-systems-our-transition-from-cmake-to-meson-2c043e93822f) relate la transition de cmake à meson, quelques notes en vrac :

- scripts cmake = assemblés avec des bouts de ficelle
- old cmake vs. new cmake
- l'abondance des posts sur "Modern cmake" montre les lacunes de la doc (et je suis d'accord avec lui)
- à l'inverse, doc et design de meson "feels right"

> why should I invest more time to become an expert in a system (CMake) that I anyway appreciate less the more I read about it ?

^ je suis _vraiment_ d'accord avec lui

Il a des requirements un poil inhabituels (mais rien de tuant non plus) :

- multi-compiler
- compiler différemment certaines libs en fonction des options
- utiliser du code auto-généré (capnproto)

> all these were easy to accomplish (for the third I use a little helper Python script), and, maybe even more importantly, in such a way that I fully understand what is going on under the hood.

^ meson est _compréhensible_

# Notes vrac à la lecture de la doc

NDM : la doc est super bien, et il y a beaucoup de petites choses utiles disséminées à-droite/à-gauche.

https://mesonbuild.com/Build-targets.html

^ builder une lib + l'utiliser dans un exécutable (à la rust).

https://mesonbuild.com/Include-directories.html

^ organisation des sources = séparer les headers publics et les sources de sa librairie (cf. mes POCs sur le sujet ou [ce post reddit](https://www.reddit.com/r/cpp_questions/comments/j5bkqj/place_header_files_in_separate_folder_or_not/g7rn53y/))

https://mesonbuild.com/Reference-manual_functions.html#project

^ version du standard = définir une default option au projet

https://mesonbuild.com/Configuration.html

^ on peut remplir un template d'include, qui contient des variables de configuration.

https://mesonbuild.com/External-commands.html

> Meson will not pass the command to the shell, so any command lines that try to use things such as environment variables, backticks or pipelines will not work. If you require shell semantics, write your command into a script file and call that with run_command.

https://mesonbuild.com/Precompiled-headers.html

> In Meson, precompiled header files are always per-target. That is, the given precompiled header is used when compiling every single file in the target.

^ bon à savoir si un jour j'utilise un precompiled header.

https://mesonbuild.com/Unity-builds.html

^ Unity builds = tout merger au sein d'une unique unité de compilation (pour avoir la LTO gratuite)

- avantage = code plus efficace + optimisations + compilation plus rapide
- inconvénient = la compilation incrémentale devient impossible

> The unity_size option allows to specify the number of source files included per unity file. The default is 4. Having more source files per unity file will speed up full builds, but slow down incremental builds. To get only one unity file per build target, you can use a very big number for unity_size.

https://mesonbuild.com/Feature-autodetection.html

ccache est utilisé par défaut (désactivable au premier lancement de meson)

Il y a des features pour facilement s'intégrer avec les tools calculant le coverage.

https://mesonbuild.com/Generating-sources.html

> Each target that depends on a generated header should add that header to its sources, as seen above with libfoo and myexe. This is because there is no way for Meson or the backend to know that myexe depends on foo.h just because libfoo does, it could be a private header.

^ headers publics et privés (pour mieux comprendre les keywords équivalents de cmake)

https://mesonbuild.com/Unit-tests.html

^ il y a un test launcher intégré, avec déjà plein plein de features avancées.

https://mesonbuild.com/Localisation.html

^ parmi les (trop ?) nombreuses features de meson : intégration des fichiers `.po` pour la localisation.

https://mesonbuild.com/Build-options.html

^ on peut configurer le build avec des options dans un fichier `meson_options.txt`.

E.g. une config indique si telle librairie est required ou non.

https://mesonbuild.com/Subprojects.html

> Some platforms do not provide a native packaging system. In these cases it is common to bundle all third party libraries in your source tree. This is usually frowned upon because it makes it hard to add these kinds of projects into e.g. those Linux distributions that forbid bundled libraries.
>
> Meson tries to solve this problem by making it extremely easy to provide both at the same time. The way this is done is that Meson allows you to take any other Meson project and make it a part of your build without (in the best case) any changes to its Meson setup. It becomes a transparent part of the project.
>
> It should be noted that this is only guaranteed to work for subprojects that are built with Meson. The reason is the simple fact that there is no possible way to do this reliably with mixed build systems. Because of this, only Meson subprojects are described here. CMake based subprojects are also supported but not guaranteed to work.

^ le besoin est de vendor des sous-projets, tout en laissant la possibilité de ne pas vendor.

Donc en quelques sortes, le vendored subproject est un FALLBACK si la dépendance n'a pas pu être résolue autrement.

https://mesonbuild.com/Code-formatting.html

> When clang-format is installed and a .clang-format file is found at the main project's root source directory, Meson automatically adds a clang-format target that reformat all C and C++ files

^ meson fabrique une target permettant de reformater le projet.

https://mesonbuild.com/Build-system-converters.html

^ si besoin, il y a un tool pour migrer de cmake à meson

https://mesonbuild.com/Configuring-a-build-directory.html

^ `meson configure` pour changer les paramètres du build.

https://mesonbuild.com/Run-targets.html

> Sometimes you need to have a target that just runs an external command. As an example you might have a build target that reformats your source code, runs cppcheck or something similar. In Meson this is accomplished with a so called run target.

^ en "compilant" une run-target, en fait on exécute une commande.

https://mesonbuild.com/Project-templates.html

^ template de projet.

https://mesonbuild.com/Dependencies.html

- en déclarant une dépendance externe, on peut utiliser une lib (NDM : d'après ma compréhension des choses, un .a ou un .so ; et non des sources)
- la gestion des flags de compilation et link sont gérés pour nous par meson
- de base, tout semble passer par `pkg-config` (et d'ailleurs, meson expose tout ce que pkg-config connaît) :
    > The dependency detector works with all libraries that provide a pkg-config file.
- mais meson semble capable de gérer les dépendances autrement :
    > Unfortunately several packages don't provide pkg-config files. Meson has autodetection support for some of these, and they are described later in this page.
- si besoin, on peut accéder à des infos (typiquement, celles fournies par `pkg-config`, mais pas que) sur la dépendance
- on peut définir une dépendance custom (sur une de ses libs, p.ex.), en pointant vers ses includes et la lib statique à linker
    - EDIT : c'est ce que j'utilise pour splitter mon projet en composants.
- on peut utiliser un subproject comme fallback d'une dépendance absente :
    > Many platforms do not provide a system package manager. On these systems dependencies must be compiled from source.
    - ^ au lieu de linker directement avec la `lib.a`, on commence par compiler les sources pour produire la `lib.a`, puis on linke contre elle
    > Meson's subprojects make it simple to use system dependencies when they are available and to build dependencies manually when they are not.
    - ^ utilisation de la lib system si elle est dispo, avec build manuel en fallback
- il y a plusieurs façons de résoudre une dépendance ; notamment [via cmake](https://mesonbuild.com/Dependencies.html#cmake) :
    > Meson can use the CMake find_package() function to detect dependencies with the builtin Find<NAME>.cmake modules and exported project configurations (usually in /usr/lib/cmake).
- tool particulier pour une dépendance :
    - https://mesonbuild.com/Dependencies.html#config-tool
    - certaines librairies / outils ne sont pas gérés avec pkg-config, mais avec leur propre outil (e.g. llvm-config)
    - meson sait gérer ces outils, et on peut donc déclarer la dépendance normalement : llvm_dep = dependency('llvm', version : '>=4.0')
