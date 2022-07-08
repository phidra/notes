**Note** : zmq n'est pas exclusivement un sujet python (d'ailleurs, c'est [une lib C++](https://github.com/zeromq/libzmq) à la base), mais comme l'utilisation de la lib n'est pas propre à un langage + son utilisation avec [pyzmq](https://github.com/zeromq/pyzmq/) est très facile et rapide, je range ça dans les notes python.

Ces notes vrac font suite à la lecture du [zmq guide](https://zguide.zeromq.org/). Concernant le guide (même si j'ai lu que le début), il est très bien (notamment, la majorité des exemples est multi-langages) mais a quelques défauts :

1. assez touffu = faut être prêt à passer du temps à lire pour retrouver l'info intéressante
2. assez confus = les infos intéressantes sont disséminées un peu partout dans les (longs) paragraphes
3. assez partisan = à la lecture, on comprend clairement que d'après l'auteur, zmq est le sauveur de l'humanité (^_^')

# zmq, c'est quoi, ça sert à quoi ?

TL;DR : zmq permet de faire communiquer deux process sans écrire son prore code réseau, et **sans nécessiter de broker**. En pratique, on manipule un objet `zmq_socket` sur lequel on appelle `zmq_send` d'un côté, et `zmq_recv` de l'autre.

----

Pour faire communiquer deux process/threads, on pourrait écrire son prore code réseau utilisant les sockets TCP, mais dès qu'on s'éloigne des cas basiques, on a vite plein de soucis ([source](https://zguide.zeromq.org/docs/chapter1/#Why-We-Needed-ZeroMQ)) :

- I/O en background
- connexions dynamiques
- gestion des messages, des queues
- gestion des messages perdus
- etc.

Une démarche courante est d'utiliser un broker, mais ça a des inconvénients :

- les protocoles qu'on met en place pour parler au broker ne sont pas forcément bien documentés
- SPOF
- bottleneck
- l'archi nécessite parfois plusieurs brokers, et devient alors complexe
- il faut maintenir et administrer le broker

Du coup, zmq a une position intermédiaire entre :

1. utiliser un broker, et avoir du mal à scaler à cause de ces problèmes
2. ne pas utiliser de broker, mais devoir écrire son propre code réseau, pas robuste et complexe à maintenir

Avec zmq, on utilise une lib qui a déjà adressé les problèmes classique si on écrit son code réseau. Du coup, elle **impose** les bons patterns pour communiquer :

- pub/sub
- req/rep (= client/server, RPC, ...)
- push/pull (= pipeline)

Et des patterns plus évolués peuvent être construits sur ces patterns de base.

zmq vise la simplicité d'utilisation : pas de dépendance, une seule lib à linker, ça marche out-of-the-box. Je remets une partie la (longue) liste de features du guide :

> - It handles I/O asynchronously, in background threads. These communicate with application threads using lock-free data structures, so concurrent ZeroMQ applications need no locks, semaphores, or other wait states.
> - Components can come and go dynamically and ZeroMQ will automatically reconnect. This means you can start components in any order. You can create “service-oriented architectures” (SOAs) where services can join and leave the network at any time.
> - It queues messages automatically when needed. It does this intelligently, pushing messages as close as possible to the receiver before queuing them.
> - It has ways of dealing with over-full queues (called “high water mark”). When a queue is full, ZeroMQ automatically blocks senders, or throws away messages, depending on the kind of messaging you are doing (the so-called “pattern”).
> - [...]
> - It lets you route messages using a variety of patterns such as request-reply and pub-sub. These patterns are how you create the topology, the structure of your network.
> - [...]
> - It handles network errors intelligently, by retrying automatically in cases where it makes sense.
> - [...]

# Notes et caveats

- on peut lancer les publishers/subscribers (ou requester/replyer) dans n'importe quel ordre, zmq se débrouille pour faire communiquer tout le monde, quitte à faire patienter les messages dans les queues en attendant
- du coup, zmq sait se reconnecter tout seul p.ex. si un publisher a été temporairement déconnecté
- côté patterns :
   - REQ/REP = modèle client-serveur
   - PUB/SUB = modèle publisher-subscriber
   - PUSH/PULL = modèle où on distribue (PUSH) ou collecte (PULL) des tâches vers/depuis un set de workers
- pour le moment, je n'ai pas bien compris la différence entre `bind` et `connect` ; `bind` doit être le plus stable = le "server", mais faudra que je revienne sur le sujet à l'occasion
- caveat PUB-SUB = **slow-joiner** = lorsqu'on démarre un publisher sur un subscriber déjà lancé, il faut un certain temps pour que la connexion se mette en place → les premiers messages publiés seront perdus à moins qu'on patiente un peu avant de les envoyer (cf. [ma POC](https://github.com/phidra/pocs/tree/ed7ea0a2da36ecc732b4cb8365884896ad1a33a9/python/LIB_zmq_04_slowjoiner))
- caveat PUB-SUB = **topics** = par défaut, si un subscriber s'abonne à un publisher, il ne reçoit rien ! Il doit préciser le topic (en vrai, le préfixe des messages) qui l'intéressent avec `zmq_socket.setsockopt(zmq.SUBSCRIBE, topic)` pour recevoir les messages (cf. [ma POC](https://github.com/phidra/pocs/tree/ed7ea0a2da36ecc732b4cb8365884896ad1a33a9/python/LIB_zmq_02_pubsub_topics_notopic))
- caveat PUB-SUB = **subscribe pour recevoir un message sans le pourrir avec le topic** = pour s'abonner à un publisher en utilisant un topic particulier, mais ne pas à avoir à préfixer les messages qu'on envoie avec le topic, il suffit d'envoyer le couple {topic+message} en multipart (cf. [ma POC](https://github.com/phidra/pocs/tree/d21ac21ad4e268a0c01ba3e13f7db9d9885ed310/python/LIB_zmq_06_sending_topic_as_multipart))
- caveat PUSH-PULL = différence entre **round-robin** et **fan-out** = à chaque fois qu'on a un message à envoyer, on l'envoie :
   - round-robin = à UN SEUL destinataire, qui change à chaque message de façon égalitaire
   - fan-out = à TOUS les destinataires en même temps
