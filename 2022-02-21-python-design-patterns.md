# Python Design Patterns

- **url** = https://python-patterns.guide/
- **type** = série de posts
- **auteur** = [Brandon RHODES](https://rhodesmill.org/brandon/), conférencier python
- **date de publication** = sans doute répartie sur plusieurs années, les premiers commits datent de 2017, et le travail est toujours en cours
- **source** = [le site dédié aux design-patterns python](https://python-patterns.guide/), dont le code-source est [sur github](https://github.com/brandon-rhodes/python-patterns)
- **tags** = language>python ; topic>design-patterns ; level>intermediate


**TL;DR** : voici ce qu'en dit l'auteur lui-même : _A site where I’m writing up all of the old Gang of Four design patterns for Python programmers, weighing which ones are still useful in a modern dynamic language._

* [Python Design Patterns](#python-design-patterns)
   * [Gang of Four: Principles](#gang-of-four-principles)
      * [The Composition Over Inheritance Principle](#the-composition-over-inheritance-principle)
         * [Position du problème](#position-du-problème)
         * [Solutions utilisant des patterns reposant sur la composition : Adapter, Bridge, Decorator](#solutions-utilisant-des-patterns-reposant-sur-la-composition--adapter-bridge-decorator)
         * [Dodge: "if" statements](#dodge-if-statements)
         * [Dodge: Multiple Inheritance](#dodge-multiple-inheritance)
         * [Dodge : Mixins](#dodge--mixins)
         * [Dodge: Building classes dynamically](#dodge-building-classes-dynamically)
   * [Python-Specific Patterns](#python-specific-patterns)
      * [The Global Object Pattern](#the-global-object-pattern)
      * [The Prebound Method Pattern](#the-prebound-method-pattern)
      * [The Sentinel Object Pattern](#the-sentinel-object-pattern)
   * [Gang of Four: Creational Patterns](#gang-of-four-creational-patterns)
      * [The Abstract Factory Pattern](#the-abstract-factory-pattern)
      * [The Builder Pattern](#the-builder-pattern)
      * [The Factory Method Pattern](#the-factory-method-pattern)

## Gang of Four: Principles

### The Composition Over Inheritance Principle

https://python-patterns.guide/gang-of-four/composition-over-inheritance/

#### Position du problème

Le problème avec les archis à base d'héritage, c'est qu'on se retrouve vite avec trop de cas à gérer quand on cherche à spécialiser par héritage selon des axes orthogonaux. Par exemple, un logger peut être spécialisé de deux façons :

- pour choisir la destination des logs parmi file, socket, ou syslog
- pour choisir le niveau de filtrage du message loggé

Si on veut spécialiser pour tous les cas possibles, on a une explosion combinatoire des sous-classes à créer pour s'adapter à toutes les situations...

#### Solutions utilisant des patterns reposant sur la composition : Adapter, Bridge, Decorator

Il donne une première "solution" intéressante utilisant le **pattern Adapter** = il créé un Adapter pour chaque type de destination possible, pour qu'ils se manipulent comme des fichiers : `FileLikeSocket` ou `FileLikeSyslog`, et ainsi, on n'a plus qu'un axe à spécialiser par héritage = le niveau de filtrage.

Il donne une seconde "solution" utilisant le **pattern Bridge** = séparer interface (capable de filtrer) et implémentation (capable de choisir où les logs finissent) en deux objets distincts, le premier utilisant le second par composition.

> Once again, the subclass explosion is avoided because two kinds of class are composed together at runtime without requiring either class to be extended.

Il donne une troisième "solution" utilisant le **pattern Decorator** : on peut alors stacker plusieurs classes filtrantes avant le logger final.

> Again, as with all Composition Over Inheritance solutions to a problem, classes are composed at runtime without needing any inheritance:
>
> There’s a crucial lesson here: design principles like Composition Over Inheritance are, in the end, more important than individual patterns like the Adapter or Decorator. Always follow the principle. But don’t always feel constrained to choose a pattern from an official list

Son message est qu'il faut retenir le principe plutôt que le pattern !

#### Dodge: "if" statements

Note : ses paragraphes "Dodge" sont des erreurs à ne pas faire, alors même qu'elles peuvent paraîtres tentantes.

Intéressante comparaison à une implémentation fonctionnellement équivalente, à base de `if` uniquement. Quelques avantages (principalement plus lisible, et tout le code est localisé à un seul endroit) mais surtout beaucoup de défauts :

- Tout le code a beau être à un seul endroit, le code spécifique à une seule feature (e.g. la gestion des sockets) est spreadée un peu partout.
- _Deletability. An underappreciated property of good design is that it makes deleting features easy. (...) In the case of our class-based solutions, we can trivially delete a feature like logging to a socket by removing the SocketHandler class and its unit tests once the application no longer needs it._
- Testing : pour tester le logging sur socket, on a besoin d'instancier plein de trucs qui n'ont rien à voir.
- _Efficiency : Even if you want a simple unfiltered log to a single file, every single message will be forced to run an if statement against every possible feature you could have enabled. The technique of composition, by contrast, only runs code for the features you’ve composed together._

> For all of these reasons, I suggest that the apparent simplicity of the if statement forest is, from the point of view of software design, largely an illusion. The ability to read the logger top-to-bottom as a single piece of code comes at the cost of several other kinds of conceptual expense that will grow sharply with the size of the codebase.

#### Dodge: Multiple Inheritance

Plutôt que d'utiliser la composition, on pourrait être tenté d'utiliser l'héritage multiple. Au final, ça ressemble beaucoup à un design à base de décorateur. Du coup, on peut comparer la façon dont les deux designs assemblent des briques de base (logger et filter) :

- bad 1 : l'héritage multiple utilise _l'implémentation_ des briques de base, là où les décorateurs utilisent uniquement leur interface ! (du coup, tester un design à base d'héritage multiples est plus difficile)
- bad 2 : construire une classe à plusieurs parents nécessite un constructeur spécifique (qui appelle les constructeurs des parents). Ce code de construction est du code supplémentaire (à tester/maintenir) par rapport à une composition à base de décorateurs.
- bad 3 : avec l'héritage multiple, on a des risques de collisions de noms d'attributs/méthodes des classes parentes. De surcroît : pas forcément facile à détecter dans des tests : _For any of these reasons, the unit tests of two base classes can stay green even as their ability to be combined through multiple inheritance is broken (...) Only by testing every combination of m×n base classes in your application can you make it safe for the application to use such classes at runtime._
- bad 4 : l'introspection est plus complexe avec héritage multiple
- bad 5 : les décorateurs peuvent être modifiés au runtime (e.g. selon les préférences de l'utilisateur, on passe tel paramètre au décorateur, voire on le shunte complètement), alors que l'héritage multiple est figé statiquement (à moins de faire de la magie noire avec le MRO)
- bad 6 : l'ordre de définition des classes parents n'est pas obvious du tout, alors que l'ordre des décorateurs l'est

> At least in this example, solving a design problem with inheritance is strictly worse than a design based on composition.

#### Dodge : Mixins

Pas grand chose à dire : une mixin est une classe dont on hérite, mais sans état, qui ne sert qu'à fournir une méthode.

Utiliser une mixin est un peu plus simple qu'un vrai design par héritage multiple, mais a tout de même certains des inconvénients de celui-ci.

#### Dodge: Building classes dynamically

> neither traditional multiple inheritance nor mixins solve the Gang of Four’s problem of “an explosion of subclasses to support every combination” — they merely avoid code duplication when two classes need to be combined.
>
> Multiple inheritance still requires, in the general case, “a proliferation of classes” with m×n class statements that each look like:
>
>     class FilteredSocketLogger(FilteredLogger, SocketLogger):

En python, on a plus de possibilités que dans d'autres langages, et il est donc envisageable de résoudre notre problème en construisant des classes dynamiquement au runtime.

> Instead of building all m×n possible classes ahead of time and then selecting the right one, we can wait and take advantage of the fact that Python not (...supports...) a builtin type() function that creates new classes dynamically at runtime

Désavantages : lisibilité, clarté du code, manque de familiarité des devs python avec les classes dynamiques, difficultés à debugger (pas de grep du nom de la classe dans le code, mypy et ide ne marcheront pas, ...), ...

En résumé, essayer de sauver l'approche par héritage multiple grâce aux classes dynamiques n'est pas une bonne idée.

Et le TL;DR de ce gros chapitre, c'est qu'avec l'héritage dynamique, on a une explosion combinatoire des classes à créer quand on veut étendre le code, alors que la composition n'a pas ces écueils.

## Python-Specific Patterns

### The Global Object Pattern

https://python-patterns.guide/python/module-globals/

> Python parses the outer level of each module as normal code. Un-indented assignment statements, expressions (...) will execute as the module is imported. This presents an excellent opportunity to supplement a module’s classes and functions with constants and data structures that callers will find useful — but also offers dangerous temptations: mutable global objects can wind up coupling distant code, and I/O operations impose import-time expense and side effects.

**Pattern Module Global** = définir quelque chose au niveau global du module.

Il est utilisé dans différents cas :

- USE 1 = Prebound method = rendre publique une méthode bindée à un objet, de façon à utiliser la méthode et son objet facilement.
- USE 2 = Sentinel Object = une valeur particulière faisant office de sentinelle peut être exposée au niveau global du module. Ça n'est pas une obligation, elle peut également être dans une classe, une closure, ...
- USE 3 = The Constant Pattern = _Modules often assign useful numbers, strings, and other values to names in their global scope. (...) Remember that these are “constants” only in the sense that the objects themselves are immutable. The names can still be reassigned. Or deleted, for that matter._

    > On rare occasions, a module global which the code clearly never intends to modify uses a mutable data structure anyway. Plain mutable sets are common in code that pre-dates the invention of the frozenset. Dictionaries are still used today because, alas, the Standard Library doesn’t offer a frozen dictionary.

    Définir des constantes est un peu plus lent mais a plein d'avantages :

    > Constants are often introduced as a refactoring: the programmer notices that the same value 60.0 is appearing repeatedly in their code, and so introduces a constant SSL_HANDSHAKE_TIMEOUT for the value instead. Each use of the name will now incur the slight cost of a search into the global scope, but this is balanced by a couple of advantages. The constant’s name now documents the value’s meaning, improving the code’s readability. And the constant’s assignment statement now provides a single location where the value can be edited in the future without needing to hunt through the code for each place 60.0 was used.

- USE 4 = Import-time computation = _Sometimes constants are introduced for efficiency, to avoid recomputing a value every time code is called._

    > A constant can also capture the result of a conditional to avoid re-evaluating it each time the value is needed

- USE 5 = Dunder Constants = Several Module Global dunder constants are set by the language itself. (...) The two encountered most often are `__name__` (...), and `__file__`

    Il y en a quelques autres, qui ne sont ni très répandues, ni très homogène dans leur utilisation... De plus, les dunder-constants sont pas forcément très adaptées : _Why not author and version instead, without the dunders? An early reader probably misunderstood dunders, which really meant “special to the Python language runtime,” as a vague indication that a value was module metadata rather than module code._


**Global Object Pattern** = instancier un objet à l'import-time, et le binder à un nom toplevel du module. Exemple : compiler une regex.

> Compiling a regular expression as a module global is a good example of the more general Global Object pattern. It achieves an elegant and safe transfer of expense from later in a program’s runtime to import time instead.

Avantage = on ne construit l'objet qu'une fois, et à l'import-time plutôt que runtime. Inconvénients = tout le monde paye ce prix, même ceux qui ne se servent pas du global-object.

Global Objects that are mutable : ça peut arriver, mais c'est pas un bon pattern, car ça fait une variable globale partagée, dont les inconvénients sont bien connus :

- Les tests ne sont plus indépendants et isolés, mais dépendent de la valeur de cette variable globale (qui peut être mutée dans un test, puis utilisée mutée dans un autre)
- Difficile de savoir qui mute la variable globale.
- Les mutations de la variable globale par un test peuvent survivre à l'exécution du test.

> My advice, to the extent that you can, is to write code that accepts arguments and returns values computed from them

Import-time I/O : TL;DR = retarder les I/O au runtime, plutôt qu'à l'import-time.

> Many of the worst Global Objects are those that perform file or network I/O at import time. They not only impose the cost of that I/O on every library, script, and test that need the module, but expose them to failure if a file or network is not available.

Exemple d'erreur = supposer que `/etc/hosts` existe. On pourrait penser que retarder le check de `/etc/host` au runtime (plutôt qu'à import global du module) ne change rien, mais en fait c'est mieux si ça échoue au runtime : déjà, au runtime, on est souvent protégé par un `try except` englobant tout le main, donc on peut survivre à l'erreur (et par exemple fonctionner en mode dégradé), ou bien la logger correctement, plutôt que printer la stacktrace sur stdout. De plus, si le check échoue à l'import, l'échec est inévitable, alors que si le check n'échoue qu'au runtime à l'appel de la fonction utilisant l'objet global, si ça se trouve, on n'appellera pas le code qui échoue.

> For all of these reasons, it’s best for your global objects to wait until they’re first called before opening files and creating sockets — because it’s at the moment of that first call that the library knows the main program is now up and running, and knows that its services are in fact definitely needed in this particular run of the program.

### The Prebound Method Pattern

https://python-patterns.guide/python/prebound-methods/

TL;DR = _A powerful technique for offering callables at the top level of your module that share state through a common object._

En gros, on veut proposer des toplevel fonctions, qui partagent un état. Solution = les toplevel fonctions sont en fait des méthodes d'une unique instance (qui permet donc de partager l'état). Exemple = les fonctions random, qui doivent partager l'état du PRNG.

> Behind the scenes these callables are, in fact, methods that have all been bound ahead of time to a single instance of Random that the module itself has gone ahead and constructed.

L'alternative à ce pattern est de stocker l'état directement dans des toplevel VARIABLES, plutôt que comme une toplevel INSTANCE, mais c'est moins pratique : déjà, comme le module est un singleton, on ne peut plus facilement en instancier plusieurs (e.g. pour des tests), à moins de faire de la magie noire avec les modules. De plus, c'est moins bugprone que les méthodes partagent implicitement leur état, plutôt qu'elles doivent s'assurer d'avoir correctement mis à jour l'état global partagé. (NdM : dit autrement, avoir une classe permet de forer le respect de ses invariants).

Une fois qu'on a encapsulé les méthodes et leur état dans une classe plutôt que dans des toplevel variables/fonctions, C'est tout à fait envisageable de NE PAS pousser plus loin à proposer des prebounds méthodes bindées à une instance globale. Dit autrement, l'encapsulation a déjà apporté sa valeur :

> The software engineer would do well to stop for a moment and to think quite seriously about the fact that in the vast majority of cases simply defining a class like this is enough. It will take only a single statement for your user to create an instance of your class. If their own application’s architecture is too deeply compromised for them to easily pass the instance everywhere it’s needed, they can always store the instance as a module global in one of their own modules for the use of the rest of their code.

Toutefois, pour un RNG, ça fait sens de pousser un cran plus loin, et que le module crée une instance de la classe au toplevel :

- déjà, instancier la classe est assez coûteux, car elle fait un appel système + consomme de l'entropie de l'OS
- de plus, partager le même RNG augmente la randomness, yay !
- enfin, dans le cas de random, l'intérêt du module réside essentiellement dans les méthodes exposées, du coup avoir instancié "à l'avance" la classe n'est jamais (ou presque) gâché.

> If the costs and benefits strike a similar balance for a module of your own you are designing, then the Prebound Method pattern can let you deliver remarkable convenience.

The **Prebound Method Pattern** :

> To offer your users a slate of Prebound Methods:
> 1. Instantiate your class at the top level of your module.
> 2. Consider assigning it a private name prefixed with an underscore _ that does not invite users to meddle with the object directly.
> 3. Finally, assign a bound copy of each of the object’s methods to the module’s global namespace.
>
> Users will now be able to invoke each method as though it were a stand-alone function. But the methods will secretly share state thanks to the common instance that they are bound to, without requiring the user to manage or pass that state explicitly

Attention à ce que la construction de l'instance ne soit pas trop coûteuse, ne fasse pas d'I/O (si c'est le cas, mieux vaut retarder la construction, et ne la faire qu'au premier appel).

> But for lightweight objects that can be instantiated without substantial delay or complication, the Prebound Methods pattern is an elegant way to make the stateful behavior of a class instance available up at a module’s global level.

### The Sentinel Object Pattern

https://python-patterns.guide/python/sentinel-object/

> The pattern most often uses Python’s built-in `None` object, but in situations where `None` might be a useful value, a unique sentinel `object()` can be used instead to indicate missing or unspecified data.

Ah, `object()` peut remplacer `None` !

Description du Pattern avec la méthode `list.find` qui renvoie `-1` si elle ne trouve pas la valeur recherchée :

> This is a classic example of a sentinel value. The value -1 is simply an integer, just like the function’s other possible return values, but with a special meaning that has been agreed upon ahead of time

Des pointeurs nuls (plutôt que des pointeurs vers des objets python, comme attendus) sont utilisés dans l'implémentation C de Python pour indiquer qu'une fonction a échoué.

Attention : un `Null object` est différent d'un `Null pointer` ! Un `Null object` est UN OBJET VALIDE et pleinement construit, mais avec une valeur particulière, convenue comme étant "nulle", sans intérêt.

Avantage 1 =  pas besoin de checker `if x is None:` un peu partout (vu que le `Null object` est un vrai objet, qui a le même type que les autres objets non-null).

Avantage 2 = le code est plus lisible : `if x is NO_MANAGER` est plus lisible et intuitif que `if x is None`.

> While not appropriate in all situations — it can, for example, be difficult to design Null Objects that keep averages and other statistics valid — Null Objects appear even in the Python Standard Library: the logging module has a NullHandler which is a drop-in replacement for its other handlers but does no actual logging.

La sentinelle qu'on utilise le plus en python est `None`, mais elle n'est pas toujours utilisable : `None` peut être une valeur ayant du sens, utile. De plus, on peut parfois passer `None` "accidentellement". Dans ces cas-là, `object()` est un bon remplaçant !

Point important : les sentinelles sont des _identity types_, et non des _value types_ !

> But whatever the application, the core of the Sentinel Object pattern is that it is the object’s identity — not its value — that lets the surrounding code recognize its significance. If you are using an equality operator to detect the sentinel, then you are merely using the Sentinel Value pattern described at the top of this page. The Sentinel Object is defined by its use of the Python is operator to detect whether the sentinel is present.

(D'où le fait que `object()` est une meilleure sentinelle que `None`)

## Gang of Four: Creational Patterns

### The Abstract Factory Pattern

https://python-patterns.guide/gang-of-four/abstract-factory/

TL;DR = **Abstract Factory Pattern** (= interface fournissant des méthodes pour construire une famille d'objets) est inutile en python, qui peut passer des classes et fonctions (qui pourront créer des objets) en tant qu'arguments de fonctions.

> The Abstract Factory is an awkward workaround for the lack of first-class functions and classes in less powerful programming languages. It is a poor fit for Python, where we can instead simply pass a class or a factory function when a library needs to create objects on our behalf.
>
> The Python Standard Library’s json module is a good example of a library that needs to instantiate objects on behalf of its caller. Consider a JSON string like this one:

L'exemple fil-rouge de cette section est la lib `json` qui prend une string (contenant du json) en entrée, et crée des objets python en sortie : listes, floats, strings, etc. On peut vouloir créer des types particuliers (par exemple, au lieu d'utiliser `float` pour stocker les nombres déserializés, on utilise `Decimal`, qui permet une meilleure précision pour les transactions financières).

Plus généralement, le problème à résoudre :

> In the course of performing its duties, a routine is going to need to create a number of objects on behalf of the caller.
> But the class that the routine would instantiate by default does not cover all possible cases.
> So instead of hard-coding that default class and making customization impossible, the routine wants to let the caller specify which classes it will instantiate.

En python, les callables sont first-class (ils peuvent être passés en paramètre comme n'importe quelle autre valeur) :

> First-class callables offer a powerful mechanism for implementing object “factories”, a fancy term for “routines that build and return new objects.”

Du coup, pour "aboutir" au Abstract Factory Pattern (qui est en fait inutile en python), il restreint petit à petit le langage :

_Restriction: outlaw passing callables_  = dans ce paragraphe, il explore l'hypothèse fictive où python ne permet plus de passer n'importe quel callable comme argument de fonction, on ne peut notamment PLUS passer de fonctions (mais uniquement des instances d'objets, ou des classes). Dans ce cas, `build_decimal` ne peut plus être une fonction, mais doit être une méthode de classe (éventuellement `staticmethod`).

_Restriction: outlaw passing classes_ = dans ce paragraphe, on continue hypothèse fictive en supposant qu'on ne peut pas non plus passer de classe (mais juste des instances). Dans ce cas, on se retrouve à devoir instancier un objet qui ne sert à rien, si ce n'est à proposer sa méthode `build_decimal`...

> a method that never uses self should not actually be a method in Python. But such are the depths to which we’ve been driven by these artificial restrictions

_Generalizing: the complete Abstract Factory_ = pour l'Abstract Factory, on généralise le principe :

- l'instance peut créer non seulement des `Decimal`, mais également des `list`, et tous les autres objets dont a besoin le json loader pour déserializer: _Every choice it needs to make about object instantiation is deferred to the factory instead of taking place in the parser itself._ Au final, l'instance est responsable de créer une _famille_ d'objets.
- de plus, pour plus de souplesse, on fait en sorte que le loader ne dépendent pas directement de la Factory pouvant créer tous les objets nécessaires, mais uniquement d'une interface (d'où le nom du pattern : Abstract Factory).

Pour conclure :

> It’s like something you might do in Python, but made overly complicated. So avoid the Abstract Factory and use callables as factories instead.

### The Builder Pattern

https://python-patterns.guide/gang-of-four/builder/

Quand on parle du **Pattern Builder**, on peut parler de trois choses différentes :

- le pattern original tel que décrit par le Gang of Four, au final peu utilisé
- un usage "secondaire" décrit par le Gang of Four, au final très utilisé car très pratique
- un autre pattern (qui n'a rien à voir) qui a pris le nom de Builder, qui permet de compenser l'absence d'argument optionnels dans certains langages, inutilisé en python.

L'article les décrit dans l'ordre 2, 1, 3.

Usage n°2 = l'usage le plus répandu du pattern Builder, pour créer facilement une hiérarchie d'objets, en masquant toutes les créations derrière un ou plusieurs appels. Exemple avec matplotlib = un seul appel `pyplot.plot` masque la création d'une douzaine d'objets de la lib !

Un point rigolo = faciliter la création d'objet est un argument "marketing" pour faire bien voir sa librairie :

> The Builder pattern is now deeply ingrained in Python culture thanks in part to the pressure that library authors feel to make the sample code on their front page as impressively brief as possible
>
> The fact that some libraries rely on their callers to tediously instantiate objects is even used as advertisement by their competitors. For example, the Requests library famously introduces itself to users by comparing its one-liner for an HTTP request with authentication to the same maneuver performed with the old urllib2 Standard Library module

Nuance entre Builder, Abstract Factory, et Facade : si les objets créés ne sont pas retournés (comme pour `pyplot.plot`), théoriquement, on est plutôt dans le **Pattern Facade** que Builder. Par ailleurs, il semblerait que le pattern Builder désigne théoriquement l'enchaînement de plusieurs étapes, et que c'est l'étape finale qui créée l'objet (alors qu'Abstract Factory ne nécessite qu'une étape pour créer chaque objet, mais peut créer toute une famille de plusieurs objets).

Usage n°1 = l'usage original souhaité par le Gof, où une même série d'appels à un builder B1 et à un builder B2 produisent toutes deux la hiérarchie d'objets souhaités (qui est différente selon qu'on a utilisé B1 ou B2). En pratique, par rapport à l'usage précédent, dans l'esprit du GoF, il y avait systématiquement PLUSIEURS implémentations concurrentes de builders, alors que dans son usage le plus répandu, il n'y a qu'un seul Builder...

Selon l'auteur, la raison pour laquelle cet usage original n'a que peu été utilisé, est que en python, plutôt que d'implémenter deux builders différents qu'on appellerait avec la même série de méthodes, on n'implémenterait qu'un unique builder qui maintiendrait un état intermédiaire construit à partir de la série des appels clients, et c'est cet unique état intermédiaire qui serait utilisé comme donnée d'entrée à plusieurs implémentations différentes :

> If a modern Python Library does want to drive two very different kinds of activity from the same series of client constructor calls, it would be very unusual for that library to offer two completely separate implementations of the same Builder interface — two builders that both have to be capable of being prodded through the same series of incremental client-driven updates to produce a coherent result.
>
> Instead, modern python libraries are overwhelmingly likely to have a single implementation of a given Builder, one that produces a single well-defined intermediate representation from the caller’s function and method invocations. That representation, whether publicly documented or private and internal to the library, can then be provided as the input to any number of downstream transformation or output routines — whose processing will now be simpler because they are free to roam across the intermediate data structure at their own pace and in whatever order they want.
>
> [...]
>
> It is, therefore, probably Python’s very rich collection of data types for representing deep compound information — tuples, lists, dictionaries, classes — and the convenience of writing code to traverse them that has produced almost an entire absence of the full-tilt Builder pattern from today’s popular Python libraries.

Usage n°3 = enfin, l'usage n°3 a le même nom "Builder" mais n'a rien à voir avec le pattern du GoF : il s'agit de pallier le fait qu'une classe avec beaucoup d'attributs immutables force son client à tous les définir à la construction :

> You can imagine the verbose and unhappy consequences. Not only will every single object instantiation have to specify every one of the dozen attributes, but if the language does not support keyword arguments then each value in the long list of attributes will also be unlabeled. Imagine reading a long list of values like None None 0 '' None and trying to visually pair each value with the corresponding name in the attribute list.

Dans ce cas, l'alternative "Builder" consiste à créer une classe jumelle, avec les mêmes attributs, MAIS mutables. Cette classe jumelle permet donc de les définir séparément plutôt que tous d'un coup, et donne accès un peu plus tard à quelque chose pour construire l'objet final d'un seul coup :

```python
# sans Builder, on est obligé de tout définir sur la même ligne :
myobject = Port(517, "myapp", "UDP", ...)

# avec Builder, on peut créer l'objet en plusieurs fois :
mybuilder = PortBuilder(517)
mybuilder.name = "myapp"
mybuilder.protocol = "UDP"
myobject = mybuilder.build()
```

Normalement, on ne doit pas voir ce pattern en python.

### The Factory Method Pattern

https://python-patterns.guide/gang-of-four/factory-method/

---

REPRENDRE À : The Factory Method Pattern
