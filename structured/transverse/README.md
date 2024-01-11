Ici, j'ai l'intention de regrouper des notes qui concerne des sujets **transverses**, i.e. des problématiques qu'on retrouve quel que soit le langage utilisé :

- gestion des erreurs :
    - comment une fonction doit-elle signaler une erreur
    - différence entre une erreur recoverable (une exception qu'on a prévue, qu'on sait gérer) et une erreur irrecoverable (bug dans le code, e.g. un use-after-free, le mieux est de panic/abort)
- gestion des dépendances d'un projet
- logging
- testing (typologie des tests, notamment ; lien avec l'architecture)
- injection de dépendance (et plus généralement, probablement pas mal de sujets liés à l'architecture)
- etc.

En attendant d'y voir plus clair sur la façon d'organiser ce contenu, je mets tout dans le présent fichier. Possiblement, je le splitterai en plusieurs sections.


* [Quotes](#quotes)
* [Gestion des erreurs](#gestion-des-erreurs)
   * [Recoverable vs. irrecoverable](#recoverable-vs-irrecoverable)
   * [Erreurs "normales"](#erreurs-normales)
* [Architecture](#architecture)
   * [Interfaces](#interfaces)
   * [Dependency Injection](#dependency-injection)
   * [Application-logic classes vs. Configuration classes](#application-logic-classes-vs-configuration-classes)
* [Coding guidelines](#coding-guidelines)
   * [Calculs vs. actions](#calculs-vs-actions)
* [Conduite du changement](#conduite-du-changement)
* [Testing](#testing)
   * [Faut-il tester des membres privés ?](#faut-il-tester-des-membres-privés-)
      * [Points de vue nuancés](#points-de-vue-nuancés)
      * [Points de vue en défaveur du testing de trucs privés](#points-de-vue-en-défaveur-du-testing-de-trucs-privés)
      * [Arguments pour tester des trucs privés](#arguments-pour-tester-des-trucs-privés)
* [Deployment](#deployment)
   * [k8s liveness readiness startup probes](#k8s-liveness-readiness-startup-probes)




# Quotes

Premature **micro**-optimization is the root of all evil. ([source](https://milen.me/writings/premature-optimization-universally-misunderstood/))

La simplicité est l'une des marques de la séniorité. ([source](https://eventuallycoding.com/2023/02/not-only-about-technique))

Pour créer de l'impact, il faut travailler sur les bons sujets, or la principale qualité des seniors, c'est justement de déterminer les bons problèmes à résoudre. ([source](https://eventuallycoding.com/2023/02/not-only-about-technique))

On the topic of comments, I like to say never send a comment to do a name's job, and never send a name to do a comment's job. ([source](https://dev.to/nadaelokaily/don-t-comment-your-code-5e9h)

Some people, when confronted with a problem, think, “I know, I’ll use threads,” and then two they hav erpoblesms. (Net BATCHELDER ; [source](https://bitbashing.io/async-rust.html#fnref:2))


# Gestion des erreurs

## Recoverable vs. irrecoverable

Au sujet des exceptions ([source](https://quuxplusone.github.io/blog/2022/12/14/my-lock-guard/#a-use-after-free-is-definitely-a)) :

> A use-after-free is definitely a logic bug in the program, not a runtime condition that can be “handled” by a catch handler.
> So rather than throw an exception, we’d really prefer to assert-fail as soon as possible, get a coredump, and go fix the bug.

## Erreurs "normales"

Notion proche de ce qui précède ([source](https://medium.com/@mattklein123/crash-early-and-crash-often-for-more-reliable-software-597738dd21c5)) :

> The only error checking a program needs are for errors that can actually happen during normal control flow.


# Architecture

## Interfaces

J'ai déjà fait [une POC](https://github.com/phidra/pocs/blob/fd9f9d9b5321433008b90bf0cc116817f33479c4/cpp/CATEGORY_archi/interface_vs_implementation/main.cpp) sur le principe d'avoir une factory qui renvoie un `IMachin*`, de sorte que le client n'ait pas connaissance (et donc ne dépende pas) de l'implémentation concrète `MyMachin`.

Par ailleurs, faire des interfaces vides peut avoir un intérêt, juste pour représenter un objet qui, par le simple fait d'être vivant, fait quelque chose d'utile en side-effect.

## Dependency Injection

> Traditionally, each object is responsible for obtaining its own references to the objects it collaborates with (its dependencies).
>
> When applying Dependency Injection, the objects are given their dependencies at creation time by some external entity that coordinates each object in the system.

([source](https://youtu.be/92ZJcxJgmmE?t=488))

^ les dépendances sont injectées par un objet ordonnanceur, il y a deux points importants derrière cette notion :

- les dépendances sont injectées (en pratique, sous forme d'objet implémentant une interface plutôt que d'un objet concret)
- un objet ordonnanceur crée les objets concrets

## Application-logic classes vs. Configuration classes

> One way to make code more testable is to use Dependency Injection. This means that an object should never instantiate its collaborator by calling the new operator. It should be passed its collaborators instead. When we work this way we separate classes in two kinds.
> - Application logic classes have their collaborators passed into them in the constructor.
> - Configuration classes build a network of objects, setting up their collaborators.
> Application logic classes contain a bunch of logic, but no calls to the new operator. Configuration classes contain a bunch of calls to the new operator, but no application logic.

([source](http://matteo.vaccari.name/blog/archives/154))

^ séparer ses classes en **Application-logic** d'un côté, et **Configuration** de l'autre.

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

# Testing

## Faut-il tester des membres privés ?


Mon avis à moi = j'aime le point de vue de Jesse DUFFIELD ci-dessous.

Sans aller jusqu'à parler de consensus, beaucoup de gens pensent que tester des fonctions privées est plutôt un code-smell. Pour ma part, je me retrouve plutôt dans les quelques avis nuancés que je croise (lorsqu'ils sont bien argumentés).

Du coup, je les mets en premier.

### Points de vue nuancés

https://jesseduffield.com/Testing-Private-Methods/

Très très bon article sur le sujet, un peu long, mais très bien écrit, avec un point de vue agréablement nuancé qui me parle énormément, et plein de trucs utiles. Je l'ai annoté ([lien](../../2023-10-21-no-consensus-on-testing-private-methods.md)), je duplique ici mon takeway = il faut rester pragmatique :

- essayer de designer ses APIs publiques de façon frugale, en y mettant le strict minimum nécessaire pour effectuer le métier
- si on se retrouve à vouloir tester une méthode privée, c'est peut-être le un avertissement d'un mauvais design : ne faudrait-il pas sortir la méthode privée dans une classe à part ?
- si la réponse _indépendamment de notre problématique de testing_ est oui, on corrige le design, ce qui règle en passant notre problème de testing
- si la réponse est non, alors on s'autorise à tester la méthode privée

----

https://stackoverflow.com/questions/8997029/i-want-to-test-a-private-method-is-there-something-wrong-with-my-design/8997128#8997128

^ très bon point de vue, bien rédigé ; morceaux choisis :

> Your class's API is flexible enough, but underneath the public methods it has some pretty complicated private methods going on underneath.
>
> [this] is the case where it's OK to test private methods. There are various tricks for that. For production code, these are better than exposing your method to the API user (making it public) just so you can test it. Or splitting related functionality into 2 classes just so you can test it.

^ les méthodes privées peuvent valoir la peine d'être testées ; quand ça arrive, c'est une mauvaise idée que de rendre publique la méthode juste pour la tester, car ça pollue l'API publique ; mieux vaut trouver une façon de les tester sans polluer l'API publique de la classe.


> Instead of thinking in terms of "code smells", which is imprecise and subjective, you can think in terms of information hiding.

^ préférer réfléchir en terme de information hiding plutôt que code-smell.

> Design decisions that are likely to change should not be exposed in your public API. Preferably, design decisions that are likely to change should not be exposed to your unit tests either -- that's why people recommend against testing private methods.

^ les design decisions susceptibles de changer ne devraient pas être exposées dans l'API publique (car ça complique alors leurs futures modifications).

C'est pour les mêmes raisons qu'on déconseille de tester les méthodes privées.

> But if you really think it's important to unit-test your private method, and if you can't do it adequately via the public methods, then don't sacrifice the correctness of your code! Test the private method.
>
> Worst case is your test code is messier, and you have to rewrite the tests when the private method changes.

^ son conseil = rester pragmatique plutôt que dogmatique : si on considère qu'il faut tester une méthode privée, le faire !

----

https://github.com/google/googletest/blob/main/docs/advanced.md#testing-private-code

> If you change your software's internal implementation, your tests should not break as long as the change is not observable by users. Therefore, per the black-box testing principle, most of the time you should test your code through its public interfaces.
>
> If you still find yourself needing to test internal implementation code, consider if there's a better design. The desire to test internal implementation is often a sign that the class is doing too much. Consider extracting an implementation class, and testing it. Then use that implementation class in the original class.
>
> If you absolutely have to test non-public interface code though, you can

^ googletest déconseille (sans proscrire) de tester des trucs privés, mais fournit de quoi le faire en cas de besoin.

### Points de vue en défaveur du testing de trucs privés


https://stackoverflow.com/questions/3676664/unit-testing-of-private-methods-in-c/3676724#3676724

> I don't think unit test cases would be required for private methods.
>
> If a method is private it can used only within that class. If you have tested all the public methods using this private method then there is no need to test this separately since it was used only in those many ways.

----

https://www.codeproject.com/Tips/5249547/How-to-Unit-Test-a-Private-Function-in-Cplusplus

> One way of thinking here is that only the public interfaces need to be tested, since the private ones are only used by the public functions of the class, so if someone is using our class from outside, she/he only uses the public interfaces and doesn’t have knowledge about the private ones.

----

https://stackoverflow.com/questions/8997029/i-want-to-test-a-private-method-is-there-something-wrong-with-my-design/8997062#8997062

> If it's private, it can't be considered part of your application's API, so testing it is indeed a code smell - when the test breaks, is that OK or not?
>
> Unit tests are supposed to be functionality oriented, not code oriented. You test units of functionality, not units of code.

^ deux choses intéressantes ici :

- on teste des fonctionnalités plutôt que du code
- quand le test d'une fonctionnalité privé pète, est-ce que c'est 1. problématique parce qu'on a introduit une régression, ou 2. pas grave car le comportement public n'a pas été modifié ?

### Arguments pour tester des trucs privés

https://www.codeproject.com/Tips/5249547/How-to-Unit-Test-a-Private-Function-in-Cplusplus

> On the other hand, one important KPI is the code coverage and to reach 100% code coverage, the easiest way is to test the private functions separately. In some cases, the private functions are really complex and it is difficult to test all the corner cases through the public functions.

----

Autre argument : certaines fonctions "utilitaires" sont beaucoup, beaucoup, beaucoup plus simples à tester unitairement que via leur utilisation dans une fonction publique.

- par exemple, une fonction privée de conversion de format, ou de gestion du datetime (e.g. prise en compte de l'heure d'été), et une fonction publique qui reçoit une requête et fabrique une réponse dont l'un des champs est le datetime ou le format...
- si on voulait tester tous les edge-cases de conversion par la fonction publique, il faudrait systématiquement wrapper ce qu'on veut tester dans un cycle requête->réponse, alors qu'on ne veut que tester la conversion...
- (certes, on pourrait argumenter qu'une telle fonction gagnerait à être publique plutôt que privée, mais ça reste une bonne illustration)


# Deployment

## k8s liveness readiness startup probes

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

En gros, :

- la **liveness probe** sert à restart un container : si elle faile, l'application est considérée comme morte, k8s tue le pod et le restarte (on peut souhaiter ce comportement même si le process tourne encore ; p.ex. si notre app est bloquée dans un deadlock)
- la **readiness probe** sert à autoriser le routage de traffic vers le container :
    > The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

Ces deux probes peuvent éventuellement être servies par le même endpoint HTTP :

> A common pattern for liveness probes is to use the same low-cost HTTP endpoint as for readiness probes, but with a higher failureThreshold. This ensures that the pod is observed as not-ready for some period of time before it is hard killed.

La **startup probe** est optionnelle, elle sert dans les cas où l'application met du temps à démarrer (e.g. elle a beaucoup de données à charger). Dans ce cas, la liveness probe ou la readiness probe peuvent ne pas être en mesure de répondre suffisamment vite avant que le container ne se fasse killer.

Dit autrement : le fonctionnement par liveness/readiness peut n'être prêt à faire son boulot que de longues minutes après le démarrage de l'application. La startup probe est là pour ça : pour désactiver le fonctionnement de liveness/readiness jusqu'à ce que le startup soit fini :

> The kubelet uses startup probes to know when a container application has started. If such a probe is configured, liveness and readiness probes do not start until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.




