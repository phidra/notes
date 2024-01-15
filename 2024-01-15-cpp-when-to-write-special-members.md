# Tutorial: When to Write Which Special Member

- **url** = https://www.foonathan.net/2019/02/special-member-functions/
- **type** = post
- **auteur** = [Jonathan MÜLLER](https://www.jonathanmueller.dev/) = dev C++ (allemand ?), membre du C++ committee
- **date de publication** = 2019-02-26
- **source** = [son blog](https://www.foonathan.net/)
- **tags** = language>cpp ; topic>special-member-functions ; level>intermediate


**TL;DR** : quels _special member functions_ doit-on implémenter/defaulter/deleter ? La majorité du temps, nos classes se retrouvent dans l'une de ces quatre situations :

- **classe "normale"** = rule of zero = on n'implémente pas de destructeur particulier ni aucun special member de copie/move
- **classe "qui gère une ressource" (e.g. `vector`)** = rule of five = on implémente le destructeur + les 4 special members de copie/move
- **classe "qui gère une ressource que ça n'a pas de sens de copier" (e.g. `unique_ptr`)** = idem mais en marquant `=delete` les special members de copie, pour ne garder que ceux de move (qui permettent de transférer la ressource)
- **classe "non-déplaçable en mémoire" (e.g. car on veut qu'un pointeur vers cette classe ne soit jamais invalidé)** = on n'implémente que le destructeur, en marquant `=delete` les autres special members

En marge de ces quatre situations, il donne quelques code-smells (e.g. définir explicitement un constructeur par défaut vide), ainsi que le cas particulier du swap qui peut être utile.

----

[Cet autre article](https://www.foonathan.net/special-member-chart/) du même auteur a le même message mais en beaucoup plus synthétique.


> A “user-declared” special member function is a special member function that is in any way mentioned in the class: It can have a definition, it can be defaulted, it can be deleted. This means that writing foo(const foo&) = default prohibits a move constructor.

^ à noter que même si on définit un constructeur comme `= default`, on considère tout de même qu'il est "user-declared".

Il introduit un tableau compliqué qui dit quand définir quoi. Le tableau est beaucoup trop compliqué pour pouvoir le retenir, mais la bonne nouvelle c'est qu'on n'a pas besoin, car la grande majorité du temps, on est dans l'une de ces 4 situations particulières :

**rule of zero** :

- c'est la majorité des cas, où la classe ne gère aucune ressource
- NDM : ça ne veut pas dire qu'on ne définit pas de constructeur ! On peut très bien définir un constructeur autre qu'un constructeur par défaut (i.e. qui attend des arguments). Par contre, on ne définit pas l'un des constructeurs spéciaux.
- La seule question à se poser dans ce cas est de savoir si la classe doit avoir un default constructor ou pas, lorsqu'on a déjà défini un constructeur :
    > If you don’t have any constructors, the class will have a compiler generated default constructor. If you have a constructor, it will not. In that case add a default constructor if there is a sensible default value.
- (un default-constructor existera toujours, compiler-defined, pour les classes qui n'ont aucun constructeur ; la question se pose pour les classes qui ont déjà un user-defined constructor, car dans ce cas, le default-constructor n'est pas défini par le compilo)
- en gros : si ça fait sens pour la classe d'avoir une valeur par défaut, alors définir un default-constructor a du sens
- et je suis bien d'accord avec cette remarque :
    > It is often not a good idea to introduce an artificial “null” state, just use std::optional<T> instead.


**rule of five (six)** :

- NDM : l'article appelle cette situation "container classes", mais je préfère voir ça comme "je gère une ressource qui doit être libérée à la destruction".
- ici, on parle d'une situation où la structure est un wrapper autour d'une ressource, qui devra être libérée à la destruction (rust appelle une partie de ces situations les SmartPointers) :
    - `std::vector` (ressource gérée = le buffer sur le heap)
    - `std::string` (ressource gérée = le buffer sur le heap)
    - `std::shared_ptr` (ressource gérée = le control-block + la structure allouée sur le heap)
- le point important est que dans ce cas, il faut écrire un destructeur custom, pour libérer correctement la ressource allouée sur le heap (là où le compiler-generated destructor se contentera de libérer le _pointeur_ vers la ressource) :
    > If you need to write a destructor — because you have to free dynamic memory, for example — the compiler generated copy constructor and assignment operator will do the wrong thing. Then you have to provide your own.
- et si on a un destructeur custom à écrire, on veut aussi écrire les membres spéciaux de copie/move :
    > This is known as the rule of five. Whenever you have a custom destructor, also write a copy constructor and assignment operator that have matching semantics. For performance reasons also write a move constructor and move assignment operator.
- détail important :
    > Strive to make them noexcept and fast.
- pourquoi `noexcept` ? Je suppose que si une exception est lancée en cours de move, on ne sait plus où on en est : qui gère la ressource ? la classe dont on vient d'essayer de la voler, ou la nouvelle classe dont la construction/assignation vient d'échouer ?
- pour notre classe qui gère une ressource, ça peut avoir du sens d'ajouter un default-constructor pour représenter une instance "vide" (e.g. un vector vide), ce qui forme le sixième item de la rule-of-six :
    > As you now have a constructor, there will not be an implicit default constructor. In most cases it makes sense to implement a default constructor that puts the class in the empty state, like the post-move one. This makes it the rule of six.

**Resource Handle Classes: Move-only**

- NDM : ce sont ces cas que l'article appelle "ressource handler" ; à titre perso, je vois ça comme un dérivé du cas précédent, mais pour la situation particulière où la ressource est "unique", non-copiable.
- ici, on parle d'une situation très très proche de la précédente (i.e. la classe est un wrapper autour d'une ressource qui devra être libérée à la destruction), mais avec une différence importante = la ressource ne peut pas être copiée :
    - `std::unique_ptr` (ressource non-copiable = la responsabilité de libérer une zone mémoire sur le heap : si on copiait la classe, on la libèrerait deux fois)
    - `std::scoped_lock` (ressource non-copiable = la responsabilité de délocker un mutex : si on copiait la classe, on le délockerait deux fois)
    - `file handle` (ressource non-copiable = la responsabilité de rendre à l'OS un descripteur de fichiers : si on copiait la classe, on rendrait le descripteur à l'OS deux fois)
- dans ces situations, cloner la structure n'a aucun sens, on veut donc empêcher les utilisateurs de copier la structure, on marque donc le copy-constructor et copy-assignment operator comme `=delete`
- ici aussi, on peut choisir de créer un default-constructor (NDM : mais je ne suis pas loin de penser que c'est une erreur de design que de permettre un `unique_ptr` vide ou un `scoped_lock` vide... Je préfèrerais un std::optional d'un unique_ptr qui n'est jamais vide) :
    > Again, it makes sense to add a default constructor that puts it in the post-move state.


**Immoveable Classes**

- classe utile si on veut qu'un objet créé à une adresse reste à cette adresse :
    > Sometimes you want that a class cannot be copied or moved. Once an object is created it will always stay at that address. This is convenient if you want to safely create pointers to that object.
- dans ce cas, on veut marquer 4 des 5 opérations spéciales (tout le monde sauf le destructeur) comme `=delete` (ou laisser le compilateur le faire pour nous, éventuellement)

----

En plus de ces 4 situations "classiques", certaines situations sont des code-smell :

**rule of three** = implémenter les copy operations sans implémenter leurs équivalents move ; ça fonctionnouille en pratique (vu qu'un move va fallback sur une copie), mais va être confusant : les utilisateurs s'attendent à ce qu'un move soit rapide (le seul cas où on voudra faire ça, c'est si on est pre-C++11).

**copy only types** = pas bon non plus : implémenter les copy operations en deletant leurs équivalents move. Ici, la classe ne sera même pas utilisable par une librairie (comme la stdlib) qui veut essayer de les move : ça ne compilera pas. NDM : il présente les choses comme "encore pire" que la situation précédente, mais je pense le contraire : avoir une erreur de compilation explicite permet d'identifier plus facilement son erreur de design, plutôt qu'avoir une inefficience (car le move fallback sur copy) plus difficile à détecter.

**Don’t: Deleted Default Constructor** = _There is no reason to = delete a default constructor, if you don’t want one, write another one._ Si on veut avoir une classe qui ne dispose pas de default-constructor, il suffit de définir un autre constructeur (avec des paramètres), et dans ce cas, le compilo ne génèrera pas de default-constructor (et il était donc inutile de le deleter explicitement).

**Don’t: Partial Implementation** = implémenter un copy constructor et deleter le copy-assignment operator qui va avec. _Copy construction and copy assignment are a pair. You either want both or none. Conceptually, copy assignment is just a faster “destroy + copy construct” cycle. So if you have copy construct, you should also have copy assignment, as it can be written using a destructor call and construction anyway._ : ne pas définir de copy-assignment operator n'empêche pas l'utilisateur de le simuler avec une destruction + construction.

----

Enfin, le cas de `swap` est particulier, techniquement indépendant, mais qui va un peu avec le reste.

Dois-je définir une méthode `swap` à ma classe ?

```cpp
class MyClass
{
    public:
    friend void swap(MyClass& lhs, MyClass& rhs) noexcept;
};
```

Je paraphase l'article : certains algos utilisent `swap()` comme le `std::move` du pauvre.

Si ma classe ne dispose pas d'une méthode `swap()`, alors c'est `std::swap()` qui sera utilisé, or elle fait 3 moves :

```
template <typename T> void swap(T& lhs, T& rhs)
{
    T tmp(std::move(lhs));
    lhs = std::move(rhs);
    rhs = std::move(tmp);
}
```

Du coup si je peux implémenter un swap plus efficace que ça, il faut le faire pour gagner en perfs.

> Of course, this only applies to classes that have a custom destructor, where you’ve implemented your own copy or move. Your own swap() should always be noexcept.
