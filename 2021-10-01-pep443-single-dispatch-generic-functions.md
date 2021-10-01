# PEP 443 -- Single-dispatch generic functions

- **url** = https://www.python.org/dev/peps/pep-0443/
- **type** = pep
- **auteur** = [Łukasz LANGA](https://lukasz.langa.pl/), core-dev python, et créateur de black
- **date de publication** = 2013-05-22
- **source** = [les PEPs python](https://www.python.org/dev/peps/)
- **tags** = language>python ; topic>interfaces ; topic>pep ; level>intermediate

**TL;DR** :
- `functools.singledispatch` permet "l'overloading" des fonctions (que python appelle _generic function_), i.e. la possibilité d'avoir des implémentations différentes en fonction du type de l'argument d'une fonction.
- cet overloading ne dépend que du type de leur **premier** argument, d'où le nom de *single* dispatch
- ce mécanisme existait déjà (e.g. la fonction `len()`), en utilisant différentes méthodes ; l'un des apports de la PEP est de standardiser la façon dont on propose de _generic functions_. Un autre apport est de permettre de répondre au besoin en respectant l'OCP.
- le principe suit deux étapes :
    - step 1 = marquer une fonction `f` comme overloadable, en la décorant de `@singledispatch`
    - step 2 = ajouter une implémentation de `f` pour un type en particulier, en décorant cette implémentation de `@f.register(TYPE)`
- la conséquence de la step 1, c'est que ça ne concerne pas **toutes** les fonctions : seules les fonctions explicitement marquées sont overloadables
- j'ai [une POC sur le sujet](https://github.com/phidra/pocs/blob/572839bedf3e4d3c76699764962755a9b337a426/python/singledispatch/poc.py), qui teste certains points, notamment le comportement de mypy


* [PEP 443 -- Single-dispatch generic functions](#pep-443----single-dispatch-generic-functions)
   * [À quel besoin ça répond ?](#à-quel-besoin-ça-répond-)
   * [D'où on part](#doù-on-part)
   * [Usage patterns](#usage-patterns)
   * [POC](#poc)
   * [Précisions techniques](#précisions-techniques)

## À quel besoin ça répond ?

Permet d'implémenter des _generic functions_, cf. [le glossaire](https://docs.python.org/3/glossary.html#term-generic-function) : pour python, une _generic function_ est un ensemble de fonctions, chacune implémentant le comportement souhaité pour UN type donné. Par exemple, `len()` est une generic-function qui existe depuis longtemps.

En gros, le besoin est de spécialiser le comportement d'une fonction plutôt générale, en proposant une implémentation particulière pour un type en particulier. La PEP donne plusieurs exemples :

- `len()`
- `pprint.pprint()`
- `copy.copy()`
- `iter()`

## D'où on part

Jusqu'ici, comment ce besoin était adressé :

- en utilisant des dunder-methods (NdM : c'est ce que fait `len()`), 
- en appelant une fonction de type "register" pour enregistrer l'implémentation alternative comme "dispatchante" (NdM : c'est la forme standardisée par la PEP 443)
- en inspectant les arguments pour dispatcher "manuellement" en fonction du type (à coup de `isinstance`), mais c'est fragile, et fermé à l'extension

Il n'y a donc pas de façon standard de faire, chacun fait à sa sauce. De plus, l'utilisateur ne peut pas facilement définir ses propres fonctions génériques.

La PEP 443 standardise la façon de faire ça, en introduisant `functools.singledispatch`.

Question : pourquoi ne pas répondre au besoin en utilisant une méthode d'objet, appelée par la fonction générique, et que tous les types implémenteraient ?

Réponse : en préambule, c'est l'un des moyens utilisé jusqu'ici pour répondre au besoin (c'est ce que fait `len()` qui appelle la _dunder-method_ `__len__`). Et surtout, faire comme ça ne respecte pas l'OCP : on n'a pas forcément la possibilité de modifier les types qu'on manipule pour leur ajouter a posteriori des méthodes.

À l'inverse, singledispatch permet d'ajouter du comportement à un type (et à une fonction, FWIF) **a posteriori**, sans modifier ni le type, ni la fonction, ce qui respecte bien l'Open-Close Principle :

> Just as a base class method may be overridden by a subclass, so too a function may be overloaded to provide custom functionality for a given type.

La seule contrainte est que la fonction générique pour laquelle on souhaite proposer une spécialisation particulière doit avoir **déjà** été décorée avec `@singledispatch` (donc elle doit déjà avoir prévu le coup).

## Usage patterns

La [section Usage patterns](https://www.python.org/dev/peps/pep-0443/#usage-patterns) de la PEP donne des points de vue intéressants :

> This PEP proposes extending behaviour only of functions specifically marked as generic.

Un point important = ce comportement n'est pas accessible à toutes les fonctions, uniquement à celles qui sont expliciement marquées comme étant génériques.

Plus précisément, la section mets en garde contre le fait de rendre toutes les fonctions génériques par défaut :

> If a module is defining a new generic operation, it will usually also define any required implementations for existing types in the same place.
> Likewise, if a module is defining a new type, then it will usually define implementations there for any generic functions that it knows or cares about.

L'idée est que ces "extensions" ne soient pas disséminées dans le code (en effet, ça compliquerait la recherche d'une implémentation en particulier d'une fonction), mais que chaque "extension" soit proche de la fonction overloadée ou du nouveau type pour lequel on overloade une fonction :

>  As a result, the vast majority of registered implementations can be found adjacent to either the function being overloaded, or to a newly-defined type for which the implementation is adding support.

Si jamais ça n'est pas le cas, c'est qu'on n'est pas censé avoir besoin de connaître le détail de la fonction overloadée :

> In the absence of incompetence or deliberate intention to be obscure, the few implementations that are not registered adjacent to the relevant type(s) or function(s), will generally not need to be understood or known about outside the scope where those implementations are defined.

La section donne également quelques billes sur d"où on vient" :

> As mentioned earlier, single-dispatch generics are already prolific throughout the standard library.
> A clean, standard way of doing them provides a way forward to refactor those custom implementations to use a common one, opening them up for user extensibility at the same time.

## POC

J'ai fait [une POC sur le sujet](https://github.com/phidra/pocs/blob/b052307751e4854457f83ba76b3f7b54179528ac/python/singledispatch/poc.py), qui addresse certaines questions :

- Q = est-ce que mypy détecte et prévient quand on essaye d'utiliser une fonction singledispatched, mais que le type n'est pas implémenté ?
- R = apparemment non...
- Q = que se passe-t-il si on appelle la fonction sur un type pour lequel on n'a pas proposé d'implémentation spécifique ?
- R = mypy ne prévient pas, et on fallback sur l'implémentation "initiale" (celle décorée par `@singledispatch`), y compris si celle-ci raise `NonImplementedError` cf. [la doc](https://www.python.org/dev/peps/pep-0443/#user-api) :

    > Where there is no registered implementation for a specific type, its method resolution order is used to find a more generic implementation. The original function decorated with @singledispatch is registered for the base object type, which means it is used if no better implementation is found.
- Q = que se passe-t-il si on essaye de register une fonction `f_registered` comme implémentation alternative de `f_initial`, mais que l'interface (nombre d'arguments, notamment) de `f_registered` ne matche pas avec celle de `f_initial` ?
- R = au lieu de refuser de register, `functools.singledispatch` autorise à enregistrer et appeler `f_registered`. Du coup, à l'appel, il faut faire attention à appeler `f_registered` d'une façon **différente** de `f_initial`, vu qu'elle attend un nombre d'argument différents que ses soeurs... ça craint je trouve.

## Précisions techniques

- **Important** : c'est uniquement sur le PREMIER argument qu'on dispatche.
- on peut soit décorer des implémentations par `@f.register(TYPE)` lors de leur définition, soit _register_ des implémentations déjà existantes (a posteriori).
- la PEP donne des exemples spécifiques, on peut notamment connaître dynamiquement l'implémentation qui sera utilisée pour un type, ou accéder à toutes les implémentation _registered_
- le fait de restreindre le multiple dispatch au premier-argument est volontaire, pour simplifier la logique ([lien vers la doc](https://www.python.org/dev/peps/pep-0443/#alternative-approaches)). [generic](https://pypi.org/project/generic/) est une généralisation de `singledispatch` qui marche pour TOUS les arguments, et plus uniquement le premier.
- d'après [la doc](https://www.python.org/dev/peps/pep-0443/#abstract-base-classes), on peut _register_ des implémentations alternatives pour les ABC (p.ex. proposer une implémentation particulière pour toutes les `Sequence`)
- pages wikipedia sur [multiple dispatch](https://en.wikipedia.org/wiki/Multiple_dispatch) et [single vs. multiple dispatch](https://en.wikipedia.org/wiki/Dynamic_dispatch#Single_and_multiple_dispatch). Le single dispatch correspond au fait de ne s'intéresser qu'à un seul argument (le premier, dans notre cas) pour choisir l'implémentation à utiliser.
