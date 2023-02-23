# You are probably testing wrong

- **url** = https://dev.to/wparad/you-are-probably-testing-wrong-4il3
- **type** = blogpost
- **auteur** = [Warren PARAD](https://github.com/wparad), tech consultant
- **date de publication** = 2023-02-02
- **source** = [son espace tech sur dev.to](https://dev.to/wparad/)
- **tags** = language>agnostic ; topic>testing ; level>intermediate

**TL;DR** = un point de vue général sur "quand ajouter des tests ?" Du bon et du moins bon dans cet article, mais il vaut clairement le coup d'oeil, pour l'approche "attention à bien réfléchir à l'objectif visé par l'ajout d'un test".

> The right number (of code coverage) is based on the problem we are trying to solve

^ Essayer de se fixer un objectif de code-coverage à atteindre, c'est se tromper d'objectif.

Les bonnes raisons d'écrire des tests :

- _Frequently changing code_
- _Preventing regressions_ (notamment, ici il dit un truc intéressant = quand on touche à une portion du code qui n'avait (légitimement) pas de test car elle n'en avait pas besoin, et que suite à notre modif la portion du code devient suffisamment complexe pour nécessiter d'être testée, alors il faut commencer par écrire le test de l'existant (pour "rattrapper le retard") avant de faire sa modif)
- _High impact code_
- _Business Logic justifications_ (NdM : en gros, ici , le test est une forme d'auto-documentation, en alternative à un commentaire)

Jusqu'ici je suis d'accord avec l'article. Mais ensuite, il donne la définition old-school des tests unitaires (= test d'un tout petit bout de code), des component/service-level tests (ce qui est sa dénomination de ce que la dénomination old-school appelle test d'intégration = des tests de plusieurs composants qui interagissent entreux) et la pyramide de tests :

> Above unit tests we have in ascending order:
>
> - component/service level
> - integration--also known as Production Tests
> - exploratory manual tests
>
> Have 2000 unit tests?, that means you want ~20 service level tests and ~1 integration test.

De plus, il dit autre chose avec lequel je ne suis pas d'accord :

> You'll notice that no where in the pyramid are E2E (End to End) tests. That's because in any real technology deployment it's impossible to test anything end to end and we actually don't need to. That's because our end to end isn't run synchronously, so we don't need synchronous tests to validate that. Further, most of this will happen any way when our users use our technology.

En gros, on attend que ça pète en prod pour vérifier si ça marche...

Il revient ensuite à un point de vue que je partage plus = viser 100% de coverage est un antipattern, puisque on finit par se concentrer sur le chiffre plutôt que ce qu'il y a derrière.

Et il reswitche de nouveau sur quelque chose qui est au mieux discutable = "si la prod n'est jamais cassée, c'est que nos tests en font trop côté qualité" (je pourrais possiblement être d'accord, mais uniquement dans des contextes très précis, p.ex. si on a un CI/CD tellement bon et réactif qu'on peut MEP en moins d'une heure).

> This brings us back to the original point of adding tests where we know we need to.

Pour savoir s'il faut ajouter des tests, il propose de se référer à deux métriques :

- CFR = Change Failure Rate = le pourcentage de déploiements qui provoquent une erreur
- MTTR = Mean Time To Resolution = temps moyen de résolution d'erreur

Ça fait partie des [métriques DORA](https://www.leanix.net/fr/wiki/vsm/dora-metrics) :

- Deployment Frequency = fréquence des MEP réussies)
- Lead time for changes = temps entre la réception d'une demande d'évolution du code et son passage à un état déployable)
- Mean Time To Recover = temps entre une erreur suite à un déploiement ou une défaillance, et le rétablissement complet
- Change Failure Rate = percentage of deployments that result in a failure in production, which ultimately requires some type of remediation (définition issue de [ce lien](https://jellyfish.co/blog/change-failure-rate/))

> if we don't know the answers to those then we also don't know how many tests we should have [...] creating arbitrary tests is the wrong solution. You simply are spraying tests everywhere hoping to hit something. Don't add tests randomly.

Il justifie derrière son point de vue sur le fait d'autoriser la prod à tomber :

> Another way of looking at this is, I want to see prod breaking in ways that don't matter, but I don't want it to break it ways that do. If production isn't breaking at all, you are violating this, and of course if it is breaking in ways that does matter, then you need to add more tests.

(mais again, mon point de vue est qu'on ne peut pas faire ce jugement de façon générale, ou plus exactement que en fonction du temps nécessaire à MEP un correctif, il peut n'exister que des "ways that does matter", et aucun "ways that don't matter").

Le mec promeut (fort logiquement) le sujet du logging :

> if we don't know when we have a problem and also details about what that problem is, then we have no idea what the fix should be

Derrière, il promeut le defensive-programming (avec un try...catch autour de son code) ; je ne suis pas emballé, mais j'ai un avis moins fort là-dessus que sur d'autres sujets.

Il encourage derrière à s'interroger sur la vraie cause des problèmes ; exemple = ajouter un test n'est pas forcément la bonne solution à un souci en prod, peut-être vaut-il mieux remplacer son module custom par une implémentation standard sur étagère.

Même si je suis loin d'être d'accord avec tout ce que dit l'article, ce message en conclusion reste intéressant :

> Adding unit tests to all our code creates a burden for future development. So we need trade-off extra burden for extra value. Arbitrarily adding tests to meet a "test coverage" always results in the wrong tests.

Il encourage à mieux répartir **reactive** (= laisser les bugs se produire et les fixer rapidement) vs. **proactive** (ajouter des tests pour essayer d'empêcher les bugs d'arriver) :

> Testing is proactive, find problems before they happen. [...] it would take an infinite of time to prevent all bugs [...]. So don't test everywhere, test only in some places. The highest value places. Those we can be proactive, but everywhere else we should optimize for being reactive.

Il encourage à plus de discernement sur "quoi tester", notamment vis-à-vis des objectifs de test coverage :

> Clever tests in the right spot are worth so much more than an arbitrary percentage. "Should this thing have automated testing" is a conversation, it definitely can't be "we have some arbitrary metric to hit".

