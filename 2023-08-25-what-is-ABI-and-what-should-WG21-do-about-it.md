# What is ABI, and What Should WG21 Do About It?

- **url** = https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2028r0.pdf
- **type** = paper
- **auteur** = le [WG21 C++](https://isocpp.org/std/the-committee) = le comité C++
- **date de publication** = 2020-01-07
- **source** = open-std = _The site www.open-std.org is holding a number of web pages for groups producing open standards_
- **tags** = language>cpp ; topic>ABI ; level>advanced


**TL;DR** : des explications sur ce qu'est l'ABI, avec des exemples concrets de changements qui ne sont pas rétro-compatibles du point de vue de l'ABI.

Le contexte du papier : doit-on faire un C++23 qui corrige des problèmes de performance, mais casse la rétrocompatibilité de l'ABI ? Mais ce qui m'intéresse surtout, c'est de mieux comprendre ce que représente l'ABI et quels changements l'impactent.

## Définition de l'ABI

Dans le papier :

> ABI is the platform-specific, vendor-specified, not-controlled-by-WG21 specification of how entities (functions, types) built in one translation unit interact with entities from another.

^ comme les translation-units sont buildées indépendamment et conduisent à du code objet, l'ABI représente les interactions entre les entités d'une translation-unit avec les entités d'une autre translation-unit.

> including but not limited to:

> - The mangled name for a C++ function (Functions in extern “C” blocks handled
> -ifferently).
> - The mangled name for a type, including instantiations of class templates.
> - The number of bytes (sizeof) and the alignment, for an object of any type.
> - The semantics of the bytes in the binary representation of an object.
> - Register-level calling conventions governing parameter passing and function invocation.

^ quelques exemples de choses qui, si elles sont modifiées, rendront deux code-objets incompatibles du point de vue de l'ABI.

## Name mangling

> For a declaration of a C function void c_func(int), when a .o/.a/.so needs to call into that function, the call is left in symbolic form in the object code.
>
> In order to ensure that the right code is called when a program is fully linked and loaded, the compiler leaves those external calls in symbolic form as a call to` _c_func` or similar. During linking and loading, the final address of `c_func` is replaced accordingly.

^ c'est la base du lien entre name-mangling et ABI : dans le code-objet, l'appel à une fonction externe (i.e. définie dans une autre translation-unit) est laissé sous une forme symbolique du genre `call c_func`. C'est le dynamic-linker (pour une lib dynamique) ou le linker (pour le linking de fichiers statiques) qui part du nom symbolique pour aboutir à une adresse utilisable = celle de la fonction à appeler.

> Changing the name of an entity, changing the parameter list of a function, or (sometimes) changing the set of template arguments for a class template is an ABI change, in that it results in changing the mangled name for that entity

^ en C++, il n'y a pas que le nom d'une fonction qui est encodé dans la "forme symbolique" décrite ci-dessus : il y a également p.ex. la liste des paramètres.

Par conséquent, la liste des paramètres d'une fonction (ou son namespace, etc.) ne doit pas être modifiée dans une code-objet A, sinon un code-objet B utilisera un nom manglé différent de celui par lequel le code-objet A désigne la fonction.

> For instance, consider the discussion of std::scoped_lock<T...> in C++17. We wished to extend std::lock_guard<T> to allow for a variadic set of heterogeneous mutexes.
>
> However, we could not change lock_guard<T> to lock_guard<T...> because on some platforms the name mangling algorithm for variadic templates is different than the mangling algorithm for class templates of fixed arity.
>
> Code built with lock_guard<T> would be unable to pass a pointer or reference to such an entity to code built with lock_guard<T...> on those platforms - an “ABI break”.

^ un exemple qui est la conséquence de ce point : on ne peut pas ajouter un paramètre template à `std::lock_guard`, car ça modifierait le name manglé, et donc serait ABI-incompatible.

> Name mangling represents an important syntactic level of compatibility for object code: do two translation units compiled at different times agree on the mangled name for an entity?

^ bon résumé : "est-ce que deux TU désignent une entité par le même nom ?"

## Object Representation

> Equally important is the semantic compatibility for objects: does an object compiled at one time and stored into memory work properly when interpreted by code compiled at another time?

^ quelle sémantique attribue-t-on à un paquet de bit ? En tout cas, ladite sémantique doit être la même pour deux TU différentes qui manipulent le même paquet de bits.

> Before C++11, (...) the class layout for std::string itself was merely a pointer: the actual data, capacity, size, and reference count were accessible at fixed offset from the pointed-to allocation.
>
> C++11 disallowed the COW behavior. A new/efficient std::string implementation without COW has fewer indirections, storing all of the relevant information directly in the object no control block) (...) Even a simple std::string is going to be roughly 3 words: a pointer to the data, and one word each for the capacity and current size of the data.

^ exemple = les `std::string`, avant et après C++11, elles ne sont pas représentées de la même façon, 1 pointeur vs. 3 words.

> Building one translation unit with the COW string and then passing a COW string to a function that accepts std::string built with the SSO implementation may link (the mangled name has not changed, necessarily), but will fail spectacularly at run time. The COW string is only passing one word to the function. The function is reading 3 words and interpreting those as (perhaps) (data, capacity, size).

