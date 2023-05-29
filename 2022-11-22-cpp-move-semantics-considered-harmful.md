# C++ Move Semantics Considered Harmful (Rust is better)

- **url** = https://www.thecodedmessage.com/posts/cpp-move/
- **type** = article
- **auteur** = [Jimmy HARTZELL](https://www.thecodedmessage.com/about/), dev multi-langage, essentiellement rust
- **date de publication** = 2021-11-03
- **source** = [son site](https://www.thecodedmessage.com/)
- **tags** = language>cpp ; language>rust ; topic>move-semantics ; level>intermediate


Préambule = j'ai lu et annoté cet excellent article deux fois (en novembre 2022 et mai 2023), je conserve les deux versions des notes.

* [C++ Move Semantics Considered Harmful (Rust is better)](#c-move-semantics-considered-harmful-rust-is-better)
   * [Premières notes en novembre 2022](#premières-notes-en-novembre-2022)
   * [Deuxièmes notes en mai 2023](#deuxièmes-notes-en-mai-2023)
      * [première partie= c++](#première-partie-c)
      * [Moves in Rust](#moves-in-rust)

TL;DR : une comparaison des move-semantics de rust et C++ ; l'article est très intéressant pour trois raisons :

- il explique très bien le POURQUOI de la move sémantique = à quel problème ça répond
- il montre que l'implémentation de la move sémantique en C++ est pas terrible, car elle répond indirectement au problème, et laisse de la place aux erreurs humaines (du coup, ça explique que l'apprentissage de la move-semantique soit compliquée)
- il montre le parallèle avec rust et les destructive move, qui répondent de façon plus directe au problème


Le move du c++ est non-destructive, i.e. un objet moved-from reste valide/utilisable, et son destructeur sera appelé.

Entre autres conséquences, le destructeur doit donc savoir gérer les cas où il est appelé mais où il doit ne rien faire (car l'objet est moved-from). C'est complexe (il faut gérer l'état moved-from), sous-optimal (il y a un check ou une destruction noop au runtime), et conceptuellement pas ce qu'on veut (tout objet movable n'a pas d'autre choix que de devenir un `Optional` devant gérer un état "nul").

A contrario, en rust, un objet moved-from n'est plus utilisable : le move est DESTRUCTIVE, ce qui évite tous ces désagréments. Dans sa conclusion, il résume les défauts de la move semantic en c++ :

> the specific design of moves in C++:
>
> - is misaligned with the purpose of moving
> - fails to eliminate all run-time cost
> - surprises programmers, and
> - forces designers of types to implement an “empty-yet-valid” state

## Premières notes en novembre 2022

Plus en détail, à quoi sert la move-semantique ?

- ça ne sert que pour des objets qui gèrent des ressources (e.g. une `std::string` qui gère un buffer heap-allocated)
- si on veut les passer dans une fonction qui doit outlive la string, sans move sémantique, on n'a pas d'autre choix que de copier le buffer, puis supprimer l'original
- la move-sémantique sert précisément à éviter cette opération sous-optimale = "copier un buffer puis détruire l'original"
- à la place, on veut "move" le buffer

Implémentation C++ pourrie ?

- pas en accord avec le besoin
- force les types qu'on veut move à avoir un état "moved-from", valide mais "vide" (i.e. force les types qui gèrent les ressources à avoir un état "toujours valide, mais sans ressource")
- (c'est dû au fait que le destructeur va obligatoirement être appelé sur cet objet moved-from, il faut donc que le destructeur soit un noop, i.e. qu'il ne cleane pas la ressource — vu que celle-ci a été movée vers son nouveau conteneur)
- pour les conteneurs (dont la ressource gérée est leur contenu), c'est pas critique, car il suffit que cet état moved-from soit l'état "empty"
- mais c'est déjà plus embêtant p.ex. pour `unique_ptr` :
    - on aurait aimé que le design de `unique_ptr` ne permette QUE de contenir un pointeur
    - or, comme on DOIT avoir un état "valide mais vide", ça force le design de `unique_ptr` à accepter aussi un pointeur nul (ce dont on se serait bien passé)

Meilleure implémentation rust = destructive move :

- a contrario, en rust, le move transfère la ressource au nouvel objet, ET MARQUE l'ancien objet comme "PSA = ne pas appeler le destructeur sur cet objet car il a été mové"
- du coup, on n'a pas besoin de gérer artificiellement un état "toujours valide mais sans ressource"
- (d'où le "destructive move" : le move "détruit" l'objet, en quelque sorte)

## Deuxièmes notes en mai 2023

### première partie= c++

Il commence par expliquer l'intérêt de l'ajout de move dans C++11 : par défaut, en C++, on a la value-semantics, du coup tout passage d'une string fait une copie du buffer.

> Which means we copied an allocation into a new heap allocation, just to free the original allocation.

^ le problème qu'on résout avec la move-semantics ; ses exemples de code expliquent très clairement le problème adressé.

> In any case, moves were not supported. And so, objects that managed resources – in this case, a heap allocation, but other resources could apply as well – could not be put onto vectors or stored in collections directly without a copy and delete of whatever resource was being managed.

^ le problème est bien lié aux objets qui gèrent des ressources, vu que ce sont ces ressources qu'on move.

On pouvait toujours faire des move manuellement, mais c'était pas pratique + moins efficace (par exemple, si un vector doit être resizé, il doit allouer + copier + désallouer, au lieu de simplement réutiliser le buffer déjà alloué via un move).

> The abstractions of only supporting “copy” and “destruct” mean that the destructor of the variable foo must be called when foo goes out of scope. This means that the “copy” operation must make an independent allocation, as it cannot control when the original goes out of scope, or will be replaced with another value. If we had instead re-used the same allocation, it would be freed by foos destructor.
>
> But copying just to destroy the original is silly – silly and ill-performant. What any programmer would naturally write in that situation results in a “move”. So this gap – and it was a huge gap – in C++ value semantics was filled in C++11 when they added a “move” operation.
>
> Because of this addition, using objects with value semantics that managed resources became possible. It also became possible to use objects with value semantics for resources that could not meaningfully be copied, like unique ownership of an object or a thread handle, while still being able to get the advantages of putting such objects in collections and, well, moving them.

^ D'une façon plus générale que cette citation, son introduction explique très bien le problème que la move-semantics vise à résoudre !

> When foo is moved into the vector, the original allocation must not freed. Instead, it is only freed when the vector itself is freed. (...) If there is to be exactly one allocation, there must be exactly one deallocation. (...) the destructor should only be called when the vector is destroyed, but not when foo goes out of scope. If foo is moved onto the vector, then the compiler should take note that it has been moved from, and simply not call the destructor. The move should be treated as having already destroyed the object, as an operation that accomplishes both initialization of the new object (the string on the vector) from the original object and the destruction of the original object

^ comment on souhaite que la move-semantics se comporte. Et c'est bien comme ça que la move-semantics fonctionne en rust :

> In Rust, the compiler would simply not output a destructor call (a “drop” in Rust) for foo because it has been moved from. (...) In destructive move-semantics, the compiler would not allow foo to be read from after the move, but in fact, the C++ compiler still does, not just for the destructor, but for any operation.

^ en rust, le move est destructif = l'objet n'est plus utilisable, alors qu'en C++, si : l'objet n'est "que" dans un état moved-from, mais on a le droit de l'utiliser (et je me souviens avoir trouvé ça bizarre quand j'ai découvert C++11), et notamment, le destructeur sera toujours appelé (et doit donc être valide) sur un objet moved-from.

TL;DR = le free a toujours lieu à la destruction, mais sur un buffer nul, donc c'est un noop. C'est bien, mais pas autant que de ne PAS appeler free sur un objet moved-from, comme le fait rust !

Conceptuellement, ce qu'il faut retenir, c'est que le C++ continue de détruire "pour rien" des objets moved-from. Il faut donc un moyen d'indiquer que l'objet a été moved-from, et qu'il n'y a rien à libérer :

> This responsibility of having some run-time indication of what resources need to be freed – rather than a one-to-one correspondence between objects and resources – is left up to the implementors of classes. For heap allocations, it is made relatively easy, but the implementor of the class is still responsible for re-setting the original object. (...) The move constructor has two responsibilities, where a destructive version would only have one: It must set up state for the new object, and it must set up a valid “moved from” state for the old object. That second obligation is a direct consequence of non-destructive moves

Ça va plus loin : comme les objets moved-from doivent rester valides, ils deviennent tous des `Optional` de facto !

> This means that any object that manages a resource now must manage either 1 or 0 copies of that resource. Collections are easy – moved from collections can be made equivalent to the “empty” collection that has no element. For things like thread handles or file handles, this means that you can have a file handle with no corresponding file. Optionality is imported to all “value types.”

C'est problématique notamment pour les smart-pointers, où on ne s'attend pas à ce qu'ils puissent être nuls, et où c'est un défaut de design :

> Nullable pointers are a serious cause of errors, as often they are used with the implicit contract that they will not be null, but that contract is not actually represented in the type.

Une définition concise de move :

> The goal of move is clear: to allow resources to be transferred when copying would force them to be duplicated.

Au final, l'ensemble du début du post explore en détail la move-semantics du C++, et le fait qu'à cause du non-destructive move, les objets gérant les ressources se retrouvent obligatoirement, à leur corps défendant, à devoir manager 0 ou 1 ressource :

> Put another way: std::unique_ptr and thread handles are therefore collections of 0 or 1 heap allocation handles or thread handles, and once defined that way, the “empty” state is not special, but it is move-semantics that force them to be defined that way.

Derrière, il analyse plusieurs réponses de Herb SUTTER concernant move, en argumentant lorsqu'il n'est pas d'accord, autour des thèmes déjà discutés = le fait que le move du C++ ne soit pas destructive ; plusieurs détails d'implémentation de move en C++ sont certes légaux techniquement, mais tellement contre-intuitifs qu'ils en deviennent dangereux :

> The misconceptions that Herb Sutter is addressing are an unfortunate consequence of the dissonance between the strict semantics of the programming language, where his statements are true, and the practical implications of how these features are used and are intended to be used, where the situation is more complicated.

### Moves in Rust

> Rust makes a special case for types that do not need move-semantics, where the value itself contains all the information necessary to represent it, where no heap allocations or resources are managed by the value, types like i32. These types implement the special Copy trait, because for these types, copying is cheap, and is the default way to pass to functions or to handle assignments

^ les types ne gérant pas de ressources, et pouvant donc être copiés à moindre coût sont explicitement identifiés comme tels en rust (via le trait `Copy`). Exemple habituel d'un tel type = un `int`.

> For types that are not Copy, such as String, the default function call uses move-semantics. In Rust, when a variable is moved from, that variable’s lifetime ends early. The move replaces the destructor call at the end of the block, at compile time

^ Pour les types qui ne sont pas `Copy`, leur passage à une fonction les move, et le compilateur empêche de les réutiliser une fois moved : le move est destructive. Exemple habituel d'un tel type = une `String`.

> Copy is a trait, but more entwined with the compiler than most traits. Unlike most traits, you can’t implement it by hand, but only by deriving from primitive types that implement copy. Types like Box, that manage a heap allocation, do not implement copy, and therefore structs that contain Box also cannot.

^ le fait d'être `Copy` ou pas est spécial et semi-automatique.

> This is already an advantage to Rust. C++ pretends that all types are the same, even though they require different usage patterns in practice.

^ c'était l'une de ses critiques sur C++ = de faire semblant qu'un `int` (objet ne gérant pas de ressources) et une `String` (qui gère un buffer) se traitent de la même façon, alors que ça n'est pas le cas.

La deepcopy plutôt que le move (p.ex. au passage d'argument à une fonction) reste possible avec `mystring.clone()`.

> Rust requires implementations for clone, but for all moves, the implementation is the same: copy the memory in the value itself, and don’t call the destructor on the original value.

^ le fait que le destructeur n'est pas appelé ne veut pas dire que la ressource n'est jamais libérée, mais plutôt qu'elle n'est libérée qu'une unique fois (ce qui est le comportement désiré). En effet, même si le destructeur de la variable moved-from n'est pas appelé, le destructeur de la nouvelle variable (celle vers laquelle on a move) sera bien appelé, lui ; et comme c'est lui qui gère la ressource maintenant (vu qu'elle a été move vers lui), l'appel de son destructeur libèrera bien la ressource.

> C++ can’t do that, because in C++, the implementation of move has to mark the moved-from value as no longer containing the resource. How this marking works depends on the details of the type.

^ une autre façon de formuler sa critique principale = le fait que les types movables doivent obligatoirement être des `Optional`.

> In any case, Rust has taken this opportunity to learn from existing programming languages, and to solve the same problems in a cleaner, more principled way.

^ je suis bien d'accord : les langages modernes bénéficient de l'expérience douloureusement acquise via les erreurs faites avec les langages plus anciens.

> And to be clear, this still has very little to do with the safety features of Rust. (...) as I continue to argue, unsafe Rust is a better unsafe language than C++.
