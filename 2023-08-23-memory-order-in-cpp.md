# Memory Order in C++

- **url** = https://www.sobyte.net/post/2022-06/cpp-memory-order/
- **type** = article
- **auteur** = ???
- **date de publication** = 2022-06-26
- **source** = [sobyte](https://www.sobyte.net/), site orienté low-level / backend / go et C++
- **tags** = language>cpp ; topic>cpp-concurrency ; topic>cpp-memory-order ; level>intermediate

TL;DR = très bonnes explications des `memory-order` en C++ avec des exemples très clairs et illustratifs.

En résumé, les memory-order sont un sujet exclusivement liés aux atomics, et régissent la façon dont deux threads différents qui utilisent le même atomic voient les opérations effectuées sur celui-ci (notamment l'ordre dans lequel les opérations sont vues).

* [Memory Order in C++](#memory-order-in-c)
   * [2. Basic Concepts](#2-basic-concepts)
   * [3. Memory order](#3-memory-order)
      * [3.2 memory_order_relaxed](#32-memory_order_relaxed)
      * [3.3 Acquire-release](#33-acquire-release)
      * [3.4* Release sequences](#34-release-sequences)
      * [3.5* memory_order_consume](#35-memory_order_consume)
   * [4. Some examples](#4-some-examples)
      * [4.1 Spin locks](#41-spin-locks)
      * [4.2 Thread-safe singleton pattern](#42-thread-safe-singleton-pattern)
   * [5. Summary](#5-summary)

----


## 2. Basic Concepts

Au chapitre des concepts basiques, il y a [un très bon exemple](https://www.sobyte.net/post/2022-06/cpp-memory-order/#21-modification-orders) appelé `modification order`, qui aide à comprendre grâce à un exemple concret que quel que soit le memory order utilisé, tous les threads **liront** les différentes opérations effectuées sur un atomic dans le même ordre final, même si cet ordre n'est pas celui qui a été utilisé à l'écriture :

- deux threads écrivent chacun les nombres pairs et impairs dans un atomic
- deux autres threads lisent régulièrement l'atomic et efficient son état
- alors l'état affiché par les deux threads qui lisent sera peut être 1. différent entre les deux threads lecteurs et 2. non-répétables, i.e. différents d'une exécution à une autre...
- ... par contre, l'ordre des lectures par les deux threads seront respectées, i.e. si 9 apparaît avant 8 dans un lecteur, alors 9 ne peut pas apparaître 8 dans l'autre lecteur

EDIT : on comprend plus bas que ceci est toujours vrai... pour un unique atomic ! Toute la question des memory model est de savoir si c'est le cas quand les threads manipulent **plusieurs** atomics différents.

Autre concept :

> Happens-before is a very important concept. If operation a “happens-before” operation b, then the result of operation a is visible to operation b. The happens-before relationship can be established between two operations using one thread, or between two operations in different threads.

^ `happens-before` = le résultat d'une opération 1 est visible par l'opération 2

Dans le cas monothreadé, c'est simple : les LOC `happens-before` dans leur ordre dans le code source car elles sont `sequenced-before`

Le cas multithreadé est plus complexe :

- deux opérations dans deux threads différents peuvent être synchronisées (la relation est appelée `synchronizes-with` ; elle est définie plus tard dans l'article)
- dans ce cas, la première opération `inter-thread happens-before` la seconde, et cette relation est **différente** de la relation `happens-before`
- `sequenced-before` étant la plus forte des relations, quand elle est accolée à une opération `inter-thread happens-before`, l'agrégat des deux devient une opération `inter-thread happens-before`

Attention que tout ceci est un modèle utilisé par le C++ : les "vraies" instructions effectuées par le processeur peuvent être différentes, tant que ça ne change pas le résultat observable.

## 3. Memory order

https://www.sobyte.net/post/2022-06/cpp-memory-order/#3-memory-order

> We mentioned earlier that C++’s six memory orders can be combined with each other to implement three sequential models

^ il y a une différence entre **memory order** et **memory model** : les premiers sont des outils pour fabriquer les seconds.

> In Section 2.1 we introduced the modification order, where the modification order of a single variable is consistent across all threads. Sequencial consistency extends this consistency to all variables

^ avec la sequential consistency, si deux opérations **sur deux atomics différents** ont lieu dans un certain ordre dans un thread, alors elles seront obligatoirement vues dans le même ordre par un autre thread (le modification order garantissait déjà ça pour un unique atomic). Dit autrement, les opérations ont lieu dans l'ordre du code source.

NDM : de mémoire, même si ce modèle est le plus simple et le plus intuitif, c'est aussi le moins efficace. EDIT : confirmé un peu plus bas.

En utilisant ce modèle, on peut obtenir la relation `synchronizes-with` entre deux threads, évoquée précédemment : un thread écrit `true` sur un atomic, et un autre thread a un `while` qui lit l'atomic en attendant qu'il soit true : alors effectivement, le deuxième thread attend le premier.

Ici aussi , l'exemple donné aide beaucoup à comprendre tout ceci.

> There is some overhead in implementing a sequencial consistent model. Modern CPUs usually have multiple cores, each with its own cache. To achieve global sequential consistency, each write operation must be synchronized to the other cores. To reduce the performance overhead, if global sequential consistency is not needed, we should consider using a more relaxed sequential model.

^ ça confirme ce que je disais plus haut.

### 3.2 memory_order_relaxed

https://www.sobyte.net/post/2022-06/cpp-memory-order/#32-memory_order_relaxed

> memory_order_relaxed can be used for store, load and read-modify-write operations to implement the relaxed order model. In this model, only atomicity and modification order of operations are guaranteed, and no synchronizes-with relationship can be achieved

^ l'exemple donné montre qu'avec relaxed, deux opérations sur deux atomic différents effectuées par un thread peuvent être vues effectuées dans un autre ordre par un autre thread : l'ordre relatif des opérations n'est plus garanti d'être le même au moment de l'écriture et au moment de la lecture.

Ce memory order ne permet donc pas de synchroniser des threads.

> The overhead of the relaxed sequential model is small

^ ça conduit à un order model moins intuitif, mais plus efficace.

> The Relaxed model can be used in scenarios where thread synchronization is not required, but care should be taken when using it. For example, std::shared_ptr is used to increase the reference count because it does not need to be synchronized; but it cannot be used to reduce the application count because it needs to be synchronized with the destructor operation.

^ un exemple concret pour expliquer dans quel cas on peut avoir besoin de l'un ou de l'autre model.

### 3.3 Acquire-release

https://www.sobyte.net/post/2022-06/cpp-memory-order/#33-acquire-release

> In the acquire-release model, the three memory orders memory_order_acquire, memory_order_release and memory_order_acq_rel are used.

^ illustration que les memory order sont des briques que l'on combine pour former un order model.

Les explications qui suivent montrent que acquire-release permet (à la différence de relaxed) de synchroniser des threads.

De ce que j'en comprends, acquire-release est moins fort que `memory_order_seq_cst` mais permet tout de même de synchroniser des threads, tout en étant plus performant (moins d'overhead) :

> Therefore, many scenarios that require a synchronizes-with relationship will use acquire-release.

### 3.4* Release sequences

https://www.sobyte.net/post/2022-06/cpp-memory-order/#34-release-sequences

Exemples plus détaillés, je skimme.

Introduit le concept de **release-sequence**

### 3.5* memory_order_consume

https://www.sobyte.net/post/2022-06/cpp-memory-order/#35-memory_order_consume

Exemples plus détaillés, je skimme.

Introduit les concepts de **carries dependency** et de **dependency-ordered before**

## 4. Some examples

https://www.sobyte.net/post/2022-06/cpp-memory-order/#4-some-examples

### 4.1 Spin locks

https://www.sobyte.net/post/2022-06/cpp-memory-order/#41-spin-locks

> In some scenarios, if the lock is occupied for a short time, we choose a spinlock to reduce the overhead of context switching

^ on utilise un spinlock quand on a besoin d'un lock, et que celui-ci n'est jamais locké très longtemps : autant attendre que le locke se libère plutôt que de risquer un context-switch.

> we want only one thread to have access to the lock at the same time, and the data protected by the lock to be always up-to-date when the lock is acquired. The former is guaranteed by atomic operations, while the latter requires consideration of memory order

^ dans ce contexte, le memory order intervient pour que quand un thread récupère le lock, il voie bien la data (que le lock protégeait) dans un état cohérent (NDM : ce qui n'est par exemple pas le cas si l'opération de mise à jour de la data et l'opération de release du lock effectuées par le thread écriveur sont vues en ordre inverse par le thread lecteur)

L'exemple qui suit montre l'implémentation d'un spinlock grâce à un atomic et le modèle "acquire-release".


### 4.2 Thread-safe singleton pattern

https://www.sobyte.net/post/2022-06/cpp-memory-order/#42-thread-safe-singleton-pattern

L'exemple montre d'abord une mauvaise implementation d'un singleton multithread avec un mutex, puis une approche correcte avec un atomic. La question est "quel memory-order" ?

Je skimme.

## 5. Summary

https://www.sobyte.net/post/2022-06/cpp-memory-order/#5-summary

Si je m'intéresse de près au sujet, le résumé est très bien.
