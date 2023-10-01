# Crash early and crash often for more reliable software

- **url** = https://medium.com/@mattklein123/crash-early-and-crash-often-for-more-reliable-software-597738dd21c5
- **type** = post
- **auteur** = [Matt KLEIN](https://mattklein123.dev/about/) = software engineer avec 20 ans d'expérience
- **date de publication** = 2019-04-07
- **source** = [son medium](https://medium.com/@mattklein123)
- **tags** = language>cpp ; topic>error-handling ; level>intermediate


**TL;DR** : plutôt que d'ajouter du code complexe avec du branching à rallong pour traiter des erreurs qui ne sont pas censées arriver, mieux vaut asserter, faire crasher, et débugger le crash : le branching pour gérer les erreurs doit être réservé aux erreurs qui peuvent légitimement arriver lors de la vie du programme.

> The only error checking a program needs are for errors that can actually happen during normal control flow.

^ p.ex. supprimer le code qui vérifie un booléen, si ce dernier n'est jamais nul dans le codeflow classique.

Et si on s'est gouré et que le booléen est nul ? Laissons le programme crasher ! Ce sera facile à investiguer et debugger.

NDM 1 = j'aime bien expliciter la pré-condition (en commentaire faute de mieux)

NDM 2 = un hic, c'est que parfois, le programme ne crashe malheureusement pas si la pré-condition n'est pas remplie : à la place, il se mets à faire des trucs dangereux...

L'un des intérêts , c'est de réduire la charge cognitive apportée par tout le code de gestion d'erreur (inutile).

> A corollary to “crash early and crash often” is “assert early and assert often.” Asserts are a tremendously powerful mechanism to verify code invariants (things that should always be true). I favor two types of asserts:
>
> - Debug asserts: These asserts are compiled out in release builds and well and truly should never happen.
> - Release asserts: These asserts are not compiled out of release builds. They are used to check situations that may theoretically happen, but if they do, it would be preferable to crash and restart instead of trying to handle the error.

^ Réponse à une de mes interrogations plus haut : forçons le crash !

NDM : selon moi, les assert encombrent tout de même un peu le code... C'est contrebalancé car 1. ils sont plus succints 2. ils sont plus faciles à lire et expriment clairement l'attendu et 3. on peut les regrouper.

> I view assertions as an incredibly powerful tool for reducing code complexity for the following reasons:
>
> Assertions act as documentation. The reader clearly understands the state the program is expected to be in when the assertion runs.
> Assertions by definition limit extraneous branches and error handling. Why handle a state which should never happen

^ Deux gros avantages de l'assertion : autodocumenter le code + simplifier le code.

> What happens if the programmer is incorrect and an assertion is not valid in all cases? Let the program crash!

^ ici aussi, on force le crash : si on a asserté à tort, ce sera facile à debugger.

> Tip: If the thought of adding the extra test coverage, logging, and stats to handle an error and continue seems ridiculous because “this should never happen”, it’s a very good indication that the appropriate behavior is to terminate the process and not handle the error.

^ tip pour savoir si on a besoin de traiter le cas d'erreur ou si on peut se contenter d'asserter : si on ne se sent pas de unittester une branche de code, c'est peut-être qu'un assert aurait suffi plutôt que de coder la branche qui gère le cas d'erreur !

> Ownership semantics used to prevent crashes can lead to complexity and bugs

^ derrière, il pousse un point de vue intéressant : certains devs utilisent parfois des `shared_ptr` pour éviter des use-after-free (et donc souvent des crashs) qu'ils auraient eus avec un `unique_ptr` ; c'est une mauvaise idée : le crash est alors signe d'un vrai bug logique, qu'il faut traiter plutôt que contourner.

> Only use shared memory ownership when the program logic actually calls for it.

^ partager l'ownership d'une variable doit être utilisé quand c'est la bonne modélisation adaptée au problème, et non pour contourner des bugs...

> letting the program crash and fixing the uncovered invariant violation is far preferable to introducing needless ownership and code complexity in an attempt to avoid crashes of this type

^ autre façon de le dire.

> Very often, invariant violations that cause a fatal crash are substantially easier to debug and fix than complex code that attempted to prevent the crashes in the first place.

^ le point le plus important de l'article = les violations d'invariants détectés avec des asserts sont plus faciles à débugger que de maintenir du code pour empêcher le crash.

>  I recommend using the following three techniques for limiting error handling and code complexity:
>
> - Limit error handling to only errors that can actually happen during normal control flow. Crash otherwise.
> - Liberally use assertions to document invariant state and crash if violated.
> - Use single owner data semantics if at all possible to limit code complexity, and if doing this using C/C++, let the program crash if an ownership invariant is violated.

^ la mise en oeuvre de ce point.
