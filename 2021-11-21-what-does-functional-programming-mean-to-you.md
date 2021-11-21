# What does "functional programming" mean to you?

- **url** = https://dev.to/awwsmm/what-does-functional-programming-mean-to-you-knh
- **type** = article
- **auteur** = [Andrew Willamson Watson aka awwsmm](https://awwsmm.com/), dev, plutôt Scala et JS d'après son profil github
- **date de publication** = 2021-11-18
- **source** = [la page dev.to du fameux Andrew](https://dev.to/awwsmm)
- **tags** = language>none ; topic>functional-programming ; level>beginner

**TL;DR** : en complément de [l'article précédent](2021-11-21-how-does-functional-programming-contribute-to-modern-languages.md), celui-ci rappelle les caractéristiques principales d'un langage fonctionnel du point de vue de l'auteur.

* [What does "functional programming" mean to you?](#what-does-functional-programming-mean-to-you)
   * [immutability and referential transparency](#immutability-and-referential-transparency)
   * [higher-order functions](#higher-order-functions)
   * [pure functions (side-effect-free functions)](#pure-functions-side-effect-free-functions)
   * [declarative programming and lambdas](#declarative-programming-and-lambdas)
   * [stream processing](#stream-processing)

À noter qu'en complément des points ci-dessous, il mentionne également la **lazy-evaluation** et la **récursion**, mais qu'il considère ces caractéristiques comme non-dimensionnantes.

## immutability and referential transparency

> **Immutability** just means you create new objects instead of mutating (changing) old ones.
>
> **Referential transparency** is a closely-related concept. In a nutshell, what it means is that -- wherever you have the value x in your code, you can replace it with whatever you initially declared x to be.\
> So if const x = 10, then x always equals 10. You can replace every instance of x with the literal 10 and your program should do the exact same thing.

## higher-order functions

> In many languages, functions and objects are separate things: you can pass objects as arguments to functions, and functions can be scoped as object methods.\
> But languages which encourage a functional style will allow you to pass functions as arguments to functions

## pure functions (side-effect-free functions)

NdM : pour ma part, c'est ceci que je considère comme la caractéristique principale des langages fonctionnels.

> A "pure" function is a function without "side effects". What does that mean? In a nutshell, it means that a function should behave similar to a mathematical function (think y = f(x)) -- it should take some value, x, and return or become some other value, y.
>
> Mathematical functions don't write to log files or print text to a terminal or change the value of other variables on the page -- they turn x into y, that's it.

Un point important en pratique :

> We can't eliminate side-effecting functions, but we should try to make it as clear as possible what the intent of our functions are. If you want to calculate a value, and log it to a file, separate those concerns into two separate routines, if possible.


## declarative programming and lambdas

> "Functional programming" also makes me think of a particular style of programming: iterating over data structures (like arrays, dictionaries, maps, sets, etc.) using declarative-style functions like map, filter, foreach, and so on.


## stream processing

> You put data in one end of a "processing pipeline", like water entering a pipe. It may be split into multiple streams, sent down different paths, siphoned off into some database, or dumped ultimately into some data lake, but the data moves from producers / sources (input), through flows / transformers / conductors, and into consumers / sinks.
