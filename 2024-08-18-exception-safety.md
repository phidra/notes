# Exception safety

- **url** = https://en.wikipedia.org/wiki/Exception_safety
- **type** = page wikipedia
- **auteur** = contributeurs wikipedia
- **date de publication** = multiples
- **source** = wikipedia
- **tags** = language>C++ ; topic>excetions ; level>intermediate


TL;DR = très bonnes explications sur le principe d'exception-safety en C++ :

- le contexte est celui où l'implémentation d'une méthode `M` peut rencontrer une exception, par exemple lancée par une sous-fonction que le corps de la méthode appelle
- si ça se produit, quelles garanties offre alors la méthode `M` ?
- par exemple, l'implémentation de `Vector::push_back` peut allouer de la mémoire ; comment se comporte `push_back` si l'allocation lance une exception ?
- **no throw** : `M` n'échouera jamais, même si l'une de ses sous-fonctions exceptionne (i.e. `M` se débrouille pour contourner l'erreur)
- **strong safety = commit-or-rollback** : tout se passe comme si `M` n'avait pas été appelée : non seulement les invariants du `Vector` resteront respectés, mais en plus son état interne n'est pas modifié
- **basic safety** : les invariants du `Vector` restent respectés (e.g. `begin()` pointera sur une adresse mémoire valide), par contre son état interne pourra être modifié (e.g. le `push_back` peut avoir pour conséquence de... vider le `Vector` !)
- **no safety** : il peut se passer n'importe quoi, y compris violer un invariant (p.ex. si l'allocation exceptionne `begin()` pointera sur une adresse mémoire invalide)

Il y a parfois conflit entre de très bonnes perfs et une forte safety (qui nécessite des copies pour être atteinte), on a donc parfois un trade-off à faire.

----

> Exception safety is the state of code working correctly when exceptions are thrown.

^ le code continue à fonctionner même si les fonctions qu'il appelle lancent des exceptions.

La situation suivante est buggée si une exception est lancée :

> - A step of an operation on a mutable data structure modifies the data and breaks an invariant.
> - An exception is thrown and control "bubbles up", skipping the rest of the operation's code that would restore the invariant
> - The exception is caught and recovered from, or a finally block is entered
> - The data structure with broken invariant is used by code that assumes the invariant, resulting in a bug
>
> Code with a bug such as the above can be said to be "exception unsafe"

Il y a plusieurs niveaux d'exception safety :

> No-throw guarantee, also known as failure transparency: Operations are guaranteed to succeed and satisfy all requirements even in exceptional situations. If an exception occurs, it will be handled internally and not observed by clients.

^ c'est le niveau le plus safe : un appel de fonction f n'échouera jamais, même si une sous-fonction appelée par f lève une exception (ce qui veut donc dire que f sait gérer ses erreurs pour faire son métier avec succès malgré des levées d'exceptions dans son implémentation)

> Strong exception safety, also known as commit or rollback semantics: Operations can fail, but failed operations are guaranteed to have no side effects, leaving the original values intact.

^ c'est un niveau de sûreté très fort, mais qui peut échouer : si l'appel à f échoue, tout se passe comme si on n'avait pas appelé f : il y a un "rollback" en cas d'erreur. Non seulement les invariants sont préservés, mais également les valeurs originales.

> Basic exception safety: Partial execution of failed operations can result in side effects, but all invariants are preserved. Any stored data will contain valid values which may differ from the original values. Resource leaks (including memory leaks) are commonly ruled out by an invariant stating that all resources are accounted for and managed.

^ c'est un niveau moins fort, c'est le niveau minimal pour qu'un code ne soit pas buggé : si une opération nécessaire à f échoue, les invariants sont certes préservés, mais l'état de l'objet a pu être modifié d'une façon qui ne nous convient pas.

> No exception safety: No guarantees are made.

^ c'est le niveau min, généralement considéré comme buggé : si une opération nécessaire à f échoue, il peut se passer n'importe quoi, y compris le non-respect des invariants.

> Higher levels of safety can sometimes be difficult to achieve, and might incur an overhead due to extra copying.Parfois, il faut arbitrer entre safety et perfs

 La suite est un exemple concret avec l'appel de la méthode `push_back` sur un `Vector`, pour lequel l'allocation peut échouer :

 - avec nothrow, l'opération est garantie de réussir (e.g. l'allocation ne peut en fait pas échouer ? ou en alternative, on change le contrat de la fonction pour considérer qu'elle peut renvoyer true ou false : l'opération "ne peut pas échouer" dans le sens où si l'allocation échoue, elle renvoie false, et ce n'est pas considéré comme un échec)
 - avec strong safety, on essaye d'allouer : si ça reussit, on y move le Vector, si ça échoue, on aborte la fonction ; ainsi, soit le `push_back` réussit (commit) soit le Vector reste inchangé (rollback) et tout se passe comme si la fonction n'avait pas été appelée, ce qui est la définition de strong safety
 - avec basic safety, la seule garantie après l'appel est que le Vector est valide (e.g. son nombre d'éléments est égal à end-begin, ou encore son buffer sera bien désalloué à la destruction), mais si l'appel a échoué, il se pourrait que faire un push_back sur un Vector contenant N éléments conduise à un Vector vide ne contenant plus aucun élément
 - avec no exception safety, tout peut arriver si l'implémentation de push_back fait face à une exception lancée : le Vector peut leaker, par exemple, ou begin peut pointer vers une adresse invalide...
