# The Single Responsibility Principle Revisited

- **url** = https://thevaluable.dev/single-responsibility-principle-revisited/
- **type** = post
- **auteur** = Matthieu CNEUDE, dev français assez actif [site perso](https://matthieucneude.com/), [twitter](https://twitter.com/Cneude_Matthieu), [github](https://github.com/Phantas0s), [son blog](https://thevaluable.dev/page/about/)
- **date de publication** = 2021-09-27
- **source** = https://thevaluable.dev/

**TL;DR** :
- très intéressant point de vue sur le SRP, et sur le fait qu'il soit érigé en sacro-sainte formule "tu violes le SRP", sans qu'on ne comprenne/définisse clairement ce qu'il y a derrière
- un point important est que "violer le SRP" regroupe des notions complexes : décomposition des modules, cohesion, couplage
- son propre TL;DR à la fin est très bien :
> The Single Responsibility Principle is ambiguous and lack preciseness.
> 
> We should decompose the problems we’re trying to solve, even before coding.
> 
> Coupling and cohesion are not booleans. We can’t achieve total cohesion with no coupling, but we should try to maximize the first while minimizing the second.
> 
> We should organize our code depending on the decomposition of the problem, with well-defined interfaces respectful of the cohesion and coupling we want between our modules.
- il donne quelques conseils concrets (cf. plus bas) pour essayer de mieux gérer la décomposition de ses modules

## Position du problème

Position du problème = le SRP est ambigü.

Il donne son propre exemple, où, jeune codeur PHP, il a voulu appliquer à la lettre le SRP, en découpant à l'extrême son application, où chaque classe n'avait qu'une seule méthode.

> What’s a responsibility? [...]
> 
> What will change? I wasn’t sure, but every single method looked like they could have one reason to change. All these responsibilities in the same class! [...]
> 
> I fragmented my Shipment class, creating many more classes with one method each: everything was light, teared apart, shredded. [...]
> 
> A class should have only one reason to change. [...]
> 
> everybody had its own personal definition of a “responsibility”, or a “reason to change” [...]
> 
> This principle has many rules the definition itself doesn’t really state.

## Notion de module

Le coeur de l'article, c'est d'abord d'expliquer que le SRP est lié à la notion de module :

> The principles we’ll see now could be applied despite the paradigm. OOP, functional, procedural, it doesn’t matter. [...]
> 
> a module is a bounded, contiguous group of statements having a single name by which it can be referred to as a unit. [...]
> 
> If the definition looks obscure to you, a module is simply a named block with some code in it. It could be a function, a class, a namespace, a package, a micro-service, even a file.

## Module decomposition

Puis, à la notion de décomposition en modules, dont il explique l'intérêt en détail :

> The principle of decomposition, also called factoring, goes way beyond computing. It touches the essence itself of our work: problem solving. [...]
> 
> Now, you’ve created more problems, but they feel already more manageable

Il rappelle qu'il n'y a pas une unique bonne décomposition, mais que celle-ci dépend fortement du contexte :

> At every step, you’ll make some decisions, depending on your goals. These decisions will be very different if you want to create a breakout clone or a full RPG in 3D. The context will heavily drive your decisions. [...]
> 
> Therefore, like we decompose our problems, we need to decompose our code in different parts.

Il en arrive à la question de "comment bien découper" :

> This brings us to an obvious question: how do we do that?

Sa réponse : c'est lié aux notions de cohésion et coupling.

## Cohesion

> Cohesion: the degree of functional relatedness of processing elements within a single module.

Ah, tiens, un intéressant (et parlant !) parallèle avec la vie réelle :

> you do that all the time in real life (depending how messy you are). You put the knives with the knives, the fork with the forks[...]

Conséquences d'une bonne cohésion :

> The benefits?
> You don’t need to reason about your whole codebase while working on a functionality, you only need to reason in the boundaries of your module.
> When you have a bug, you know in what module to search.

## Coupling

> Coupling as an abstract concept - the degree of interdependence between modules - may be operationalized as the probability that in coding, debugging, or modifying one module, a programmer will have to take into account something about another modules.

Intéressante définition.

> one of the main problem in software development: consequence of changes, even simple, can spread like crazy in the entire codebase.

C'est l'un des problèmes qu'on essaye d'adresser

> coupling and cohesion are linked:
> - If you have a good cohesion in your module, everything belongs together.
> - If everything which belongs together are together, changing the module won’t affect other module.
> - If changing something in module A affects module B, it means that this something from module A should be in module B.

Le troisième point est exprimé de façon un peu trop simpliste à mon goût : dans beaucoup de cas, si on essaye de résoudre notre couplage en déplaçant ce _something_ de A à B comme suggéré, on se rend alors compte qu'un changement dans B va affecter A (on n'a alors pas supprimé le couplage). Peut-être parce qu'il fallait en fait sortir ce qui lie A à B dans un troisième module C, peut-être pour d'autres raisons...

