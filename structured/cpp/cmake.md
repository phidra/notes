D'une façon générale, sur cmake, cf. mes nombreuses POCs sur le sujet, assorties de notes.

(même si je préfère meson, cmake semble être établi comme un stardard de fait...)

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

Je reporte ici plusieurs extraits vrac :

https://hsf-training.github.io/hsf-training-cmake-webpage/04-targets/index.html

- un mot sur la visibilité des properties des target (le cas typique, c'est le cas où la target est une librairie) :
    - `PRIVATE`   = les propriétés propres à la librairie (e.g. le cpp de cette librairie a besoin de C++14)
    - `INTERFACE` = les propriétés propres à une target qui utilise la librairie (pas hyper clair les cas d'usages où je voudrais utiliser ça ?)
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
