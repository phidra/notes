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
         * [Explications dans la page Memory Concepts](#explications-dans-la-page-memory-concepts)
         * [Explications dans la page de manuel dédiée](#explications-dans-la-page-de-manuel-dédiée)
         * [mmap](#mmap)
         * [munmap](#munmap)
         * [msync](#msync)
         * [mremap](#mremap)
         * [madvise](#madvise)
         * [shared_memory shm_open](#shared_memory-shm_open)
         * [shared_memory shm_unlink](#shared_memory-shm_unlink)
         * [shm_open n'est rien d'autre qu'un open dans un tmpfs](#shm_open-nest-rien-dautre-quun-open-dans-un-tmpfs)
         * [System-V shared_memory](#system-v-shared_memory)
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

#### Explications dans la page Memory Concepts

Excellent résumé du memory-mapping dans [la page Memory Concepts](https://www.gnu.org/software/libc/manual/2.36/html_node/Memory-Concepts.html) :

> Memory-mapped I/O is another form of dynamic virtual memory allocation. Mapping memory to a file means declaring that the contents of certain range of a process’ addresses shall be identical to the contents of a specified regular file. The system makes the virtual memory initially contain the contents of the file, and if you modify the memory, the system writes the same modification to the file. Note that due to the magic of virtual memory and page faults, there is no reason for the system to do I/O to read the file, or allocate real memory for its contents, until the program accesses the virtual memory.

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

#### Explications dans la page de manuel dédiée

Par ailleurs, il y a [une page du manuel dédiée aux memory-mapped I/O](https://www.gnu.org/software/libc/manual/2.36/html_node/Memory_002dmapped-I_002fO.html) avec plein de trucs intéressants :

> On modern operating systems, it is possible to mmap (pronounced “em-map”) a file to a region of memory. When this is done, the file can be accessed just like an array in the program. This is more efficient than read or write, as only the regions of the file that a program actually accesses are loaded. Accesses to not-yet-loaded parts of the mmapped region are handled in the same way as swapped out pages.

La libc mets en avant ceci comme intérêt principal = charger le contenu du fichier "à la demande" plutôt qu'intégralement.

> Since mmapped pages can be stored back to their file when physical memory is low, it is possible to mmap files orders of magnitude larger than both the physical memory and swap space. The only limit is address space.

Du coup, une memory page peut-être backée par le disque ou la RAM selon les besoins, et même alterner entre les deux → on peut donc charger morceau par morceau un énorme fichier, plus gros que la RAM disponible.

> Memory mapping only works on entire pages of memory. Thus, addresses for mapping must be page-aligned, and length values will be rounded up. To determine the default size of a page the machine uses one should use:
    ```
    size_t page_size = (size_t) sysconf (_SC_PAGESIZE);
    ```

C'est un détail important : les adresses qu'on souhaite mmapper doivent être alignées sur une page mémoire.

#### mmap

> `void * mmap (void *address, size_t length, int protect, int flags, int filedes, off_t offset)` \
> The mmap function creates a new mapping, connected to bytes (offset) to (offset + length - 1) in the file open on filedes. A new reference for the file specified by filedes is created, which is not removed by closing the file.

Plusieurs trucs intéressants ici
- on crée un mapping entre des addresses virtuelles et un fichier
- comme `mmap` attend un file_descriptor, c'est que mmap travaille sur un fichier DÉJÀ ouvert préalablement par `open`
- c'est pas hyper-clair, mais on dirait que mmapper le fichier crée une référence sur le file_descriptor (et que celle-ci peut leaker si on oublie de la relâcher avant de quitter le programme)
- plus bas, il est dit que fermer le fichier n'invalide pas le mapping

Concernant la protection de la mémoire, il s'agit de savoir si on peut lire la mémoire, écrire dessus ou les deux (voire aucun des deux), comme décrit [sur cette autre page](https://www.gnu.org/software/libc/manual/2.36/html_node/Memory-Protection.html)

Les flags peuvent contenir plein de trucs utiles (e.g. `MAP_HUGETLB`) mais le point principal, c'est que l'un des deux flags `MAP_PRIVATE` ou `MAP_SHARED` doit obligatoirement être passé :

> MAP_PRIVATE This specifies that writes to the region should never be written back to the attached file. Instead, a copy is made for the process, and the region will be swapped normally if memory runs low. No other process will see the changes.

NdM = ma compréhension, c'est que ça sert à un process pour se créer un genre de buffer pour son usage perso ? Comme un genre d'alternative à un malloc/new ?

> MAP_SHARED This specifies that writes to the region will be written back to the file. Changes made will be shared immediately with other processes mmaping the same file. Note that actual writing may take place at any time. You need to use msync, described below, if it is important that other processes using conventional I/O get a consistent view of the file.

Ici aussi plusieurs trucs intéressants :
- déjà, ce qu'on écrit dans la mémoire finit par être persisté dans le fichier (peut être utile pour persister la donnée mmappée)
- les autres process qui mappent le même fichier pourront lire une modification que j'écris aussitôt qu'elle est écrite (pas besoin d'attendre qu'elle soit synchronisée avec le disque)
- en revanche, pour qu'un autre process qui n'a pas mmappé le fichier (i.e. qui souhaite lire le fichier sur disque de façon traditionnelle) puisse lire ce que j'écris, il faut attendre... "plus tard" (et si nécessaire, on peut forcer explicitement ce "plus tard" à "tout de suite" avec `msync`)

#### munmap

> `Function: int munmap (void *addr, size_t length)` \
> munmap removes any memory maps from (addr) to (addr + length). length should be the length of the mapping.

`munmap` défait un précédent mappint (je laisse de côté les cas particuliers).

#### msync

> `Function: int msync (void *address, size_t length, int flags)` \
> When using shared mappings, the kernel can write the file at any time before the mapping is removed. To be certain data has actually been written to the file and will be accessible to non-memory-mapped I/O, it is necessary to use this function.

Cet appel est nécessaire si un process veut que les modifications qu'il a lui-même apportées à un fichier mmappé (écriture sur la mémoire) soient accessibles par le monde extérieur = un autre process.

(c'est inutile dans le cas des shared_memory, vu que de ce que j'en comprends, les shared_memory ne sont rien d'autre que des mmap sur des fichiers dans un tmpfs)

#### mremap

> `Function: void * mremap (void *address, size_t length, size_t new_length, int flag)` \
> This function can be used to change the size of an existing memory area.

NdM : on dirait que ça permet d'agrandir une région mmappée, mais d'un autre côté, on a plus bas :

> This function is only available on a few systems. Except for performing optional optimizations one should not rely on this function. 

Du coup, quel serait l'alternative pour agrandir une région mmappée ? Pas clair...

#### madvise

> `Function: int madvise (void *addr, size_t length, int advice)` \
> This function can be used to provide the system with advice about the intended usage patterns of the memory region starting at addr and extending length bytes. 

Plein de trucs cools, permettant au kernel d'optimiser ses IO, e.g. :

- `MADV_RANDOM` The region will be accessed via random page references. The kernel should page-in the minimal number of pages for each page fault.
- `MADV_SEQUENTIAL` The region will be accessed via sequential page references. This may cause the kernel to aggressively read-ahead, expecting further sequential references after any page fault within this region.

#### shared_memory shm_open

> `Function: int shm_open (const char *name, int oflag, mode_t mode)` \
> This function returns a file descriptor that can be used to allocate shared memory via mmap. \
> Unrelated processes can use same name to create or open existing shared memory objects.

Plusieurs processus peuvent se mettre d'accord en avance de phase sur le fichier à utiliser, avant même la création de la shared-mem.

> A name argument specifies the shared memory object to be opened. In the GNU C Library it must be a string smaller than NAME_MAX bytes starting with an optional slash but containing no other slashes.

Typiquement, un nom pourrait être `/pouet`, qui crééera probablement la shared_memory en tant que fichier mmappé dans `/dev/shm/pouet` (où `/dev/shm` est un tmpfs, donc tout en RAM).

> shm_open returns the file descriptor on success or -1 on error. On failure errno is set.

Attention à bien vérifier qu'il n'y a pas eu d'erreur !

#### shared_memory shm_unlink

> `Function: int shm_unlink (const char *name)` \
> This function is the inverse of shm_open and removes the object with the given name previously created by shm_open. \
> shm_unlink returns 0 on success or -1 on error. On failure errno is set.

Fonction à appeler symétriquement à shm_open.

#### shm_open n'est rien d'autre qu'un open dans un tmpfs

**DISCLAIMER** : ce point n'est pas directement dans le manuel de la libc, mais autant centraliser mes notes sur le partage de mémoire entre différents processes avec `mmap`.

`shm_open` est simplement un `open` sur un fichier choisi automatiquement par l'OS dans un tmpfs en RAM plutôt sur disque dur : https://www.sololearn.com/Discuss/2785097/shared-memory-with-mmap-shm_open-vs-open

> The difference between them is shm_open must put the file in the tmpfs(temporary file storage or virtual storage) file system which removes the extra i/o overheads as the data is not being written in a physical file and thus would be much faster than opening and using mmap() on a regular file.
>
> Now of-course one can also use open() to create a file in tmpfs (for example "/dev/shm" in LINUX) to get exactly same results as that with shm_open() 

Autre ressource sur le même sujet = https://stackoverflow.com/questions/24875257/why-use-shm-open/24875686#24875686

> If you open and mmap() a regular file, data will end up in that file. \
> If you just need to share a memory region, without the need to persist the data, which incurs extra I/O overhead, use shm_open().

Dit autrement : il est tout à fait possible de partager de la mémoire entre deux process en utilisant `mmap` sur un fichier _classique_ sur disque-dur (plutôt que dans un `tmpfs`), et dans ce cas, on peut tout à fait se passer de `shm_open`, et utiliser `open` à la place.

#### System-V shared_memory

**DISCLAIMER** : ce point n'est pas directement dans le manuel de la libc, mais autant centraliser mes notes sur le partage de mémoire entre différents processes avec `mmap`.

Attention, en plus de `shm_open` il existe une autre façon de créer une shared_memory : `shmget`, de System-V (qui est [une variante d'UNIX](https://fr.wikipedia.org/wiki/UNIX_System_V)).

Le **TL;DR** est donné par `man shm_overview`, il faut utiliser `shm_open` et pas `shmget` :

> System V shared memory (shmget(2), shmop(2), etc.) is an older shared memory API. \
> POSIX shared memory provides a simpler, and better designed interface; on the other hand POSIX shared memory is somewhat less widely available (especially on older systems) than System V shared memory.

[Cette autre page](https://comp.os.linux.development.apps.narkive.com/sPuJ4fI1/shmget-vs-shm-open) est un peu moins péremptoire, mais va plutôt dans le même sens. En gros, avec l'appel POSIX, on peut "nommer" la shared_memory (et donc la retrouver dans un autre process avec un nom convenu à l'avance), ce qui n'est pas le cas avec shmget :

> shm_open() allows multiple un-related processes to access the same shared memory - since it can be accessed by a well know name. \
> shmget() requires some way for the creating process to give the 'key' used to create the memory to other processes so they can access it. sometimes the creating process writes the 'key' to a well known file name so that other un-related processes can read the key.
>
> if the creating process only needs to share the memory with child processes created via fork(), then shmget() is ok; otherwise, shm_open() is often used.

La façon de lister les shared_memory est donc incompatible entre les deux API !

Pour lister les segments de mémoire partagée, on utilise `ipcs -m`, ou `ipcs -m -p` pour connaître le process qui a créé la shared-mem. Si le process créateur existe encore, on peut donc lister les shared-mem SYSTEM-V et leur process créateur avec :

```sh
ipcs -m -p |grep myself|awk -F"myself" '{print $2}'|cut -d" " -f 6|sort -u | xargs ps -o comm= -p
# ibus-ui-gtk3
# xfwm4
# xfce4-panel
# xfdesktop
# panel-1-whisker
# panel-5-notific
# panel-9-pulseau
# xfce4-notifyd
# nm-applet
# gnome-terminal-
# firefox
```

A contrario, `man shm_open` :

> On successful completion shm_open() returns a new file descriptor referring to the shared memory object.

On voit donc qu'avec `shm_open`, les shared_memory sont identifiées par un fichier et connaître les actuelles shared_memory revient à lister ces fichiers : si l'utilisateur a utilisé la localisation par défaut `/dev/shm`, il suffit de :

```
ls -lh /dev/shm
```


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