## Conseils sur cohesion/coupling

> How do we achieve high cohesion and low coupling between our modules?”. [...]
>
> To achieve less coupling, each module should know the minimum about each other. [...]
>
> Now, how do we hide information between our modules? Using abstractions with well-defined interfaces. [...]
>
> For example, public methods and properties of a class are its interface; if everything is public, nothing is hidden. A micro-service has an interface too: its API.


> If you really did a good job, you could even use part of your codebase on a different one. This is difficult to achieve, and it shouldn’t be a priority; just a nice bonus.

Point de vue intéressant : la réutilisabilité est importante surtout en tant que critère, mais n'est pas une fin en soi.

## critique du SRP


> It Provides One Solution Without Stating the Problem

(je trouve qu')il a raison, et ce point est justement adressé par le présent article, qui rappelle le lien avec la décomposition en modules, la cohésion et le coupling.

> It’s the Wrong Abstraction

En gros, pour lui, "SRP violation" est une façon trop simpliste de décrire une mauvaise gestion de ces 3 concepts : décomposition, cohérence, coupling .

Il donne d'ailleurs plus loin un avis beaucoup plus nuancé que "SRP violation" :

> Coupling and cohesion are scales, not booleans. That’s why we speak about “high” and “low” cohesion, “high” and “low” coupling.

Et il donne une autre guideline que j'aime beaucoup, et qui fait le lien avec son exemple en php du début de l'article :

> If you need to retain something from this article, it’s the following: the goal is not to be 100% decoupled and 100% cohesive, it’s doing our best to avoid unnecessary coupling and making our modules as cohesive as possible.

Par ailleurs :

> Your decisions will depend on the context. By order of importance:
> The business model and the problem domain of your company.
> The outcomes the software should have.
> The technology used, and the reason you use them.

Je trouve intéressante son envie, déjà exprimée dans d'autres articles de revenir à "à quoi sert le code qu'on écrit".

Il suggère des questions à se poser pour aider à choisir sa décomposition en modules :

> Asking these questions might be a good starting point:
> Here’s the problem we’re trying to solve with this software. How should we decompose this problem? What are the sub-problems?
> What kind of relationships these problems have? Is this problem depending on the solution of this one?
> After decomposing the problem, what design decisions can we make to support this decomposition? What interface should have our modules?
> What new challenges do we face while designing? While coding? Is it possible to represent the solution of the problems using an abstraction in a cohesive way?
> What could make this module more cohesive? How to call it, to show this cohesion?
> I see some coupling between two modules. Is this coupling necessary? Why?
> This part is difficult to understand. Should we isolate it in its own class or function?

Je note que ça dépend fortement de la compréhension qu'on a de notre problème -> je fais le lien avec un sentiment que j'ai déjà = quand tu défriches un problème, c'est pas forcément très pertinent de réfléchir d'entrée de jeu à une super archi top-moumoute : mieux vaut peut-être faire une première implémentation qui aidera à mieux comprendre le problème, avant de trouver la bonne archi.

C'est d'ailleurs **exactement** ce qu'il dit ici, et on est bien d'accord sur ce point :

> Try to make sense of it, code your solutions, and come back to it later when you’ll have more knowledge about the context, to improve your design.

(mais le souci, c'est qu'à cause des contraintes de temps et de pression, on ne prend pas toujours le temps de revenir modifier cette première implémentation maintenant qu'on a mieux compris le problème)

> This is what this article is about: we need to be precise when we state the problem, or we won’t understand how to solve it.

Ce qu'il dit est corrélé aux points précédents : il faut correctement **formuler** un problème avant de pouvoir le résoudre.

Dit autrement, se poser la question "quel est le problème que j'essaye de résoudre" n'est jamais une mauvaise idée :-)

> I’m thinking in terms of relationships between components, in the business and therefore in my code, and I make incremental and, if possible, deferred decisions till I understand more the problems and their context.

J'aime vraiment beaucoup sa façon de revenir à du concret, lorsqu'il raisonne sur son archi.
