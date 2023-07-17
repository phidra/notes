# Test-induced design damage

- **url** = https://dhh.dk/2014/test-induced-design-damage.html
- **type** = post
- **auteur** = [David HEINEMEIER HANSSON](https://dhh.dk/), créateur de Ruby On Rails, no less.
- **date de publication** = 2014-04-29
- **source** = [son site](https://dhh.dk/)
- **tags** = language>agnostic ; topic>design ; topic>testing ; level>intermediate


Article mentionné dans [la prez de Mark SEEMANN pits of success](./2023-07-13-function-architecture-pits-of-success.md), c'est pour ça que je l'ai lu.

> that's a far ways off from declaring that hard-to-unit-test code is always poorly designed, and always in need of repair.

^ En gros, le message de l'article est dérivé de cet extrait : on ne devrait pas laisser le caractère "facilement unit-testable" guider TOUTES les décisions de design.

> It's from this unfortunate maxim that much of the test-induced design damage flows. Such damage is defined as changes to your code that either facilitates a) easier test-first, b) speedy tests, or c) unit tests, but does so by harming the clarity of the code through — usually through needless indirection and conceptual overhead. Code that is warped out of shape solely to accomodate testing objectives.

^ Le message principal est "si on essaye à tout prix de tout rendre unit-testable, alors on cause des dommages".

> While you're watching the presentation, listen to the justifications for the design. They're all about testing! It's about having faster tests, without touching the database, and it's about being able to test controller logic without dependent context. (...) The code has suffered tremendous design damage to achieve two testing goals: Faster tests and easy-to-mock unit tested controllers.

^ Un exemple concret de dommages créés parce qu'on n'a designé que pour les tests.

> The hexagonal pattern is being misapplied to Rails applications, alongside it companions of boundaries and ports'n'adapter-like patterns, for the purpose of testing. This is test-driven pattern application.

^ Son message (qu'il généralise un peu après) = attention à ne pas faire tourner le design de l'application QUE autour des tests : p.ex. l'archi hexagonale a d'autres avantages.

> One conclusion of this is that I think it's a mistake to try to unit test controllers in Rails (or similar MVC setups). The purpose of the controller is to integrate the requests from the user with the response from the models within the context of session. (...) Controllers are meant to be integration tested, not unit tested. (...) The view is at the top layer of the MVC cake, the appropriate testing metaphor for the vast bulk of that is system testing: End-to-end.

^ on n'a pas une obligation que TOUT soit unit-testable ! Certains bouts de code ont plutôt vocation à être testés par des tests d'intégration, voire des tests e2e !

> Finally, the fear of letting model tests talk to the database is outdated. This decoupling is simply not worth it any more, even if it may once have been.

^ autant je partage à 100% son avis sur l'extrait précédent, autant là je ne suis pas d'accord...

> The bottom line for Rails/MVC testing is imo this:
>
> Test models with intricate logic through model tests, but don't bother trying to separate them from the database. (...)
> Test controllers using integration tests. Don't even try to make it fit under the unit testing paradigm. (...)
> Test views through system/browser testing. This is particularly true if your view is powered in part by JavaScript. You have to make sure that all that stuff works too. (...)

^ son résumé.

> But the most important part is where to place the emphasis. If you have a very simple model layer, but your UI is complex, then you should indeed be system-test heavy, and model-test light! This is not a sin. If your controllers do very little interesting work, maybe you don't need to test drive them at all, if you're covered by the system tests. And if you hardly have any JavaScript of note, maybe integration tests are actually the place to be.

^ je suis 100% d'accord : soyons pragmatiques et mettons l'effort de test là où il y a de la complexité / de la valeur ajoutée !

> Above all, you do not let your tests drive your design, you let your design drive your tests! The design is going to point you in the right direction of what layer in the MVC cake should get the most test frosting. (...) The design integrity of your system is far more important than being able to test it any particular layer. Stop obsessing about unit tests, embrace backfilling of tests when you're happy with the design, and strive for overall system clarity as your principle pursuit.

Son point de vbue = et pour ce faire, garder le design de l'application focusé sur l'application elle même (plutôt que sur son testing), n'envisager le testing que dans un second temps, comme ça on verra clairement quelles parties de l'application en auront le plus besoin.
