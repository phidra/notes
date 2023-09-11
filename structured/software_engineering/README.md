Ici, j'ai l'intention de regrouper des notes qui concerne des sujets **transverses**, i.e. des problématiques qu'on retrouve quel que soit le langage utilisé :

- gestion des erreurs :
    - comment une fonction doit-elle signaler une erreur
    - différence entre une erreur recoverable (une exception qu'on a prévue, qu'on sait gérer) et une erreur irrecoverable (bug dans le code, e.g. un use-after-free, le mieux est de panic/abort)
- gestion des dépendances d'un projet
- logging
- testing (typologie des tests, notamment ; lien avec l'architecture)
- injection de dépendance (et plus généralement, probablement pas mal de sujets liés à l'architecture)
- etc.

En attendant d'y voir plus clair sur la façon d'organiser ce contenu, je mets tout dans le présent fichier.

* [Quotes](#quotes)
* [Gestion des erreurs](#gestion-des-erreurs)
   * [Recoverable vs. irrecoverable](#recoverable-vs-irrecoverable)
* [Architecture](#architecture)
   * [Interfaces](#interfaces)
* [Coding guidelines](#coding-guidelines)
   * [Calculs vs. actions](#calculs-vs-actions)
* [Conduite du changement](#conduite-du-changement)



# Quotes

Premature **micro**-optimization is the root of all evil. ([source](https://milen.me/writings/premature-optimization-universally-misunderstood/))

La simplicité est l'une des marques de la séniorité. ([source](https://eventuallycoding.com/2023/02/not-only-about-technique))

Pour créer de l'impact, il faut travailler sur les bons sujets, or la principale qualité des seniors, c'est justement de déterminer les bons problèmes à résoudre. ([source](https://eventuallycoding.com/2023/02/not-only-about-technique))

On the topic of comments, I like to say never send a comment to do a name's job, and never send a name to do a comment's job. ([source](https://dev.to/nadaelokaily/don-t-comment-your-code-5e9h)


# Gestion des erreurs

## Recoverable vs. irrecoverable

Au sujet des exceptions ([source](https://quuxplusone.github.io/blog/2022/12/14/my-lock-guard/#a-use-after-free-is-definitely-a)) :

> A use-after-free is definitely a logic bug in the program, not a runtime condition that can be “handled” by a catch handler.
> So rather than throw an exception, we’d really prefer to assert-fail as soon as possible, get a coredump, and go fix the bug.


# Architecture

## Interfaces

J'ai déjà fait [une POC](https://github.com/phidra/pocs/blob/fd9f9d9b5321433008b90bf0cc116817f33479c4/cpp/CATEGORY_archi/interface_vs_implementation/main.cpp) sur le principe d'avoir une factory qui renvoie un `IMachin*`, de sorte que le client n'ait pas connaissance (et donc ne dépende pas) de l'implémentation concrète `MyMachin`.

Par ailleurs, faire des interfaces vides peut avoir un intérêt, juste pour représenter un objet qui, par le simple fait d'être vivant, fait quelque chose d'utile en side-effect.

# Coding guidelines

## Calculs vs. actions

Séparer les fonctions en deux catégories générales, et ne pas mélanger les deux :

- les actions (avec I/O)
- les calculs (fonctions pures)

Une source parmi d'autres sur le sujet : https://rusty-ferris.pages.dev/blog/fp-actions-vs-calculations/

# Conduite du changement

Approche souvent intéressante pour la conduite des refactos = garder le truc pourri, mais l'isoler dans un périmètre restreint, de sorte que le monde extérieur se mette à utiliser un truc propre, même si en sous-main on continue d'utiliser le truc pourri :

- situation = tout le monde dépend de `TrucPourri`
- état souhaité = on se passe de `TrucPourri` (ou en tout cas, on est libre de changer l'implémentation)
- conduite du changement = exprimer les comportements de `TrucPourri` dans des interfaces, de sorte qu tout le monde ne dépende plus des classes concrètes mais d'une interface
- le point important = au lieu de tout changer d'un coup, on commence par se donner les moyen de faire ce qu'on veut sans que rien ne soit impacté

Si je généralise : quand quelque chose est pourri, au lieu de ne voir que deux possibilités :

1. tout garder pourri car trop coûteux de changer
2. tout remettre au propre...

... il faut essayer de trouver un état intermédiaire permettant d'introduire des changements dans le bon sens, sans tout refaire.

J'ajoute que l'état intermédiaire est "pérenne" (au moins autant que l'état pourri = c'est améliorable, mais on peut vivre longtemps avec si on ne veut pas investir plus de billes pour le corriger).
