# Type Systems Explained With Examples

- **url** = https://thevaluable.dev/type-system-explained/
- **type** = post
- **auteur** = Matthieu CNEUDE, dev français assez actif [site perso](https://matthieucneude.com/), [twitter](https://twitter.com/Cneude_Matthieu), [github](https://github.com/Phantas0s), [son blog](https://thevaluable.dev/page/about/)
- **date de publication** = 2020-09-25
- **source** = https://thevaluable.dev/
- **tags** = language>agnostic ; topic>type-system ; base-concept ; programming-languages



**TL;DR** un article très clair et très bien construit sur la notion de type :
- **définition d'un type** :
    - set de valeurs possibles
    - set d'opérations possibles
    - représentation en mémoire (i.e. comment interpréter un bitset)
- un type permet d'attribuer du sens (sémantique) à une variable, qui, sinon, pourrait représenter n'importe quoi : une variable de type "color" ne véhicule pas la même information qu'une variable de type "float"
- **type-cheking** = algorithme permettant de vérifier que les règles associées aux types sont bien respectées (e.g. qu'on n'applique à une variable QUE des opérations supportées par son type)
    - static type-checking (au compile-time) vs. dynamic type-checking (au runtime)
    - intéressant : l'un des intérêts du typage statique = permettre des optimisations qui se servent de l'information de type
- Déclaration de type : implicite ( **type inference** ) vs explicite.
- Modification de type : **type casting** (explicite : le changement de type est clairement indiqué dans le code) vs **type coercion** (implicite)
- **type-strength / type-safety**
    - ces notions sont floues, et pas définies formellement !
    - (à noter qu'il ne donne pas très clairement la différence qu'il fait entre les deux notions)
    - attention, il y a beaucoup de fausses idées trop simplistes sur les types (e.g. si un langage est statiquement typé, alors il est fortement typé).
    - il propose un point de vue que j'apprécie beaucoup et que je partage : bien préciser les notions dont on parle et qu'on souhaite voir dans un type-system (e.g. dire "je n'aime pas ce langage car il autorise les cast implicites" plutôt que "je n'aime pas ce langage car il est faiblement typé")
    - la type-safety n'est pas un booléen, mais un spectre nuancé.
    - en gros, un language type-safe s'assure que les règles des différents types sont bien respectées
    - certaines personnes considèrent que la notion de type-safety est associée à un programme plutôt qu'à un langage.
- Son propre résumé de fin d'articles est très bon :
    - Types and data types are two names for the same idea.
    - Types represent a set of value, a set of rules, and gives semantics (sense) to your data.
    - To enforce the rules of a type system, type checking often occurs at compile time (static type checking) or at runtime (dynamic type checking).
    - Changing a type can be implicit (type coercion), or explicit (type casting).
    - Declaring a type can be implicit (type inference), or explicit.
    - Types of a function signature can be implicit or explicit, too.
    - Type strength (“strong” or “weak”) has no formal definition, and people speak about it with many different meanings in mind.
    - A type system makes only sense in the context of a programming language.
    - You should learn how the type system of the language you’re using behave, to avoid bad surprises.
- Last but not least, l'une des illustrations de l'article m'a conduit à réfléchir sur (et écrire dans les présentes notes) la différence entre un type et un état.

## What’s a Type?

> we often hear the words “type systems”, “data type”, “type inference”, “static typing”, “weak typing”, “coercion”, and more.
> 
> First, let’s clarify the difference between the syntax and the semantics of a programming language.

En gros, syntaxe = comment le langage est construit mathématiquement, sémantique = ce qu'il fait, le sens qu'on lui attribue.

> When you assign 65 to the variable $integer, you give it a semantics it didn’t have when we declared it, on the second line. This semantics will let you know what you can do with the variable.

Si on ignore son type, une variable peut être n'importe quoi : elle n'a pas de sémantique. En lui attribuant un type, on lui donne du sens.

## Why Do We Need Types?

> Indeed, in memory, the two values of $character and $integer are exactly the same: 1000001

En gros, toute cette partie explique ça : les types permettent d'interpréter (de donner du sens à) une représentation binaire

> this interpretation is not always accurate for some types, like floating point numbers.

C'est important de le garder en mémoire?

> A type will determine as well how you store a value in memory.
>
> The memory storage is nicely abstracted by the type system for us, developers, not to think about these confusing 0 and 1.

## A Set of Rules

> A type system is as well a set of rule, more or less strict. You can’t do everything you want with some types.

Autre aspect important des types (en plus de l'aspect "set of values" et de l'aspect "représentation en mémoire")

> When you violate the rules of a type system, the outcome can range between these two extremes:
>   - The interpreter or compiler will silently try to fix the problem and continue.
>   - The interpreter or compiler will throw an error and stop.

## Built-in Types vs Our Own Abstractions

> Programming languages, more often than not, have a whole set of types you can use, out of the box. These types are called primitive types. For example: integer, boolean, float, and more.

Séquence émotion : je me souviens que ça a été une grande découverte quand j'ai réalisé que tout membre d'une classe cpp, aussi évolué soit son type, n'était qu'un assemblage plus ou moins complexe de types de base :-)

