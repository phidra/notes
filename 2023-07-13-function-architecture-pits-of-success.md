# Functional architecture - The pits of success

- **url** = https://www.youtube.com/watch?v=US8QG9I1XW0
- **type** = vidéo
- **auteur** = [Mark SEEMANN](https://blog.ploeh.dk/about/) aka [ploeh](https://github.com/ploeh), auteur prolifique : _Helps programmers make code easier to maintain._
- **date de publication** = 2016-08-26
- **source** = [NDC Conferences](https://www.youtube.com/@NDC) = conférence initialement .NET, maintenant généraliste
- **tags** = language>csharp ; language>haskell ; topic>functional-programming ; topic>architecture-hexagonale ; topic>testing ; topic>entity-services ; level>intermediate

Le message principal de la prez est : la programmation fonctionnelle aide aux bonnes pratiques, avec trois exemples concrets :

- **archi hexagonale** (car les fonctions faisant des I/O appellent des fonctions pures et pas le contraire, donc les fonctions avec I/O sont forcément sur les couches externes de l'archi)
- **entities services** (car la séparation entre data d'un côté et comportements de l'autre est inhérent à la FP)
- **testing** (car les fonctions pures sont naturellement isloées de leurs dépendances, donc testables facilement)
- (il y en a d'autres, mais ce sont les plus importantes selon lui ; EDIT : il en reparle [dans un post de son blog](https://blog.ploeh.dk/2023/03/27/more-functional-pits-of-success/))

Overall une très très bonne prez à mes yeux ! Notamment des exemples très bien craftés pour rester simples tout en étant illustratifs.


* [Functional architecture - The pits of success](#functional-architecture---the-pits-of-success)
   * [Introduction](#introduction)
   * [PARTIE 1 = PORTS AND ADAPTERS](#partie-1--ports-and-adapters)
   * [PARTIE 2 = SERVICES AND DATA](#partie-2--services-and-data)
   * [PARTIE 3 = Testability](#partie-3--testability)

## Introduction

8 premières minutes du talk

Langages du talk = F# et Haskell (mais pas vraiment besoin de savoir coder dans ces langages)

Objectif = avoir du code toujours maintenable même trois ans plus tard (plutôt que de tout réécrire tous les trois ans) (NDM = architecture logicielle).

Il prend une image intéressante avec un rocher au sujet d'une colline (équilibre instable) vs au fond d'un creux (pit) = équilibre stable : ce qu'on veut en tant que tech lead, c'est un équilibre stable = on ne veut pas que la qualité du code se dégrade dès que le tech-lead a le dos tourné ou est en vacances (ce qui est un "équilibre" instable), on veut plutôt que la qualité du code reste constante même si le tech-lead n'est pas là (équilibre stable).

Ses trois outils qu'il va détailler dans la prez :

- Ports and adapter
- Services, Entités, Value objects
- Testability

## PARTIE 1 = PORTS AND ADAPTERS

[08:20](https://youtu.be/US8QG9I1XW0?t=500)

11:40 pour filer la métaphore avec l'équilibre instable : c'est problématique d'avoir besoin que tous les membres de l'équipe doivent connaître et maîtriser l'archi hexagonale afin de l'utiliser et voir la qualité du code s'améliorer.

Par exemple, si le tech lead part en vacances et que l'équipe ne maîtrise pas le sujet, quelqu'un va se mettre à appeler la database directement depuis la couche business-model = depuis l'hexagone (au lieu de le faire depuis un adapter).

12:50 selon lui, le caractère instable de l'archi hexagonale est surtout vrai pour les langages OO, et moins vrai pour les langages fonctionnels (NDM = c'est pas la première fois que je vois l'idée que le business model = l'hexagone devrait être codé en paradigme fonctionnel).

18:00 il donne un exemple de ce qui va être son fil rouge = app e réservation de restaurant avec 5 étapes :

- validation de la requête de réservation = pure
- requêter la DB pour connaître le nombre de siègres déjà réservés = impure
- décider s'il y a assez de places libres pour accepter la requête = pure
- enregistrer la réservation dans la DB = impure
- créer une réponse à renvoyer au client = pure

Ce qui est intéressant : certaines étapes sont pures, d'autres impures.

Derrière, il donne ses 5 fonctions (ou juste leur signature) en Haskell pour expliquer la syntaxe des fonctions pures/impures.

NDM = rien à voir, mais ici aussi il appelle le tuple vide "unit" : cette dénomination n'est donc pas spécifique à rust.

25:00 sa dernière fonction est juste la traduction d'un état interne ("a-t-on accepté la réservation ?") en une réponse HTTP. NDM = cette fonction irait dans un adapter en archi hexagonale.

26:30 il a également une fonction supplémentaire (impure) de haut niveau qui fait l'ordonnancement de ses 5 fonctions (NDM = lien avec le `main` qui est souvent le grand oublié des explications sur l'archi hexagonale).

29:00 il en arrive à des cercles concentriques, avec les fonctions pures au centre , et le point important : c'est **obligatoirement** dans ce sens que les fonctions sont organisées (= les fonctions impures dessinées à l'extérieur car elles appellent (=ont connaissance de) les fonctions pures, ces dernières étant dessinées à l'intérieur, car elles n'ont pas connaissances des fonctions impures), car Haskell ne compilea pas le programme sinon (car une fonction pure n'a pas le droit d'appeler une fonction impure).

^ c'est donc l'explication à son assertion "la programmation fonctionnelle rend plus facile le fait de respecter l'archi hexagonale".

> IO happens at the boundary of the system, because Haskell forces you this way

31:45 du coup, il peut partir en vacances sereinement : personne ne va appeler la database depuis le code du business model, car c'est rendu impossible par la programmation fonctionnelle (à moins de transformer une fonction pure en fonction impure).

## PARTIE 2 = SERVICES AND DATA

[33:30](https://youtu.be/US8QG9I1XW0?t=2010)

Pour beaucoup de gens, la POO, c'est "data with behaviour + encapsulation (= ne pas exposer les rouages internes)"

Il prend un exemple = une classe User, qui encapsule des attributs (e.g. name et id), et des behaviour (e.g. les fonctions CRUD pour interagir avec la BDD, mais également une fonction pour envoyer un email, etc.)

Le design pattern Active Record (= on mélange dans une classe les attributs qui constituent l'objet, et les fonctions permettant d'interagir avec, notamment toutes les fonctions CRUD pour interagir avec la BDD) est souvent enseigné comme une bonne pratique, mais c'est un reliquat du passé, et un antipattern. Notamment, on se retrouve à mettre dans l'objet beaucoup, beaucoup de behaviour, elle devient de moins en moins cohésive (et n'est plus maintenable).

Il fait beaucoup référence au beaucoup de DDD d'Eric EVANS.

36:50 il y a trois catégories catégories d'objets :

- **Entities** : c'est l'objet qu'on retrouve dans les cours de POO, qui modélise le monde réel. Ils ont une long lasting identité : User, Contract, Order...
- **Value Objects** : ils sont des simples containers de données, mais ils n'ont aucune business value. Si on y place de la business value, ça se passe mal.
- **Services** = à la place, il faut mettre la business value dans des classes qui sont des collections de behaviour = des services (qu'on connait parfois sous le nom de "Manager")

Il reprend son exemple de classe User, en essayant de la découper selon ces catégories d'objets :

- une entity `User` n'ayant que les attributs (id et name)
- un service ` SqlRepository ` pour les fonctions CRUD d'un user dans la BDD
- un service `EmailSender` pour envoyer un email à un `User`

Note : les fonctions de ces services prennent et renvoient des instances de `User` en input/output .... Mais elles sont séparées de la classe `User`.

40:00 si on veut une codebase maintenable sur plusieurs années, c'est ce découpage qu'il faut suivre selon lui. (Certaines personnes disent que ce n'est plus de la POO).

41:00 et ici aussi, la programmation fonctionnelle aide à avoir ce découpage propre car elle sépare naturellement data et fonctions. On retrouve l'équilibre stable de "personne ne risque de mélanger data et comportement en mon absence".

Il donne des exemples concrets de son user en F#

## PARTIE 3 = Testability

[43:30](https://youtu.be/US8QG9I1XW0?t=2610)

1995 = Kent Beck introduit TDD, et déjà à l'époque (et encore maintenant), c'est un sujet controversé...

45:00 il cite un autre gus qui dit : "TDD apporte du Test Induced Damage" (EDIT : c'est à [cet article](https://dhh.dk/2014/test-induced-design-damage.html) qu'il fait référence ; TL;DR = si ce sont les tests qui guident le design, on se retrouve à faire des trucs aberrants par rapport à notre produit ; à la place, il faut garder le produit qui guide le design, et adapter les tests en fonciton).

Il prend un exemple d'une classe (qui devrait dès le début être une fonction IMO, car elle n'a qu'une méthode utile ; mais peut être que C# n'autorise que les classes ? Ou peut être que comme l'un des arguments de la fonction est elle même une fonction (pour requêter le repository de réservation), on considère que c'est mieux d' utiliser une classe ?) et montre que la transformer en fonction (dont l'un des arguments est une fonction) est mieux.

Au passage, le comportement est polymorphique sur la fonction passée en argument : toute fonction qu'on peut utiliser pour récupérer un nombre de sièges entier a partir d'une date sera acceptée comme higher-order function (c'était le cas aussi pour le code C# via du nominal subtyping = l'interface `IReservationRepository`)

Le caractère pur ou impur de `getReservedSeats` ne dépend que de la higher order function passée en argument pour interroger la database au sujet du nombre de seats disponibles : si celle-ci est pure, `getReservedSeats` est pure (et vice versa).

L'intérêt ? En mockant la higher ordre function avec une fonction pure, `getReservedSeats` devient pure, et est donc facilement testable.

50:00 pure functions are always testable.

50:30 il donne un exemple de test sur cette fonction, qui dans ce cas est pure car la higher order function qu'on lui passe est pure.

52:00 les fonctions pures sont toujours facilement testables, car elles respectent toujours L'ISOLATION (= la seule info que la fonction a sur le monde extérieur provient de ses arguments). Certaines fonctions impures peuvent avoir l'isolation aussi, mais pour les fonctions pures, c'est systématiquement le cas.

54:00 pourquoi c'est important pour une fonction d'avoir l'isolation ? Parce que le principe même des unit tests est de tester un bout de code de façon indépendante de ses dépendances.

55:00 son message est donc que si on part du principe que l'isolation est une bonne chose pour le unit testing, alors le principe même des méthodes d'une classe en POO est mauvais, car elle reçoit de l'information par d'un instance en plus de ses arguments.

Derrière, il montre un diagramme de Venn entre encapsulation et isolation : le recouvremnet est petit. Son interpétation = en POO, il est possible d'avoir des fonctions qui respectent l'isolation, mais c'est compliqué, et ça nécessite beaucopu de travail.

En FP, a contrario, toutes les fonctions pures ont l'isolation : le recouvrement est total.

> Just write as many pure functions as you can
