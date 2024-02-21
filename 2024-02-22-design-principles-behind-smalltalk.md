# Design Principles Behind Smalltalk

- **url** = https://www.cs.virginia.edu/~evans/cs655/readings/smalltalk.html
- **type** = article
- **auteur** = [Daniel INGALLS](https://fr.wikipedia.org/wiki/Daniel_Henry_Holmes_Ingalls,_Jr.), co-créateur de SmallTalk
- **date de publication** = 1981-08-??
- **source** = [Byte magazine](https://fr.wikipedia.org/wiki/Byte_(magazine)) = un magazine papier des années 1980 dédié aux micro-ordinateurs
- **tags** = language>smalltalk ; topic>language-design ; level>advanced



TL;DR : l'article présente les principes qui guident le design de SmallTalk. Ces concepts orientés objets étaient encore novateurs à l'époque. Quand on parle de "message passing pensé par Alan KAY", c'est de SmallTalk qu'on parle ; l'intérêt pédagogique de l'article est de comprendre cette notion de "message-passing", que j'interprète grosso-modo comme le fait que les méthodes présentent une certaine interface, une signature, et qu'on n'a besoin que de connaître celle-ci pour utiliser la méthode : l'implémentation est cachée, voire privée.

NDM complémentaires :

- le texte parle de smalltalk-80, version standardisée de SmallTalk, sortie en 1980
- en complément, voici ce que [la page wikipédia de SmallTalk](https://fr.m.wikipedia.org/wiki/Smalltalk) en dit :
    > Smalltalk a été d'une grande influence dans le développement de nombreux langages de programmation, dont : Objective-C, Actor (en), Java et Ruby.
    >
    > Un grand nombre des innovations de l'ingénierie logicielle des années 1990 viennent de la communauté des programmeurs Smalltalk, tels que les design patterns (appliqués au logiciel), l’extreme programming (XP) et le refactoring.
- seul le mécanisme de message passing fait partie du langage : même les structures de contrôles du langage sont implémentées avec du message passing :
    > les décisions sont prises en envoyant un message `ifTrue` à un objet booléen, et en passant un fragment de code à exécuter si le booléen est vrai

## Language

> The interaction between two individuals is represented in figure 1 as two arcs. The solid arc represents explicit communication: the actual words and movements uttered and perceived. The dashed arc represents implicit communication: the shared culture and experience that form the context of the explicit communication. In human interaction, much of the actual communication is achieved through reference to shared context, and human language is built around such allusion. This is the case with computers as well.

^ c'est intéressant puisque ça fait un parallèle avec la notion de fonction (où tout est explicite) vs méthode (où il y a un shared contexte partagé).

## Communicating Objects

Il faut un moyen unique de référencer des objets, i.e. un id.

> A way to find out if a language is working well is to see if programs look like they are doing what they are doing. If they are sprinkled with statements that relate to the management of storage, then their internal model is not well matched to that of humans.

^ le programme doit ressembler à ce qu'il veut faire (NDM : ça me fait penser à la _screaming architecture_)

> Smalltalk provides a much cleaner solution: it sends the name of the desired operation, along with any arguments, as a message to the number, with the understanding that the receiver knows best how to carry out the desired operation.

^ par opposition au fait qu'il existe une routine (soit dupliquée dans le code, soit centralisée, mais à laquelle tout le monde, y compris les codeurs débutants, ont accès) qui sait comment "ajouter 5 à un entier", SmallTalk propose plutôt d'envoyer le message "ajoute toi 5" à l'entier, et rien de plus : c'est l'entier lui-même qui sait comment réagir à ce message, en ajoutant 5 à sa propre valeur. C'est le principe d'une méthode, et le "message" est l'interface exposée par cette méthode, sa signature.

> Every object in Smalltalk, even a lowly integer, has a set of messages, a protocol, that defines the explicit communication to which that object can respond. Internally, objects may have local storage and access to other shared information which comprise the implicit context of all communication. For instance, the message + 5 (add five) carries an implicit assumption that the augend is the present value of the number receiving the message

^ principe du **message passing** dont on entend beaucoup parler avec l'orienté objet.

## Organization

> Modularity: No component in a complex system should depend on the internal details of any other component. (...) This principle is depicted in figure 2. If there are N components in a system, then there are roughly N-squared potential dependencies between them. If computer systems are ever to be of assistance in complex human tasks, they must be designed to minimize such interdependence. The message-sending metaphor provides modularity by decoupling the intent of a message (embodied in its name) from the method used by the recipient to carry out the intent. Structural information is similarly protected because all access to the internal state of an object is through this same message interface.

^ pour ne pas faire exploser de façon quadratique la complexité, il faut minimiser les interdépendances entre les modules. Le message passing aide à cela, puisque les modules ne dépendent plus que de l'interface (= le message), et non plus des détails d'implémentation (= comment une méthode implémente l'interface).

(NDM : la clé pour ça = encapsulation + abstraction : me A et le E de APIE utilisé pour représenter l'orienté objet)

>  The complexity of a system can often be reduced by grouping similar components. Such grouping is achieved through data typing in conventional programming languages, and through classes in Smalltalk. A class describes other objects -- their internal state, the message protocol they recognize, and the internal methods for responding to those messages. The objects so described are called instances of that class. Even classes themselves fit into this framework; they are just instances of class Class, which describes the appropriate protocol and implementation for object description.

^ l'une des premières introductions historiques des classes, NDM : qui sont un moyen de répondre au besoin A et E défini juste au dessus.

> Polymorphism: A program should specify only the behavior of objects, not their representation A conventional statement of this principle is that a program should never declare that a given object is a SmallInteger or a LargeInteger, but only that it responds to integer protocol

 ^ est présenté juste après le besoin de polymorphisme (NDM : qui s'accompagne des interfaces, on a bien le P et le I qui manquaient) comme quelque chose permettant notamment d'étendre un système facilement, sans risquer d'introduire de bugs (car on ne touche pas au système qui utilise des objets qui respectent l'interface `IVehicle` : on se contente d'ajouter un nouvel objet de type `IVehicle`)

 Derrière, un message qui a mal vieilli est poussé en avant par l'article = mutualiser le code et les comportements par le truchement de l'héritage.

> Take the case of sorting an ordered collection of objects. In Smalltalk, the user would define a message called sort in the class OrderedCollection. When this has been done, all forms of ordered collections in the system will instantly acquire this new capability through inheritance

 ^ c'est pas hyper clair, mais le message passing me conduit à penser qu'on parle plutôt ici "d'héritage" d'interface, i.e. du fait d'implémenter une interface.

##  User Interface

L'UI est décrite selon le même principe = des trucs qui savent se dessiner.
