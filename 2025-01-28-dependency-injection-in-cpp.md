# Dependency Injection in C++ - A Practical Guide - Peter Muldoon - C++ on Sea 2024

- **url** = https://www.youtube.com/watch?v=_UTgOC6jW8o
- **type** = vidéo
- **auteur** = [Peter MULDOON](https://meetingcpp.com/2024/Speaker/items/Peter_Muldoon.html), Senior Ingeneering Lead chez Bloomberg
- **date de publication** = 2024-11-16
- **source** = chaîne YouTube de [C++ on Sea](https://www.youtube.com/@cpponsea)
- **tags** = language>C++ ; topic>dependency-injection ; level>intermediate

**TL;DR** = très intéressante prez sur l'injection de dépendance, qui déborde sur ce qui va avec : testabilité + bonne architecture de son code

Première demi-heure = revue des bases de **DI = Dependency Injection**

5 façons de faire de la DI :

- linktime (à éviter, pas pratique de recompiler partout où on veut injecter)
- virtual functions/interfaces
- templates
- type erasure via std::function
- null objets

Concrètement, comment injecter la dépendance :

- **Setter** = une méthode de notre classe permet de setter la dépendance, dans un membre (à éviter, car on modifie l'interface de la classe juste pour le test)
- **Argument supplémentaire** = ajouter la dépendance comme argument de la méthode qui utilise la dépendance ; inconvénient = ça change la signature de la méthode
- **À la construction** = on injecte la dépendance à la construction ; inconvénient = ça change la signature du constructeur

Deuxième demi-heure : real world = quelques conseils pour se faciliter la DI dans la vraie vie :

- **Law of Demeter** = ne parler qu'à son voisin immédiat : si une méthode utilise une sous-classe d'une sous-classe d'un de ses attributs, lorsqu'on va vouloir stubber l'attribut pour un test, il va falloir stubber plusieurs niveaux en profondeur, ce qui a beaucoup nous compliquer la vie.
- **éviter l'entanglement** où on mélange les lectures et écritures sur une classe ; sinon on ne sait plus comment elle est utilisée et donc comment on devra la mocker
- **limiter le nombre de dépendances** : sinon on doit mocker 4000 trucs pour pouvoir tester sa classe
    - à noter qu'agréger toutes les dépendances d'une classe dans une unique structure n'est pas une solution : ça ne change pas grand chose
    - si on doit regrouper des dépendances, il faut le faire _parce que ça a du sens du point de vue fonctionnel_
    - la bonne solution est donc d'extraire les différents comportement de la grosse classe dans plein de petites classes
    - la grosse classe devient alors une classe "main" qui n'a pas d'autre métier propre que d'agréger les comportements

En résumé, un message important est : _si la DI est difficile, c'est que l'app est mal découpée par domaines métier_ !

44:00 pour faire de la DI, il faut sortir la création de la dépendance à l'extérieur de la logique (et soit injecter la dépendance directement, soit injecter un privider de dépendance)

Son contexte est celui d'une très grande codebase, où il nous est de facto interdit de modifier l'API de fonctions/constructeurs. Pour cela, deux astuces :

- injecter la dépendance comme argument de la fonction, avec une valeur par défaut (ainsi, on ne casse pas les callsites qui ne la précisent pas). Pas parfait ; notamment, casse `std::function`
- avoir une fonction `F1` qui attend la dépendance en paramètre, et avoir une fonction wrapper `F2` qui a la même signature mais SANS la dépendance : son body n'est qu'un appel à `F1` qui harcode la dépendances ; les callsites pré-existants utilisent `F2` (on n'a donc pas à les modifier), et les nouveaux callsites utilisent `F1`. NDM : il faut prévoir de transitionner les anciens callsites de F2 vers F1.

Ces deux astuces fonctionnent aussi pour les constructeurs.

Sa conclusion est double :

- DI et découpage correct de son code vont ensemble : si le code est bien découpé, on pourra facilement faire de la DI (p.ex. pour tester), et à l'inverse, structurer son code pour autoriser de la DI facilement nous conduit à le découper correctement.
- in fine, le point important est de minimiser le nombre de dépendance de ses classes
