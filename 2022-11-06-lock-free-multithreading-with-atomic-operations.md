# Lock-free multithreading with atomic operations


- **url** = https://www.internalpointers.com/post/lock-free-multithreading-atomic-operations
- **type** = article
- **auteur** = difficile à dire sur ce site
- **date de publication** = 2019-11-13
- **source** = [le site internal pointers](https://www.internalpointers.com/)

- **tags** = language>C++ ; topic>multithreading ; topic>concurrency ; topic>lock-free-programming ; level>advanced


**TL;DR** : explications des opérations lock-free, des algos qui vont avec.

Mon contexte était très précis : si j'ai un `std::atomic_bool` dans une shared-memory, comment différent process peuvent-ils le partager s'ils n'ont pas de structure commune pour se synchroniser, et que l'atomic_bool lui-même n'en a pas ? [Cette page](https://stackoverflow.com/questions/51463312/are-lock-free-atomics-address-free-in-practice) et [celle-ci](https://stackoverflow.com/questions/50298358/where-is-the-lock-for-a-stdatomic) indiquent que ça n'est que si l'atomic est lock-free qu'il peut être partagé entre différents process.

La réponse que j'avais supposée (et qui tend à être confirmée par le présent article) = avec un atomic lock-free, pas besoin de structure de synchronization : c'est le CPU lui-même qui garantit que la lecture ou l'écriture de la donnée se fait par une instruction qui est atomique au niveau hardware.

* [Lock-free multithreading with atomic operations](#lock-free-multithreading-with-atomic-operations)
   * [lock-free programming et atomic operations](#lock-free-programming-et-atomic-operations)
   * [Exemple 1 d'une incrémentation](#exemple-1-dune-incrémentation)
   * [algos lock-free pour des structures plus évoluées](#algos-lock-free-pour-des-structures-plus-évoluées)
   * [Exemple 2 compare-and-swap](#exemple-2-compare-and-swap)
   * [spinlock is a gentle form of locking](#spinlock-is-a-gentle-form-of-locking)
   * [Lock-freedom vs. wait-freedom](#lock-freedom-vs-wait-freedom)
   * [Attention à bencher](#attention-à-bencher)

L'article fait partie d'une série d'articles (e.g. un autre où les primitives plus classiques de synchronization sont abordées, ou encore un sur les memory-barriers).

## lock-free programming et atomic operations

Problème avec les mécanismes bloquants (comme les mutex) =
- bloquent le thread
- ça peut bloquer (et deadlock)
- pas de contrôle sur quel thread dort/est réveilé


Il y a d'autres façon de faire des tâches concurrentes qu'avec des primitives de synchronization bloquantes = lock-free programming.

Inconvénient = c'est bas-niveau, et compliqué à faire correctement.


> Lock-free programming relies upon atomic instructions, operations performed directly by the CPU that occur atomically.

^ Il existe bien des instructions qui sont atomiques au niveau hardware.

> Depending on the CPU architecture, some machine instructions are atomic, that is they are performed in a single, uncuttable and uninterruptible step.

^ ditto (et comme il s'agit d'instructions hard, ça dépend de l'architecture)


> More specifically, atomic instructions can be grouped into two major classes: store and load and read-modify-write (RMW).

Deux catégories :

- store and load : on parle d'une écriture ou d'une lecture de donnée ; ce sont les briques de base. Pour certains types de données (e.g. je suppose un byte voire un int32 aligné), une lecture/écriture est atomique.
- read-modify-write : quelques exemples :
    - `test-and-set` — writes 1 to a memory location and returns the old value in a single, atomic step;
    - `fetch-and-add` — increments a value in memory and returns the old value in a single, atomic step;
    - `compare-and-swap` (CAS) — compares the content of a memory location with a given value and, if they are equal, modifies the contents of that memory location to a new given value.

> All these instructions perform multiple things in memory in a single, atomic step. This is an important property that makes read-modify-write instructions suitable for lock-free multithreading operations.

Je reviens plus bas avec un exemple sur CAS.


> All the instructions seen above belong to the hardware: they require you to talk directly to the CPU.

^ On en vient à ce qui m'intéressait à la base : ces instructions atomiques sont fournies par le hardware lui-même.

> Climbing up to the software level, many operating systems provide their own versions of atomic instructions. Let's call them atomic operations, since we are abstracting away from their physical machine counterpart. (...) The best way to perform portable atomic operations is to rely upon the ones provided by the programming language of choice.

^ Les opérations atomiques qu'on va utiliser concrètement sont plutôt les wrappers fournies par l'OS, voires celles fournies par l'implémentation de C ou C++.

> For example, GCC — a C++ compiler — usually transforms C++ atomic operations and objects straight into machine instructions

En C++, le compilo génère directement les instructions hardware nécessaires pour les opérations atomiques.


## Exemple 1 d'une incrémentation

Avec des primitives de synchronisation :

```
reader_thread()
    mutex.lock()
    print(x)
    mutex.unlock()

writer_thread()
    mutex.lock()
    x++
    mutex.unlock()
```

Avec des primitives lock-free :

```
reader_thread()
    print(load(x))

writer_thread()
    fetch_and_add(x, 1)
```

> The atomicity of load() makes sure that no reader thread will read the shared value half-complete, as well as no writer thread will damage it with a partial write thanks to fetch_and_add().

NdM : c'est sans doute ce qui permet à `atomic_bool` d'être utilisé sans risque même par plusieurs process : chaque process utilise une opération atomique pour lire ou écrire l'atomic-bool dans la zone mémoire partagée.


## algos lock-free pour des structures plus évoluées

> Now, this example reveals us an important property of atomic operations: they work only with primitive types — booleans, chars, shorts, ints and so on.
>
> You don't protect a shared resource with atomic operations directly, as you would do with a mutex or a semaphore. Rather, you build lock-free algorithms or lock-free data structures, based on atomic operations to determine how multiple threads will access your data.

Pour des structures plus évoluées, on a des algos lock-free dédiés (qui utilisent les primitives lock-free comme briques de base)

> Writing these atomic weapons from scratch is hard, let alone making them work correctly. This is why most of the time you may want to employ existing, battle-tested algorithms and structures instead of rolling your owns.

Rule of thumb = ne pas écrire soi-même ses algos lock-free !

## Exemple 2 compare-and-swap

Intéressant exemple de lock-free fetch-and-add basé sur compare-and-swap

En gros, **compare-and-swap** veut dire "attribue atomiquement cette nouvelle valeur à la donnée", mais avec un twist ! Le twist, c'est que c'est plutôt :

> attribue atomiquement cette nouvelle valeur à la donnée, à condition que l'ancienne valeur soit bien celle que tu crois qu'elle est

Typiquement, pour utiliser CAS pour incrémenter :

- on fait un premier `load` pour regarder la valeur actuelle `V` (qui est la future ancienne valeur)
- on compare-and-swap : ancienne valeur = V, nouvelle valeur = V +1 :
    - cas nominal : si aucun autre thread n'a modifié V entre temps (l'ancienne valeur est bien toujours V), et on lui sette V+1 de façon atomique
    - contention : si un autre thread a modifié V entre temps, l'ancienne valeur n'est plus V, compare-and-swap ne fait rien (et on peut refaire un nouveau cycle de read + compare-and-swap)

> As said before, the CAS loop introduces a recurring pattern in many lock-free algorithms:
> - create a local copy of the shared data;
> - modify the local copy as needed;
> - when ready, update the shared data by swapping it with the local copy created before.
>
> Point 3) is the key: the swap is performed atomically through an atomic operation. The dirty job is done locally by the writer thread and then published only when ready. This way another thread can observe the shared data only in two states: either the old one, or the new one. No half-complete or corrupted updates, thanks to the atomic swap.

Un point intéressant, c'est le laps de temps où la donnée est "protégée" :

>  This is also philosophically different from the locking approach: in a lock-free algorithm threads get in touch only during that tiny atomic swap, running undisturbed and unaware of others for the rest of the time. The point of contact between threads is now shrinked down and limited to the duration of the atomic operation.


##  spinlock is a gentle form of locking

> The spin until success strategy seen above is employed in many lock-free algorithms and is called spinlock: a simple loop where the thread repeatedly tries to perform something until successful. It's a form of gentle lock where the thread is up and running — no sleep forced by the operating system, although no progress is made until the loop is over. Regular locks employed in mutexes or semaphores are way more expensive, as the suspend/wakeup cycle requires a lot of work under the hood.

spinlock = essayer en boucle de faire quelque chose jusqu'à ce qu'on réussisse. L'intérêt est d'éviter d'avoir à endormir le thread, ce qui nécessite un coûteux context-switch.

(je crois me souvenir avoir lu que certaines implémentations faisaient les deux : commençaient par essayer de spinlock pour quelques cycles le temps de voir s'ils accèdent à la ressource, puis endorment le thread sinon)

## Lock-freedom vs. wait-freedom

- lock-free = au moins un thread progresse (mais un thread donné peut starve indéfiniment ; c'est le cas dans l'implémentation de fetch-and-add avec un CAS ci-dessus : d'autres threads pourront modifier la variable derrière notre dos indéfiniment)
- wait-free = tous les threads finiront en un nombre fini d'étapes

## Attention à bencher

> Sometimes a good old mutex can outperform fancier synchronization primitives, especially when the concurrent task complexity is high.

Parfois (notamment quand il y a beaucoup de contention), un mutex sera plus efficace que lockless -> il faut toujours bencher !