> composite types, a type containing multiple values, and possibly multiple primitive types. It’s what we call more commonly data structures.
> 
> For example, an array is a composite type:

Faut que je fasse attention : cet exemple n'est vraiment pas celui auquel j'aurais pensé en premier lieu quand je pense aux types composites.

> Abstract Data-Type (ADT)

Notion intéressante, que je maîtrise peu, à creuser.

> When you think about it, a type can be seen as a set of possible values, a category, or a group

In fine, un type, c'est juste ça (si on rajoute la représentation en mémoire, et les opérations qu'on peut faire avec, deux notions qui vont avec la catégorie/groupe)

## Type Checking

> a programming language need an algorithm to check if we respect them. This is called type checking.

Vue comme ça, la définition du type checking est assez simple : vérifier qu'on respecte les règles du type qu'on manipule.

> There are two important categories of type checking: static type checking and dynamic type checking.
>
> If your programming language doesn’t have any static type checking, it’s normally said that it’s a “dynamically typed language”.

Point de vue intéressant :  il y a forcément du type checking (même dans un langage à typage faible)

> For a static type system, type checking occurs before runtime, during compilation
>
> It implies that the compiler needs to know the exact data type for each data in use in your program, before even running it. This can be a problem when the data type can only be determined at runtime. That’s why, more often than not, a statically typed language has some dynamic type checking as well!

Hum, je ne savais pas. Quels langages sont concernés ? Peut-être des langages genre C# ou java ?

> Since most of the types don’t have to be checked at runtime, the performance of your program will be often better. The compiler can optimize your code, knowing that the rules of type system are respected.

Intéressant : l'un des intérêts du typage statique = permettre des optimisations qui se servent de l'information de type.

## Changing (NdM : and declaring) Types

**type casting** = explicit type change (i.e. le changement de type est clairement indiqué dans le code)

**type coercion** = implicit type change

Le fait de déclarer le type (notion corrélée, mais à ne pas confondre avec le fait de modifier le type ci-dessus) peut également être **implicite** (type inference) ou **explicite**.

Il y a également une petite section sur le type des fonction, qui se résume à ça + des exemples :

> You can declare types for function arguments in some languages, as well as for function outputs.

## Type Strength and Type Safety

> Some developers will categorize languages as “strong” or “weak”, based on different properties of a type system; if the type system coerce your values, for example. But it seems that nobody has the same definition of what should be “strong” or “weak”, which makes the communication difficult.

Point très important : la notion de *type strength* est floue !

> Many developers will think that it exists some mapping similar to the one below
> 
> [...] plein de fausses idées associées au typing, par exemple : "un langage statiquement typé est obligatoirement fortement typé" [...]
> 
> In short, the mapping above is wrong.

Point important, même si je n'y apprend rien de nouveau.

Il propose un point de vue que j'apprécie beaucoup et que je partage :

> However, speaking about “weak” or “strong” languages is confusing, because there is no clear definitions for these terms. Instead, we should specifically say what we like in a typing system, and why it’s better than another, using the vocabulary we saw above. It would add precision and clear meanings to the conversation.

## Type Safety

> When people speak about “weakly typed” and “strongly typed” languages, they often refer to the type safety of a language. Commonly speaking, it’s “how much” your compiler (or your interpreter) will check for type errors, and how often it will throw them to your sorry face.
>
> This “how much” is not really well defined.

Même combat que la *type-strength* : la notion de *type-safety* est floue.

Ceci dit, il ne précise pas très clairement dans son article la différence qu'il fait entre les deux notions.

> We reduced the possibilities of type coercion by declaring a strict_type, but we didn’t get rid of them totally. Type safety still improved: our confidence about knowing the types of our variables improved.

La type-safety n'est pas un booléen, mais un spectre nuancé.

> A more precise definition of a safely typed language is a language which prevent you to access memory you shouldn’t access, or to perform “impossible” operations

Ma compréhension : en gros, un language type-safe s'assure que les règles des différents types sont bien respectées (et au passage, ne pas accéder à de la mémoire à laquelle tu n'as pas accès va avec ces règles, normalement).

> Most high level languages nowadays respect this definition of type safety, so saying that language X is safer than language Y is, by this definition, often nonsense.

----

