Ici sont regroupées des notes qui concerne des sujets **transverses**, i.e. des problématiques qu'on retrouve quel que soit le langage utilisé.

En attendant d'y voir plus clair sur la façon d'organiser ce contenu, je mets tout dans le présent fichier. Possiblement, je le splitterai en plusieurs sections.

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
   * [Property-based testing](#property-based-testing)
* [Deployment](#deployment)
   * [k8s liveness readiness startup probes](#k8s-liveness-readiness-startup-probes)
      * [Quel intérêt de la sonde de startup ?](#quel-intérêt-de-la-sonde-de-startup-)
* [Télémétrie avec OpenTelemetry](#télémétrie-avec-opentelemetry)
* [stdout vs stderr](#stdout-vs-stderr)
* [Gestion de projet BUILD vs RUN](#gestion-de-projet-build-vs-run)
   * [BUILD](#build)
   * [RUN](#run)


# Gestion des erreurs

En résumé (à l'occasion de l'annotation de [cet article](../../2024-10-15-ultimate-guide-error-handling-in-python.md)) :

- si mon code est face à une erreur "récupérable", normale :
    - s'il sait comment y réagir, il adresse l'erreur lui-même (typiquement : il catche une exception et ne la propage pas)
    - s'il ne sait pas comment y réagir, il peut propager l'exception (i.e. il ne catche rien) dans l'espoir qu'un appelant saura le faire
        - mais attention : c'est peut-être aussi le signe que mon code devrait gérer ce type d'erreur lui-même plutôt que propager ?
- si mon code est face à une erreur non-récupérable (bug) :
    - en dev = je plante salement le plus tôt possible, avec une stacktrace pour investiguer (objectif = détecter les soucis et les corriger)
    - en prod = propager l'erreur vers l'appelant pour 1. logger proprement le souci 2. permettre à un appelant de haut-niveau d'essayer de contourner le souci, et ainsi continuer à traiter (si possible) en évitant le crash/downtime
        - e.g. pour un serveur web, un appelant de haut-niveau réagira à une exception non-traitée en répondant `HTTP 500`, mais juste pour cette requête : sans pour autant faire planter tout le serveur

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


## Property-based testing

Quelques notes issues de [100% test coverage is not enough](https://blog.robertroskam.com/p/100-test-coverage-is-not-enough ), lu le 6 novembre 2023 :

- 100% code coverage ne garantit pas que le programme est correct
- l'exemple illustratif est parfait pour illustrer ce point, car simplissime :
   ```py
   def inverse_of(x: int) -> float:
       return 1/x
   ```
- property basée testing peut être complémentaire au 100% coverage
- c'est un genre de fuzzing pour les tests U
- plutôt adapté aux tests des fonctions bas niveau, plutôt qu'aux tests d'une API de haut niveau

# Deployment

## k8s liveness readiness startup probes

TL;DR = les sondes permettent de répondre à une question différente :

- `liveness` = le container doit-il être killé ?
- `readiness` = le container doit-il être sorti du load-balancer ?

La sonde optionnelle `startup` permet d'inhiber les deux autres jusqu'à ce que le container ait fini de démarrer : k8s attendra que la sonde de `startup` ait répondu au moins une fois **OK** avant de commencer à utiliser les sondes `liveness` et `readiness`.

----

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

En gros, :

- la **liveness probe** sert à restart un container : si elle faile, l'application est considérée comme morte, k8s tue le pod et le restarte (on peut souhaiter ce comportement même si le process tourne encore ; p.ex. si notre app est bloquée dans un deadlock)
- la **readiness probe** sert à autoriser le routage de traffic vers le container, si elle faile, l'application est sortie du flux du load-balancer :
    > The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

Ces deux probes peuvent éventuellement être servies par le même endpoint HTTP :

> A common pattern for liveness probes is to use the same low-cost HTTP endpoint as for readiness probes, but with a higher failureThreshold. This ensures that the pod is observed as not-ready for some period of time before it is hard killed.

^ la sonde `readiness` est moins forte que `liveness`, de sorte qu'on essaye d'abord de sortir l'application du flux et de la laisser retrouver un état stable, avant de finalement la killer.

La **startup probe** est optionnelle, elle sert dans les cas où l'application met du temps à démarrer (e.g. elle a beaucoup de données à charger). Dans ce cas, la liveness probe ou la readiness probe peuvent ne pas être en mesure de répondre suffisamment vite avant que le container ne se fasse killer.

Dit autrement : le fonctionnement par liveness/readiness peut n'être prêt à faire son boulot que de longues minutes après le démarrage de l'application. La startup probe est là pour ça : pour désactiver le fonctionnement de liveness/readiness jusqu'à ce que le startup soit fini :

> The kubelet uses startup probes to know when a container application has started. If such a probe is configured, liveness and readiness probes do not start until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

### Quel intérêt de la sonde de startup ?


Prenons l'exemple d'une app qui doit charger des données en RAM pendant cinq minutes au démarrage. Pourquoi l'app ne choisirait pas de répondre `liveness=true + readiness=false` pendant toute la durée de son chargement ?

Il y a deux raisons pour laquelle ça ne marchera pas forcément : d'abord, toutes les apps n'ont pas le loisir de pouvoir répondre aux sondes _en même temps_ qu'elles effectuent le chargement nécessaire au démarrage ; dans ce cas, les sondes ne répondront pas tant que les données ne seront pas chargées, elles seront donc considérées comme en échec pendant les 5 premières minutes : possiblement, c'est suffisant pour que k8s considère l'application comme morte, et la kille.

k8s permet au devops de configurer un temps d'attente initial avant de commencer à prober `liveness+readiness`, ce qui nous amène à la deuxième raison : si le temps de chargement est variable, p.ex. entre 30 secondes et 5 minutes, comment le devops doit-il configurer le délai initial ?

- s'il configure à 30 secondes, l'app sera parfois killée avant même d'avoir fini de démarrer
- s'il configure à 5 minutes, l'app sera parfois laissée vivante alors même qu'elle a démarré depuis 30 secondes, et est en deadlock sur les 3 dernières minutes

Conclusion = pas facile de prendre en compte le temps de démarrage de façon externe... Ce qu'on voudrait, c'est configurer le délai initial à "tout pile le temps nécessaire au démarrage", i.e. commencer les liveness probes uniquement après le démarrage. C'est exactement à ça que sert la startup probe : tant qu'elle n'a pas répondu **OK** au moins une fois, les liveness+readiness probes ne démarrent pas.


# Télémétrie avec OpenTelemetry

cf. mes notes spécifiques sur OpenTelemetry, et mes POCs.

# stdout vs stderr

- **stdout** = sortie "normale" → peut-être redirigée vers un fichier ou un autre programme → contient du contenu potentiellement parsable par un ordinateur
- **stderr** = sortie "en erreur" (mais pas forcément une erreur !) → a vocation à être lu par un humain → contient du contenu human-readable

C'est un peu contre-intuitif, parce que la sortie sur stderr ne correspond pas forcément à une erreur !

C'est plus simple avec un exemple concret : un programme qui convertit un fichier CSV en JSON :

- le json fabriqué en sortie ira sur **stdout** (ça c'est intuitif)
- un log d'erreur du genre `Input file 'toto.csv' is not a CSV file !` ira sur **stderr** (ça aussi c'est intuitif)
- un log pas d'erreur du genre `Reading input file 'toto.csv'` ira aussi sur **stderr** (ça c'est moint intuitif !)
- résultat : on peut rediriger stdout vers un fichier pour enregistrer le json produit, tout en ayant les "logs" (normaux, ou en erreur) disponibles sur stderr


# Gestion de projet BUILD vs RUN


(j'ai pris les deux premiers liens google : [source1](https://fr.linkedin.com/pulse/le-run-et-build-en-gestion-de-projet-ma%C3%AEtriser-les-deux-moueza-sihbe) + [source2](https://blog-gestion-de-projet.com/passage-projet-build-au-run/))

## BUILD

**BUILD** = le projet est en cours de développement :

- Analyse des besoins
- Conception
- Développement
- Tests
- Déploiement initial

ENJEU = respecter qualité + délais.

LIVRABLES :

- doc projet + doc utilisateur + formation
- PV de recette validé
- backlog résiduel (reste à faire)
- PV de réception des livrables

À la fin du BUILD, pour transitionner vers RUN : livraison du projet au client + recette (prioriser les anomalies selon 2 axes : reproductibilité + criticité)

## RUN

**RUN** = le projet est réalisé, il est en phase d'exploitation :

- Support utilisateur
- Maintenance corrective
- Maintenance préventive
- Supervision
- Gestion des incidents

ENJEU = garantie du Maintien en Conditions Opérationnelles (MCO)

LIVRABLES :

- contrat SLA
- dimensionnement + constitution de l'équipe de maintenance
- matrice RACI = qui fait quoi (Responsible / Accountable / Consulted / Informed)
- processus + matrice d'escalade
- liste des KPI à suivre
- registre des demandes de changements
- rapports de suivi


