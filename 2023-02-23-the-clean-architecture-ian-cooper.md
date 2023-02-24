# The Clean Architecture — Ian COOPER

- **url** = https://youtu.be/SxJPQ5qXisw
- **type** = vidéo
- **auteur** = [Ian COOPER](https://github.com/iancooper), dev avec 20 d'expérience, particulièrement en .NET
- **date de publication** = 2019-12-20
- **source** = [DevTernity Conference](https://www.youtube.com/@DevTernity/about), _Turning developers into architects and engineering leaders_
- **tags** = language>agnostic ; topic>architecture ; topic>clean-architecture ; level>intermediate

**TL;DR** = une présentation de la clean architecture, avec notamment une progression logique depuis une layered architecture.

04:00 la clean architecture n'est pas particulièrement novatrice : c'est une synthèse d'idées existantes.

Sommaire :

-  Layer
-  Domaine model
-  Clean Architecture
-  Implementing

05:30 layer = les couches du dessus utilisent les couches du dessous.

06:10 exemple : UI layer qui utilise domaine model layer qui utilise infrastructure layer (parfois appellée data layer) ; mais ce n'est que l'exemple le plus répandu : on peut avoir autant de layer qu'on veut, et les nommer comme on veut.

Chaque layer expose une **façade** présentant ce qu'elle offre à la couche du dessus.

07:30 on ne peut dépendre que des couches du dessous.

09:30 intérêt d'une archi avec des layers :

- On peut se limiter à ne s'intéresser qu'à l'implémentation d'un unique layer (en se contentant de ne connaître les autres layers que sous forme d'abstraction = les façades qu'elles présentent)
- On peut switcher d'implémentation pour un layer sans impacter les autres layers (c'est le pouvoir des interfaces)
- On peut retarder les décisions sur les implémentations des layers sous-jacents
- On limite les dépendances entre layers

12:10 j'aime bien cette citation :

> change is quite constant in a long lived system

13:00 couplage (on veut le RÉDUIRE) :

> coupling is the property that one module is forced to change because another does.

Avec des layers, le couplage est limité à la jonction entre deux layers, la fameuse façade.

13:00 cohésion (on veut l'AUGMENTER) :

> cohesion is the property that a module is subject to the same forces of change

Les layers permettent de réduire les couplages et d'augmenter la cohésion.

15:00 ce qu'on appelle **business logic** est splitté en deux :
- **Domain Logic** = les règles du métier (e.g. le calcul d'un tarif d'assurance en fonction des infos du client), au sujet desquelles on peut interroger un expert du domaine.
- **Application Logic** = le workflow propre à son application, e.g. pour calculer le tarif, il faut commencer par aller chercher dans la base les infos du client pour savoir s'il fume, puis calculer le tarif, puis remplir telle page visualisée et envoyer un email, puis etc.

NdM : ma compréhension, c'est que deux applications différentes pourront éventuellement utiliser un workflow différent, mais elles calculeront le tarif de la même façon → l'application logic est propre à l'application, alors que le domain logic est propre au domaine quelle que soit l'application.

16:30 = il est courant de splitter le layer domaine en deux : **service** et **entity**, et on a deux implémentations courantes de ce splitting :

Façon 1 de le faire = ce sont les entities qui possèdent toute la logique, le service layer n'est qu'une fine façade permettant de manipuler les entities.

Façon 2 = _opération script_ = le service layer fait le gros du taf, car les entities n'ont pas spécialement de logique, elles se contentent de stocker un état (c'est le cas par exemple d'une application qui fait simplement du CRUD sur les données).

19:15 il passe à ports-and-adapters architecture, et a un point de vue intéressant :

- l'archi hexagonale est à voir comme un exemple particulier de layered architecture
- la couche que l'archi hexagonale appelle "application" est mal nommée, car on a tendance à comprendre le terme "application" comme "l'ensemble de ce qu'on a codé", alors que dans ce cas, il ne s'agit "que" du domain model.
- la couche adapters parle au monde extérieur
- la couche port est entre les deux, et c'est la couche qu'utilisent les adapters pour parler à la couche domaine (= "l'application").
- les "ports" s'appellent ports, car c'est ce que quoi on va plugger, brancher quelque chose, ce quelque chose étant les adapters.

NdM : dans son point de vue, il ne distingue pas particulièrement driver-port and driven-port.

23:30 = à la différence d'une layered architecture (où la communication entre deux couches adjacentes est bidirectionnelle), dans l'archi hexagonale, les dépendances sont dans un seul sens = vers le centre (les ports ne dépendent pas des adapters, c'est le contraire).

Pour cela, il est nécessaire de faire de l'inversion de dépendance.

28:30 use case diagram. L'idée est de revenir aux cas d'utilisation de l'app, et par qui. Exemple avec les cas d'utilisation dans un restaurant, avec 3 users : client, waiter, cashier

NdM : la notion de **use-case** est particulièrement importante.

29:40 il dit un truc proche de la screaming architecture = ce sont les use cases (et non les contraintes techniques telles que le framework utilisé) qui doivent guider la façon dont on organise le code.

30:00 boundary controller entity = autre façon d'architecturer le code qui suit des principes similaires à la clean architecture / ports-and-adapters architecture :
- Boundary = un objet qui s'interface avec un **acteur** (NdM = le monde extérieur) : ui, proxy, gateway, ...
- Entity = objets représentant les system data : customer, transaction, ...
- Controller (=interactor) = interface entre les deux = il orchestre l'exécution des commandes en provenance des boundaries.

Ce sont dans les controllers qu'on trouve les use cases, apparemment : leur implémentation SONT les use cases.

32:00 Alistair Cockburn a dit que les ports sont là où on représente les uses case 

33:00 les tests sont un autre adapter qui drivent le port = le use case. Au lieu de driver l'application par une ui, on la drive par le test :

> tests should focus on the behaviour expressed by a use case, not a unit of code

33:30 n'écrivez pas des tests qui ont besoin de votre framework : non seulement si on change de framework les tests sont à jeter, mais surtout on rend plus difficile la compréhension de l'objectif du test = tester les use cases.

34:00 on en arrive à la clean architecture, en couches concentriques :
- Centre = entities = domain logic business rules
- Puis = use cases = application logic business rules
- Puis = controllers = adapters = ce qui drive l'application (driver) ou qui présente les résultats (driven)
- Puis = framework et driver = le code qui gère p.ex les devices, la DB, l'ui, ...

Les dépendances vont toutes dans le même sens = vers le centre.

Le gros intérêt : toutes les couches business sont sans IO → elles sont faciles à tester.

39:00 commande pattern + command dispatcher (je skimme)

41:30 on implementing = exemple concret en .NET (je skimme)

51:30 le key-takeaway de la clean architecture = séparer l'application domain code (et les use-cases qui sont autour), des préoccupations liées à l'infrastructure ou aux IOs.
