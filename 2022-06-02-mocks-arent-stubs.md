# Mocks Aren't Stubs

- **url** = https://martinfowler.com/articles/mocksArentStubs.html
- **type** = post
- **auteur** = [Martin FOWLER](https://martinfowler.com/) = auteur de bouquins (dont [Refactoring](https://martinfowler.com/books/refactoring.html), une référence), conférencier
- **date de publication** = 2007-01-02
- **source** = [son blog](https://martinfowler.com/)
- **tags** = language>agnostic ; topic>design ; topic>testing ; topic>classicists-vs-mockists ; level>intermediate


**TL;DR** : un article ancien mais très intéressant (mais aussi très dense en infos pertinentes : presque chaque paragraphe du post a une petite pépite de sagesse) sur l'approche classicist vs. mockist pour le testing, et il explique très bien les différences entre les deux, ce qui permet de se forger une opinion sur leurs avantages/inconvénients. En deux mots :

- Classicist = Detroit School of Testing = **state verification** : on vérifie que l'état final d'un collaborator est bien celui attendu
- Mockist = London School of Testing = **behaviour verification** : sans s'intéresser à l'état final, on vérifie plutôt que le mock du collaborator a bien été utilisé comme attendu

Les deux approches sont différentes non seulement sur la façon dont implémente les tests (qu'on vérifie que le SUT se comporte correctement), mais également sur les conséquences sur le design : comment les classicistes et les mockistes utilisent TDD différemment pour designer leur application.

Une notion très importante à retenir de ce post également (primordiale pour affiner son avis vis-à-vis des tests U) est la notion de **SUT = system-under-test** vs. **collaborator**.

* [Mocks Aren't Stubs](#mocks-arent-stubs)
   * [Regular Tests](#regular-tests)
   * [Tests with Mock Objects](#tests-with-mock-objects)
   * [The Difference Between Mocks and Stubs](#the-difference-between-mocks-and-stubs)
   * [Classical and Mockist Testing](#classical-and-mockist-testing)
   * [Choosing Between the Differences](#choosing-between-the-differences)
      * [State verification vs. behaviour verification](#state-verification-vs-behaviour-verification)
      * [Driving TDD](#driving-tdd)
      * [Fixture Setup](#fixture-setup)
      * [Test Isolation](#test-isolation)
      * [Coupling Tests to Implementations](#coupling-tests-to-implementations)
      * [Design Style](#design-style)
   * [So should I be a classicist or a mockist?](#so-should-i-be-a-classicist-or-a-mockist)

## Regular Tests

Dans ce chapitre, il commence par donner un exemple d'approche classiciste, avec un fil rouge simple mais clair :

- un objet `Warehouse` contient des articles (chaque article dans l'entrepot est associé à une quantité disponible)
- un objet `Order` permet de passer commande d'une certaine quantité d'articles précis
- la méthode `Order.fill` permet de remplir la commande :
   + s'il y a assez d'objets dans le warehouse, on remplit la commande, et on vide l'entrepôt
   + s'il n'y a pas assez d'objets, on ne touche pas à l'entrepôt ni à la commande

Il donne un exemple de test associé _dans le style classicist = state vérification_, ce qui lui permet de définir les pratiques et la nomenclature utilisé :

- 4 phases (**setup**, **exercise** = l'objet testé effectue le traitement qu'on veut tester, **verify** = là où on a les assertions qui vérifient si l'objet testé s'est bien comporté, **teardown**)
- deux types d'objets : **system-under-test = SUT** (= l'objet dont on veut vérifier le comportement, ici l' `Order`) + **collaborators** (= les objets dont le SUT a besoin pour fonctionner, ici le `Warehouse`)
- à noter que même si c'est le SUT qu'on teste, on peut tout de même avoir besoin de faire des assertions sur les collaborators (e.g. ici, le fait de tester `Order.fill` implique de vérifier que la méthode a bien muté le `Warehouse` pour lui retirer les articles)

À retenir = classicist = on utilise le vrai objet `Warehouse` (mais en lui passant des fausses données forgées juste pour le test), et on asserte l'état du `Warehouse` (qui doit avoir maintenant des articles en moins)

## Tests with Mock Objects

Ici, il reprend le même exemple, et change les tests pour présenter l'approche mockiste (avec la librairie jMock, mais c'est anecdotique) :

- la phase de setup est différente : déjà, on a DEUX trucs à setup = les données de test comme avant, mais également le comportement attendu du SUT vis-à-vis de ses collaborators
- du coup, on n'utilise plus le vrai objet Warehouse, mais un mock (qui permettra de vérifier que l'Order l'a utilisé correctement)
- après avoir exercised le SUT, il passe aux vérifications, et ici aussi il y en a maintenant de deux types :
   + le SUT lui-même est toujours vérifié en state-verification (ici, on assert l'état de l'Order)
   + mais pour les collaborators, on n'assert plus le collaborator : on assert plutôt que le mock a bien été utilisé comme attendu
   + très concrètement, on vérifie que appeler `Order.fill` a bien appelé `MockWarehouse.remove(50 articles)` (et on n'asserte plus l'état du collaborator !)
   + et si le SUT n'est même pas muté quand on l'exercise (comme dans le cas d'une fonction qui se contente d'orchestrer des sous-systèmes), alors il n'y a pas d'assert du tout sur lui, et le test ne consiste qu'en des assert sur le behaviour du SUT vis-à-vis de ses collaborateurs

## The Difference Between Mocks and Stubs

Les mocks ne sont qu'UN type de test doubles, mais il y en a d'autres !

Problèmatique des **test doubles** = quand on test un module unique (test unitaire), la plupart du temps, ce module a besoin d'autres modules (e.g. le `Warehouse` pour notre `Order`) :
- classicist = on utilise un vrai objet si possible, et un test double sinon
- mockist = on utilise un test double particulier (=mock) quoiqu'il arrive, vu qu'on veut pouvoir asserter la façon dont il est utilisé, on a donc besoin d'un objet capable d'intercepter ses utilisations

Mais il y a d'autres types de **test double** :

- dummy objects = un objet passé mais non utilisé (passé uniquement pour satisfaire un paramètre attendu par une fonction)
- fake objects = un objet qui est implémenté juste pour permettre le test, donc qui prend des raccourcis (e.g. in-memory database)
- stubs = objet qui ne fait pas réellement son boulot, mais qui remplace son travail par des réponses toutes faites (généralement, spécifiquement conçues pour faire marcher le test)
- spies = c'est un stub spécial qui enregistre des infos (e.g. le nombre d'appel) pour pouvoir faire des vérifications plus tard
- mocks = (NdM = je vois ça comme une généralisation des spies) = objets disposant d'une spécification des appels qu'ils sont supposés recevoir

Parmi ceux-ci, seuls les mocks accordent une place importante à la **behaviour verification** (NdM : et un peu les spies aussi, IMO), les autres utilisent la **state verification**.

En fait, les mocks fonctionnent comme les autres test doubles lors de la phase d'exercice (= ils permettent au SUT de fonctionner et d'être exercised), c'est aux phaes de setup et verification qu'ils diffèrent.

Pour mieux illustrer la différence entre mock et les autres test-doubles, il prend un exemple plus complexe :
- si jamais il n'y a pas assez d'articles dans l'entrepôt, on veut envoyer un mail
- problème = on ne veut pas utiliser le vrai objet `MailSender`, car dans le cadre des tests on ne veut PAS envoyer de vrais mails
- mais comme l' `Order` a tout de même besoin d'un `MailSender`, il nous faut utiliser... un **test-double** \o/

Avec une approche classiciste :
- on créé un `MailSenderStub` qui implémente l'interface `MailSender`, qui n'envoie pas réellement des mails, et qui se contente juste de mémoriser le nombre de mails envoyés
- lors du setup, on créée une `Order` qui utilise le `MailSenderStub`
- lors de l'exercise, on essaye de fill une `Order` quand l'entrepôt n'a pas assez d'articles
- on vérifie en fin de test le STATE du **collaborator**, en assertant sur le nombre de mails envoyés par le stub

Avec une approche mockiste :

- on a un **MOCK** de `MailSender`, qui non seulement n'envoie pas réellement des mails, mais en plus permet d'asserter sur les méthodes appelées
- lors du setup, on créée une `Order` qui utilise le `MailSenderMock`
- lors du setup, on définit également les attentes d'utilisation du `MailSenderMock`
- lors de l'exercise, on essaye de fill une `Order` quand l'entrepôt n'a pas assez d'articles
- on vérifie en fin de test que le MailSenderMock a bien été utilisé comme attendu par l' `Order` (et à la différence de l'approche classiciste, on ne vérifie plus son état final)

Dans les deux cas, on passe un **test-double** (un faux `MailSender`) à l' `Order`... la différence est dans la façon dont le test vérifie que l' `Order` se comporte comme souhaité.

## Classical and Mockist Testing

La vraie différence, c'est la façon dont un classicist et un mockist font du TDD :

- classicist = utilise des vrais objets si possible (ici, un vrai `Warehouse`), et des test-doubles sinon, i.e. si c'est pas facile d'utiliser des vrais objets (ici, un test-double pour le `MailSender`)
- mockist = n'utilise jamais de vrais objets, n'utilise que des mocks (aussi bien pour `Warehouse` que pour `MailSender`)

(ça n'empêche pas les frameworks de mocks d'être utilisable par les classicists pour créer facilement des test-doubles)

Ici, une corrélation est faite avec BDD (avec [ce lien](https://dannorth.net/introducing-bdd/), qui grosso-modo présente le given-when-then), mais je n'annote pas ce point.

## Choosing Between the Differences

Le chapitre le plus intéressant une fois que les deux premiers ont permis de comprendre ce que sont les deux approches, puisqu'il permet de comprendre les différences entre les deux approches.


### State verification vs. behaviour verification

D'une façon générale, les collaborators sont de deux types : easy (`Warehouse`) et awkward (`MailSender`).

Dans le cas d'une easy collaboration :

- le classicist utilise le vrai objet + **state verification**
- le mockist utilise un mock + **behaviour verification**

Dans le cas d'une awkward collaboration :

- le classicist a le choix entre utiliser un **test-double** + **state verification**, ou bien utiliser un mock + **behaviour verification** (et il choisit le plus simple à mettre en place)
- le mockist utilise systématiquement un mock + **behaviour verification**

Autre exemple (pas vraiment awkward, mais très dur à tester en **state verification** = un cache. Dans ce cas, ne pas hésiter à utiliser le **behaviour verification**.

### Driving TDD

Les deux approches conduisent à deux façons très différentes de faire du TDD.

Mockist = **outside-in** :

- tu commences par définir l'utilisation externe de ton système (e.g. son UI) et tu créées ton premier test sur ce sujet
- comme il teste les interactions de ton SUT avec ses collaborators, ce test te force à réfléchir à ses attentes vis-à-vis de ce qu'il utilise, vu qu'il faut les vérifier sur des mocks dans la **behaviour verification**
- puis, chacune de ces vérifications sur des mocks peut être la prochaine étape du processus TDD : convertie en test U, puis implémentée pour faire passer le test U
- style **OUTSIDE-IN** (on démarre de l'extérieur, et on avance en couche vers l'intérieur), bien adapté aux layered systems

classicist = **middle-out** :

- à partir d'une feature demandée par le PO, on définit une fonction bas-niveau, dans la couche métier (domain), qui sera nécessaire à cette feature
- tu utilises TDD pour tester + implémenter la couche métier dont tu as besoin, et tu fabriques ton UI par dessus après coup (d'où le no **MIDDLE-OUT**)
- sur le principe, on peut donc aussi avancer étape par étape (en utilisant des stubs hard-codant le comportement souhaité pour implémenter les collaborators, puis en testant+implémentant réellement chaque stub pour la prochaine étape)
- avantage 1 = si tu commences suffisamment bas, il se peut que tu n'aies rien à remplacer par des test-doubles (en effet, les objets nécessaires au SUT ont déjà été implémentés, vu qu'ils sont plus bas dans les couches, et que tu as commencé par les couches inférieures)
- avantage 2 = ça évite que la couche métier leake dans la couche UI (i.e. ça évite qu'on implémente dans l'UI quelque chose qui aurait plutôt sa place dans la couche métier)

Attention que ce n'est pas une remise en question du principe TDD : dans les DEUX cas, on garde l'approche TDD itérative (une feature à la fois, une story à la fois).

### Fixture Setup

classicist = comme on essaye d'utiliser de vrais objets (au moins pour le **SUT**, mais possiblement aussi les collaborators), il faut les créer dans le setup (et certains **SUT** de haut-niveau peuvent nécessiter (récursivement) beaucoup de collaborators) :

- pour simplifier ces créations, on a tendance à les mutualiser dans des setup de test, qui sont réutilisés
- du coup, le code qui setup le test devient un code complexe, à maintenir également, et difficile à faire évoluer (rigide)
- de plus, le setup des fixtures peut possiblement être coûteux en perfs (mais au pire, on peut utiliser des doubles pour remplacer les vrais objets qui sont coûteux à créer)

mockist = seul le **SUT** doit être un vrai objet, tous les collaborators sont des mocks (et du coup, pas de récursion).


Le truc rigolo, c'est que chaque camp s'accuse de nécessiter trop de boulot :
- les classicistes considèrent que les mockistes doivent implémenter des mocks pour chaque test (au lieu de réutiliser des fixtures)
- les mockistes considèrent que les classicistes ont besoin de setup des fixtures complexes pour pouvoir tester
- (et sur ce point, je suis plutôt d'accord avec les mockistes, je préfère ne pas mutualiser de setup entre mes tests)

### Test Isolation

Ce qui diffère entre mockists et classicists, ce sont les tests qui vont se mettre à échouer quand on introduit un bug dans un module :

- mockist = seuls les tests dont le **SUT** est le module buggé vont se mettre à échouer
- classicist = TOUS les tests qui utilisent le module buggé (soit directement comme **SUT**, soit indirectement comme **collaborator** — i.e. le **SUT** est un client du module buggé, éventuellement récursif) vont échouer

En théorie, c'est très problématique, c'est dû au fait que dans l'approche classique, un même test teste en même temps PLUSIEURS vrais objets (le SUT et les collaborators), ce que ne fait pas l'approche mockist.

En pratique c'est pas trop gênant car un classiciste retrouvera facilement le module qui contient le bug (d'autant plus vrai que la régression fait suite à un seul petit commit unitaire).

Mais c'est tout de même un inconvénient de l'approche classiciste = on a la possibilité de ne faire que des tests grossiers (englobant beaucoup d'objets réels différents) en oubliant de les tester unitairement eux aussi, alors que l'approche mockiste rend facile le fait de se rendre compte qu'on a oublié de tester unitairement un **collaborator**, vu qu'il n'est JAMAIS testé sauf s'il est lui-même le SUT. L'idée est que dans ce cas, le problème vient plutôt une mauvaise mise en place de l'approche classiciste, qui a "oublié" de tester unitairement les collaborators (et ne les a testé que via leur utilisation par un module de plus haut-niveau).

Puis, il dit un truc assez intéressant (que je partage), concernant la dénomination "test unitaire" :

> classic xunit tests are not just unit tests, but also mini-integration tests. As a result many people like the fact that client tests may catch errors that the main tests for an object may have missed, particularly probing areas where classes interact

Déjà, j'aime bien le fait de revenir sur la dénomination "unitaire" (qui reste floue pour moi). De plus, l'idée est que c'est en fait un AVANTAGE de l'approche classiciste qu'il y ait plusieurs tests différents qui échouent quand on introduit un module buggé : en effet, d'une certaine façon, le fait qu'on utilise de vrais objets pour les collaborators permet de détecter des bugs que l'approche mockist (qui n'utilise que des mocks comme collaborators) ne détecte pas.

Autre inconvénient de l'approche mockiste = c'est plus facile de se gourer sur l'expression du comportement à vérifier que sur l'expression de l'état final à vérifier.

Il conclut ce paragraphe en insistant sur le fait qu'il faut associer les tests unitaires (même "unitaires de haut-niveau") avec des tests end-to-end.

### Coupling Tests to Implementations

L'approche mockist fonctionne en mode boîte blanche vis-à-vis du SUT, puisqu'elle vérifie ses appels sortants (pour vérifier qu'elle passe les bons).

Elle est donc couplée à l'implémentation (ce que n'est pas l'approche classiciste, qui ne s'intéresse qu'à l'état final, peu importe comment on l'obtient).

Entre autres défauts, ça complique le refactoring, qui fera péter un test mockist, même si le résultat final est correct.

### Design Style

Point important : quand utilisé avec TDD, le choix de l'approche mockist ou classicist a une influence sur le **design** de l'application !

Déjà, influence illustrée par l'exemple plus haut avec les layers (outside-in en partant de l'utilisation externe vs. middle-out en partant du domaine métier).

D'après lui, les mockists ont tendance à préférer les fonctions qui mutent leur paramètre (vu qu'on peut les mocker pour vérifier le comportement) plutôt que les fonctions qui retournent leur résultat. NdM : sur ce point, je suis super super pas fan de l'approche mockiste... je préfère des fonctions qui n'ont pas de side-effect, et qui se contentent de renvoyer des valeurs.

Autre point mis en avant, lié au principe [Tell Don't Ask](https://martinfowler.com/bliki/TellDontAsk.html) (Tell Don't Ask = plutôt qu'un module expose ses données pour permettre des traitements, il expose plutôt une fonction permettant le traitement directement. Dit autrement : on regroupe le comportement avec les données) : l'approche mockist encourage à vérifier que telle méthode a été appelée (plutôt que de vérifier la valeur de tel résultat), donc encourage le tell don't ask (appel de méthodes plutôt que l'utilisation directe des données).

NdM : mais je suis plutôt d'accord avec cette phrase :

> Classicists argue that there are plenty of other ways to do this.

Problème de l'approche classiciste = parfois, on se retrouve à exposer un état inutile dans l'interface du module, et qui n'existe que pour permettre au test classicist de vérifier l'état (NdM : c'est avéré, j'ai été confronté plusieurs fois à la situation). En théorie, c'est vrai ; en pratique, ces expositions sont généralement mineures.

Autre avantage supposé de l'approche mockiste (NdM : pas super claire à mes yeux) = elle encouragerait l'interface segregation principle (je suppose, en forçant à bien distinguer les différents usages que le SUT fait des collaborators ?)

Enfin, il conclut que ces impacts de l'approche mockiste sur le design ne sont pas anecdotiques : ils sont un argument important des mockistes en faveur de leur approche : "l'approche mockiste est meilleure car elle encourage un bon design".

## So should I be a classicist or a mockist?

Ici, il donne son avis = il est classiciste, car il ne voit pas d'avantage à l'approche mockiste, mais voit un inconvénient de taille :

> A mockist is constantly thinking about how the SUT is going to be implemented in order to write the expectations. This feels really unnatural to me.

(mais il le tempère par le fait qu'il n'a pas beaucoup essayé l'approche mockist...)

NdM : je suis plutôt d'accord, et sa rule of thumb plus haut me va = utiliser l'approche classiciste (avec **test doubles** si besoin) la plupart du temps, et réserver l'utiliation de mocks aux cas qui s'y prêtent mal, comme le `MailSender` ou le cache.

----

Cet [autre post qui n'a rien à voir](https://agilewarrior.wordpress.com/2015/04/18/classical-vs-mockist-testing/) (qui est en fait un post de notes brutes sur l'article de Martin Fowler) a également un conseil intéressant :

> keep your calculations separate from your orchestrations

Avec comme conséquences :

> If you can do this you get the best of both worlds. Calculations can be classically testing. Thin orchestrations mocked. Should keep both set easy to read, and easy to setup.


