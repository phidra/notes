# From dependency injection to dependency rejection

- **url** = https://blog.ploeh.dk/2017/01/27/from-dependency-injection-to-dependency-rejection/
- **type** = post
- **auteur** = [Mark SEEMANN](https://blog.ploeh.dk/about/) aka [ploeh](https://github.com/ploeh), auteur prolifique : _Helps programmers make code easier to maintain._
- **date de publication** = 2017-01-27
- **source** = [son blog](https://blog.ploeh.dk/)
- **tags** = language>csharp ; language>fsharp ; topic>functional-programming ; topic>dependency-injection ; level>advanced


TL;DR = série de 4 articles + une introduction ; le résumé revient à chercher l'équivalent FP de l'injection de dépendance, qui se trouve être un sandwich impure+pur+impure, où on fait les IO au début et à la fin seulement.

Il y a aussi [une prez vidéo](https://youtu.be/cxs7oLGrxQ4) dont je m'épargne le visionnage.

* [From dependency injection to dependency rejection](#from-dependency-injection-to-dependency-rejection)
   * [Introduction](#introduction)
   * [Dependency injection is passing an argument](#dependency-injection-is-passing-an-argument)
   * [Partial application is dependency injection](#partial-application-is-dependency-injection)
   * [Dependency rejection](#dependency-rejection)
   * [Pure interactions](#pure-interactions)

## Introduction

https://blog.ploeh.dk/2017/01/27/from-dependency-injection-to-dependency-rejection/


> The problem typically solved by dependency injection in object-oriented programming is solved in a completely different way in functional programming.

^ Son résumé de la série

> This article series contains the following parts:
>
> - Dependency injection is passing an argument
> - Partial application is dependency injection
> - Dependency rejection
> - Pure interactions
> The first three articles revolve around a common example, which is one of my favourite scenarios: on-line restaurant reservation (...) The fourth article on pure interactions is a gateway to another article series on free monads.

^ Son sommaire

> I should point out that nowhere in this article series do I reject dependency injection as a set of object-oriented patterns. In object-oriented programming, dependency injection is a well-known and comprehensively described way to achieve decoupling and testability.

^ il n'a rien à redire à la dependency injection en POO, son propos porte sur la FP.

## Dependency injection is passing an argument

https://blog.ploeh.dk/2017/01/27/dependency-injection-is-passing-an-argument


Il reprend son exemple (que j'ai déjà vu dans [l'excellente prez "pits of success"](2023-07-13-function-architecture-pits-of-success.md)) de la réservation de places dans un restaurant, qui accepte une requête de réservation en entrée, et qui retourne un status-code HTTP indiquant si la réservation est accepté ou  refusée.

Il montre que la méthode POST a des dépendances (notamment une pour la logique métier permettant de faire la réservation), et que celles ci ne peuvent pas être injectées, car on ne peut pas modifier la signature de la méthode POST pour faire l'injection, vu que celle-ci est imposée par le framework.

En poussant un cran plus loin, il montre que la fonction business qui fait la réservation a le repository des available seats comme dépendance, et que celle ci n'est pas injectable non plus pour les mêmes raisons : la signature de la fonction est imposée par son interface (qu'on ne peut pas modifier pour ajouter la dépendance non plus, d'une part car le code client qui l'utilise ne l'utilise que via cette interface, d'autre part ce serait une mauvaise idée de toutes façons, car toutes les implémentations de cette interface n'auront pas forcément besoin de cette dépendance).

> Dependency injection is, in a sense, only a specific way for objects to take arguments. Often, however, objects have roles defined by the interfaces they implement. Such objects may need collaborators that are not available via the APIs defined by these interfaces, so you'll have to supply dependencies via members that belong to the concrete class in question.

^ résumé de ce premier article.

## Partial application is dependency injection

https://blog.ploeh.dk/2017/01/30/partial-application-is-dependency-injection

> The equivalent of dependency injection in F# is partial function application, but it isn't functional.

^ son résumé en tête d'article.

Ma compréhension :

- quand une fonction f utilise une (higher order) fonction g de type T (où T est la signature de g), qu'elle attend en argument
- pour que g se conforme au type exact T attendu par f, elle n'a pas le choix sur sa signature qui doit respecter T
- MAIS le`partial` d'une fonction h qui a une signature "plus étendue" (i.e. la même que celle de T, mais avec des arguments supplémentaires) peut convenir, car en en prenant le `partial`, on peut ramener sa signature à g
- du coup, si h fait le travail que f attend d'elle mais a besoin d'une dépendance supplémentaire, on peut injecter cette dépendance à h, puis en prendre le `partial` pour se conformer à T et pouvoir être passé en argument à f
- (Et c'est en quelque sorte l'équivalent en FP (mais pas réellement FP, comme on va le voir) au fait de construire un objet en lui passant ses dépendances à la construction, et dont la classe implémente une interface)

> Partial application is equivalent to dependency injection. It's just not a functional solution to dealing with dependencies.

^ même si ma compréhension est correcte, ce n'est pas la bonne façon de gérer les dépendances dans un langage fonctionnel.

>  In order to evaluate if my F# code is properly functional, I sometimes port it to Haskell. If I can get it to compile and run in Haskell, I take that as confirmation that my code is functional.

^ F# n'est pas purement fonctionnel, donc pour vérifier si son code est correct (= fonctionnel), son astuce est d'essayer de le traduire en Haskell : si ça passe, c'est que le code F# était fonctionnel, sinon, c'était qu'on avait juste écrit du code non-fonctionnel en F#.

Et effectivement, comme les dépendances de h qu'on masque avec `partial` sont impures, la fonction g résultante est impure, et on ne peut pas la passer à f, qui attend une fonction pure... On n'avait donc pas écrit du code fonctionnel.

> Partial application in F# can be used to achieve a result equivalent to dependency injection. It compiles and works as expected, but it's not functional. The reason it's not functional is that (most) dependencies are, by their very nature, impure.

^ résumé de l'article.

> Functional programming solves the problem of decoupling (side) effects from program logic another way.

^ petit teaser sur le suivant.

## Dependency rejection

https://blog.ploeh.dk/2017/02/02/dependency-rejection

> In functional programming, the notion of dependencies must be rejected. Instead, applications should be composed from pure and impure functions.

^ son résumé de l'article.

> dependency injection can't be functional, because it makes everything impure

^ Résumé de l'article précédent.

> When a unit of operation queries a dependency for data, the data returned from the dependency is indirect input. (...) Likewise, when a unit invokes a dependency, all arguments passed to that dependency constitute indirect output. 

^ les dépendances forment des inputs et outputs indirects des fonctions.

Je skimme la suite, mais l'idée est d'appliquer [une technique décrite dans un autre article](https://blog.ploeh.dk/2016/09/26/decoupling-decisions-from-effects) pour decoupler la prise de décision et les IO.

Il y a quand même une fonction chapeau qui fait le mélange des fonctions pures et impures. Au final, l'ordre des opérations est un peu modifié, mais la fonction principale est maintenant pure et accepte/renvoie les trucs en lien avec l'outside world.

> The code exhibits a common pattern for Haskell: First, gather data from impure sources. Second, pass pure data to pure functions. Third, take the pure output from the pure functions, and do something impure with it.

^ résumé de la technique.

Le truc intéressant, c'est que ça conduit naturellement à l'archi hexagonale :

> Dependencies are, by nature, impure. They're either non-deterministic, have side-effects, or both. Pure functions can't call impure functions (because that would make them impure as well), so pure functions can't have dependencies. Functional programming must reject the notion of dependencies.
>
> Obviously, software is only useful with impure behaviour, so instead of injecting dependencies, functional programs must be composed in impure contexts. Impure functions can call pure functions, so at the boundary, an application must gather impure data, and use it to call pure functions. This automatically leads to the ports and adapters architecture.

## Pure interactions

https://blog.ploeh.dk/2017/07/10/pure-interactions

> Long-running, non-deterministic interactions can be modelled in a pure, functional way.

^ son résumé de l'article.

> In a previous article, you can read why Dependency Injection and (strict) functional programming are mutually exclusive. Dependency Injection makes everything impure, and if nothing is pure, then it's hardly functional. In Dependency rejection, you can see how you can often separate impure and pure code into an impure/pure/impure sandwich.

^ son résumé des épisodes précédents.

La suite de l'article est trop orientée FP pour que j'y consacre beaucoup de mon temps pour le moment, mais l'idée est que le modèle "en sandwich" où on relègue les impuretés au début et à la fin marche bien p.ex. pour une appli web, mais pas pour tous les types de programmes :

> For such software, the impure/pure/impure sandwich architecture is no longer possible. Just think of a UI-based program, like an email client. You compose and send an email, receive a response, then compose a reply, and so on. Every send and receive is impure, as is all the user interface rendering. 

Ce quatrième article est en fait l'introduction d'une autre série d'articles sur le sujet, que je ne lis pas pour le moment.
