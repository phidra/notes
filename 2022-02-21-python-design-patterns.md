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
         * [Position du problème et pattern](#position-du-problème-et-pattern)
         * [Alternative 1 = use Dependency Injection](#alternative-1--use-dependency-injection)
         * [Alternative 2 = use a Class Attribute Factory](#alternative-2--use-a-class-attribute-factory)
         * [Alternative 3 = use an Instance Attribute Factory](#alternative-3--use-an-instance-attribute-factory)
         * [Alternative 5 = Instance attributes override class attributes](#alternative-5--instance-attributes-override-class-attributes)
         * [Complément = Any callables accepted](#complément--any-callables-accepted)
         * [Pattern Factory Method](#pattern-factory-method)
      * [The Prototype Pattern](#the-prototype-pattern)
      * [The Singleton Pattern](#the-singleton-pattern)
      * [The Composite Pattern](#the-composite-pattern)
      * [The Decorator Pattern](#the-decorator-pattern)
      * [The Flyweight Pattern](#the-flyweight-pattern)

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

#### Position du problème et pattern

Supposons qu'on ait un objet A (e.g. `HttpConnectionPool`), dont une partie du métier nécessite d'être responsable de créer d'autres objets X (e.g. `HttpConnection`). Supposons également que les `HttpConnection` créées doivent suivre une configuration particulière (p.ex. parce qu'elles seront utilisées derrière un proxy particulier).

Le problème est : comment customiser `HttpConnectionPool` pour créer ces `HttpConnection` particulières ?

Le **Pattern Factory Method** conssite à ce que le code métier de `HttpConnectionPool` fasse appel à une méthode `create_http_connection` pour créer les objets qu'il va utiliser, et à ce que l'utilisateur puisse dériver `HttpConnectionPool` : la classe dérivée va surcharger `create_http_connection` pour lui faire créer des `HttpConnection` capables de traverser le proxy.

Dans le reste de la section, il présente des alternatives plus pythoniques.

#### Alternative 1 = use Dependency Injection

Principe = pour éviter que l'objet A doivent CRÉER d'autres objets X, utiliser de l'injection de dépendance pour lui passer directement les objets X. Exemple = `json.load` accepte un file-like object, plutôt qu'un chemin, pour éviter d'avoir à instancier lui-même le file-like object à partir du chemin. Les avantages sont nombreux :

> - Decoupling: the library doesn’t need to know all the parameters accepted by the open() method, and doesn’t need to accept every one of them as a parameter itself. If the file object were to grow more creation parameters in the future, json.load() won’t need to change.
> - Efficiency: If you already have the file open for other reasons, you can go ahead and provide the open file object. The library won’t insist on re-opening the file.
> - Flexibility: You can pass any file-like object you want. It can be a subclass of the standard file object, or be completely independent. You could pass a StringIO that operates directly on data in RAM instead of needing the JSON written to disk first.

Au final, on shunterait complètement la nécessité pour A de créer des X.

> So (a) if your class always needs a particular object built, and (b) if users will normally want to at least customize the parameters with which it is instantiated, you should strongly consider Dependency Injection.

#### Alternative 2 = use a Class Attribute Factory

Principe = _The class that needs to be created can be attached as an attribute on the class that will be doing the creating._

On spécialise la classe A en A' qui utilise un attribut différent, ainsi, les méthodes de A qui utilisent directement l'attribut de classe pour instancier les objets X utiliseront en fait l'attribut de A'.

(NdM : dériver A en A' n'est pas indispensable, cf. plus bas)

#### Alternative 3 = use an Instance Attribute Factory

> Why should you have to subclass an object merely to customize its behavior?

(NdM : ah ben il a mieux formulé que moi mes critiques avec les approches OO)

Principe = la façon pour A de créer les objets X est d'appeler un attribut d'instance `A.create_x()`, qui est passée à A à la construction (ou pas, si `create_x` accepte une valeur par défaut).

On tire ainsi parti du fait que les fonctions et classes sont des first-class objects en python, et peuvent être passés comme arguments de fonctions, ou stockés dans des variables.

Et en passant `create_x` à la construction de `A`, on n'a plus besoin de la dériver pour la customiser :

```python
my_decoder = JSONDecoder(parse_float=Decimal)
```

#### Alternative 5 = Instance attributes override class attributes

Principe = A peut très bien avoir un attribut de classe `create_x` ; et pour que le client customise le comportement, au lieu de dériver A en A', il se contente de définir un attribut d'instance `self.create_x` : comme les attributs d'instances ont priorité sur les attributs de classe, le code de A qui utilise `create_x` utilisera en fait l'attribut d'instance.

#### Complément = Any callables accepted

Comme python n'utilise pas de syntaxe particulière pour créer des objets, `create_x` peut être n'importe quoi : une classe (et `create_x(...)` instanciera la classe), mais également un callable quelconque qui renvoit un objet : fonction, bound-method, partial, foncteur, etc.

#### Pattern Factory Method

Le principe du pattern = la classe `A` a un métier complexe, et une partie de celui-ci implique de créer des `X`. Pour ce faire, `A` a implémente une méthode `create_x`. Lorsque l'utilisateur veut spécialiser la création des X, il dérive `A` et surcharge `create_x`.

Vue la puissance de python, ce pattern ne devrait jamais être nécessaire en python, mais il l'est dans des langages moins évolués :

> Imagine that you were using a language where:
>
> - Classes are not first-class objects. You are not allowed to leave a class sitting around as an attribute of either a class instance or of another class itself.
> - Functions are not first-class objects. You’re not allowed to save a function as an instance of a class or class instance.
> - No other kind of callable exists that can be dynamically specified and attached to an object at runtime.
>
> Under such dire constraints, you would turn to subclassing as a natural way to attach verbs — new actions — to existing classes, and you might use method overriding as a basic means of customizing behavior.

### The Prototype Pattern

https://python-patterns.guide/gang-of-four/prototype/

D'après [la page wikipedia](https://fr.wikipedia.org/wiki/Prototype_(patron_de_conception)), le pattern consiste à remplacer les instanciations explicites d'une classe, jugées coûteuses (ou bien introduisant du couplage à la construction de la classe), par l'appel à une méthode `clone()` sur une instance déjà existante. Exemple d'application :

> Exemple où prototype s'applique : supposons une classe pour interroger une base de données. À l'instanciation de cette classe on se connecte et on récupère les données de la base avant d'effectuer tous types de manipulation. Par la suite il sera plus performant pour les futures instances de cette classe de continuer à manipuler ces données que de réinterroger la base. Le premier objet de connexion à la base de données aura été créé directement puis initialisé. Les objets suivants seront une copie de celui-ci et donc ne nécessiteront pas de phase d'initialisation.

L'auteur a une définition différente du problème, que je comprends moins :

> It is this situation — supplying a framework with a menu of classes that will need to be instantiated with pre-specified argument lists — that the Prototype pattern is designed to address.

Sa solution est donc de lister les différentes classes à instancier avec leurs arguments :

```python
menu = {
    'whole note': (Note, (1,)),
    'half note': (Note, (2,)),
    'quarter note': (Note, (4,)),
    'sharp': (Sharp, ()),
    'flat': (Flat, ()),
}
```

Python permet beaucoup de variations autour de ce thème.

Le pattern du GoF qu'il décrit retombe sur celui décrit par wikipedia : les différentes notes peuvent être créer les unes à partir des autres, chaque classe dérivée implémente sa propre version de `clone()`.

### The Singleton Pattern

https://python-patterns.guide/gang-of-four/singleton/

Le terme "singleton" peut avoir plusieurs sens en python :

- un tuple qui ne contient qu'un élément
- un module (vu qu'un module d'un nom donné ne sera importé qu'une fois)
- une instance globale à un module (cf. Global Object Pattern)
- _Individual flyweight objects that are examples of The Flyweight Pattern are often called “singleton” objects by Python programmers._
- l'objet correspondant réellement au pattern Singleton

Il n'y avait pas de "vrais" singletons en python2 : il y avait des objets-singletons (e.g. `None` ou `Ellipsis`) mais pas de classes permettant de les créer, qui renvoyaient toujours la même instance. En python3, ces classes existent, mais ce n'est pas très utilisé :

> This makes life easier for programmers who need a quick callable that always returns None, though such occasions are rare. In most Python projects these classes are never called and the benefit remains purely theoretical. When Python programmers need the None object they use The Global Object Pattern and simply type its name.

Dans le livre du GoF, ils visaient C++, qui a une syntaxe particulière (`new`) pour instancier des classes. Or, le standard C++ impose que `new` renvoie toujours une nouvelle instance, du coup, comment renvoyer un singleton ?

Le **Pattern Singleton** = on rend le constructeur protégé ou privé (de sorte qu'on ait une erreur de compilation si on l'utilise), et on fournit une méthode de classe pour créer les objets, qui renvoie toujours la même instance = le singleton. Une autre possibilité (non-retenue) aurait été que le singleton soit accessible dans l'espace de nom global.

> In one sense, Python started out better prepared than C++ for the Singleton Pattern, because Python lacks a new keyword that forces a new object to be created. Instead, objects are created by invoking a callable, which imposes no syntactic limitation on what operation the callable really performs

Pour simuler ce comportement en python, l'auteur montre un exemple où `__init__` raise une `RuntimeError`, et où à la place on peut utiliser une `@classmethod` nommée `instance()` qui permet de récupérer le singleton (en le créant au premier appel).

Le pattern original du GoF n'est pas directement applicable à python (car pas de protected/private), mais on peut en avoir un équivalent à base de `__new__`. Ces implémentations sont :

1. moins lisibles (car `__new__` est peu connu des devs)
2. moins intuitives (une opération qui ressemble à une instanciation n'en est pas vraiment une)
3. radicales (si on décide qu'un objet est singleton, on ne peut plus revenir en arrière, alors qu'avec le Global Object Pattern, rien n'empêche de créer d'autres instances en plus de l'instances globale partagée, ce qui s'avère très précieux pour les tests).

La conclusion de la section, c'est donc qu'au lieu d'utiliser un singleton, il vaut mieux utilier des toplevel variables dans un module, comme [préconisé par la doc python](https://docs.python.org/3/faq/programming.html#how-do-i-share-global-variables-across-modules) :

> The canonical way to share information across modules within a single program is to create a special module (often called config or cfg). Just import the config module in all modules of your application; the module then becomes available as a global name. Because there is only one instance of each module, any changes made to the module object get reflected everywhere.

### The Composite Pattern

https://python-patterns.guide/gang-of-four/composite/

L'auteur donne une bonne description du **Pattern Composite** :

> The Composite Pattern suggests that whenever you design “container” objects that collect and organize what we’ll call “content” objects, you will simplify many operations if you give container objects and content objects a shared set of methods and thereby support as many operations as possible without the caller having to care whether they have been passed an individual content object or an entire container.

Principe = en deux mots, l'idée est de pouvoir manipuler un conteneur et son contenu de la même façon, ce qui permet de travailler sur les différents items d'une hiérarchie d'objets de façon homogène.

Exemple classique = les fichiers et répertoires d'un filesystem. C'est d'ailleurs l'exemple suivi par la section : `ls` accepte indifféremment en entrée un chemin vers un fichier ou un répertoire.

Avantages :

- ça permet de ne pas avoir à switcher entre deux commandes (l'une pour les fichiers, l'autre pour les répertoires)
- un wildcard tel que `/etc/logrotate*` peut matcher indifféremment avec des fichiers (`/etc/logrotate.conf`) et des répertoires (`/etc/logrotate.d/`), ce qui n'aurait pas été possible avec des commandes différenciées
- plus généralement, les scripts qui se fichent de savoir s'ils travaillent sur un fichier ou un répertoire peuvent passer à `ls` un argument sans avoir à vérifier son type réel

> This not only lowers cognitive overhead, but makes shell scripts easier to write — there are many operations that a script can simply run blindly without needing to stop and check first whether a path names a file or a directory.

En python, le **Pattern Composite** est plus simple que dans d'autres langages, et notamment on peut se reposer sur le duck-typing (structural subtyping) plutôt que sur l'héritage (nominal subtyping) pour que le container et les objets contenus présentent un comportement homogène.

La partie difficile, c'est de savoir quand choisir de ne PAS manipuler fichiers et répertoires de façon homogène... Par exemple, la commande de création est différente pour les deux : `touch` et `mkdir` :

> But the designers thought that the two operations were conceptually different enough to deserve separate commands.

Il prend l'exemple du troisième bit `x` des permissions de fichiers/réperotires pour illustrer que c'est pas toujours facile de choisir entre manipuler les deux entités de façon homogène, ou bien avoir des comportements propres à chacun. Si on se goure et qu'on mutualise là où il n'y a pas lieu d'être, on va créer des cas particuliers bizarres...

Il prend l'exemple concret de "lis-moi le contenu de ce fichier/répertoire" pour illustrer que quand les opérations sont différentes (et notamment quand les opérations renvoient un type de données différentes : byte-stream pour un fichier, liste de strings pour un répertoire), il faut rompre la symétrie et manipuler les deux entités de façon différente. C'est très visible s'il faut que le code client analyse la valeur retournée à coup de `if` ou `isinstance` :

> if your desire to create symmetry between container and content leads you to engineer calls that require an if statement or isinstance() to safely handle their return value, then the desire for symmetry has led you astray

L'auteur note que là où par le passé les hiérarchies avaient la cote (notamment dans les noms de pacakges tels que `zope.app/i18n`, ou dans les structures de données tels que les arbres binaires), de nos jours, elles sont tombées en désuétude au profit de structures plus flat, ce qui se retrouve même dans le zen de python = _flat is better than nested_. Exception = les hiérarchies sont encore bien présentes dans les documents.

Exemple d'application du pattern Composite = les GUI construites avec TKinter : pour éviter d'avoir à tester que le widget a des enfants et implémente `winfo_children()` (soit par `isinstance`, soit avec `getattr`), TOUS les widgets implémentent `winfo_children()` : ceux pour qui ça n'a pas de sens (comme les labels) renvoient simplement une liste vide.

À noter que cette façon de faire est controversée : certes elle facilite la manipulation des widgets, mais on pourrait arguer en retour que c'est contre-intuitif d'avoir un `winfo_children()` sur un widget qui n'aura jamais d'enfants...

L'intérêt est de rendre les différents objets interchangeables :

> The benefits of the symmetry that the Composite Pattern creates between containers and their contents only accrue if the symmetry makes the objects interchangeable.

Dans certains langages statiquement typés, ça veut obligatoirement dire hériter d'une interface (voire, d'une implémentation !) commune. En python, on peut soit hériter d'une classe parente, soit se reposer sur le duck-typing, avec les subtilités si souhaitées, comme l'utilisation d'`abc`, de `zope.interface` ou encore des protocoles...

En conclusion, en python, il recommande de se reposer sur le structural subtyping pour que conteneur et contenu présentent les mêmes comportements.

### The Decorator Pattern

https://python-patterns.guide/gang-of-four/decorator-pattern/

Ce paragraphe commence par un warning : le **Pattern Decorator** tel que décrit par le GoF est _différent_ de ce que python appelle un decorator (genre `@mydecorator`).

Son verdict = le pattern Decorator tel qu'imaginé par le GoF reste utile en python ! _Use it on the rare occasion when you need to adjust the behavior of an object that you can’t subclass but can only wrap at runtime._

Principe = un décorateur wrappe une classe tout en présentant exactement la même interface qu'elle ; il délègue ses appels de méthodes à la classe wrappée, en modifiant ou étendant leur comportement. Exemple typique = le décorateur ajoute un logge à l'appel de la méthode wrappée.

Ce type d'ajout de comportement est également possible en dérivant la classe, mais le **Pattern Decorator** a un énorme avantage = il est accessible sur un objet déjà existant, alors que dériver une classe n'est possible que si on contrôle la création de l'instance :

> For example, it isn’t helpful to subclass the Python file object if a library you’re using is returning normal file objects and you have no way to intercept their construction — your new MyEvenBetterFile subclass would sit unused. A decorator class does not have that limitation. It can be wrapped around a plain old file object any time you want, without the need for you be in control when the wrapped object was created.

Le reste de la section explore diverses implémentations.

Il donne un premier exemple intéressant où il commence par implémenter un décorateur "statique" d'un file-object à l'ancienne (pour logger les appels à `write`), où le décorateur implémente statiquement _toutes_ les méthodes du file-object (et même un triplet getter/setter/deleter pour chaque attribut !). Très verbeux, ce décorateur n'a que peu de valeur ajoutée, puisqu'il se contente d'être un passe-plat pour 98% de ses 100 lignes de code, et seul le wrapping de `write` fait quelque chose utile = logger. Conclusion : lourd, et pas pérenne par dessus le marché (puisqu'il ne faut pas oublier de le modifier dès que le file-object gagne des méthodes ou attributs).

En seconde approche, il réduit son wrapper à uniquement ce qui est nécessaire = le wrapping de `write()` pour logger, en supposant que les autres méthodes ne sont pas appelées dans notre cas. En taillant ainsi dans le lard, on est certes plus concis, mais on prend le risque de ne pas avoir wrappé assez (et de rencontrer une `RuntimeError` lorsqu'un usage aura besoin d'autre chose que `write()`).

En troisième approche, il fait un wrapper dynamique, à base de `__getattr__` et `__setattr__` (ainsi que d'autres dunder méthodes qui s'avèrent en fait nécessaires, telles que `__iter__` !) : les méthodes réellement implémentées par le wrapper (comme `write()`) sont appelées directement, et les autres sont déléguées via le wrapping dynamique à l'objet wrappé. Même si c'est marginalement plus lent, cette façon de faire est plus concise et plus robuste que le premier wrapper statique.

C'est cette troisième approche qu'il recommande.

WARNING : comme python permet de l'introspection poussée (en utilisant `dir()` ou `__dict__`), il sera toujours possible de détecter que l'objet qu'on utilise est un wrapper et non le vrai objet original. Par exemple, lister les attributs de l'instance donnera des résultats différents. Normalement, quand on fait de la _programmation_, on n'a pas besoin d'introspecter les entrailles des instances qu'on utilise. Mais si on fait de la _metaprogrammation_, ça peut être utile (e.g. debugger, framework, testing, etc.) et dans ce cas, aucun décorateur, pas même notre troisième décorateur dynamique, ne se comportera exactement comme l'instance wrappée...

Une avant-dernière façon hacky de wrapper est en fait de redéfinir dynamiquement l'attribut `write` de l'instance, pour que la nouvelle fonction `write` fasse à la fois appel à l'ancienne `write`, et ajoute le logging. Le problème, c'est que quand on redéfinit dynamiquement une méthode comme ça, elle ne reçoit pas `self` comme premier argument ; or, on en a besoin pour appeler l'ancienne fonction `write`. Il y a certes un moyen de contourner (capturer `self.write` dans une closure, et la nouvelle fonction qui ne reçoit pas self y aura accès tout de même), mais c'est crado.

Il mentionne une dernière façon encore plus crado = créer une nouvelle classe (dérivée de celle de l'instance, mais qui possède le comportement qui nous intéresse) et l'attribuer comme `__class__` de notre objet. Dans le cas général, ça ne fonctionne pas.

### The Flyweight Pattern

https://python-patterns.guide/gang-of-four/flyweight/

---

REPRENDRE À : The Flyweight Pattern
