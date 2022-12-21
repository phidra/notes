* [address-free](#address-free)
* [utilisation des atomic dans une shared-memory](#utilisation-des-atomic-dans-une-shared-memory)
* [notes vrac issues de la doc de boost::interprocess](#notes-vrac-issues-de-la-doc-de-boostinterprocess)
* [Keypoints appris lors de mes POCS](#keypoints-appris-lors-de-mes-pocs)
   * [principe général du partage de mémoire entre deux process](#principe-général-du-partage-de-mémoire-entre-deux-process)
   * [synchronisation entre les process - qui construit/détruit quoi quand ?](#synchronisation-entre-les-process---qui-construitdétruit-quoi-quand-)
   * [implémentation des boost::interprocess::named_mutex](#implémentation-des-boostinterprocessnamed_mutex)
   * [xxx_unlink et suppression des structures interprocess](#xxx_unlink-et-suppression-des-structures-interprocess)
   * [forker après avoir mmappé](#forker-après-avoir-mmappé)

**Contexte** : fin 2022, je m'intéresse à ce que permet boost pour faire communiquer deux process via une shared-memory ; j'ai fait plusieurs POCs sur le sujet, parmi lesquelles [la dernière en date](https://github.com/phidra/pocs/tree/59976b5105c639b04eca7af87802df07468521ef/cpp/CATEGORY_ipc/shmem_07_simpler_boost_ipc_shared_memory) est la plus utilisable. Ces notes un peu vrac sont complémentaires aux POCS.

# address-free

Ce qu'on peut mettre dans une shared-memory est connexe au sujet _address-free_ (que j'avais abordé dans le cadre de la sérialization/déserialization de structures de données).

Notamment, dès que je veux partager quelque chose entre plusieurs process, ce quelque chose doit être _address-free_, puisque les adresses sont propres à un espace virtuel d'un process en particulier.

# utilisation des atomic dans une shared-memory

**TL;DR** : en pratique, si les atomic sont lock-free (ce qu'on ne peut vérifier qu'au runtime, avec [is_lock_free](https://en.cppreference.com/w/cpp/atomic/atomic/is_lock_free)), alors ils sont address-free et on peut les utiliser librement dans une shared-memory.

L'intérêt est de pouvoir utiliser un atomic partagé dans une shared-memory, p.ex. pour synchroniser deux process.

Ma compréhension des choses :
- les atomic non lock-free utilisent une hashtable, et chaque process a sa propre hash-table de locks (c'est donc logique que ces atomic là ne soient pas utilisables dans une shared-mem)
- pour les atomic qui SONT lock-free, on n'a pas besoin de structure de donnée externe pour la synchronisation : dans leur cas, le CPU propose des instructions spéciales permettant de faire des read et des write de façon atomique (et c'est même sans doute ça qui caractérise les atomic lock-free : ceux qui portent sur un type pour lequel il existe une instruction assembleur hardware de lecture/écriture atomique)
- par exemple, dans le cas d'un `atomic<bool>`, l'atomicité étant garantie par les instructions hardware (e.g. `compare_and_swap`), pas besoin de stocker en RAM autre chose que le bool : l'atomic n'occupe qu'un seul byte en mémoire (pour stocker le bool), et c'est normal

Quelques extraits de références :

https://subscription.packtpub.com/book/iot-&-hardware/9781838821043/7/ch07lvl1sec58/using-atomic-variables-in-shared-memory

> atomic variables can also be used to synchronize independent applications that run as separate processes.

---

https://stackoverflow.com/questions/51463312/are-lock-free-atomics-address-free-in-practice

^ TL;DR : les atomic lock_free sont address-free en pratique, et peuvent être utilisés avec plusieurs processes.

---

https://en.cppreference.com/w/cpp/atomic/atomic_is_lock_free

> Determines if the atomic object pointed to by obj is implemented lock-free
>
> The C++ standard recommends (but does not require) that lock-free atomic operations are also address-free, that is, suitable for communication between processes using shared memory.
>
> Atomic types are also allowed to be sometimes lock-free (...), whether atomics are lock-free may not be known until runtime.

---

https://stackoverflow.com/questions/50298358/where-is-the-lock-for-a-stdatomic

^ Ressource intéressante sur le fonctionnemnet des atomic = pour les atomic lockfull (donc pas les atomic lock-free), le lock n'est pas stocké dans l'atomic lui-même, mais dans une hashtable utilisant l'addresse de l'atomic comme key → cette implémentation n'est donc pas partageable entre plusieurs process.

> The usual implementation is a hash-table of mutexes (...), using the address of the atomic object as a key.

# notes vrac issues de la doc de boost::interprocess

Il y a de quoi créer un segment de mémoire (= l'équivalent du fichier dans un `ramfs`) puis le mmapper dans le virtual-address-space (VAS) du process.

https://www.boost.org/doc/libs/1_80_0/doc/html/interprocess/sharedmemorybetweenprocesses.html

^ la page de la doc qui précise comment utiliser les shared-memory boost IPC

- instancier un shared_memory_object = créer le segment de mémoire (l'équivalent de `shm_open` = créer le fichier en ramfs)
   ```cpp
   shared_memory_object shm_obj (open_or_create ,"my_super_shared_memory" ,read_only);
   ```
- When a shared memory object is created, its size is 0. To set the size of the shared memory, the user must use the truncate function call :
   ```cpp
   shm_obj.truncate(10000);
   ```
- derrière, pour mapper une shared-memory créée dans le VAS du process :
    - _Once created or opened, a process just has to map the shared memory object in the process' address space. (...) The mapping process is done using the mapped_region class._
   ```cpp
   mapped_region region(shm_obj, read_write);
   auto my_ptr = region.get_address();
   auto my_size = region.get_size();
   ```
- l'équivalent de `shm_unlink` est la fonction statique `remove` :
   ```cpp
   shared_memory_object::remove("my_supershared_memory");
   ```

---

https://www.boost.org/doc/libs/1_80_0/doc/html/interprocess/sharedmemorybetweenprocesses.html#interprocess.sharedmemorybetweenprocesses.mapped_region_object_limitations

^ notamment, ceci est TRÈS intéressant, et précise les règles pourqu'un objet C++ puisse être stocké dans un segment de mémoire

- pointeurs = ne pas utiliser de pointeur, mais utiliser plutôt des `offset_ptr`
- pointeurs = ne "pointer" que vers des objets qui sont dans la même shared-mem
- références = elles sont interdites (car elles sont implémentées sous le capot avec des pointeurs)
- fonctions virtuelles = elles sont interdites (car la vtable n'est pas dans la shared-mem mais dans le VAS du process ; elle est donc propre à chaque process)
- static class members = attention : chaque process aura sa propre copie du membre statique ! (p.ex. ce n'est pas problématique s'ils ont des valeurs constantes initialisées une seule fois)

---

https://www.boost.org/doc/libs/1_80_0/doc/html/interprocess/synchronization_mechanisms.html

> As mentioned before, the ability to shared memory between processes through memory mapped files or shared memory objects is not very useful if the access to that memory can't be effectively synchronized.

^ la problématique de synchronisation est inhérite à l'IPC (j'en ai fait l'expérience dans mes POCs)

Deux types de synchronization dans boost = **named** et **anonymous** :

- named :
    - Named utilities: When two processes want to create an object of such type, both processes must create or open an object using the same name.
    - Each process uses a different object to access to the resource, but both processes are using the same underlying resource.
    - NdM : ici, inutile de créer explicitement le mécanisme de synchronisation dans une shared-mem : le partage entre process se fait avec le nom.
- anonymous :
    - Anonymous utilities: Since these utilities have no name, two processes must share the same object through shared memory or memory mapped files.
    - NdM : ici, les deux process utilisent le même mécanisme de synchronisation, qui vit dans une mémoire partagée.

D'après la doc, l'intérêt de l'anonyme, c'est de pouvoir sérializer le mécanisme de synchro sur disque pour réutilisation (je n'ai pas cet usage) :

> One could construct [an anonymous] synchronization utility in a memory mapped file, reboot the system, map the file again, and use the synchronization utility again without any problem. This can't be achieved with named synchronization utilities.

boost::ipc propose plusieurs mécanismes de synchronisation :
- mutex
- condition-variable
- semaphore
- sharable mutex
- upgradable mutex
- file lock
- message queue

D'une façon générale, les exemples boost::ipc tendent à confirmer que le mécanisme que j'ai utilisé pour mes POCs est le bon : on construit dans une shared_mem un Payload qui contient à la fois la donnée partagée et de quoi synchroniser son accès

Leurs résumés sont bien, je les reproduis ici.

- **What's A Mutex?**
    - Mutex stands for mutual exclusion and it's the most basic form of synchronization between processes. Mutexes guarantee that only one thread can lock a given mutex. If a code section is surrounded by a mutex locking and unlocking, it's guaranteed that only a thread at a time executes that section of code. When that thread unlocks the mutex, other threads can enter to that code region:
    - A mutex can also be recursive or non-recursive:
        - Recursive mutexes can be locked several times by the same thread. To fully unlock the mutex, the thread has to unlock the mutex the same times it has locked it.
        - Non-recursive mutexes can't be locked several times by the same thread. If a mutex is locked twice by a thread, the result is undefined, it might throw an error or the thread could be blocked forever.
- **What's A Condition Variable?**
    - In the previous example, a mutex is used to lock but we can't use it to wait efficiently until the condition to continue is met. A condition variable can do two things:
        - wait: The thread is blocked until some other thread notifies that it can continue because the condition that lead to waiting has disappeared.
        - notify: The thread sends a signal to one blocked thread or to all blocked threads to tell them that they the condition that provoked their wait has disappeared.
    - Waiting in a condition variable is always associated with a mutex. The mutex must be locked prior to waiting on the condition. When waiting on the condition variable, the thread unlocks the mutex and waits atomically.
    - When the thread returns from a wait function (because of a signal or a timeout, for example) the mutex object is again locked.
- **What's A Semaphore?**
    - A semaphore is a synchronization mechanism between processes based in an internal count that offers two basic operations:
        - Wait: Tests the value of the semaphore count, and waits if the value is less than or equal than 0. Otherwise, decrements the semaphore count.
        - Post: Increments the semaphore count. If any process is blocked, one of those processes is awoken.
    - If the initial semaphore count is initialized to 1, a Wait operation is equivalent to a mutex locking and Post is equivalent to a mutex unlocking. This type of semaphore is known as a binary semaphore.
    - Although semaphores can be used like mutexes, they have a unique feature: unlike mutexes, a Post operation need not be executed by the same thread/process that executed the Wait operation.
- **What's a Sharable and an Upgradable Mutex?**
    - Sharable and upgradable mutex are special mutex types that offers more locking possibilities than a normal mutex. Sometimes, we can distinguish between reading the data and modifying the data. If just some threads need to modify the data, and a plain mutex is used to protect the data from concurrent access, concurrency is pretty limited: two threads that only read the data will be serialized instead of being executed concurrently.
    - If we allow concurrent access to threads that just read the data but we avoid concurrent access between threads that read and modify or between threads that modify, we can increase performance. This is specially true in applications where data reading is more common than data modification and the synchronized data reading code needs some time to execute. With a sharable mutex we can acquire 2 lock types:
        - Exclusive lock: Similar to a plain mutex. If a thread acquires an exclusive lock, no other thread can acquire any lock (exclusive or other) until the exclusive lock is released. If any thread other has any lock other than exclusive, a thread trying to acquire an exclusive lock will block. This lock will be acquired by threads that will modify the data.
        - Sharable lock: If a thread acquires a sharable lock, other threads can't acquire the exclusive lock. If any thread has acquired the exclusive lock a thread trying to acquire a sharable lock will block. This locking is executed by threads that just need to read the data.
    - With an upgradable mutex we can acquire previous locks plus a new upgradable lock:
        - Upgradable lock: Acquiring an upgradable lock is similar to acquiring a privileged sharable lock. If a thread acquires an upgradable lock, other threads can acquire a sharable lock. If any thread has acquired the exclusive or upgradable lock a thread trying to acquire an upgradable lock will block. A thread that has acquired an upgradable lock, is guaranteed to be able to acquire atomically an exclusive lock when other threads that have acquired a sharable lock release it. This is used for a thread that maybe needs to modify the data, but usually just needs to read the data. This thread acquires the upgradable lock and other threads can acquire the sharable lock. If the upgradable thread reads the data and it has to modify it, the thread can be promoted to acquire the exclusive lock: when all sharable threads have released the sharable lock, the upgradable lock is atomically promoted to an exclusive lock. The newly promoted thread can modify the data and it can be sure that no other thread has modified it while doing the transition. Only 1 thread can acquire the upgradable (privileged reader) lock. 
     - NDM : avec ces mutex plus évolués vient de la complexité additionnelle, que je laisse de côté pour le moment, cf. Lock Transfers Through Move Semantics
- **What's A File Lock?**
    - A file lock is an interprocess synchronization mechanism to protect concurrent writes and reads to files using a mutex embedded in the file. This embedded mutex has sharable and exclusive locking capabilities. With a file lock, an existing file can be used as a mutex without the need of creating additional synchronization objects to control concurrent file reads or writes.
    - Generally speaking, we can have two file locking capabilities:
        - Advisory locking: The operating system kernel maintains a list of files that have been locked. But does not prevent writing to those files even if a process has acquired a sharable lock or does not prevent reading from the file when a process has acquired the exclusive lock. Any process can ignore an advisory lock. This means that advisory locks are for cooperating processes, processes that can trust each other. This is similar to a mutex protecting data in a shared memory segment: any process connected to that memory can overwrite the data but cooperative processes use mutexes to protect the data first acquiring the mutex lock.
        - Mandatory locking: The OS kernel checks every read and write request to verify that the operation can be performed according to the acquired lock. Reads and writes block until the lock is released.
- **What's A Message Queue?**
    - A message queue is similar to a list of messages. Threads can put messages in the queue and they can also remove messages from the queue. Each message can have also a priority so that higher priority messages are read before lower priority messages. Each message has some attributes:
        - A priority.
        - The length of the message.
        - The data (if length is bigger than 0).
    - A thread can send a message to or receive a message from the message queue using 3 methods:
        - Blocking: If the message queue is full when sending or the message queue is empty when receiving, the thread is blocked until there is room for a new message or there is a new message.
        - Try: If the message queue is full when sending or the message queue is empty when receiving, the thread returns immediately with an error.
        - Timed: If the message queue is full when sending or the message queue is empty when receiving, the thread retries the operation until succeeds (returning successful state) or a timeout is reached (returning a failure).
    - A message queue just copies raw bytes between processes and does not send objects. This means that if we want to send an object using a message queue the object must be binary serializable.

---

https://lists.boost.org/Archives/boost/2015/04/221802.php

^ même si la doc n'en fait pas mention, il y a bien des spurious wakeup (et il faut utiliser une boucle while pour utiliser une CV).

# Keypoints appris lors de mes POCS

## principe général du partage de mémoire entre deux process

L'idée est de faire un `mmap` sur un fichier dans un ramfs.

Commençons par voir ce qui se passe quand deux process mmappent depuis un fichier dans un stockage classique (sur disque-dur, pas en RAM) :

- le fichier est sur disque, et on suppose qu'il n'a jamais été lu (donc il n'est pas en RAM)
- mmap par le process A  :
    - le mmap dans le process A réserve une plage d'adresses virtuelle du process A, backée par le fichier sur disque (pour simplifier, on suppose que cette plage est représentée par une unique memory-page)
    - au premier accès en lecture de cette memory-page, on se mange une page-fault
    - le fichier est alors mis en RAM par le kernel, i.e. le kernel créée une memory-frame MF qui contient le contenu du fichier
    - à ce stade, la memory-page virtuelle du process A est backée par la memory-frame MF
    - le process A peut donc accéder à la mémoire dans la memory frame
- mmap par le process B :
    - le mmap dans le process B réserve une plage d'adresses virtuelle de B (pour simplifier : une unique memory-page)
    - ici, le fichier est déjà en RAM dans le kernel : la memory-page de B est backée par la memory-frame MF
    - c'est bien la MÊME memory-frame MF qui backe à la fois la page de A et la page de B !
- la mémoire est donc effectivement partagée entre les process, CQFD

À noter que dans le cas ci-dessus, le partage de la memory-frame entre les process est effectif sans attendre que les modifications soient écrites sur le disque-dur, cf. [ce qu'en dit la libc](https://www.gnu.org/software/libc/manual/2.36/html_node/Memory_002dmapped-I_002fO.html) :

> MAP_SHARED  Changes made will be shared immediately with other processes mmaping the same file. Note that actual writing may take place at any time.

Le partage interprocess via une shared-memory suit le même principe, sauf que le fichier est dans un ramfs plutôt que sur disque (et encore, on peut très bien faire de l'IPC avec un fichier sur disque).

À noter que je peux utiliser deux APIs pour faire ces opérations, system-V et POSIX, mais que l'utilisation de l'API POSIX (`shm_open`, `shm_unlink`) est recommandée, cf. `man shm_overview`. Et bien sûr, si j'utilise boost::interprocess, je n'ai pas à me soucier de ça.

Voir également les primitives de synchronisations POSIX : `man sem_overview` et `man mq_overview`.

## synchronisation entre les process - qui construit/détruit quoi quand ?

**TL;DR** : il faut que les process se synchronisent pour construire/détruire la shared-memory et son payload. Dans mes POCs, j'ai utilisé les moyens suivants :

La question de qui construit (resp. détruit) quoi et quand n'est pas un sujet facile :

- quand et par quel process est créée la shared-mem ?
- comment s'assurer que la shared-mem est crée et initialisée (le Payload est construit) au moment où on l'utilise ?
- quand et par quel process est détruite la shared-mem ?
- comment s'assurer que le Payload n'est plus utilisée par personne au moment où son destructeur est appelé ?
- et ces questions se généralisent aux structures interprocess partagées : named_mutex, message_queue, named_cv, etc.

C'est *très* facile de faire foirer le système (notable exemple = je n'avais pas `remove` en début de programme, or certains named-mutex avaient survécu, lockés) ; pire : quand quelque chose se passe mal, ça n'est pas toujours reproductible, et pas toujours débuggable facilement.

Dans mes POCs, j'ai utilisé les moyens suivants :

- **process "parent"** : la construction de la shared-mem / du payload ont été fait AVANT de forker. Au moment où les process forkés démarrent, tout est déjà initialisé, et ils peuvent utiliser les structures sans se poser de question.
- **mutex** : le process le plus rapide créée la shared-mem + construit le payload, le mutex s'assure que les deux opérations sont faites ensemble, les process suivants voient obligatoirement un ensemble déjà construit
- **message-queue** : l'un des process en particulier est responsable de créer+initialiser la shared-mem puis prévient l'autre via un message sur une queue ; l'autre process attend le message avant de commencer à bosser (à éviter car tordu : le mutex est supérieur IMO)
- **"dernier à bosser"** : l'un des process en particulier sera forcément le dernier à travailler → il peut être responsable du cleaning des structures interprocess sans se soucier de qui les utilise.
- (par contre, on ne peut pas utiliser de condition-variable, car une CV nécessite de partager une ressource, un flag évitant les spurious-wakeup, ce qu'on cherche justement à faire)
- (il y a sans doute des trucs smart à faire à base de registering de process, mais clairement c'est hors-scope de ces notes d'en discuter)

Pour construire (au sens C++ du terme) un Payload dans une shared-memory déjà existante, il faut utiliser [placement-new](https://en.cppreference.com/w/cpp/language/new) (qui se contente de construire un objet sans allouer de mémoire) ; symmétriquement, on détruit un objet construit avec placement-new par un inhabituel appel explicite au constructeur `ptr->~Payload();`.

À noter que les structures interprocess (e.g. les mutex partagés) **survivent** à la destruction de tous les process les utilisant, donc le sujet du cleaning en début de programme (pour reset un état préalable mal cleané) ou en fin de programme (pour laisser un état propre pour les suivants) est crucial.

## implémentation des boost::interprocess::named_mutex

J'ai fait une [POC spécifique](https://github.com/phidra/pocs/tree/59976b5105c639b04eca7af87802df07468521ef/cpp/CATEGORY_ipc/shmem_05_boost_ipc_named_mutex_experiences) pour mieux comprendre les named-mutex boost. Mes constatations :

- un named_mutex a une existence INDÉPENDANTE des process qui l'utilisent
- un process peut tout à fait locker un named_mutex, puis quitter sans le délocker
- on se retrouve alors dans un état où le named_mutex existe et est locké (!) alors même qu'aucun process ne l'utilise
- (d'où la nécessité de partir d'un état propre, ce qu'on peut faire a priori facilement via `remove` grâce au comportement spécial de `sem_unlink`, qui n'invalide pas les sémaphores existants, tout en permettant de réutiliser le nom pour un nouveau sémaphore)

Sur Linux, les mutex de boost::interprocess semblent implémentés avec les sémaphores POSIX ; du coup, une fois créé, un named_mutex apparaîtra dans :

```
/dev/shm/sem.my_super_mutex
```

(c'est un moyen de savoir si un mutex existe sans passer par un programme utilisant boost::ipc)

J'ai également vérifié que les condition-variables vivaient également sous `/dev/shm` (et obviously, c'est le cas aussi pour les shared-memory).

Pour les queues, c'est particulier : les queues POSIX vivent sous `/dev/mqueue`, consultables avec :

```
sudo mount -t mqueue none /dev/mqueue
```

Mais les queues de boost::interprocess [ne semblent pas utiliser les queues POSIX](https://stackoverflow.com/questions/63302676/boost-message-queue-with-posix-message-queue/63305924#63305924), et effectivement, je retrouve mes queues boost non-pas dans `/dev/mqueue`, mais plutôt également sous `/dev/shm/`, donc elles sont sans doute implémentées via des shared-memory.

## xxx_unlink et suppression des structures interprocess

La méthode statique `named_mutex::remove` appelle probablement `shm_unlink`, qui a un comportement particulier vis-à-vis de la suppression. Extraits de `man sem_unlink` :

> The sem_unlink() function shall remove the semaphore named by the string name

^ un process tiers peut donc supprimer un mutex créé par un précédent process, ce qui permet de faire un remove préalable

> If the semaphore named by name is currently referenced by other processes, then sem_unlink() shall have no effect on the state of the semaphore.
>
> If one or more processes have the semaphore open when sem_unlink() is called, destruction of the semaphore is postponed until all references to the semaphore have been destroyed by calls to sem_close(), _exit(), or exec.
>
> Calls to sem_open() to recreate or reconnect to the semaphore refer to a new semaphore after sem_unlink() is called.

^ C'est le point le plus intéressant :

- une fois qu'on a appelé sem_unlink, tout nouveau process peut "réutiliser le nom" avec `sem_open`, car le nouveau sémaphore aura certes le même nom, mais pointera sur un AUTRE sémaphore
- les anciens process qui utilisaient encore l'ancien sémaphore peuvent également continuer de le faire de façon safe
- le principe sous-jacent, c'est que le fichier est certes le même grâce (vu qu'il dépend du nom), mais qu'après `sem_unlink` + `sem_open`, le nouveau fichier est chargé dans le kernel dans des _nouvelles_ memory-frame
- les anciennes memory-frame existent toujours (ce qui explique que les anciens process peuvent toujours utiliser l'ancien sémaphore) mais elles ne sont plus associées au fichier : ce sont les nouvelles memory-frame qui sont associées au fichier

> The sem_unlink() call shall not block until all references have been destroyed; it shall return immediately.

^ dans tous les cas, `sem_unlink` ne bloque pas : il se contente éventuellement de retarder la suppression effective du mutex.

---

Et le fonctionnement est similaire pour `shm_unlink`, cf. `man shm_unlink` :

> The operation of shm_unlink() is analogous to unlink(2):
>
> it removes a shared memory object name, and, once all processes have unmapped the object, de-allocates and destroys the contents of the associated memory region.
>
> After a successful shm_unlink(), attempts to shm_open() an object  with  the  same name fail (unless O_CREAT was specified, in which case a new, distinct object is created)

---

Sous ces hypothèses, je me fixe comme bonne pratique d'attaquer toute nouvelle POC avec un `remove` préliminaire :

- il n'aura pas d'impact si le mutex n'existe pas (il se contentera de renvoyer false)
- si le mutex existe et n'est utilisé par personne, il remets les choses d'équerre
- si le mutex existe et est utilisé par des anciens process, ces anciens process pourront continuer de l'utiliser, et le nouveau process utilisera un mutex tout neuf
- le seul hic, c'est que s'ils existent toujours les anciens process risquent d'appeler `remove` à leur tour, ce qui n'invalidera non pas leur ancien mutex, mais le nouveau !


Un truc rigolo et bigrement utile pour s'assurer de ne pas oublier de supprimer des structures interprocess, c'est qu'on peut `shm_unlink` aussitôt que tous les process ont bien chargé la shared-mem dans leur VAS :

- en `shm_unlink` au plus tôt, on a moins de chances d'oublier de le faire en fin de programme
- même si on `shm_unlink` tôt, les process existants continueront d'utiliser la shared-mem sans souci (c'est bien après que tous les process ont unmappé la shared-memory qu'elle sera effectivement supprimée et désallouée)
- un éventuel nouveau programme voit un état propre sans aucune shared-mem, et peut même créer sa propre shared-mem du même nom
- par contre, plus aucun programme ne pourra mapper l'ancienne shared-memory dans un nouveau VAS : il ne faut `shm_unlink` qu'une fois que tous les process souhaité ont bien mmappé la shared-mem

C'est expliqué plus en détail ici : https://stackoverflow.com/questions/65860154/behaviour-of-shm-unlink/65860605#65860605

## forker après avoir mmappé

Lorsqu'on forke un process, des questions se posent sur mmap / munmap / le close des file-descriptors :

- peut-on forker après avoir mmappé (ou doit-on mmapper après le fork) ?
- si oui, chaque process fils doit-il `munmap` ou bien un seul doit-il le faire ?
- tous les process fils doivent-ils fermer les file-descriptors ou bien un seul doit-il le faire ?

TL;DR :

- on peut forker après avoir mmappé
- les `munmap` + `close` de file-descriptors sont bien à faire dans chaque process (le parent et le forké)
- en revanche, l'appel à `shm_unlink` ne doit être fait qu'une seule fois
- en généralisant, la destruction des structures interprocess ne doit être faite qu'une seule fois
- de même, l'appel au destructeur `~Payload` ne doit être fait qu'une seule fois

https://unix.stackexchange.com/questions/91058/file-descriptor-and-fork/91061#91061

> When a child is forked then it inherits parent's file descriptors, if child closes the file descriptor what will
> happen ? It inherits a copy of the file descriptor. So closing the descriptor in the child will close it for the
> child, but not the parent, and vice versa.

https://unix.stackexchange.com/questions/687403/how-does-a-process-and-its-children-use-memory-in-case-of-mmap/687413#687413

> On fork() the memory space of the parent process is cloned into the child process.
>
> "Both memory spaces have the same content" includes memory allocated with mmap().
> The memory mappings get cloned and mmap() or munmap() after the fork don't affect the other process anymore.
