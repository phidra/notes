# Meetup C++

- **url** = https://www.meetup.com/fr-FR/User-Group-Cpp-Francophone/events/280843919
- **type** = meetup
- **auteur** = l'hôte est [Fred TINGAUD](https://github.com/FredTingaud), senior UI développeur chez Murex
- **date de publication** = 2021-09-27
- **source** = [User Group C++ Francophone](https://www.meetup.com/fr-FR/User-Group-Cpp-Francophone)
- **tags** = language>C++ ; topic>wasm ; topic>web-assembly ; meetup ; level>beginner

**TL;DR** : je n'ai assisté qu'au premier talk, sur WebAssembly.

* [Meetup C++](#meetup-c)
   * [outils intéressants](#outils-intéressants)
   * [talk WASM](#talk-wasm)

## outils intéressants

(c'est indépendant du talk WASM)

- https://quick-bench.com = outil du style de compiler-explorer, développé par Fred TINGAUD pour bencher du code C++
- https://cppp.fr/ = conf C++ francophone qui va être relancée
- autre concurrent de conan pour build/packaging = [build2](https://build2.org/), commande `bdep` qui a l'air magique

## talk WASM

**Titre** = Intro to webassembly (with some C++)  ([slides](https://docs.google.com/presentation/d/11yXAErlqfThh2AHiNvxgpyOnVpCuTvbihcvjdDFKJTo/edit#slide=id.p), [github du talk](https://github.com/Klaim/talk-wasm-2021))

L'auteur est Joël LAMOTTE (Klaim, [son site perso, très fourni](https://klaimsden.net/)), qui bosse chez https://www.jellynote.com/fr

Notes très vrac :

- WebAssembly = trompeur : pas spécifique au web, et pas vraiment assembleur
- format d'instruction binaire, pour une VM stack-based (comme C et C++, qui se basent aussi sur cette VM)
- W3C, donc un gros truc du web
- disponibles dans tous les gros browsers (1.0), mais pas que
- caractéristiques :
    - format binaire (bytecode) ou textuel
    - VM stack-based (comme C, C++, rust)
    - l'API dépend du type d'application hôte (NdM : hôte = sans doute le client web qui runne le wasm ?)
    - les modules peuvent être library ou exécutable ou les deux (pas vraiment de différence, à part qu'il y a ait un main et/ou un autorun)
    - c'est juste du code avec des points d'entrée
    - la VM est sandboxée, sécurisée, isolée : conçu dès la conception avec en tête la sécurité + le caractère embarqué
    - awesome WASM : [lien](https://github.com/mbasso/awesome-wasm)
    - poussé par des grands du web -> c'est "durable"
- il faut voir ça comme une "plate-forme cible" (comme windows ou linux) avec peu d'adaptations (les mêmes que pour windows ou linux)
- besoin = besoin de portage d'application vers le browser meilleur que javascript (NaCL, Flash, Silverlight, etc.)
    - ça permet de créer des apps web depuis n'importe quel language qui peut compiler vers du wasm
    - (mais C/C++/rust sont les 1st-class citizen → facile à utiliser avec C/C++, les seuls efforts à faire sont côté UI)
- de base, plus rapide que javascript (généralement)
- utilisable aussi en dehors du web : on peut embarquer une VM wasm qui va exécuter du code wasm
    - (exemple d'intérêt 1 = ne pas se taper les ABI issues)
    - (exemple d'intérêt 2 (plutôt théorique) = remplacer les langages de script, pour ne pas "forcer" un langage : on peut utiliser ce qu'on veut tant que ça compile en wasm)
    - sandboxé by design
- ça permet des "binaires universels" (un peu comme une VM java, mais plus bas-niveau)
- "near-native" performances performance 
- quelques exemples, dont un exemple de jeu Unity (qui est un moteur C++ qui embarque une VM .NET)
- comment ça marche :
    - code C++ --> toolchain (compiler+linker) --> code WASM --> VM WASM qui exécutera le code
    - la toolchain a besoin d'une app-spécifique API, qui est liée à une application hôte (NdM : je suppose que c'est le browser ? ou bien emscripten ?)
    - emscripten est la toolchain génère du js et du wasm, utilisable directement dans le browser
    - le même code (i.e. le même js+wasm) peut être exécuté dans une autre application hôte (e.g. node, qui permet de tester le wasm sans ouvrir le browser)
- emscripten
    - clang + special linker (et wrapping python)
    - fournit une API HTML5 en C++
    - fournit également des traductions d'APIs "classiques" (opengl, openal, sdl2, pthread) vers leur équivalent browser (webgl, html5audio, html5canvas, webworker)
    - il y a des outils pour binder du code C++/JS, et des outils de testing (e.g. pour pouvoir tester avec node plutôt que dans le browser)
- note : on peut générer des infos de debug pour que le wasm soit débuggable dans le browser
- utilisation emscripten :
    - script à sourcer pour mettre em++ (et un nodejs de debug) dans le path
    - em++ pour compiler : `em++ hello.cpp -o hello.js`
- accès au filesystem pas forcément simple depuis le browser -> emrun addresse le problème
- on peut compiler directement un HTML, et il mettra tout dedans (le html, le js, et le wasm)
- on peut utiliser des libs externes (boost, etc. y compris qui ne sont pas header-only) : webassembly va les linker en statique et les intégrer au wasm
- il y a un support du multithreading dans le browser

Un mec a posé une question intéressante :

> !question Y a-t-il un goulot d'étranglement dans une communication wasm -> canvas ? Dans un cas d'ecriture directe facon buffer.

La réponse fournie est bien détaillée et enrichissante :

> je peux développer un peu sur la question précédente. Il n'y a pas de différence d'accès entre Wasm et Javascript niveau bande passante, parce que l'accès est exactement le même.
>
> Ce qui est important à voir c'est que le Wasm n'a aucun accès, à rien.
>
> Tout le principe de emscripten c'est de mettre en place un protocole de communication entre le code wasm et le monde externe. Quand tu fais par exemple un glTexImage2D(), ce qui se passe c'est en gros :
>
>   → le code c++ appelle le stub wasm fourni par emscripten. \
>   → ce code sérialise les paramètres de la fonction à un emplacement mémoire convenu \
>   → puis il interrompt l'exécution du javascript (un peu comme un trap sur un système embarqué). \
>   → le glue code emscripten lit les paramètres depuis l'emplacement connu \
>   → et il appelle WebGL2RenderingContext.texImage2D() \
>   → puis il relance l'exécution du js
>
> La conversion des paramètres / interruption+relance est assez coûteuse, en revanche il n'y a pas d'impact du tout sur la bande passante mémoire, c'est l'appel js habituel qui fait le taff.
