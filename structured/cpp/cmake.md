D'une façon générale, sur cmake, cf. mes nombreuses POCs sur le sujet, assorties de notes.

(même si je préfère meson, cmake semble être établi comme un stardard de fait...)

* [Connaître les options de config du build](#connaître-les-options-de-config-du-build)
* [PUBLIC vs PRIVATE vs INTERFACE](#public-vs-private-vs-interface)
   * [Résumé de ma compréhension pour PUBLIC ET PRIVATE](#résumé-de-ma-compréhension-pour-public-et-private)
   * [Et INTERFACE ?](#et-interface-)
   * [Guidelines d'utilisation](#guidelines-dutilisation)
   * [Les liens et extraits utiles](#les-liens-et-extraits-utiles)
* [Effective Modern CMake](#effective-modern-cmake)


# Connaître les options de config du build

Le contexte = je voulais savoir comment builder OpenTelemetry sans builder ses tests unitaires. Au final, la commande était :

```
cmake -DBUILD_TESTING=OFF ..
```

Mais comment pouvais-je avoir connaissance de cette option `BUILD_TESTING` ?

3 pistes de réponses :

- d'abord, si elle est bien faite, la doc du build le mentionnera
- on peut toujours aller voir le `CMakeLists.txt` (notamment grepper les instructions `option`), mais ça peut éventuellement être un peu complexe
- reste [ccmake](https://cmake.org/cmake/help/latest/manual/ccmake.1.html) qui fournit une GUI permettant de paramétrer le build, et qui liste donc implicitement les options de build
    - installation :
    ```
     sudo apt install cmake-curses-gui
    ```
    - utilisation = `ccmake .` après un cmake
    - `t` pour passer en mode avancé, et voir aussi les options cmake non-spécifiques au projet
    - la config semble enregistrée, i.e. on n'a plus qu'à builder après avoir lancé ccmake
    - cf. [cette réponse StackOverflow](https://stackoverflow.com/questions/1224627/cmake-ccmake-or-cmake/52757472#52757472) :
    > Usage: After you have generated configure files using `cmake ..` in build folder. Then do `ccmake ..` , to see all the options and paths which can be be modified by pressing Enter for editing. After editing press c again to configure. This will save the files in build folder with your edited settings. Then do make -j4 , for building the MakeFile.

# PUBLIC vs PRIVATE vs INTERFACE

## Résumé de ma compréhension pour PUBLIC ET PRIVATE

Lorsqu'on associe une propriété à une librairie `L`, les keywords `PUBLIC` et `PRIVATE` influent non pas sur la façon dont cmake va compiler `L` (qui aura de toutes façons toujours la propriété peu importe le keyword), mais plutôt sur la façon dont cmake compile un **utilisateur** de `L`, p.ex. un exécutable `E` utilisant la librairie.

Exemples de propriétés concernées :

- les répertoires à inclure pour compiler la lib, via [target_include_directories](https://cmake.org/cmake/help/latest/command/target_include_directories.html)
- les flags de compilation pour compiler la lib, via [target_compile_definitions](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html)
- les librairies dont `L` a besoin pour compiler, via [target_link_librairies](https://cmake.org/cmake/help/latest/command/target_link_libraries.html)

Prenons l'exemple d'un exécutable `E` qui utilise une librairie `L`, elle-même utilisant une dépendance `D` (e.g. une autre librairie). On aura donc quelque part dans le `CMakeLists.txt` de `L` quelque chose comme :

```cmake
target_include_directories(L "/path/to/headers/of/D")
target_link_libraries(L D)
```

Si on utilise plutôt la forme complète, la question est de savoir quelle est la différence entre les deux formes suivantes :

```cmake
# forme 1 = avec PUBLIC :
target_include_directories(L PUBLIC "/path/to/headers/of/D")
target_link_libraries(L PUBLIC D)

# forme 2 = avec PRIVATE :
target_include_directories(L PRIVATE "/path/to/headers/of/D")
target_link_libraries(L PRIVATE D)
```

Réponse : du point de vue de `L`, les deux formes suivantes seront **identiques** : `L` sera compilé de la même façon (en incluant les headers de `D`, et en liant `D`). Ce qui change, c'est la façon dont `E` sera compilé :

- avec `PRIVATE`, `E` sera compilé sans utiliser `D` (ni pour les headers, ni au link)
- avec `PUBLIC`, `E` sera compilé en utilisant `D` (pour les headers et pour le link)

## Et INTERFACE ?

Le cas `INTERFACE` est un peu particulier ; pour reprendre la mise en situation ci-dessus, si on définit :

```cmake
target_include_directories(L INTERFACE "/path/to/headers/of/D")
```

C'est que `L` n'a pas besoin des headers de `D` pour être compilée, mais que `E` en aura besoin (par exemple, si `L` est une librairie header-only).

Dans ce cas, `L` sera compilé **SANS** les includes, et `E` sera compilé **AVEC**.

En résumé, pour chacun des 3 keywords `PUBLIC+PRIVATE+INTERFACE`, on a un tableau à deux colonnes :

|             | `L` sera compilé avec la propriété  | `E` sera compilé avec la propriété  |
|-------------|-------------------------------------|-------------------------------------|
| `PUBLIC`    |                 OUI                 |                 OUI                 |
| `PRIVATE`   |                 OUI                 |                 NON                 |
| `INTERFACE` |                 NON                 |                 OUI                 |

## Guidelines d'utilisation

Du coup, on peut en déduire les guidelines suivantes (que je reprends d'une référence ci-dessous) :

- dans le cas général, on considère qu'une dépendance n'est utilisée que dans l'implémentation d'une librairie : elle doit être privée, et déclarée avec `target_link_libraries()` + le keyword `PRIVATE`
- mais si la dépendance est utilisée dans un header public de la librairie, elle doit être déclarée avec le keywoard `PUBLIC`
- et si la dépendance n'est pas utilisée par l'implémentation de la librairie, mais uniquement dans ses headers publics, elle doit être déclarée avec le keyword `INTERFACE`

## Les liens et extraits utiles

Je reporte ici plusieurs extraits de différents liens qui m'ont aidé à construire ma compréhension. Par ailleurs, voici quelques autres références utiles, que je n'ai pas annotées :

- https://cmake.org/pipermail/cmake/2016-May/063400.html
- https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library
- https://cmake.org/cmake/help/latest/command/target_sources.html


----

https://hsf-training.github.io/hsf-training-cmake-webpage/04-targets/index.html

- un mot sur la visibilité des properties des target (le cas typique, c'est le cas où la target est une librairie) :
    - `PRIVATE`   = les propriétés propres à la librairie (e.g. le cpp de cette librairie a besoin de C++14)
    - `INTERFACE` = les propriétés propres à une target qui utilise la librairie (e.g. pour définir un compile-flag d'une lib header-only)
    - `PUBLIC`    = les propriétés utiles à la fois à la librairie, et à la fois à un exécutable qui l'utilise (e.g. le chemin vers les headers à inclure)
- [cette page](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html) liste de toutes les commandes cmake, notamment les `target_XXX` permettant de définir des propriétés des targets
- si on veut définir une target représentant une librairie qui n'a rien à builder (e.g. une lib header-only) :
    ```
    add_library(some_header_only_lib INTERFACE)
    ```
- si on veut définir une target représentant une librairie DÉJÀ buidlée, il faut utiliser `IMPORTED`

----

https://leimao.github.io/blog/CMake-Public-Private-Interface/

Cas typique :

- une target `L` est une librairie, et une target `E` est un exécutable qui dépend de `L`
- on s'intéresse à des propriétés qu'on associe à la target `L` ; par exemple, ça peut être `target_include_directories` ou `target_link_libraries`
- en l'occurence, notre fil rouge pour bien comprendre va être l'application de `target_include_directories` à `L` :
    - avec la visibilité `PUBLIC` = tout se passera comme si on avait ÉGALEMENT défini `target_include_directories` pour `E`
    - avec la visibilité `INTERFACE` = tout se passera comme si on avait UNIQUEMENT défini `target_include_directories` pour `E` (et pas pour `L`)
    - avec la visibilité `PRIVATE` = tout se passera comme si on avait UNIQUEMENT défini `target_include_directories` pour `L` (et pas pour `E`)

Voici un exemple de cas d'application pour chacun :

- `PRIVATE` : une fonction statique dans le cpp de L dépend d'une dépendance D1 -> quand on builde L, on veut builder également D1, mais E n'a pas à avoir connaissance de D1 (car la fonction est privée, statique dans le cpp) :
    ```
    target_include_directories(L PRIVATE "/path/to/headers/of/D1")
    target_link_libraries(L PRIVATE D1)
    ```
- `PUBLIC` : pour définir les headers publics de L. Dit autrement, non seulement lorsqu'on builde L, on veut inclure son répertoire de headers, mais lorsqu'on builde un main qui utilise L (tel que E), on veut AUSSI inclure le répertoire des headers de L :
    ```
    target_include_directories(L PUBLIC "/path/to/headers/of/L")
    target_link_libraries(E L)
    # après ces deux lignes, lorsqu'on builde E, cmake utilisera le chemin vers les headers de L
    ```
- mix des deux précédents :
    ```
    target_include_directories(L PRIVATE "/path/to/headers/of/D1")
    target_link_libraries(L PRIVATE D1)
    target_include_directories(L PUBLIC "/path/to/headers/of/L")
    target_link_libraries(E L)
    # après ces lignes, lorsqu'on builde E, cmake utilisera le chemin vers les headers de L (mais pas ceux de D1)
    ```
- `INTERFACE` : si L est une lib header-only, qui dépend d'une lib D2 header-only -> on n'a pas besoin de buidler L (car header-only), mais les target nécessitant L doivent avoir connaissance de D2
    ```
    target_include_directories(L INTERFACE "/path/to/headers/of/D2")
    ```

----

https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories

> The INTERFACE, PUBLIC and PRIVATE keywords are required to specify the scope of the following arguments. \
> PRIVATE and PUBLIC items will populate the INCLUDE_DIRECTORIES property of <target>. \
> PUBLIC and INTERFACE items will populate the INTERFACE_INCLUDE_DIRECTORIES property of <target>. \
> The following arguments specify include directories.

^ `PUBLIC` publie les includes de la target ET de son client utilisateur, `PRIVATE` ne peuple que la target, `INTERFACE` ne peuple que le client utilisateur.

----

https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html

> The usage requirements of a target can transitively propagate to the dependents. The target_link_libraries() command has PRIVATE, INTERFACE and PUBLIC keywords to control the propagation.

Le snippet est très illustratif :

```cmake
# NDM : on associe la compile_definition USING_ARCHIVE_LIB à la target `archive` (qui est une lib) avec le scope INTERFACE
#       du coup, la lib archive n'utilise pas elle-même ce flag (sauf erreur de ma part, elle sera compilée SANS le flag)
add_library(archive archive.cpp)
target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

# NDM : idem pour la target `serialization` et USING_SERIALIZATION_LIB :
add_library(serialization serialization.cpp)
target_compile_definitions(serialization INTERFACE USING_SERIALIZATION_LIB)

# NDM : on définit une target `archiveExtras` (qui est ici aussi une lib), avec deux choses intéressantes : upstream et downstream.
#       upstream : la target doit linker avec les libs `archive` et `serialization` ; le fait d'avoir utilisé `INTERFACE`
#                  précédemment pour la compile_definition fait que `archiveExtras` est compilé AVEC les flags
#                  `-DUSING_ARCHIVE_LIB` et `-DUSING_SERIALIZATION_LIB`.
#       downstream : la target a indiqué qu'elle linkait avec `archive` en `PUBLIC` et `serialization` en `PRIVATE`.
#                    Du coup, les utilisateurs de `archiveExtras` utiliseront le flag `-DUSING_ARCHIVE_LIB` mais pas
#                    le flag `-DUSING_SERIALIZATION_LIB`.
add_library(archiveExtras extras.cpp)
target_link_libraries(archiveExtras PUBLIC archive)
target_link_libraries(archiveExtras PRIVATE serialization)

# NDM : l'exécutable est utilisateur de `archiveExtras`, il sera buildé avec `-DUSING_ARCHIVE_LIB` mais pas avec `-DUSING_SERIALIZATION_LIB` :
add_executable(consumer consumer.cpp)
target_link_libraries(consumer archiveExtras)
```

EDIT : je l'ai testé [dans une POC](https://github.com/phidra/pocs/tree/133e02c20694e5feb0ca13ba18964405d5b764d5/cpp/TOOL_cmake/cmake_21_PUBLIC_PRIVATE_INTERFACE_compile_definitions), et malheureusement, je n'arrive pas à obtenir le comportement souhaité : l'exécutable se voit propagé les DEUX compile_defs... je ne sais pas où je me suis trompé.

> Because the archive is a PUBLIC dependency of archiveExtras, the usage requirements of it are propagated to consumer too. \
> Because serialization is a PRIVATE dependency of archiveExtras, the usage requirements of it are not propagated to consumer.

^ si une lib déclare qu'elle dépend d'une autre lib en PUBLIC, alors l'utilisateur de la lib en dépendra aussi (et c'est l'inverse pour PRIVATE)

> Generally, a dependency should be specified in a use of target_link_libraries() with the PRIVATE keyword if it is used by only the implementation of a library, and not in the header files.
>
> If a dependency is additionally used in the header files of a library (e.g. for class inheritance), then it should be specified as a PUBLIC dependency.
>
> A dependency which is not used by the implementation of a library, but only by its headers should be specified as an INTERFACE dependency.

^ les guidelines d'utilisateur de PUBLIC vs PRIVATE vs INTERFACE.

----

https://cmake.org/cmake/help/latest/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES

> Targets may populate this property to publish the include directories required to compile against the headers for the target.
>
> The target_include_directories() command populates this property with values given to the PUBLIC and INTERFACE keywords.

Beaucoup de choses dans ces deux phrases. Quand on définit une target (surtout une lib en fait) avec un `INTERFACE_INCLUDE_DIRECTORY` alors ce sont des includes qui sont nécessaire pour utiliser les headers de la librairie. En d'autres termes, ces includes définissent des choses utilisées dans les headers publics de la librairie.

NDM : et ces includes peuvent ou non être nécessaires pour compiler la librairie elle-même :

- s'ils le sont, ces includes auront été associés à la librairie mylib via `target_include_directories(mylib PUBLIC myinclude)`
- s'ils ne le sont pas, ces includes auront été associés à la librairie mylib via `target_include_directories(mylib INTERFACE myinclude)`
- (et le troisième cas est celui où des includes sont nécessaires à l'implémentation privée de la librairie, sans apparaître dans ses headers publics : `target_include_directories(mylib PRIVATE myinclude)` )

> When target dependencies are specified using target_link_libraries(), CMake will read this property from all target dependencies to determine the build properties of the consumer.

Important à comprendre : lorsqu'on associe quelque chose (un include_directory, une compile_definition, etc.) à une librairie, le mot-clé `PUBLIC/PRIVATE/INTERFACE` n'a pas d'influence sur la façon dont cmake compile la lib en elle-même, mais ne joue que sur la façon dont cmake compile un **UTILISATEUR** de la lib.

# Effective Modern CMake

[Effective Modern CMake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1) = équivalent spirituel de Effective C++ pour cmake.

[Think in terms of targets and properties](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1#think-in-terms-of-targets-and-properties) :

> By defining properties (i.e., compile definitions, compile options, compile features, include directories, and library dependencies) in terms of targets, it helps the developer to reason about the system at the target level. The developer does not need to understand the whole system in order to reason about a single target. The build system handles transitivity.

----

[Imagine targets as objects](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1#imagine-targets-as-objects) :

> Calling the member functions modifies the member variables of the object.
>
> Analogy to constructors:
> - add_executable
> - add_library
>
> Analogy to member variables:
> - target properties (too many to list here)
>
> Analogy to member functions:
> - target_compile_definitions
> - target_compile_features
> - target_compile_options
> - target_include_directories
> - target_link_libraries
> - target_sources
> - get_target_property
> - set_target_property

----

[Declare direct dependencies with target_link_libraries](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1#declare-direct-dependencies-with-target_link_libraries) :

> This propagates usage requirements from the dependent target to the depending target. The command also resolves transitive dependencies.

