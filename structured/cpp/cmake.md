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

