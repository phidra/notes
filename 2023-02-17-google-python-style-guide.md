# Google Python Style Guide


- **url** = https://google.github.io/styleguide/pyguide.html
- **type** = document tech
- **auteur** = N/A
- **date de publication** = N/A
- **source** = le [github google](https://github.com/google)
- **tags** = language>python ; topic>guidelines ; level>beginner


**TL;DR** = les trucs notables dans les recommandations de style de google pour coder en python


Proscrire `staticmethod`, et préférer à la place une top-level fonction dans le même module.

Éviter les power-features = les trucs compliqués et puissants.

Utiliser `from __future__ import xxx` est **encouragé** ! (ça simplifie la transition vers des versions plus modernes)

4 spaces plutôt que tabs

le point sur les docstrings des fonctions a quelques trucs suffisamment intéressants pour les reproduire :

> A docstring is mandatory for every function that has one or more of the following properties:
>
> -  being part of the public API
> -  nontrivial size
> -  non-obvious logic
>
> A docstring should give enough information to write a call to the function without reading the function’s code. The docstring should describe the function’s calling syntax and its semantics, but generally not its implementation details, unless those details are relevant to how the function is to be used. For example, a function that mutates one of its arguments as a side effect should note that in its docstring. Otherwise, subtle but important details of a function’s implementation that are not relevant to the caller are better expressed as comments alongside the code than within the function’s docstring.
>
> Certain aspects of a function should be documented in special sections, listed below. [...]
>
> - Args
> - Returns
> - Raises

De même, la docstring pour les classes a quelques points :

> If your class has public attributes, they should be documented here in an Attributes section

Accumulation dans une string :

> Avoid using the + and += operators to accumulate a string within a loop. In some conditions, accumulating a string with addition can lead to quadratic rather than linear running time. [...] Instead, add each substring to a list and ''.join the list after the loop terminates, or write each substring to an io.StringIO buffer.

Concernant les fichiers à close :

> Explicitly close files and sockets when done with them.

^ NDM (parce que j'étais passé à côté en première lecture) = ça veut dire ne pas les laisser se fermer tout seuls une fois l'objet détruit, mais les fermer explicitement "plus tôt".

> There are no guarantees as to when the runtime will actually invoke the `__del__` method. Different Python implementations use different memory management techniques, such as delayed garbage collection, which may increase the object’s lifetime arbitrarily and indefinitely. [...] Relying on finalizers to do automatic cleanup that has observable side effects has been rediscovered over and over again to lead to major problems, across many decades and multiple languages. The preferred way to manage files and similar resources is using the with statement:

J'apprends qu'il existe un tool (= `contextlib.closing`) pour les objets qui ont `close` mais qui ne supportent pas `with`.

> If your TODO is of the form “At a future date do something” make sure that you either include a very specific date (“Fix by November 2009”) or a very specific event (“Remove this code when all clients can handle XML responses.”) that future code maintainers will comprehend.

^ très important IMO = quand on laisse un TODO, il faut savoir quand il n'est plus d'actualité, pour éviter qu'il traîne ad vitam eternaem dans le code.

De façon curieuse (et que je n'approuve pas), ils autorisent ce one-liner :

```py
if foo: bar(foo)
```

Ceci est intéressant : à leurs yeux, un setter/getter véhicule une sémantique de complexité, et devrait donc être utilisée plutôt qu'un attribut public lorsque le set est complexe :

> Getter and setter functions (also called accessors and mutators) should be used when they provide a meaningful role or behavior for getting or setting a variable’s value.
>
> In particular, they should be used when getting or setting the variable is complex or the cost is significant [...]
>
> If, for example, a pair of getters/setters simply read and write an internal attribute, the internal attribute should be made public instead. By comparison, if setting a variable means some state is invalidated or rebuilt, it should be a setter function. The function invocation hints that a potentially non-trivial operation is occurring.

Nommage à éviter :

> names that needlessly include the type of the variable (for example: id_to_name_dict) \
> we discourage [prepending a double underscore] as it impacts readability and testability, and isn’t really private. Prefer a single underscore.

Toujours pouvoir importer un module :

> In Python, pydoc as well as unit tests require modules to be importable. If a file is meant to be used as an executable, its main functionality should be in a main() function, and your code should always check if __name__ == '__main__' before executing your main program, so that it is not executed when the module is imported.

Recommandation (plutôt que hard-rule) pour la longueur des fonctions :

> We recognize that long functions are sometimes appropriate, so no hard limit is placed on function length. If a function exceeds about 40 lines, think about whether it can be broken up without harming the structure of the program.

Virgule "inutile" sur le dernier paramètre :

> After annotating, many function signatures will become “one parameter per line”. To ensure the return type is also given its own line, a comma can be placed after the last parameter.

String qui peut être de plusieurs types :

> Use [AnyStr] for multiple annotations that can be bytes or str and must all be the same type.

Imports qui ne sont nécessaires que pour le type-checking :

> Imports that are needed only for type annotations can be placed within an if TYPE_CHECKING: block.

Dépendances circulaires :

> Circular dependencies that are caused by typing are code smells. Such code is a good candidate for refactoring.

Leur mot de la fin, écrit en MAJUSCULES :

**BE CONSISTENT.**


