# Explaining Why Interfaces Are Great

- **url** = https://glyph.twistedmatrix.com/2009/02/explaining-why-interfaces-are-great.html
- **type** = post
- **auteur** = [Glyph LEFKOWITZ](https://glyph.twistedmatrix.com/pages/about.html), le créateur de Twisted
- **date de publication** = 2009-02-16, mais l'article a été mis à jour en 2018
- **source** = [le blog de Glyph](https://glyph.twistedmatrix.com/)
- **tags** = language>python ; topic>python>interfaces ; topic>python>abc ; topic>python>zope ; level>intermediate

**TL;DR** : une bonne explication des avantages des interfaces zope par rapport aux ABC (meilleure que celle [de l'article précédent](./2021-10-15-tutswiki-abstract-classes-and-interfaces-in-python.md)). Faut reconnaître que les zope.interfaces ont l'air chouettes. À froid, ce qui me bloque, c'est le manque de vérification (tu peux register ce que tu veux, sans vérification). Mais c'est à tester avec le plugin mypy, justement.

À la base, article assez ancien puisque le contexte d'écriture initiale de l'article est l'introduction des "nouvelles" ABC, en 2009 ! En pratique, l'article a été remis à jour en 2018, et il reste très intéressant..

> Let's say we have an idea of something called a "vehicle".  We can represent it as one of two things: a real base class (Vehicle), an ABC (AVehicle) or an Interface (IVehicle).

L'idée est donc de comparer ABC + classe-fille d'un côté, avec zope.interface + classe-implementer de l'autre.

En python (pour le cas qui m'intéresse, en tout cas), c'est surtout important au niveau du type-checking.

> There are a set of operations that interfaces and base-classes share.

- `isinstance(something, Vehicle)`   a pour équivalent :   `isinstance(something, Vehicle)`
- `issubclass(Something, Vehicle)`   a pour équivalent :   `IVehicle.implementedBy(Something)`

> However, there are some questions you can't quite cleanly ask of the ABC system.

> In other words, types have two jobs: they might be ABCs, or they might be types, or they might be both, and it's impossible to separate those responsibilities.

En gros, quand on définit une ABC, c'est apparemment compliqué de voir ce qui relève de l'interface, et ce qui relève soit de la tuyauterie python (e.g. attribut `__doc__`), soit de l'implémentation partagée (qui sera commune à toutes les filles)

C'est visible par un `dir(AVehicle)` qui renvoie plein de choses non liées à l'interface, alors que `list(IVehicle)` renvoie la liste des attributs d'un `Vehicle`. Et il y a des moyens programmatiques `IVehicle.namesAndDescriptions()`  et  `Method.getSignatureInfo()` spécifiquement dédiés à ça.

En résumé, les `zope.interfaces` définissent ce qu'est un `Vehicle` d'une façon suffisamment formelle pour que ça puisse être utilisé programmatiquement.

> zope.interface.verifyClass and zope.interface.verifyObject can tell you, both for error-reporting and unit-testing purposes, whether an object looks like a proper vehicle or not, without actually trying to drive it around.

Du coup, cette utilisabilité programmatique est utilisée pour la vérification de type.

> If you want to change the answer to isinstance(), you need to register a type by using ABCMeta.register or overriding __instancecheck__ on a real subclass of type. \
> If you want to change the answer to providedBy, for example for a unit test, all you need is an object with a providedBy method.

Le mécanisme lui-même (qui vérifie si une classe est une fille d'une interface) est plus facilement customizable avec zope qu'avec ABC.

> With an interface, however, there is a specific facility for this: you put a moduleProvides(IVehicle) declaration at the top of your module.

Ça a l'air plus facile de dire qu'un module implémente une interface avec zope qu'avec ABC (en effet, si on veut utiliser les ABC pour indiquer qu'un module implémente une interface, il faut faire hériter le module d'une ABC ce qui est bizarre...)

> In zope.interface, there is a very clear conceptual break between implements and provides.  A module may provide an interface — i.e. be an object which satisfies that interface at runtime — without there being any object that implements that interface — i.e. is a type whose instances automatically provide it.

C'est ce qui permet de dire qu'un module implémente une interface avec zope (alors qu'avec ABC, pour dire que quelque chose implémente une interface, il faut automatiquement le faire dériver d'une classe particulière = son ABC).

MAIS tout ceci repose sur une déclaration statique (le fait de register le module comme providant l'interface). De plus, je ne sais pas si ça suffit à type-checker le module avec mypy... Note : ça reste possible avec ABC (distinction entre `issubclass` et `isinstance`), mais d'après l'article c'est plus vague (?)

> you can adapt from one interface to another

Ah, l'exemple donné ici est déjà plus parlant que dans [l'article précédent](./2021-10-15-tutswiki-abstract-classes-and-interfaces-in-python.md) :

- on a une interface `IHTMLRenderer` représentant les objets représentables en HTML
- on a une interface `IVehicle` représentant les véhicules
- si on veut représenter un véhicule en HTML, on peut convertir "quelque chose qui implémente l'interface véhicule" en "quelque chose qui implémente l'interface HTML" avec :  `my_renderer = IHTMLRenderer(my_vehicle)`

> If there are no rules for adapting an object like the one you have passed to an IHTMLRenderer, then you will get an exception

En gros, ça reste à nous de définir comment on peut adapter un véhicule en HTMLRenderer (ce qui est logique ^^)

> Your code for rendering now doesn't need any special-case knowledge about particular types.  It is written to an interface, and it's very easy to figure out which one; it says "IHTMLRenderer" right there.

> In a super-dynamic language like Python, you don't need a system for explicit abstract interfaces.

Indeed, à mes yeux, c'est surtout utile pour le type-checking statique

> formal interface definitions serve many purposes.\
> - They can function as documentation, \
> - as a touchstone for code which wants to clearly report programming errors ("warning:  MyWidget claims to implement IWidget, but doesn't implement a 'doWidgetStuff' method"),\
> - and a mechanism for indirection when you know what contract your code wants but you don't know what implementation will necessarily satisfy it (adaptation).
