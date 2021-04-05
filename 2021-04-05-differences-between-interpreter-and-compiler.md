# The Differences Between Interpreter and Compiler Explained

- **url** = https://thevaluable.dev/difference-between-compiler-interpreter/
- **type** = post
- **auteur** = Matthieu CNEUDE, dev français assez actif [site perso](https://matthieucneude.com/), [twitter](https://twitter.com/Cneude_Matthieu), [github](https://github.com/Phantas0s), [son blog](https://thevaluable.dev/page/about/)
- **date de publication** = 2021-10-27
- **source** = https://thevaluable.dev/

**TL;DR** : rappel des différences entre langage compilé et langage interprété
- *If you need to remember only one thing from this article, it’s this one: every programming language can be interpreted or compiled.*
- bon résumé des avantages/inconvénients de AOT-compilation et interprétation (résumés à la fin des présentes notes)
- une side-note de cet article m'aide à mieux comprendre en quoi un binaire donné est dépendant de l'OS

## intro

> For example, PHP is very often interpreted in real life projects, that’s why we put it in the “interpreted programming language” category. But if you’re Facebook and you write a compiler for PHP, you have now a “compiled programming language”.

Explications sur comment les CPU exécutent les programmes :

> What does a CPU do? It executes a bunch of instructions! [...]
>
> different CPUs don’t speak the same machine code. Each family of CPU understand a precise set of instructions, called as well ISA (Instruction Set Architecture), or simply CPU architecture. Even in the same CPU family, the architecture can be a bit different from one CPU to another.

Réponse à l'une des questions que je me posais (sur comment l'OS intervenait dans le fait qu'on puisse ou non exécuter un binaire sur une machine donnée) :

> If every program written by everybody could do everything on your computer, it would lead to huge security problems.
>
> That’s where your operating system (OS) comes in the mix (Windows, Linux, Android, MS DOS, Plan 9… you name it). If your program want to do any direct operation on the hardware, say writing to the disk, it needs to ask your OS the permission via system calls.
> If the permission is granted, your OS will then instruct the CPU to do whatever you want to do. Since the OS plays such an important role between your source code and the CPU, the machine code needs to use some APIs from your OS to get the authorizations it needs. That’s one of the reason why so many programs can run on a specific OS, but not on another.

----

> It stays very close to the instruction set of the CPU however, that’s why an assembly language will often be tied to a specific CPU and its instruction set.

Ceci est une question que j'essaye de creuser en profondeur ces temps-ci :

> If you want to give your ambitious program to a friend who has a different CPU architecture, you might think it won’t work.

Cross-compiling :

> , gcc will create machine code which can run on different (most common) CPU architectures! It’s called cross-compiling. It has some performance drawbacks, but

J'aime bien cette phrase, car la précision **AOT** est souvent oubliée dans les ressources sur le sujet :

> This specific compilation process is called Ahead Of Time compilation (AOT), because your entire source code is translated to machine code without even be executed once

## What’s an Interpreter ?

Contexte historique :

> Historically, an interpreter is a program reading the source code you wrote, line by line. [...]
> 
> Each line was translated to machine code and fed directly to the CPU while executing the program. [...]
> 
> This was pretty slow and costly, so the way we interpret our high level code changed over the years. [...]
> 
> Nowadays, to interpret your code, you’ll need to compile it into another form of code, called bytecode. This code is designed to be interpreted efficiently by a specific interpreter, or virtual machine.

L'un des avantages des langages interprétés = le bytecode est indépendant de la machine : si la VM existe pour mon archi cible, alors elle pourra exécuer le même bytecode qu'une autre archi :

> The fact that you use a very generic bytecode makes the virtual machine dependent of the OS, but not the bytecode itself!

Au passage, plusieurs langages peuvent produire le même bytecode :

> If you write a compiler to translate another programming language into JVM bytecode, you can even run your programs on the JVM without writing a line of Java.

NdM : c'est une approche retenue par beaucoup de langages, par exemple scala, kotlin, clojure. Et j'apprends à cette occasion que ces langages permettent tous également de produire du javascript.

Définition de la JIT :

> Some interpreters will compile the bytecode it needs into machine code while executing your program. This is called Just In Time compilation (JIT), as opposed to Ahead of Time compilation (AOT)

## Avantages / inconvénients de l'AOT compilation

Toute cette section est intéressante, je résume/traduis :
- **avantage** : à l'AOT-compilation, on connaît déjà l'archi cible, on peut optimiser spécifiquement pour le couple ISA+OS (alors que quand on produit du bytecode, il peut s'exécuter partout)
- **avantage** : on ne dépend pas d'une VM pour exécuter le programme, il est auto-porteur (aux autres dépendances près, of course)
- **inconvénient** : la cross-compilation est un sujet difficile. De plus, si on veut diffuser le programme sur beaucoup de couples archi+OS, il faut générer beaucoup de binaires différents (alors qu'un seul bytecode sera exécutable PARTOUT où la VM sera portée).
- **inconvénient** : l'AOT compilation est (un peu) plus longue que la compilation en bytecode.

Notamment, corrélé à ma question générale sur "qu'est-ce qui fait qu'un programme tourne sur une machine et pas sur une autre". :

> In most cases, you’ll still need different executables for every OS you target, and other ones for some different CPU architectures. You might end up with a growing list of programs to download, and your users need to choose the good one, depending on their system.

## Avantages / inconvénients de l'interpretation

Toute cette section est intéressante, je résume/traduis :
- **avantage** : le code est plus portable : il suffit que la VM soit portée sur le couple OS+ISA (ce qui est souvent le cas).
- **avantage** : la compilation en bytecode est souvent plus rapide que l'AOT compilation
- **inconvénient** : le bytecode est souvent plus lent à exécuter (NdM : en effet, il reste interprété : on a un switch case sur le bytecode)
- **inconvénient** : l'utilisateur doit installer une VM sur sa machine (et la mettre à jour), celle-ci consomme des ressources.