^ du coup, si une fonction dans une TU attends une nouvelle `std::string` de 3 words et qu'une autre TU lui passe une `std::string` d'un seul pointeur, ça va faire boum car la nouvelle TU va lire 2 words de trop.

Dans le même genre, on n'itère pas sur un `vector<string>` de la même façon s'il contient des nouvelles ou d'anciennes `std::string` : dans un cas on avance 3 words par 3 words, et dans l'autre un pointeur par un pointeur.

> Changing object representation is obviously an ABI break. This includes functional no-ops like reordering fields, changing padding or object packing.

^ d'autres exemples qui ont l'air innocents car ils sont source-compatibles, mais qui ne sont pas ABI-compatibles car ils changent la façon dont on interprète les bits représentant un objet.

## Object Representation and Semantics

> Importantly, it isn’t only changes to object layout that may result in this sort of ABI incompatibility. Any change to the semantic meaning of the binary representation of an object in memory is an ABI break.

^ d'une façon générale, tout changement dans la façon dont **on interprète** les bits représentant un objet peut poser problème, même si les bits eux-mêmes ne changent pas.

> Among other things, this means that we cannot change std::hash without an ABI incompatibility. The hash value for an object computed in one translation unit is embedded in how it is stored in an unordered_map. If that unordered_map is passed to a TU that has a different definition of std::hash for the key, then we will be unable to find that object in the map.

^ l'exemple donné est celui d'une hashmap : si on change la façon dont on hashe un type, une hashmap le contenant changera.

Dit autrement : pour un groupe de bits représentant une hashmap, la façon dont on l'interprète est importante.

> Both translation units must agree perfectly on the number of bits for every object and the meaning of those bits.

^ le résumé du lien entre ABI et object representation.

## Calling Convention

> there is the language-level ABI, which for these purposes largely boils down to the question of “How do we call a function that was built in a different translation unit?” What values are passed in registers, how many registers can be used in that fashion, what type properties govern whether a type may be passed in registers vs. in memory?

^ pour qu'une TU puisse appeler une fonction d'une autre TU, les deux doivent être d'accord sur comment passer les paramètres (stack ou RAM).

Sous Linux, on utilise [l'ABI Itanium](https://github.com/itanium-cxx-abi/cxx-abi).

L'exemple donné est celui d'un `unique_ptr<T>` : comme il a un destructeur, il ne peut pas être passé dans un registre, mais ne peut être passé qu'en mémoire (sur la stack), à la différence d'un `T*` ; du coup, une fonction acceptant des `unique_ptr<T>` sera (un peu) moins performante que la même fonction acceptant des `T*`.

Changer la façon dont on passe un `unique_ptr<T>` à une fonction (afin de résoudre ce souci de performances) est un changement non-ABI-compatible (NDM : puisqu'une TU utilisant l'ancienne convention va passer le `unique_ptr` en mémoire plutôt qu'en registre).

## It is “Just” a Binary Format

> In many respects, ABI definitions are similar to the way we define a binary file format or network protocol. If the sender and receiver disagree on how to interpret a file or a network message, there is going to be an error.

^ j'aime bien ce parallèle : l'ABI est un format d'échange, qu'utilisent deux TU qui doivent communiquer = l'une appelle une fonction de l'autre.

> Those disagreements can be syntactic (framing, message size, structure) or semantic (is this sequence of bits a checksum or a size).

^ au passage, il faut distinguer deux types de différences :

- syntaxiques = la forme n'est pas compatible (e.g. une TU source passe 3 bytes là où la TU target attends 4 bytes)
- sémantiques = même si la forme est compatible (e.g. les deux TU utilisent 2 bytes), le sens attribué au message n'est pas le même (e.g. une TU attribue le sens `origin` au byte 1 et `destination` au byte 2, et l'autre TU fait le contraire)

> Every function is a server, every function call is a client. Clients and servers must be updated in lock-step, or not updated, or must be updated with great care to ensure that clients never transmit something to the server that will be misinterpreted.

^ la suite de la métaphore = c'est compliqué d'introduire des changements dans le format d'échange.

> Following this client/server comparison, it is noteworthy that many binary formats explicitly encode a version number in order to properly identify version skew issues. Our current ABIs may or may not have such identification, which makes ABI changes even more troublesome.

^ toujours pour filer la métaphore, comme n'importe quel format d'échange, on peut versionner l'ABI.

## Why Would We Consider ABI Changes ?

>  in aggregate, providing a stable ABI for the past decade costs C++ generally and the standard library in particular perhaps 5-10% aggregate performance cost . It prevents many API changes, because of changes to mangling, object size, or memory representation. It precludes adoption of new optimizations for existing types

^ garder une ABI stable empêche tellement d'évolutions que ça a un coût énorme en termes de perfs.

Jusqu'à présent, le comité C++ a préféré privilégier la stabilité de l'ABI (i.e. qu'un exécutable récent puisse tout de même utiliser/linker avec une lib ou tout fichier objet buildé il y a longtemps, avec une ancienne ABI) au détriment des perfs.

Il enchaîne en listant des modifs d'ABI (pour la plupart, source-compatibles, i.e. qui ne nécessitent pas de modifier le code-source de l'appelant) qui amélioreraient les perfs, mais qui sont jusqu'ici refusées pour garder l'ABI stable :

- changer les `std::unordered_map` et `std::hash` pour des implémentations plus efficaces
- TO BE CONTINUED
