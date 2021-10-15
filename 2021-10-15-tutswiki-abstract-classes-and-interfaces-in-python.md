# Abstract classes and interfaces in Python

- **url** = https://tutswiki.com/abstract-classes-and-interfaces-in-python/
- **type** = post
- **auteur** = [Chankey PATHAK](https://github.com/chankeypathak) (d'après le code-source de la page), dev senior
- **date de publication** = ???
- **source** = [TutsWiki](https://tutswiki.com/) : _TutsWiki is like Wikipedia focused for tutorials_
- **tags** = language>python ; topic>python>interfaces ; topic>python>abc ; topic>python>zope ; level>intermediate

**TL;DR** : notes en vrac lors de mes pérégrinations autour des interfaces python. Parle des différences entre les ABC et les zope.interfaces.

> Both the first and second are a peculiar way of documenting the code and help to limit (decouple) the interaction of individual abstractions in the program (classes).

> In the simplest case, using zope.interfaces is similar to ABC:

NdM : notion de `implementedBy` une classe vs. `providedBy` une instance (et comme en python les classes sont des objets, une classe peut à la fois implement et provide un comportement, ce qui aide à confuser un peu le tout).

> In fact, the implementation declaration (IVehicle) is a convention; just a promise that a given class and its objects behave that way. No real checks will be made.

Hum... déclarer une classe comme implémentant une zope.interface n'a pas d'impact concret, en dehors de register... Or, je recherche justement à ce que ça autorise le type-checking statique ?!

> The component architecture of Zope includes another important concept - adapters. Generally speaking, this is a simple design pattern that corrects one class for use somewhere where a different set of methods and attributes is required.

NdM : la partie `adapter` est pas claire... Il me faut une POC sur le sujet, sans doute... ce qui n'est pas clair, c'est le besoin auquel ça répond : pourquoi on voudrait qu'un `Guest` soit un `Desk` ?!

À noter tout de même : on peut dynamiquement récupérer les adapters d'une classe donnée.

> This infrastructure is useful for splitting code into components and linking them together.

La partie "linking them together" n'est justement pas encore claire...

> Upon closer inspection, it turns out that interfaces and abstract base classes are two different things.

Ça aussi ça n'est pas encore clair :-/

> Abstract classes basically hardcode the required front-end part. Checking an object against the interface of an abstract class is checked using the built-in isinstance function; class - issubclass. An abstract base class should be included in the hierarchy as a base class or mixin.

ceci est clair : utiliser ABC veut dire jouer avec `isinstance` / `issubclass`

> Interfaces are a declarative entity, they do not set any boundaries; simply asserts that the class implements and its object provides the interface.

Du coup, je ne vois pas l'intérêt, surtout par rapport aux protocols... Peut-être parce qu'il me manque la partie `adapter` ?
