# Back to Basics: The Abstract Machine - Bob Steagall - CppCon 2020

- **url** = https://www.youtube.com/watch?v=ZAji7PkXaKY
- **type** = vidéo
- **auteur** = [Bob STEAGALL](https://github.com/BobSteagall), membre du comité C++
- **date de publication** = 2020-09-22
- **source** = [la chaîne youtube de la CppCon](https://www.youtube.com/channel/UCMlGfpWw-RUdWX_JbLCukXg)


**TL;DR** : talk sur les similitudes et différences entre la différence entre _abstract machine_ (dans le modèle mental du développeur, c'est ce qui fait tourner le code qu'on écrit) et _physical machine_ (qui exécute réellement le programme).

* 03:00 définition d'une abstract machine : processor (instruction set, register set, memory model) conçu non pas pour être implémenté physiquement, mais pour supporter l'exécution d'un langage intermédiaire
* 03:20 C++ abstract machine = portable abstraction of your os, kernel and hardware
* 04:20 variety of computing Platforms :
    *  général purpose cpu
    *  GPU
    *  embedded processors
* 05:00 quand on écrit des programmes, on ne veut pas écrire pour un hardware particulier
* 08:00 there is no room for another langage between C++ and the hardware : C++ se mappe bien sur le hardware (e.g. les types fondamentaux se mappent naturellement sur des entités mémoire hardware)
* 08:40 : C++ defines how programs work in terms of an abstract machine. Cette abstract machine est délibérément proche du hardware
* 09:00 our programs describe operations performed on the abstract machine
    * C'est le point principal du talk : _When we write C++ code, we are writing to the C++ abstract machine_
    * En résumé, le dev est en charge d'écrire du code pour l'abstract machine, le compilo est en charge d'écrire du code pour la physical machine (fort heureusement nommé code machine)
    * L'observable behaviour de l'abstract machine et de la physical machine sera identique.
* 13:30 implementation-defined behaviour = marge laissée à l'implémentation (le compilo) sur son comportement. Elle doit être documentée. Certains aspects de l'abstract machine sont non specifies, et non déterministes
* 16:00 définition de observable behaviour
* ~23:00 implementation defined (= au choix de l'implémentation, mais déterministe et documenté, e.g. la taille d'un pointeur = `sizeof(void*)`) et unspecified (non-deterministe,p.ex. l'ordre d'évaluation des arguments d'une fonction)
* 28:00 undefined behaviour. S'il y en a dans le programme, le programme ne sert plus à rien car il peut faire absolument n'importe quoi.
* 29:00 ill-formed = l'implémentation doit avertir l'utilisateur. (il existe aussi une catégorie "no diagnostic required")
* 31:30 structure de l'abstract machine :
    * Memory
    * Objects
    * Threads
* 32:00 pour l'abstract machine, la mémoire est flat et homogène
* 34:00 objects : size, alignement, storage duration (automatic/static/threadlocal), lifetime, type, value (possiblement indéterminé), name. At most ONE memory location.
* ??:?? As-if rule : le compilo peut faire ce qu'il veut, si ça ne modifie pas le comportement observable du programme
* 38:30 intéressant exemple concret d'arithmétique des pointeurs autorisée ou non par l'abstract machine : autorisé dans un tableau, mais pas en dehors.
* 43:45 threads. Chaque thread a une toplevel fonction (celle du thread principal est `main`)
* 52:40 le truc à retenir :

> When we write C++ code, we are writing to the C++ abstract machine

* (Et c'est l'implémentation qui traduit les opérations effectuées par l'abstract machine en des opérations effectuables par la physical machine)

