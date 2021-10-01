# Implementing an Interface in Python

- **url** = https://realpython.com/python-interface/
- **type** = article
- **auteur** = [William MURPHY](https://realpython.com/team/wmurphy/), un auteur quidam sur realpython
- **date de publication** = 2021-??-??
- **source** = https://realpython.com/
- **tags** = language>python ; topic>interfaces ; level>intermediate

## Notes brutes :

InformalInterface = https://realpython.com/python-interface/#informal-interfaces

- simplement hérite d'une classe (dont on surchargerait les méthodse) ne suffit pas
- en effet, rien n'oblige à redéfinir dans les filles les méthodes qu'on est censé surcharger
- dans l'exemple, EmlParser et PdfParser héritent de InformalParserInterface, mais EmlParser ne surcharge pas extract_text (alors qu'il devrait !)
- pourtant, issubclass(EmlParser, InformalParserInterface) renvoie bien True (c'est logique, mais c'est pas ce qu'on veut)
- du coup, InformalInterface ne marche pas, car rien n'oblige la classe fille à redéfinir ce qu'elle devrait

Using MetaClasses = https://realpython.com/python-interface/#using-metaclasses

- pour corriger, issubclass devrait à la place VÉRIFIER si les conditions (notamment, la surcharge de extract_text) sont remplies, avant de renvoyer True
- le moyen de faire ça, c'est d'utiliser une métaclasse custom pour définir la classe d'interface
- au lieu de proposer des méthodes à réimplémenter, la classe d'interface ne propose aucun code, elle se contente d'utiliser la métaclasse qui sait vérifier si une classe implémente ce qu'il faut pour être une subclass ou non
- avantage = on n'a pas besoin de définir EXPLICITEMENT les subclasses, ça se fait "tout seul" lors du issubclass
- QUESTION : en quoi "issubclass" m'intéresse ? Parce que c'est le moyen pour mypy de vérifier si les conditions de type sont réunies ?

Using Virtual Base Classes = https://realpython.com/python-interface/#using-virtual-base-classes

- virtual base class = classe "parente" (dans le sens où "issubclass" va renvoyer True), mais n'apparaissant pourtant pas dans le MRO
- de mon point de vue, ça ressemble un peu aux protocol
- j'ai l'impression que ce chapitre n'apporte rien de nouveau, et se contente de comparer ce qu'on a fait avant (utilisation des metaclasses) avec le fait d'hériter directement (de PersonSuper)
- en gros, avec ma compréhension, l'utilisation des metaclass pour définir une interface EST une virtual base class (et je ne vois pas ce que vient faire PersonSuper ici)

Formal Interfaces = https://realpython.com/python-interface/#formal-interfaces

Il y a tout une partie sur "register" pour influer sur "issubclass" (alternative à subclasshook) : https://realpython.com/python-interface/#using-abc-to-register-a-virtual-subclass

mieux expliqué ici : https://docs.python.org/fr/3/library/abc.html#abc.ABCMeta.__subclasshook__

Au sujet des ABC :

- en gros, pour qu'une classe soit une ABC, elle doit utiliser abc.ABCMeta comme métaclasse (ou bien hériter de ABC qui le fera pour nous)
- derrière, le mécanisme à base de subclasshook ou register permet de tuner le comportement de "issubclass"
- et les abc.abstractmethod / abc.abstractproperty permettent de définir des "choses toujours vraies" sur les filles de l'ABC
- notamment : il sera interdit au runtime d'instancier une classe fille de mon ABC si la fille n'implémente pas toutes les abstractmethod (note importante : pour que ce comportement soit actif, il reste nécessaire pour les filles d'hériter explicitement de l'ABC (ce qui est un point négatif), cf. [la POC que j'ai faite sur ce sujet](https://github.com/phidra/pocs/blob/9664c5d7a6adc084a30a6b7412ff3526d7dc12f1/python/type_checking/poc05_abstractmethod_runtime_behaviour/poc.py) ; en revanche, cet héritage explicite n'est pas nécessaire pour que mypy râle si on a oublié de redéfinir une fonction décorée par `@abc.abstractmethod`)
