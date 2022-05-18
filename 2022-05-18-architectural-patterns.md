# Architectural Patterns

- **url** = https://www.ou.nl/documents/40554/791670/IM0203_03.pdf/ ([copie locale](./LOCALCOPIES/IM0203_03.pdf), md5sum=`ba983ccb8dfc9a231ab13200473e85bc`)
- **type** = cours PDF, semble être un morceau d'un [cours plus général intitulé Software Architecture](https://www.ou.nl/en/-/software-architecture), dont [ce lien](https://www.ou.nl/documents/40554/791670/IM0203_co.pdf/) donne la structure
- **auteur** = ?
- **date de publication** = ?
- **source** = [Open University](https://www.ou.nl/en/over-ons) = université online belge et néerlandaise.
- **tags** = language>none ; topic>architecture ; topic>architecture-patterns ; level>intermediate

**TL;DR** : liste de façon succinte des patterns architecturaux, en donnant des exemples. Pas inintéressant, mais je m'attendais à beaucoup mieux au vu du TOC.

* [Architectural Patterns](#architectural-patterns)
   * [Intro](#intro)
   * [Layers pattern](#layers-pattern)
   * [Client-Server Pattern](#client-server-pattern)
   * [Master-Slave Pattern](#master-slave-pattern)
   * [Pipe-Filter pattern](#pipe-filter-pattern)
   * [Broker pattern](#broker-pattern)
   * [Peer-to-Peer pattern](#peer-to-peer-pattern)
   * [Event-Bus pattern](#event-bus-pattern)
   * [Model-View-Controller pattern](#model-view-controller-pattern)
   * [Blackboard pattern](#blackboard-pattern)
   * [Interpreter pattern](#interpreter-pattern)
   * [Reste du PDF](#reste-du-pdf)

## Intro

Différence entre design-pattern (réponse à un problème spécifique, granularité = classe) et architectural-pattern (orgnaisation d'un système, granularité = module)

## Layers pattern

**Principe** = organiser le code en couches :

- une couche supérieure utilise une couche inférieure (et en a donc connaissance puisqu'elle en dépend)
- une couche inférieure n'utilise jamais une couche supérieure (elle n'en dépend pas)

Exemple : la couche TCP peut être utilisée inchangée à la fois par telnet et par ftp.


- note = comme personne ne dépend des couches supérieures, un changement dans les couches supérieures n'impacte personne ; à l'inverse, un changement dans les couches inférieures impacte tout le monde
- **avantage** = mutualisation des couches inférieures (qui peuvent être utilisées par plusieurs couches supérieures différentes)
- **avantage** = on se force à définir proprement les interfaces des couches inférieures
- **avantage** = la couche supérieure ne dépend que de l'INTERFACE de la couche inférieure -> on peut en modifier l'implémentation (possiblement par une autre team)
- **inconvénient** = si on doit changer la couche inférieure, faut changer tout le monde au dessus
- **inconvénient** = la couche inférieure pourra faire du travail inutile pour la couche supérieure (NdM : par exemple, si la couche inférieure récupère tout plein d'infos sur un fichier, et que la couche supérieure l'utilise pour connaître sa taille, la couche inférieure aura par exemple récupéré les permissions du fichier pour rien)

VARIANTE = Relaxed Layered System = idem, mais une couche supérieure a accès à TOUTES les couches inférieures, et pas uniquement celles qui sont immédiatement en desouss d'elles.

VARIANTE = la couche supérieure peut passer une callback à la couche inférieure, afin de permettre la communication d'une couche inférieure vers la couche supérieure.

## Client-Server Pattern

**Principe** = _a server component provides services to multiple client components ; a client component requests services from the server component. Servers are permanently active,_

Exemple : remote database access (psql est le client, postgresql est le serveur), web-based applications (browser est le client, application tornado est le server)

- mécanisme d'IPC nécessaire (dans le web : réseau), puisque client et serveur vivent dans des process différents, voire des machines différentes
- on peut voir client-server comme un layered system à deux couches : client = high layer, server = low layer
- **inconvénient** = Requests are typically handled in separate threads on the server.
- **inconvénient** = il faut passer par une sérialization des données pour l'envoyer du client au serveur (ou le contraire)
- **inconvénient** = devoir connaître le serveur n'est pas terrible, idéalement, on préfère que le client tape sur un service "générique" sans qu'il sache exactement quelle machine lui répond
- **inconvénient** = parfois, le serveur aimerait notifier le client

- note = sessions :
    + si les requêtes ne sont pas indépendantes, il y a session, avec deux cas
    + cas 1 = stateful server : c'est le serveur qui a connaissance de l'état, et le client fournit à chaque requête un cliendid pour que le serveur retrouve la session du client (inconvénient = ne scale pas très bien quand il y a beaucoup de clients)
    + cas 2 = stateless server : c'est le client qui a connaissance de l'état, et qui l'envoie au serveur à chaque requête (inconvénient = si le client perd son état, il perd définitivement sa session + security issues)

VARIANTE = REST = Representational State Transfer : _A REST architecture is a client-server architecture where clients are separated from servers by a uniform interface, servers offer addressable resources and communication is stateless.  A server may be stateful, but in that case, each server-state should be addressable (for instance, by a URL)_

## Master-Slave Pattern

**Principe** = _The master component distributes the work among identical slave components and computes a final result from the results the slaves return._

NdM : tel que présenté, dans ce pattern, plusieurs slaves différents effectuent le même calcul (à charge pour le master de  réconcilier les différents résultats, e.g. en choisissant le résultat obtenu le plus rapidement, par la majorité, ou encore la moyenne)

Utilisé p.ex. dans des systèmes qui doivent être fault-tolerants ou des systèmes fortement parallèles. Autre utilisation = distribuer un calcul lourd entre différents workers (NdM : pour moi, on est dans deux usages assez différents : l'un redondé, l'autre non). Dernier exemple d'utilisation (douteuse, IMO) = faire faire plusieurs fois le même calcul à un même service implémenté différemment, pour avoir une meilleure précision.

- note : les slaves sont indépendants (ils ne dépendent de personne, pas même du master, vu que celui-ci est à l'origine de la requête)
- **avantage** = _The Master-Slave pattern supports fault tolerance and Master-Slave pattern parallel computation._
- **inconvénient** = c'est fault-tolerant si le SLAVE a une erreur, mais pas si le master échoue
- **inconvénient** = ne peut s'appliquer que si le problème est décomposable

## Pipe-Filter pattern

**Principe** = la donnée passe dans des filtres successifs. Bien adapté à des streams de données.

- **avantage** = permet facilement le parallélisme puisqu'un filtre peut commencer à bosser avant même que le précédent ait fini de tout traiter :
    + _A filter consumes and delivers data incrementally (another name for the same concept is lazily):_
    + dès qu'un filtre reçoit des données, il traite (et le reste du temps, il ne fait rien)
    + du coup, on peut traiter chunk par chunk, ce qui permet de facto la parallélisation
    + par exemple, un filtre qui mettrait du texte tout en majuscule peut traiter au fur et à mesure que le fichier d'entrée est lu
- **avantage** = permet facilement la modularité, en ajoutant/supprimant des filtres
- **avantage** = permet facilement la réutilisation des filtres
- **avantage** = les filtres peuvent être développés de façon indépendante (donc possiblement par plusieurs équipes) si tant est que leur interface est claire
- **inconvénient** = possible overhead de conversion, car les filtres doivent accepter des données à leur format attendu + les produires au format souhaité :
    + _when the input and the output have the form of a string, for instance, and filters are used to process real numbers, there is a lot of overhead from data transformation_
- **avantage** = les filtres se comportent de façon indépendante les uns des autres, ils n'ont pas à partager d'état
- **avantage** = modèle conceptuel simple
- **avantage** = comportement prévisible en terme de perfs (on aura les perfs du filtre le plus lent)
- **inconvénient** = certains filtres cassent cet aspect incrémental, en nécessitant l'ensemble des données d'un coup (e.g. un filtre qui trie les données)
- **inconvénient** = pas facilement utilisable pour des applis interactives
- **inconvénient** = NdM = pas toujours faciles d'accéder à l'état intermédiaire

## Broker pattern

**Principe** = quand les différents composants d'un système se connaissent les uns les autres, c'est pas très flexible. À l'inverse, avec le Broker Pattern, chaque composant ne connaît que le broker, publie un message sur le broker, et lit la réponse sur le broker :

> A broker component is responsible for the coordination of communication among components: it forwards requests and transmits results and exceptions

- Sur le principe, les services disponibles sont déclarés dynamiquement au broker :
    + _Servers publish their capabilities (services and characteristics) to a broker. Clients request a service from the broker, and the broker then redirects the client to a suitable service from its registry_
    + le broker maintient donc un registry des serveurs qu'il connaît et des services que ces serveurs proposent
- (stricto-sensu, ce n'est pas obligatoire, on peut hardcoder qu'il faut requêter telle queue pour obtenir tel service)
- **avantage** = seul le composant broker doit connaître des détails de communication IPC (les autres composants peuvent se contenter de connaître les détails permettant la communication avec le broker)
- note = on peut utiliser un IDL = interface definition language pour qu'un serveur déclare au broker les services qu'il est capable de fournir (et que le broker les trouve donc automatiquement). e.g. CORBA
- **avantage** = les services accessibles sont transparents pour le développeur
- **avantage** = c'est dynamique : les services accessibles via ce broker peuvent évoluer (ajout, suppression, modification)
- note : deux brokers peuvent coopérer (le premier broker peut forwarder au second broker une requête demandant un service précis)

## Peer-to-Peer pattern

**Principe** = _The Peer-to-Peer pattern can be seen as a symmetric Client-server pattern: peers may function both as a client, requesting services from other peers, and as a server, providing services to other peers._

Exemple : file-sharing bittorrent

- comme la communication est bidirectionnelle, un "server" peut prendre l'initiative de notifier un "client" (ce qui n'est pas facile en modèle client-server classique)
- **avantage** = un node d'un système peer-to-peer peut profiter de l'ensemble du système, en ne lui apportant que sa propre capacité
- **avantage** = peu d'administration, le système est auto-organisé (et la configuration peut être dynamique)
- **avantage** = résistant à la failure d'un node individuel + scalable
- **inconvénient** = pas de garantie de qualité de service ou de sécurité
- **inconvénient** = s'il y a peu de participants, les perfs ne seront pas bonnes

## Event-Bus pattern

**Principe** = les sources publient des évènements sur un channel particulier d'un event-bus, les listeners écoutent les channels, et sont notifiés quand un évènement se produit.

- **avantage** = la génération du message et sa notification sont asynchrones : le générateur se contente de produire le message et continue son taf sans attendre : il se fiche qu'il soit reçu par 0, 1 ou 999 listeners.
- **avantage** = dynamique, on peut ajouter de nouveaux publishers, subscribers, ou types d'évènements
- **inconvénient** = il devient compliqué de maîtriser quand les messages sont distribués (on n'a pas toujours de garantie de préservation de l'ordre des messages, ni même de délivrance)
- **inconvénient** = l'event-bus centralise tous les messages -> il devient un bottleneck qui peut empêcher la scalabilité.

## Model-View-Controller pattern

**Principe** = concerne les applications interactives, et les découpe en 3 morceaux :

- model = data et fonctionnalités du coeur de métier
- view = affichage de l'information à l'utilisation
- controller = gestion des inputs de l'utilisateur

Concrètement :
- le controller reçoit les inputs de l'user, et mets à jour le modèle
- le changement de modèle (soit par le controller, soit par une autre cause) trigge la mise à jour de la vue
- la vue est capable de requêter le modèle pour choisir comment s'afficher

- **avantage** = la vue est découplée du reste -> on peut en avoir plusieurs différentes
- **inconvénient** = tous les éléments visibles ne se découpent pas très bien en model+view+controller
- **inconvénient** = possiblement, beaucoup d'updates inutiles
- **inconvénient** = tout ce beau monde reste assez couplé : ajouter un champ email nécessite de modifier la view (pour le montrer), le controller (pour réagir aux inputs sur ce email), et le modèle (pour stocker l'email)

## Blackboard pattern

**Principe** = pas hyper clair, on dirait que le principe est que quand on ne peut pas trouver une réponse parfaitement déterministe, plusieurs sous-systèmes mettent en commun leur connaissance pour construire la meilleure réponse possible.

- blackboard = shared data store
- les exemples donnés sont : speech recognition, submarine detection, etc.
- **avantage** = facile d'ajouter des application ou d'étendre les données gérées.
- **inconvénient** = modifier des données partagées est difficile, car elles peuvent être déjà utilisées par des applications. Plus généralement, les process doivent se mettre d'accord (synchronisation ?) sur les données dans le blackboard.

## Interpreter pattern

**Principe** = un interpréteur interprète un langage (e.g. la VM python qui interprète le bytecode)

- **avantage** = le "programme" interprété par l'interpréteur peut être remplacé facilement

NdM : je ne comprends pas trop pourquoi ce "pattern" est inclus aux patterns architecturaux, pour moi ça n'a pas grand chose à voir avec la façon d'architecturer un système en plusieurs composants...

## Reste du PDF

La suite du doc est plutôt intéressante (quoique moins qu'espérée, ici aussi) et montre comment adresser un problème concret selon différents patterns architecturaux.

Le cas d'application concrète est [KWIC = Keyword In context](https://en.wikipedia.org/wiki/Key_Word_in_Context), ce qui est apparemment un problème classique :

- en entrée, on a un fichier avec plusieurs lignes ; chaque ligne contient une liste de mots
- en sortie, on veut une liste avec M lignes : chaque ligne en entrée produit plusieurs lignes en sortie (les circular shift)
- p.ex. les circular shift de la ligne "man eats dog" sont : "dog man eats / eats dog man / man eats dog"
