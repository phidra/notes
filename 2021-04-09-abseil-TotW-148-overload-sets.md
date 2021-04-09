# abseil / Tip of the Week #148: Overload Sets

- **url** = https://abseil.io/tips/148
- **type** = tip
- **auteur** = [Titus WINTERS](https://www.linkedin.com/in/tituswinters), Senior Staff Software Engineer chez Google, fondateur d'Abseil, membre du comité C++
- **date de publication** = 2018-05-03 (updated 2020-04-06)
- **source** = [Abseil's C++ Tip of the Week](https://abseil.io/tips/)
- **tags** = language>cpp ; topic>overload-sets ; best-practices

**TL;DR** : les bonnes pratiques autour de l'overloading :
- il ne faut overloader des fonctions que si toutes les fonctions de l'overload-set se comportent de façon homogène
- dit autrement, on devrait pouvoir considèrer l'overload-set comme une seule et même "super fonction"
- c'est valable aussi pour les constructeurs !
- bon exemple = overloader une fonction pour accepter indifféremment une `std::string` ou un `char*`
- bon exemple = overloader `operator[]` de vector pour avoir une version const qui renvoie `T const&` et une version non-const qui renvoie `T&` 
- mauvais exemple = les deux fonctions ci-dessous :
```cpp
// remove obstacle from garage exit lane
void open(Gate& g);

// open file
void open(const char* name, const char* mode ="r");
```

## Notes vrac

> Use overloaded functions (including constructors) only if a reader looking at a call site can get a good idea of what is happening without having to first figure out exactly which overload is being called.”
>
> If the documented behavior of the members of an overload set varies, then a user implicitly has to know which function is actually being called.

En gros, deux fonctions d'un même overload-set doivent avoir le même comportement.

Le **contre-exemple** parfait est celui copié-collé ci-dessus dans le TL;DR.

Et au passage, la bonne façon d'éviter ce genre d'overloading malencontreux est le namespace :

> Hopefully, namespace differences suffice to disambiguate functions like these from actually forming an overload set.

Le point au coeur de l'article, c'est que si l'overloading est correctement utilisé, on doit pouvoir traiter toutes les fonctions overloadées comme une seule et même fonction :

> It has never actually been a single function - we just treat it conceptually as one entity.
>
> it will always be the “convert to string and concatenate” tool.

Autre exemple qui respecte le même principe : overloader pour une lvalue-reference et une rvalue-reference :

> Another technique that the standard uses and that comes up in a lot of generic code is to overload on const T& and T&& when passing a value that will be stored: a value sink. Consider std::vector::push_back():
>
> Interestingly, these overloads are semantically the same as a single method – push_back(T)

Autre exemple qui respecte le même principe : overloader pour avoir une version const et une version non-const :

> For methods on a class (especially a container or a wrapper), it is sometimes valuable to provide an overload set for accessors.
> 
> In the case of vector::operator[], two overloads exist: one const and one non-const, which accordingly return a const or non-const reference, respectively. This matches our definition nicely; a user doesn’t need to know which thing is invoked. The semantics are the same, differing only in constness 

Dernier point abordé = constructeurs :

> The most important overload sets for types are often their set of constructors, especially the copy and move constructors. Copy and move, done right, form an overload set in all senses of the term: the reader should not need to know which of those overloads is chosen, because the semantics of the newly-constructed object should be the same in either case (assuming both constructors exist).

La rule of thumb à retenir, même si je trouve que la forme opposée (= les fonctions overloadées doivent représenter la même superfonction) est plus intuitive :

> don’t produce overloads where anyone might need to know which function was chosen

