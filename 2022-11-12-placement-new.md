# placement new


- **url** = https://h-deb.clg.qc.ca/Sujets/Divers--cplusplus/new_positionnel.html
- **type** = article
- **auteur** = [Patrice ROY](https://h-deb.clg.qc.ca/) professeur Montréalais, qui en plus d'être *très* pédagogique, de faire partie du [Working Group normalisant le C++](http://www.open-std.org/jtc1/sc22/wg21/), est super gentil :-)
- **date de publication** = ?
- **source** = [son site](https://h-deb.clg.qc.ca/)
- **tags** = language>cpp ; topic>placement-new ; level>intermediate

TL;DR : notes sur placement new (en réalité, issues d'un peu plus de sources que juste le post de Patrice ROY). Sa conclusion :

> le new positionnel est puissant et polyvalent, mais demande de la délicatesse. Mieux vaut l'encapsuler.

---

`new` fait deux choses :
- réserve avec malloc un bloc de mémoire brute
- appelle le constructeur d'un objet sur ce bloc mémoire

`delete` fait l'opération inverse :
- appeler le destructeur sur la mémoire de l'objet (ce qui transforme la mémoire de l'objet en mémoire "brute")
- désallouer la mémoire

On dirait qu'il faut distinguer :
- la `new expression` ([lien cppreference](https://en.cppreference.com/w/cpp/language/new)) ; ma compréhension, c'est qu'elle appelle la fonction d'allocation pour allouer la mémoire, puis appelle le constructeur.
- la fonction d'allocation (= l'operator new) ce qui est utilisé par la new-expression pour allouer de la mémoire ([lien cppreference](https://en.cppreference.com/w/cpp/memory/new/operator_new)) : tout comme malloc, elle prend en argument la quantité de mémoire à allouer :
	> When calling the allocation function, the new-expression passes the number of bytes requested as the first argument, of type std::size_t, which is exactly sizeof(T) for non-array T.

On peut spécialiser cette fonction d'allocation :

> As described in allocation function, the C++ program may provide global and class-specific replacements for these functions. If the new-expression begins with the optional :: operator, as in ::new T, class-specific replacements will be ignored (the function is looked up in global scope).

L'operator-new gère uniquement l'allocation et pas la construction.

Lorsqu'il n'est pas customisé, placement new correspond simplement à la construction d'un objet (i.e. appel du constructeur) sur une mémoire brute déjà allouée, donc sans allocation de mémoire :

> Placement new \
> If placement-params are provided, they are passed to the allocation function as additional arguments. \
> Such allocation functions are known as "placement new", after the standard allocation function void* operator new(std::size_t, void*), which simply returns its second argument unchanged. This is used to construct objects in allocated storage.


La page de Patrice montre qu'on peut tout à fait faire un make_unique d'un tableau de char (ce qui revient à faire un `new char[]`, je suppose).

En revanche, j'ai l'impression que le `reinterpret_cast` est inévitable.

L'équivalent du placement new (qui construit un objet sur une mémoire déjà allouée, i.e. qui transforme une zone de mémoire brute en un objet) est `std::destroy` : elle détruit un objet sans désallouer la mémoire (i.e. elle transforme la mémoire d'un objet en mémoire brute).

Il mentionne au passage un truc que je ne savais pas : un vector est typiquement plus rapide qu'un tableau de T !

Les risques à gérer la mémoire manuellement avec placement-new :

- avoir des accès non-alignés
- destruction : attention à ce qu'on détruit (un tableau de char ? un tableau de T ?)

Autres références :
- https://stackoverflow.com/questions/222557/what-uses-are-there-for-placement-new
    > Placement new allows you to construct an object in memory that's already allocated.
    - Exemple d'usage 1 = si on veut créer / détruire plusieurs fois un objet, autant ne pas désallouer/réallouer la mémoire à chaque fois, et plutôt la réutiliser.
    - Exemple d'usage 2 = s'assurer que la partie "allouer la mémoire" de la construction n'échouera pas (vu qu'on construit l'objet sur une mémoire déjà allouée)
    - Comment désallouer la mémoire qu'on a utilisée avec placement-new = comme la construction a été faite en deux étapes (1. allocation RAM brute avec `new char[]` 2. construction avec placement-new), alors il faut détruire/désallouer en deux étapes :
        1. détruire les objets sans désallouer (e.g. `std::destroy` ou appel explicite au destructeur `p->~T()`)
        2. désallouer la mémoire brute avec un `delete[]`
- https://stackoverflow.com/questions/57948950/placement-new-on-already-existing-object-in-sharedmemory = explique un peu une key difference entre mes pocs 1 et 2 sur la shmem = la différence entre un placement-new et un cast de pointeur :
    - le cast de pointeur se contente d'interpréter une zone de mémoire comme un objet : il n'y a PAS d'appel au constructeur (ce que fait le post qui a inspiré ma POC 2)
    - le placement-new fait un truc en plus : il appelle le constructeur sur la zone de mémoire (ce que fait le post qui a inspiré ma POC 1)
