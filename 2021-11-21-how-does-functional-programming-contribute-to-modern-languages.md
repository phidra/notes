# How Does Functional Programming Contribute to Modern Languages ?

- **url** = https://dev.to/typeable/how-does-functional-programming-contribute-to-modern-languages-4gap
- **type** = article
- **auteur** = [Vladislav LAGUNOV](https://lagunoff.github.io/), dev expérimenté, notamment en FP
- **date de publication** = 2021-11-15
- **source** = [la page dev.to de Typeable](https://dev.to/typeable), _Functional programming and digital transformation consultants_
- **tags** = language>none ; topic>functional-programming ; level>beginner

**TL;DR** : intéressant article sur ce que la programmation fonctionnelle a apporté aux autres langages de programmation. En d'autres termes, l'article est un listing des features caractéristiques des langages fonctionnels. Ces notes sont assez brutes.

## First-class functions 

The distinctive feature of FP style, in general, is the wide use of functions which become one of the main development tools.

- **Higher-order** function is the function that either takes another function as an argument or returns some function as the result.
- **First-class functions** are the ones which you can manipulate in the same way as any other values: pass as arguments, return as results, assign to variables and structure fields.
- **Lambda function** is the anonymous function.
- **Closure** is the function that can capture some variables from the context it was declared in, without letting the garbage collector erase the data which can be used in this function for as long as the application has the reference to the function itself.

(NdM : même en l'absence de garbage-collector, les closures permettent un accès à des données hors-scope)

## List comprehensions

> List comprehension allows writing down concisely the processing or generation of lists using existing ones.

## Algebraic data types 

Aussi connu comme **ADT** : sum-types (= unions), product-types (= classes, tuple, ou tout ce qui permet d'agréger plusieurs valeurs).

NdM sur la terminologie :

- _sum_ parce que les valeurs possible de `Union[A, B]` est la somme des valeurs possibles de `A` et des valeurs possibles de `B`
- _product_ parce que les valeurs possible d'une classe ayant deux attributs, de types A et B, sont toutes les combinaisons possibles des valeurs possibles pour `A` et des valeurs possibles pour `B`, et il y en a `|A| x |B|`

## Pattern Matching

> Pattern Matching resembles the switch-case operator you all know from imperative languages. However, its main advantage is that the compiler checks the access to alternative fields statically by using the information about the expression type, while the switch-case doesn’t prevent errors related to incorrect access to the fields, missing cases or redundant checks.

## Lazy evaluations 

> In most programming languages evaluation is performed when the value is assigned to the variable; all arguments are evaluated before the function call (strict evaluations).
>
> The alternative approach is “lazy” implying that the evaluation is postponed until the value is actually used.

## Continuations

> Continuation is the “remaining evaluation”, i.e. the description of what is to be done with the result of expression for each subexpression in the program.

C'est le concept qui m'est le moins clair dans le tas... Peut-être à cause de ceci :

> CPS is rarely found immediately in the software source code. Compilers are one of the main areas of its application – as the intermediate format before generating the machine code.

##  Futures and promises

> Futures, Promises, or Deferred, are a construct containing the evaluation of asynchronous value.

## Monadic interfaces

> To put it simply, “monad” is just an interface with two methods that allow combining evaluations into a chain in the same way it’s done in the promise chain example.

## Async

> [...] you can notice that despite all advantages of the promises, the call chain looks no better than callbacks.\
> The async-await syntactic structure allows taking it a step further and improving the code with the promise chain, turning it into something much like the code with blocking calls.
