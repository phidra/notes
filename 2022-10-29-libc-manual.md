# The GNU C Library

- **url** = https://www.gnu.org/software/libc/manual/2.36/html_node/index.html
- **type** = manuel
- **auteur** = multiple
- **date de publication** = N/A
- **source** = [le site GNU](https://www.gnu.org)

- **tags** = language>C ; topic>libc ; level>intermediate


**TL;DR** : mes notes sur quelques pages intéressantes du manuel de la libc.

* [The GNU C Library](#the-gnu-c-library)
   * [Memory](#memory)
      * [Memory concepts](#memory-concepts)
      * [Memory-mapped IO](#memory-mapped-io)
      * [Autres notes en vrac sur la mémoire](#autres-notes-en-vrac-sur-la-mémoire)
      * [Autres extraits du manuel](#autres-extraits-du-manuel)
   * [Processes](#processes)
      * [Principes et création](#principes-et-création)
      * [Identification](#identification)
      * [Créer un process pour exécuter un fichier](#créer-un-process-pour-exécuter-un-fichier)
      * [Attendre un processus fils](#attendre-un-processus-fils)
      * [Inputs d'un process](#inputs-dun-process)
      * [Syscalls](#syscalls)
      * [Terminaison d'un process](#terminaison-dun-process)

Contexte dans lequel je m'y suis intéressé = je me renseignais sur les shared_memory. Du coup, les pages qui ont retenu mon attention étaient celles liées à la mémoire, et aux processus.

## Memory

### Memory concepts

https://www.gnu.org/software/libc/manual/2.36/html_node/Memory-Concepts.html

Cette première page résume de façon parfaite ce qu'il me faut pour comprendre la gestion de la mémoire :

- chaque process a un espace **virtuel** d'adressage ([_virtual address space_ = VAS](https://en.wikipedia.org/wiki/Virtual_address_space), déjà relativement grand en 32 bits = 2³² bits = ~4 Gio, il est titanesque en 64 bits = 2⁶⁴ bits = 16 millions de Tio)
- cet espace virtuel est découpé en **pages** (typiquement d'une taille de 4 kio) = plus petite unité de mémoire allouable → quand on parle de **page mémoire**, on parle toujours de **mémoire virtuelle**
- chaque **page** de mémoire virtuelle a vocation à être backée par quelque chose de "réel" :
    - page de mémoire réelle (appelée **memory frame**)
    - fichier sur disque-dur
    - espace de swap sur disque-dur
    - juste des zéros (dans ce cas, le kernel peut ne backer la page par "rien", et se contenter de marquer quelque part qu'elle est remplie de zéro)
- deux process indépendants ont chacun leur espace d'adressage propre, donc les pages mémoires ne sont jamais partagées
- en revanche, ce qui backe une page mémoire _peut_ être partagé entre deux process ; par exemple la **memory frame** qui stocke un bout de la libc a de bonne chance de backer plusieurs pages mémoire différentes de plusieurs process différents, vu que la libc est utilisée par presque tous les process.


> There are a lot of different ways systems organize memory, but in a typical one, each process has one linear virtual address space, with addresses running from zero to some huge maximum. It need not be contiguous; i.e., not all of these addresses actually can be used to store data.
>
> The virtual memory is divided into pages (4 kilobytes is typical). Backing each page of virtual memory is a page of real memory (called a frame) or some secondary storage, usually disk space. The disk space might be swap space or just some ordinary disk file. Actually, a page of all zeroes sometimes has nothing at all backing it – there’s just a flag saying it is all zeroes.
>
> The same frame of real memory or backing store can back multiple virtual pages belonging to multiple processes. This is normally the case, for example, with virtual memory occupied by GNU C Library code. The same real memory frame containing the printf function backs a virtual memory page in each of the existing processes that has a printf call in its program.
>
> In order for a program to access any part of a virtual page, the page must at that moment be backed by (“connected to”) a real frame. But because there is usually a lot more virtual memory than real memory, the pages must move back and forth between real memory and backing store regularly, coming into real memory when a process needs to access them and then retreating to backing store when not needed anymore. This movement is called paging.
>
> When a program attempts to access a page which is not at that moment backed by real memory, this is known as a page fault. When a page fault occurs, the kernel suspends the process, places the page into a real page frame (this is called “paging in” or “faulting in”), then resumes the process so that from the process’ point of view, the page was in real memory all along. In fact, to the process, all pages always seem to be in real memory. Except for one thing: the elapsed execution time of an instruction that would normally be a few nanoseconds is suddenly much, much, longer (because the kernel normally has to do I/O to complete the page-in).

^ RAS, résumé très clair de comment les choses fonctionnent !

> Within each virtual address space, a process has to keep track of what is at which addresses, and that process is called memory allocation. Allocation usually brings to mind meting out (NdM = distribuer, répartir) scarce resources, but in the case of virtual memory, that’s not a major goal, because there is generally much more of it than anyone needs. Memory allocation within a process is mainly just a matter of making sure that the same byte of memory isn’t used to store two different things.

Ce point est intéressant = comme on ne manquera pas de mémoire virtuelle, l'allocation, c'est simplement s'assurer qu'on ne place pas deux fois au même endroit (i.e. dans la même page virtuelle) des trucs différents (i.e. des memory frames différentes).


> Processes allocate memory in two major ways: by exec and programmatically. Actually, forking is a third way, but it’s not very interesting. See Creating a Process.
>
> Exec is the operation of creating a virtual address space for a process, loading its basic program into it, and executing the program. It is done by the “exec” family of functions (e.g. execl). The operation takes a program file (an executable), it allocates space to load all the data in the executable, loads it, and transfers control to it. That data is most notably the instructions of the program (the text), but also literals and constants in the program and even some variables: C variables with the static storage class

L'espace d'addressage virtuel est créé par `exec`, et au moment de sa création, on commence déjà à allouer des choses dedans = ce qu'on peut allouer à partir du fichier ELF du programme : segment `text`, variables statiques, etc.

À ce stade, l'exécution du programme n'a pas encore commencé, mais son espace d'addressage virtuel a pourtant déjà commencé à être utilisé (et les pages sont bakcées soit par les éléments du fichier ELF sur disque-dur, soit par les memory-frame qui leur correspondent après qu'elles ait été faulted-in).


> Once that program begins to execute, it uses programmatic allocation to gain additional memory. In a C program with the GNU C Library, there are two kinds of programmatic allocation: automatic and dynamic.

Aussi bien la stack que le heap permettent "d'allouer" de la mémoire (i.e. de marquer certaines addresses du virtual address space comme "réservées" pour certaines memory frames).

> Just as it programmatically allocates memory, the program can programmatically deallocate (free) it. You can’t free the memory that was allocated by exec. When the program exits or execs, you might say that all its memory gets freed, but since in both cases the address space ceases to exist, the point is really moot.

RAS : on `free` pour désallouer (avec le caveat décrit ci-dessus), mais on ne peut pas désallouer ce qui a été alloué par `exec` : ça n'est que quand le programme se termine que ça se désalloue tout seul, juste avant que l'espace d'addressage lui-même soit détruit.

> A process’ virtual address space is divided into segments. A segment is a contiguous range of virtual addresses. Three important segments are:
>
> - The text segment contains a program’s instructions and literals and static constants. It is allocated by exec and stays the same size for the life of the virtual address space. \
> - The data segment is working storage for the program. It can be preallocated and preloaded by exec and the process can extend or shrink it by calling functions as described in See Resizing the Data Segment. Its lower end is fixed. \
> - The stack segment contains a program stack. It grows as the stack grows, but doesn’t shrink when the stack shrinks.

### Memory-mapped IO

> Memory-mapped I/O is another form of dynamic virtual memory allocation. Mapping memory to a file means declaring that the contents of certain range of a process’ addresses shall be identical to the contents of a specified regular file. The system makes the virtual memory initially contain the contents of the file, and if you modify the memory, the system writes the same modification to the file. Note that due to the magic of virtual memory and page faults, there is no reason for the system to do I/O to read the file, or allocate real memory for its contents, until the program accesses the virtual memory.

Excellent résumé du memory-mapping :
- `mmap` dit "tel plage d'addresses virtuelles" a une relation un-pour-un avec tel fichier
- lorsque le programme lit les addresses virtuelles, il lit le contenu du fichier
- lorsque le programme écrit les addresses virtuelles, l'OS écrit les modifications dans le fichier sur le disque
- juste après le `mmap`, les addresses virtuelles sont réservées, mais elles ne sont pas encore faulted-in -> le fichier n'a pas encore été lu sur disque :
    - il n'y a pas encore eu d'IO disque (qui a lieu plutôt "à la demande")
    - le contenu du fichier n'a pas encore été transféré dans une memory frame (du coup on n'a pas encore consommé de mémoire physique)
- lorsque le programme essaye de LIRE les addresses virtuelles mmappées, alors le contenu du fichier est faulted-in dans une memory-frame qui backe alors les addresses virtuelles : c'est à ce moment qu'on fait une I/O et qu'on consomme de la mémoire physique
- dans l'autre sens, lorsque le programme essaye d'ÉCRIRE dans les addresses virtuelles mmappées, l'OS fera des I/O pour transférer ce qu'on écrit dans le fichier sur le disque.
- attention toutefois, ça peut ne pas e faire quand on le souhaite, cf. [man msync](https://man7.org/linux/man-pages/man2/msync.2.html) :
    > msync() flushes changes made to the in-core copy of a file that was mapped into memory using mmap(2) back to the filesystem. Without use of this call, there is no guarantee that changes are written back before munmap(2) is called.


### Autres notes en vrac sur la mémoire

- lorsque le système boote, la mémoire physique ne contient rien d'autre que le kernel
- le procédé par lequel l'OS remplit la mémoire physique est le _page fault_ : quand un programme essaye de lire une memory page, et que celle-ci est backée par un fichier sur disque, il va lire le contenu du fichier pour le placer dans la mémoire physique
- c'est pas indiqué dans le manuel de la libc, mais de mémoire, il y a deux types de page-faults :
    - hard = le fichier sur disque n'a jamais été transféré dans une memory-frame -> il faut faire une I/O pour le lire et placer son contenu dans une memory-frame
    - soft = le fichier sur disque existe déjà dans une memory-frame, mais cette memory-frame n'est pas mappée dans l'address-space du processus qui souhaite l'utiliser -> pas besoin de faire l'I/O pour le lire : on peut se contenter d'associer une memory page à la memory frame contenant le fichier
    - (et of course : si la memory page du process est DÉJÀ associée à la memory frame contenant le fichier, il n'y a pas de page-fault du tout)
- quand on parle "allouer" de la mémoire, on parle toujours d'allouer de la mémoire virtuelle
- si malloc échoue, c'est bien la mémoire VIRTUELLE qui est exhausted
- ça n'est PAS l'allocation qui consomme de la mémoire physique ! Ce qui consomme de la mémoire physique, c'est le fait de fault-in une memory-frame pour backer une memory page
- le process de page-fault rend les choses transparentes pour le process, qui a toujours l'impression que ses pages mémoires sont chargées dans la mémoire physique :
    - première situation = c'est déjà le cas -> rien à faire
    - deuxième situation = ça n'est pas encore le cas, l'exécution du process est interrompue par le kernel, qui fault-in la page manquante avant de reprendre l'exécution -> du point de vue du process, il ne se rend pas compte que la page a un jour été absente de la mémoire physique
    - petite entorse : il peut s'en rendre compte car dans la seconde situation, son instruction CPU (qui a nécessité l'accès à la memory page) prendra longtemps à s'exécuter, possiblement très longtemps car nécessite peut-être des IO disque, au lieu de quelques cycles
- malloc a deux façons d'allouer de la mémoire :
    - en allouant des chunks de mémoire virtuelle (i.e. en marquant des adresses de l'espace d'adressage virtuel comme "réservées", et en les backant par des memory-frames), et en allant piocher dedans quand l'utilisateur appelle malloc
    - en associant des adresses virtuelles à un fichier sur disque avec mmap
- de façon contre-intuitive, dans le cas le plus général, `free` ne libère **pas** la mémoire virtuelle, il se contente de la marquer en interne comme disponible pour un prochain malloc
    - ma compréhension, c'est que dans ce cas, le chunk de mémoire virtuelle est bien alloué (i.e. du point de vue de l'OS, la page mémoire correspondante n'est pas libre)
    - mais elle n'est pas utilisée par le programme, elle est utilisée par le runtime malloc qui la garde disponible dans une freelist
- ce n'est que dans certains cas particuliers (je suppose : si on `free` les régions les plus externes du heap) que le runtime `malloc+free` "rend" à l'OS les pages mémoires, et leurs memory-frames associées
- un autre article de blog sur le sujet = https://lemire.me/blog/2020/03/03/calling-free-or-delete/
    > Of course, there are ways to force the memory to be released to the system (...), but you should not expect that it will do so by default.
    > What are the implications?
    > - You cannot measure easily the memory usage of your data structures using the amount of memory that the processes use.
    > - It is easy for a process that does not presently hold any data to appear to be using a lot of memory.


### Autres extraits du manuel

https://www.gnu.org/software/libc/manual/2.36/html_node/Malloc-Examples.html

> The block that malloc gives you is guaranteed to be aligned so that it can hold any type of data. On GNU systems, the address is always a multiple of eight on 32-bit systems, and a multiple of 16 on 64-bit systems. Only rarely is any higher boundary (such as a page boundary) necessary; for those cases, use aligned_alloc or posix_memalign

^ Alignement

> Note that the memory located after the end of the block is likely to be in use for something else; perhaps a block already allocated by another call to malloc. If you attempt to treat the block as longer than you asked for it to be, you are liable to destroy the data that malloc uses to keep track of its blocks, or you may destroy the contents of another block. If you have already allocated a block and discover you want it to be bigger, use realloc

^ De ce que j'en comprends, ça explique qu'un use-after-free n'entraine pas nécessairement de segfault : en effet, du point de vue de l'OS, la mémoire utilisée after `free` peut très bien continuer à être mappée de façon valide, et l'utiliser ne provoquera pas de segfault.

> In the GNU C Library, a failed malloc call sets errno, but ISO C does not require this and non-POSIX implementations need not set errno when failing.

^ Différence de comportement entre la libc GNU (qui, ici, semble suivre POSIX), et ISO C

https://www.gnu.org/software/libc/manual/2.36/html_node/Freeing-after-Malloc.html

> Occasionally, `free` can actually return memory to the operating system and make the process smaller. Usually, all it can do is allow a later call to malloc to reuse the space. In the meantime, the space remains in your program as part of a free-list used internally by malloc.

^ Ce point est intéressant : `free` ne libère pas la mémoire (virtuelle, je suppose), par défaut.

> There is no point in freeing blocks at the end of a program, because all of the program’s space is given back to the system when the process terminates.

https://www.gnu.org/software/libc/manual/2.36/html_node/Malloc-Tunable-Parameters.html

On peut paramétrer le comportement de malloc avec `mallopt`. Ex : choisir le seuil à partir duquel `mmap` sera utilisé au lieu de renvoyer un chunk dans la freelist gérée par malloc.

https://www.gnu.org/software/libc/manual/2.36/html_node/Heap-Consistency-Checking.html

^ Il existe des fonctions (`mcheck`)  pour tester que le heap n'a pas été corrompu (corruption qui peut arriver p.ex. en écrivant après la fin d'un bloc, ce qui écrase des infos de bookkeeping nécessaires à `free`)

https://www.gnu.org/software/libc/manual/2.36/html_node/Statistics-of-Malloc.html

^ On peut avoir des stats d'utilisation de `malloc`.

https://www.gnu.org/software/libc/manual/2.36/html_node/Summary-of-Malloc.html

^ Un bon résumé pratique qui liste les différentes fonctions liées à malloc :

- `*malloc (size_t size)` : Allocate a block of size bytes.
- `free` : Free a block previously allocated by malloc.
- `realloc` : Make a block previously allocated by malloc larger or smaller, possibly by copying it to a new location.
- `reallocarray` : Change the size of a block previously allocated by malloc to nmemb * size bytes as with realloc.
- `calloc` : Allocate a block of count * eltsize bytes using malloc, and set its contents to zero.
- `valloc` : Allocate a block of size bytes, starting on a page boundary.
- `aligned_alloc` : Allocate a block of size bytes, starting on an address that is a multiple of alignment.
- `posix_memalign` : Allocate a block of size bytes, starting on an address that is a multiple of alignment.
- `memalign` : Allocate a block of size bytes, starting on an address that is a multiple of boundary.
- `mallopt` : Adjust a tunable parameter.
- `mcheck` : Tell malloc to perform occasional consistency checks on dynamically allocated memory, and to call abortfn when an inconsistency is found.
- `mallinfo2` : Return information about the current dynamic memory usage.

https://www.gnu.org/software/libc/manual/2.36/html_node/Allocation-Debugging.html

^ Il existe du tooling pour debugger les allocations


## Processes

### Principes et création

https://www.gnu.org/software/libc/manual/2.36/html_node/Processes.html

> Processes are the primitive units for allocation of system resources. Each process has its own address space and (usually) one thread of control. A process executes a program; you can have multiple processes executing the same program, but each process has its own copy of the program within its own address space and executes it independently of the other copies.
>
> Processes are organized hierarchically. Each process has a parent process which explicitly arranged to create it. The processes created by a given parent are called its child processes. A child inherits many of its attributes from the parent process.

https://www.gnu.org/software/libc/manual/2.36/html_node/Running-a-Command.html

La fonction `system` permet de lancer un sous-process et d'attendre son résultat.

https://www.gnu.org/software/libc/manual/2.36/html_node/Process-Creation-Concepts.html

Cette page explique le principe de création des process :

> - A new processes is created when one of the functions posix_spawn, fork, \_Fork or vfork is called. (...)
> - Each new process (the child process or subprocess) is allocated a process ID, distinct from the process ID of the parent process.
> - After forking a child process, both the parent and child processes continue to execute normally.
> - If you want your program to wait for a child process to finish executing before continuing, you must do this explicitly after the fork operation, by calling wait or waitpid
> - A newly forked child process continues to execute the same program as its parent process, at the point where the fork or \_Fork call returns. You can use the return value from fork or \_Fork to tell whether the program is running in the parent process or the child.

^ Ceci explique comment un programme peut s'exécuter en plusieurs sous-process (dit autrement : un même programme forke plusieurs processes pour faire son taf).

Mais un autre usage classique est de forker un process pour lui faire exécuter un **autre** programme :

> Having several processes run the same program is only occasionally useful. But the child can execute another program using one of the exec functions; see Executing a File. The program that the process is executing is called its process image. Starting execution of a new program causes the process to forget all about its previous process image; when the new program exits, the process exits too, instead of returning to the previous process image.

(exemple : un programme de type shell autorise l'utilisateur à lancer des sous-programmes)

https://www.gnu.org/software/libc/manual/2.36/html_node/Creating-a-Process.html

> fork makes a complete copy of the calling process’s address space and allows both the parent and child to execute independently

^ juste après le `fork`, les deux process ont chacun deux copies indépendantes, mais identiques du même VAS (NdM : le copy-on-write fait même en sorte qu'il n'y ait de duplication que quand elle est nécessaire)


### Identification

https://www.gnu.org/software/libc/manual/2.36/html_node/Process-Identification.html

> Each process is named by a process ID number, a value of type pid_t. A process ID is allocated to each process when it is created.

Un process a obligatoirement un PID.

> The lifetime of a process ends when the parent process of the corresponding process waits on the process ID after the process has terminated. See Process Completion. (The parent process can arrange for such waiting to happen implicitly.)

^ Intéressant : ça n'est donc PAS quand un processus de termine que son cycle de vie finit, mais uniquement quand son parent le `wait` ! Tel que je comprends les choses, ça explique la notion de zombie. Cf. https://www.linuxjournal.com/content/how-kill-zombie-processes-linux :

> Also known as “defunct” or “dead” process – In simple words, a Zombie process is one that is dead but is present in the system’s process table. Ideally, it should have been cleaned from the process table once it completed its job/execution but for some reason, its parent process didn’t clean it up properly after the execution.
>
> In a just (Linux) world, a process notifies its parent process once it has completed its execution and has exited. Then the parent process would remove the process from process table. At this step, if the parent process is unable to read the process status from its child (the completed process), it won’t be able to remove the process from memory and thus the process being dead still continues to exist in the process table – hence, called a Zombie


https://www.gnu.org/software/libc/manual/2.36/html_node/Process-Identification.html

> A process ID uniquely identifies a process only during the lifetime of the process. As a rule of thumb, this means that the process must still be running.

^ Les pids sont réutilisés

> On Linux, threads created by pthread_create also receive a thread ID. The thread ID of the initial (main) thread is the same as the process ID of the entire process. Thread IDs for subsequently created threads are distinct. They are allocated from the same numbering space as process IDs.
>
> In contrast to processes, threads are never waited for explicitly, so a thread ID becomes eligible for reuse as soon as a thread exits or is canceled.

Un thread-id ne risque donc pas d'avoir la même valeur qu'un PID.

> You can get the process ID of a process by calling getpid. The function getppid returns the process ID of the parent of the current process


### Créer un process pour exécuter un fichier

https://www.gnu.org/software/libc/manual/2.36/html_node/Executing-a-File.html

> This section describes the `exec` family of functions, for executing a file as a process image. You can use these functions to make a child process execute a new program after it has been forked.

^ Ma compréhension = l'enchaînement `fork` + `exec` + `waitpid` est équivalent à exécuter un subprocess de façon synchrone (EDIT : confirmé par [cette page](https://www.gnu.org/software/libc/manual/2.36/html_node/Process-Creation-Example.html))

> The functions in this family differ in how you specify the arguments, but otherwise they all do the same thing. The execv function executes the file named by filename as a new process image.
>
> Executing a new process image completely changes the contents of memory, copying only the argument and environment strings to new locations

Le process qui utilise `exec` se met à exécuter le programme contenu dans le fichier passé en argument, et oublie son état actuel (du coup, si on veut que le process actuel survive, il faut que ce soit un processus forké qui appelle `exec`).

### Attendre un processus fils

https://www.gnu.org/software/libc/manual/2.36/html_node/Process-Completion.html

> The functions described in this section are used to wait for a child process to terminate or stop, and determine its status.
>
> The waitpid function is used to request status information from a child process whose process ID is pid. Normally, the calling process is suspended until the child process makes status information available by terminating.
>
> If status information for a child process is available immediately, this function returns immediately without waiting.

Le parent peut "attendre" un processus fils (on dirait qu'on ne choisit pas prcisément le child avec `wait`)

### Inputs d'un process

https://www.gnu.org/software/libc/manual/html_node/Program-Basics.html

> Though it may have multiple threads of control within the same program and a program may be composed of multiple logically separate modules, a process always executes exactly one program.
>
> A program starts another program with the exec family of system calls. This chapter looks at program startup from the execee’s point of view

https://www.gnu.org/software/libc/manual/html_node/Environment-Variables.html

Les deux moyens pour un programme de recevoir des trucs en entrée, c'est les arguments, et les envvars

> The argv mechanism is typically used to pass command-line arguments specific to the particular program being invoked. The environment, on the other hand, keeps track of information that is shared by many programs, changes infrequently, and that is less frequently used.

^ rule-of-thumb pour savoir si on passe quelque chose à un process par un argument ou par une envvar. Envvar à privilégier si :

- valeur partagée avec plusieurs process
- valeur assez stable, changeant peu
- valeur peu utilisée

### Syscalls

https://www.gnu.org/software/libc/manual/html_node/System-Calls.html

> A system call is a request for service that a program makes of the kernel. The service is generally something that only the kernel has the privilege to do, such as doing I/O.

^ définition claire d'un syscall

> Programmers don’t normally need to be concerned with system calls because there are functions in the GNU C Library to do virtually everything that system calls do. These functions work by making system calls themselves. For example, there is a system call that changes the permissions of a file, but you don’t need to know about it because you can just use the GNU C Library’s chmod function.

^ aha, ce que j'appelle "syscall" (comme la fonction `open`) n'est peut-être pas un syscall stricto-sensu, mais plutôt un wrapper de la libc autour d'un syscall.

> However, there are times when you want to make a system call explicitly, and for that, the GNU C Library provides the syscall function. syscall is harder to use and less portable than functions like chmod, but easier and more portable than coding the system call in assembler instructions.
>
> syscall is most useful when you are working with a system call which is special to your system or is newer than the GNU C Library you are using.

Donc le chemin des écoliers est de coder le syscall en assembleur. Suivi par l'appel à la fonction `syscall`, suivi par l'appel au wrapper de haut-niveau par la libc. Dans ma vie de tous les jours, je ne devrais pas avoir besoin d'autre chose que du wrapper.

### Terminaison d'un process

https://www.gnu.org/software/libc/manual/html_node/Program-Termination.html

> The usual way for a program to terminate is simply for its main function to return. The exit status value returned from the main function is used to report information back to the process’s parent process or shell.
>
> A program can also terminate normally by calling the exit function.
>
> In addition, programs can be terminated by signals;

https://www.gnu.org/software/libc/manual/html_node/Exit-Status.html

^ quelques conventions sur les exit status.

Un truc intéressant, c'est que le programme ne se termine pas toujours "normalement" :

> Don’t confuse a program’s exit status with a process’ termination status. There are lots of ways a process can terminate besides having its program finish. In the event that the process termination is caused by program termination (i.e., exit), though, the program’s exit status becomes part of the process’ termination status.

^ ma compréhension, c'est que "finir avec un exit status de 1" est différent de "avoir une terminaison anormale".

Exemple de terminaison anormale = https://www.gnu.org/software/libc/manual/html_node/Aborting-a-Program.html

> The abort function causes abnormal program termination. This does not execute cleanup functions registered with atexit or on_exit.
>
> This function actually terminates the process by raising a SIGABRT signal, and your program can include a handler to intercept this signal;

https://www.gnu.org/software/libc/manual/html_node/Termination-Internals.html

Intéressante page sur ce qui se passe quand un processus termine. Ma compréhension des choses, c'est que toutes ces actions qui ont lieu lorsqu'un processus termine font partie du "runtime C" :

- Fermeture des file descriptors
- Sauvegarde de l'exit status pour le renvoyer au processus parent du celui appelle waitpid
- Les enfants orphelins sont reattribués comme fils de init
- Etc.


