# Three reasons to pass std::string_view by value

- **url** = https://quuxplusone.github.io/blog/2021/11/09/pass-string-view-by-value/
- **type** = post
- **auteur** = [Arthur O’DWYER](https://quuxplusone.github.io/blog/about/)
- **date de publication** = 2021-11-09
- **source** = [le blog de l'auteur](https://quuxplusone.github.io/blog/)
- **tags** = language>C++ ; topic>pass-by-value ; topic>pass-by-reference ; topic>std::string_view ; level>intermediate

**TL;DR** : pass-by-reference est une optimisation de pass-by-value pour les gros objets coûteux à copier ; en dehors des cas où cette optimisation est valable, c'est pass-by-value qu'on devrait tendre à utiliser.

* [Three reasons to pass std::string_view by value](#three-reasons-to-pass-stdstring_view-by-value)
   * [Message principal = passage par const-ref à réserver aux objets coûteux à copier](#message-principal--passage-par-const-ref-à-réserver-aux-objets-coûteux-à-copier)
   * [avantage de perf 1 = pass-by-value autorise le passage d'arguments dans les registres](#avantage-de-perf-1--pass-by-value-autorise-le-passage-darguments-dans-les-registres)
   * [avantage de perf 2 = pass-by-ref force le passage d'arguments sur la stack](#avantage-de-perf-2--pass-by-ref-force-le-passage-darguments-sur-la-stack)
   * [avantage de perf 3 = pass-by-value n'a pas d'aliasing donc permet plus d'optimisations du compilo](#avantage-de-perf-3--pass-by-value-na-pas-daliasing-donc-permet-plus-doptimisations-du-compilo)
   * [il ne faut pas non plus proscrire le passage par référence](#il-ne-faut-pas-non-plus-proscrire-le-passage-par-référence)
   * [les types modernees sont conçus pour être passés par valeu](#les-types-modernees-sont-conçus-pour-être-passés-par-valeu)

## Message principal = passage par const-ref à réserver aux objets coûteux à copier

L'article prend les `std::string_view` comme exemple, mais le message ci-dessus est plus général.

Au sujet de [std::string_view](https://en.cppreference.com/w/cpp/string/basic_string_view), c'est un équivalent plus efficace et plus puissante d'une `string const&` :
- plus efficace... cf. les raisons dans l'article :-)
- plus puissante, car on peut modifier la view sans modifier la string sous-jacente, par exemple, en "supprimant" un préfixe (en fait, on ne supprime pas vraiment le préfixe, on se contente de modifier la vue sur une string qui, elle, n'a pas été modifiée)
- de ce que j'en vois, le point le plus crucial avec les string_view, c'est d'éviter de faire en sorte que la string_view outlive la string sous-jacente (sinon, UB)
- dispo à partir de C++17

> It is idiomatic to pass std::string_view by value.
>
> In C++, everything defaults to pass-by-value [...] But copying big things can be expensive. So we introduce “pass-by-const-reference” as an optimization of “pass-by-value,”
>
> But for small cheap things — int, char*, std::pair<int, int>, std::span<Widget> — we continue to prefer the sensible default behavior of pass-by-value.

Le passage par (const-)référence n'existe qu'en tant qu' **optimisation** du passage par valeur, pour les objets trop gros à copier ; pour les situations où cette optimisation n'a pas d'intérêt, il **faut** continuer à passer par valeur, car le passage par const-ref devient alors inutile.

Pire (et c'est le cœur de l'article), ça en devient contre-productif, car le passage par valeur a 3 avantages de perfs, qu'il va illustrer avec `std::string_view` ; à noter que chacun des exemples est illustré avec godbolt et de l'assembleur.

Note : souvent, le compilo peut corriger du code passant à tort par const-ref, mais ça n'est pas parce que le compilo peut corriger nos bêtises qu'il ne faut pas essayer de faire propre dès le début :-)

## avantage de perf 1 = pass-by-value autorise le passage d'arguments dans les registres

pass-by-value autorise à passer les arguments directement dans les registres, alor que passer par référence oblige à déréférencer le pointeur (car les références sont gérées en interne par un pointeur).

On le voit dans l'assembleur généré = avec le déréférencement de pointeur, on doit faire un memory load, qu'on n'a pas à faire sinon.

Pour certains arguments, on déréférencera tout de même, car l'argument sera passé sur la stack quoi qu'il arrive, mais ici aussi, autant essayer de faire propre dès le début.

## avantage de perf 2 = pass-by-ref force le passage d'arguments sur la stack

pass-by-ref oblige à ce que l'argument ait une adresse, donc FORCE l'argument à être passé sur la stack.

Dans certain cas où le tail-call optimization pourrait avoir lieu, ce passage par const-ref est le seul élément empêchant la tail-call optimization... Ma compréhension est donc que :

- avec passage par const-ref, on n'aura JAMAIS de tail-call optimization.
- avec passage par valeur, si le reste des planètes sont alignées, le passage par valeur n'empêchera pas le TCO

## avantage de perf 3 = pass-by-value n'a pas d'aliasing donc permet plus d'optimisations du compilo

En passant par ref, on passe un pointeur, et le compilateur n'a pas d'infos sur ce qui est pointé. Notamment, il n'a pas d'infos sur qui pointe vers le même objet (ou vers des morceaux du même objet), par conséquent, il doit être très prudent dans les optimisations qu'il lui applique.

À l'inverse, en passant par valeur, le compilateur sait que personne d'autre que la fonction n'aliase l'objet en argument (vu que c'est une copie qui n'est utilisée que dans la fonction), du coup, il peut optimiser aggressivement.

L'article donne un exemple concret de cas où le compilo rate des optimisations à cause de ça.

## il ne faut pas non plus proscrire le passage par référence

> Pass-by-value has at least three performance benefits, detailed above. But if the performance cost of making a copy outweighs all of these benefits — for large and/or expensive-to-copy types like string and vector — then prefer to pass by const reference. Pass-by-const-reference is an optimization of pass-by-value

## les types modernees sont conçus pour être passés par valeu

Certains types modernes sont spécialement conçus pour être passés par valeur :

- string_view
- span
- function_ref
