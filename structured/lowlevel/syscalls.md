Notes issues de la lecture de [cette excellente série d'articles](https://cpu.land/the-basics), à la base pour expliquer comment fonctionne un CPU ; mais j'y trouve un bon complément à mes investigations d'il y a quelques temps, et notamment je comprends mieux le lien entre syscall, libc et hardware.

* [Interrupt](#interrupt)
* [CPU modes](#cpu-modes)
* [Syscalls et libc](#syscalls-et-libc)

# Interrupt

**Interrupt** = le CPU interrompt l'exécution séquentielle du programme à l'emplacement de l'actuel _instruction pointer_, pour aller exécuter un handler d'interruption :

- Software interrupt = triggé par un programme
- Hardware interrupt = triggé par le matériel (e.g. un timer arrive à échéance)

# CPU modes

Côté hardware, le processeur permet deux modes : normal et admin :

- Admin a accès à tout.
- Normal a certaines fonctionnalités restreintes, telles que la possibilité de remapper la virtual memory (nécessaire pour accéder à la mémoire d'un autre process), ou la possibilité d'enregistrer des handlers d'interrupts.

En pratique, le kernel tourne en admin mode, les process classiques (y compris ceux de l'OS lui même tels que `init`) tournent en normal mode.

Et au passage, les process sont une abstraction fournie par l'OS. Le CPU n'a pas connaissance de process : juste de code qu'il exécute séquentiellement, à l'emplacement de son instruction pointer.

# Syscalls et libc

De base, le CPU n'a pas vraiment connaissance des syscalls : il se contente d'exécuter des instructions séquentiellement. Les syscalls sont une abstraction de l'OS, et non du CPU (en vrai, les syscalls sont tellement fréquents que les CPU ont parfois des instructions spéciales pour les traiter, telles que `sysenter`).

En pratique, les syscalls sont gérés avec des interrupts soft :

- lors du démarrage, le kernel enregistre un handler d'interruption pour gérer les syscalls
- plus tard, les programmes génèrent des interrupts soft pour "appeler" le syscall (en vrai, le programme userland appelle plutôt une fonction de la libc en userland, et c'est cette fonction qui trigge l'interrupt)

La gestion des syscalls dépend à la fois :

- de l'OS : obviously, vu que c'est une abstraction proposée par l'OS : la liste des syscalls disponibles dépend de l'OS
- du hardware : car comme il s'agit d'interrupts, et que la façon d'enregistrer un interrupt en le déclarant auprès du CPU, puis plus tard de déclencher un interrupt + de lui passer des arguments, dépend du hardware sur lequel on s'exécute, alors les syscalls dépendent du hardware

Du coup, pour simplifier la vie du programmeur, la libc gère les multiples implémentations possibles de déclenchement de l'interrupt `syscall`, et les programmes en userland dépendent de la libc, en ayant accès pour chaque syscall à une unique fonction, plutôt que d'avoir à implémenter eux-mêmes la bonne façon d'appeler un syscall en fonction du couple {OS+hardware}.

> If you were curious, the interrupt ID used for system calls on Linux is 0x80

^ tous les syscalls linux passent par le même interrupt : `0x80`

> Programs can’t directly switch privilege levels; software interrupts are safe because the processor has been preconfigured by the OS with where in the OS code to jump to. The interrupt vector table can only be configured from kernel mode

^ j'aime bien cette vision : c'est l'OS qui pré-configure le CPU en définissant des handlers d'interruption destinés à gérer les syscalls.

Ça m'éclaire aussi sur la dépendance d'un programme à un OS (et m'aide donc à répondre à la question "qu'est ce qui fait qu'un programme binaire qui tourne sur ma machine ne tournera pas sur telle autre machine) : comme 1. c'est l'OS qui définit les interrupts et les syscalls, 2. qu'un programme a besoin de passer des syscalls pour faire quelque chose d'utile (en effet : tous les IO passent par des syscalls), et 3. qu'il faut utiliser une librairie telle que la libc pour faire des syscalls, alors la conclusion qui s'ensuit est que notre programme est marié à la fois à l'OS, et à la libc qui permet de l'utiliser.

NDM : si on contourne le fait de passer par une lib pour faire des syscalls, alors c'est qu'on commence à implémenter soi même les fonctionnalités du kernel, ou bien qu'on fait du dev spécifique à l'hardware, a.k.a de l'embarqué.

Autre lien avec le hardware : la façon d'invoquer un syscall (e.g. quoi passer dans des registres, quoi passer sur la stack) dépend du hardware (NDM : vu que deux hardwares différents n'ont pas les mêmes registres) :

> The variance in how system calls are called across devices means it would be wildly impractical for programmers to implement system calls themselves for every program.

^ du coup, si on veut faire un programme qui peut tourner sur plusieurs hardwares différents, il faut abstraire le syscall.

> This would also mean operating systems couldn’t change their interrupt handling for fear of breaking every program that was written to use the old system.

^ si le programme gère lui même le code appellant l'interrupt du syscall, les OS ne peuvent plus le changer librement sans casser plein de programmes dans la nature (NDM : alors que si c'est la libc qui gère les syscall, lorsque l'OS veut modifier sa table d'interrupt, il n'a qu'un seul endroit avec lequel se synchroniser = la libc).

> Finally, we typically don’t write programs in raw assembly anymore — programmers can’t be expected to drop down to assembly any time they want to read a file or allocate memory.

^ last but not least des raisons de passer par une lib pour faire les syscalls : c'est plus simple.

> So, operating systems provide an abstraction layer on top of these interrupts. Reusable higher-level library functions that wrap the necessary assembly instructions are provided by libc on Unix-like systems

^ conclusion de tout ce qui précède = il nous faut la libc :-)

> When you call exit(1) from C running on a Unix-like system, that function is internally running machine code to trigger an interrupt, after placing the system call’s opcode and arguments in the right registers/stack/whatever.

^ résumé de ce qui se passe quand on fait un syscall, qui montre que 1. c'est un interrupt, et 2. c'est forcément hardware-dependent

> Many CISC architectures like x86-64 contain instructions designed for system calls, created due to the prevalence of the system call paradigm.
>
> Intel and AMD managed not to coordinate very well on x86-64; it actually has two sets of optimized system call instructions. SYSCALL and SYSENTER are optimized alternatives to instructions like INT 0x80. Their corresponding return instructions, SYSRET and SYSEXIT, are designed to transition quickly back to user space and resume program code.

^ les instructions assembleur `SYSCALL` (ou `SYSENTER`) sont des alternatives optimisées au fait de trigger manuellement l'interrupt `0x80`