> To make matters even more complicated, some consider type safety as a property of a program, and not of a programming language. This is, I think, a better definition, because you can always create type errors in your code, even with strict type systems.

Je vois l'idée, mais c'est pas très concret, et ça irait mieux avec un exemple. Est ce que un program *type-safe* serait un programme où on s'assure de respecter les règles des types, alors qu'un programme non safe serait un programme où (par exemple) on utiliserait un équivalent de `reinterpret_cast` ?

# What Type System Should We Choose?

> Even if many studies tried to finally sort out if a static type system with explicit typing is better than a dynamic type system with implicit one, there’s no definitive, absolute truth.
>
> That said, the type system is only one (important) variable in the equation. Other things should be considered when choosing a language to use: the libraries available, the tooling, or the way of thinking the language try to push you into (the paradigm).
>
> Programming languages are tools to express your thinking, to bring life to solutions of specific problems. It’s a mean to an end, not the end itself. Choosing a language will largely depends on the business problems your software needs to solve.

Le type-system n'est pas l'alpha et l'oméga de la programmation, et ne devrait être qu'un critère parmi d'autres dans le choix du langage à utiliser pour un projet donné. D'une façon générale, j'aime bien sa façon de voir la programmation.

# Learn How The Type System Of Your Language Behave

> Bugs can appear because of the misconceptions we have about the type systems we use. Some of these systems are so inconsistent that you’ll likely to end up with logical contradictions and unforeseen values. This can be a real pain to debug.
>
> On the other hand, strict type systems can create some rigidity in your design, which will make the code more difficult to change when the messy real world, the context, evolve.

En résumé, il n'y a pas de silver-bullet : un type-system trop lâche c'est bad, un type system trop rigide c'est bad aussi.

Son conseil = s'assurer surtout de bien comprendre comment fonctionne le type-system de son propre langage !

> Looking at different type systems from different programming languages can be an interesting exercise

## Bonus = ma propre réflexion sur l'une des illustrations de l'article

L'illustration représente quelqu'un qui dit à un fantôme *"your type is 'dead'..."* et le fantôme répond : *"It's my state, not my type. I'm a ghost."*

Les notes qui suivent ne sont pas issues du post, elles sont juste le produit de mes réflexions sur cette illustration.

D'où vient la confusion ? Du fait que le *type* comme le *state* permettent tous deux de faire la même chose = catégoriser les objets dans un même groupe :

Reprenons son illustration, et supposons qu'on ait, dans un monde fantasy, différents objets :
- des personnes en vie (qui peuvent être des hommes et des femmes) :
    - **magiciens** = ils peuvent lancer des sorts
    - **archers** = ils peuvent tirer à l'arc
    - **rois/reines** = ils peuvent porter une couronne
- des personnes mortes (qui peuvent être des hommes et des femmes) :
    - **fantômes** = ils peuvent traverser les murs
    - **zombies** = ils peuvent manger de la chair humaine
    - **morts classiques** = ils ont une tombe et n'en sortent pas !

Si on regarde quelques exemples de comment on peut catégoriser (au sens large) ces objets :
- Catégorie = **les femmes**, qui regroupe :
    - magicienne
    - archère
    - reine
    - femme fantôme
    - femme zombie
    - femme morte "classique"
- Catégorie = **les vivants** qui regroue:
    - magicien
    - magicienne
    - archer
    - archère
    - roi
    - reine
- Catégorie = **les fantômes** :
    - homme fantôme
    - femme fantôme

La question (légitime) est : parmi ces catégories, qu'est-ce qui est de l'ordre du **state**, qu'est-ce qui est de l'ordre du **type** ?

Rappelons la définition d'un type donnée dans l'article : jeu de valeurs acceptées + opérations possibles sur un objet (+ interprétation en mémoire, mais c'est moins utile ici)

Alors :
- les objets de catégorie **les femmes** ne peuvent pas tous faire les mêmes choses (certaines femmes peuvent lancer des sorts, d'autres peuvent passer au travers des murs). **femme** n'est pas un type.
- les objets de catégorie **les vivants** ne peuvent pas tous faire les mêmes choses (certains vivants peuvent lancer des sorts, d'autres peuvent tirer à l'arc). **vivant** n'est pas un type.
- les objets de catégorie **les fantômes** peuvent tous faire les mêmes choses (passer au travers des murs) ou partagent les mêmes données (e.g. la zone qu'ils hantent). **fantôme** est un type.

Note : si on se mettait à attribuer des opérations spécifiques à **tous** les objets de la catégorie **vivant** (e.g. tous les vivants respirent) ou des données spécifiques (e.g. tous les vivants ont une fréquence cardiaque moyenne), alors être **vivant** pourrait correspondre à un type.
