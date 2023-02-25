# An Ode to Unit Tests: in Defense of the Testing Pyramid

- **url** = https://www.infoq.com/articles/unit-tests-testing-pyramid/
- **type** = blogpost
- **auteur** = [Guilherme FERREIRA](https://gsferreira.com/), minimalist software craftsman
- **date de publication** = 2022-12-19
- **source** = [infoq](https://www.infoq.com/), news aggregator tech
- **tags** = language>agnostic ; topic>testing ; level>intermediate


**TL;DR** : un EXCELLENT article qui réfléchit à quoi mettre derrière les dénominations **tests unitaires** et **tests d'intégration**, pourquoi préférer les premiers aux seconds, et comment tester correctement.

Il aboutit à ce que j'appelle les dénominations "modernes" des tests unitaires et tests d'intégration :

- les tests unitaires testent l'application isolée du monde extérieur, et testent le comportement publiquement exposé (blackbox testing) plutôt que les détails internes de l'implémentation (whitebox testing).
- les tests d'intégration testent uniquement les adapters, i.e. les modules responsables de communiquer avec le monde extérieur
- NdM : ce n'est pas explicitement précisé, mais les end-to-end-tests testent tout le monde en même temps = l'ensemble {adapters + hexagon} : certes on en a toujours besoin, MAIS ces tests peuvent se limiter à des smoke-tests (vu que les détails ont été testés en tests unitaires + tests d'intégration).

Il souligne le lien entre architecture et testing : l'architecture doit être pensée dès le début avec la problématique de "comment tester mon app" en tête ; c'est notamment ce que fait l'hexagonal architecture.

---

## Historique

NdM : dans l'article, il fait le lien entre (ce que j'appelle) la dénomination old-school des tests d'intégration = le fait de tester plusieurs modules de code en même temps, avec (ce que j'appelle) la dénomination "moderne" des tests d'intégration = le fait de tester uniquement le module de code responsable de la communication avec le monde extérieur).

> Historically, integration testing was the stage when different development units were tested together. Those units were developed in isolation, often by multiple teams. That was the phase when we guaranteed that the defined interfaces were well implemented and worked accordingly.
>
> Nowadays, we see integration tests applied to code units developed by the same team. This implies that each source code file is a system boundary. As if each code file had been developed by an autonomous team. This is blurring the lines between unit and integration tests.

^ Tout l'enjeu est de savoir quelle est la boundary testée : historiquement, les tests unitaires testaient l'interface extérieure d'un module développé par une équipe (et les tests d'intégration testaient l'intégration de plusieurs modules) ; mais on a malheureusement shifté vers "un test unitaire teste une unité de code" ; c'est ce dernier shift que j'appelle les "dénominations old-school" des tests U / tests d'intégration.


Du coup :

> That moment was the epicenter of a new wave. A wave that took us to today, where unit tests are losing importance in favor of integration tests.

^ À partir de 2014, on abandonne les tests unitaires en faveur des tests d'intégration → la **testing pyramid** devient le **testing diamond**, vu qu'on veut maintenant plus de tests d'intégration.

Ce que dit l'auteur de l'article, c'est que cette mouvance a ses origines surtout dans le fait qu'on a mal compris les tests unitaires (en les réservant à des unités de code plutôt que des unités comportementales), et promeut au contraire des "tests unitaires" selon "ma" dénomination moderne = qui teste plusieurs modules de code.

C'est parce qu'on n'a pas suivi ce principe (en testant unitairement le code plutôt que les use-cases = le comportement) que les tests unitaires ne sont plus aimés, et qu'on leur préfère les tests d'intégration, qui, eux, testent le comportement.

Mais ça n'est pas un problème de "unitaire" vs. "intégration" ! C'est un problème de "tester le code" vs. "tester le use-case" !

## Lien avec l'archi hexagonale + que doivent représenter les tests "unitaires" et les tests "d'intégration"

> In (Hexagonal Architecture), (Alistair COCKBURN) describes a system as having two sides. The internal and the external ones. [...]  It is this internal/external relationship that makes it clear the relationship between unit and integration tests. The unit tests are responsible for testing the boundary from an outside-in perspective. While the integration tests will test the boundary from an inside-out perspective. In concrete words, we can say that integration tests ensure the correct behavior of the Adapters, [...] that mediate the relationship with other development units (such as APIs, Plugins, Databases, and Modules).

Pour lui (et ça correspond avec la dénomination moderne des tests U / tests d'intégration) :

- tests unitaires = test du "domain model" = l'intérieur de l'hexagone dans l'archi hexagonale (donc **indépendamment** des adapters i.e. du monde extérieur)
- tests d'intégration = test des adapters = des modules qui interagissent avec le monde extérieur

> What does the unit in unit tests mean? It means a unit of behavior. There's nothing in that definition dictating that a test has to focus on a single file, object, or function. Why is it difficult to write unit tests focused on behavior?

^ Le point important, c'est ce qu'on désigne par "unitaire" :

- avec la dénomination oldschool, "unitaire" désigne une "unité de code" = un fichier, une classe, un module, ...
- avec la dénomination moderne, "unitaire" désigne une "unité de comportement" = un use-case, un usage de port, ...

> A common problem with many types of testing comes from a tight connection between software structure and tests. That happens when the developer loses sight of the test goal and approaches it in a clear-box (sometimes referred to as white-box) way. 
>
> Clear-box testing means testing with the internal design in mind to guarantee the system works correctly. This is really common in unit tests. The problem with clear-box testing is that tests tend to become too granular, and you end up with a huge number of tests that are hard to maintain due to their tight coupling to the underlying structure.
>
> Part of the unhappiness around unit tests stems from this fact. Integration tests, being more removed from the underlying design, tend to be impacted less by refactoring than unit tests. 

^ Du coup, ceci suit logiquement (et est en lien avec la _screaming architecture_) : l'organisation du code — et donc le testing : il y a vraiment un lien fort entre archi et testing — doit refléter les use-cases de l'application, plutôt que de suivre l'implémentation.

C'est parce qu'on n'a pas suivi ce principe (en testant unitairement le code plutôt que les use-cases = le comportement) que les tests unitaires ne sont plus aimés, et qu'on leur préfère les tests d'intégration, qui, eux, testent le comportement.

Mais ça n'est pas un problème de "unitaire" vs. "intégration" ! C'est un problème de "tester le code" vs. "tester le use-case" !

## Reduce the Public Surface of Your Code

> Another side effect of clear-box testing is that it leads to exposing more code than needed. [...]
>
> Once a piece of code is accessible from the outside, it becomes harder to change, and tests become required. This will lead to code where maintainability is a problem and refactoring is almost impossible without rewriting a ton of unit tests.
>
> On the surface that looks like an argument in favor of integration tests, since integration tests focus on the outer layer where many of these implementation details don't leak. [...]
>
> If we implement unit tests in an opaque-box way, ignoring the internal design decisions and only being concerned with what consumers need, it will lead to a smaller contract. A contract that is easier to test, with fewer tests, and tests that are easier to maintain.

^ Ici aussi, on en vient au même principe = tester le comportement public, ce qui évite d'avoir des tests friables et qu'on doit systématiquement mettre à jour dès qu'on touche à l'implémentation de l'application.

## Architecture as the Guiding Principle

> Tests tend to grow around architecture. We design our systems, then we think about testing. When we do that, systems can become harder to test. [...] That can easily be avoided by adopting an architecture with test isolation in mind. From Hexagonal Architecture to Clean Architecture.

^ l'architecture doit être pensée dès le début avec la problématique de "comment tester mon app" en tête ; c'est ce que fait l'hexagonal architecture.

> This type of architecture will make unit testing comfortable and lead you to use integration tests for what they should be: testing adapters to the outside world.

On en vient à la dénomination moderne :

- les unit-tests testent l'application isolée du monde extérieur
- les integration-test testent les adapters qui communiquent avec le monde extérieur
- NdM : ce n'est pas explicitement précisé, mais les end-to-end-tests testent tout le monde en même temps = les adapters pluggés à l'hexagon

> We can still run tests with all components connected. The difference is that those become "smoke tests" and don't need to test every single corner case. 

Ce point est important à mes yeux : on a toujours besoin de tests qui vérifient toute la chaîne, MAIS ces tests peuvent se limiter à des smoke-tests (vu que les détails ont été testés en tests unitaires + tests d'intégration).

---

Au final, il recommande l'utilisation "moderne" des tests unitaires dans ses key takeaways :

- Opaque-box tests are not exclusive to testing through public interfaces of the system. A system is composed of many boundaries, and all of them benefit from behaviour-focused tests.
- By avoiding clear-box testing, the need for heavy mocking and public interfaces will drop significantly. This leads to a more maintainable test portfolio.
- Avoid publicly accessible code at all costs. The less code you have that is accessible, the easier it is to maintain, evolve, and refactor your code.
- Build architectures with a testing strategy in mind. How easy it is to test them will dictate the success of the architecture.
