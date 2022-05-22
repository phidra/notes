# Modern Python performance considerations

- **url** = https://lwn.net/Articles/893686/
- **type** = Article rendant compte d'un talk à la [pycon 2022](https://us.pycon.org/2022/schedule/presentation/37/)
- **auteur** = [Jake EDGE](https://www.ukuug.org/bios+profiles/JEdge.shtml), éditeur [LWN](https://lwn.net/), rendant compte d'un talk de [Kevin Modzelewski](https://us.pycon.org/2022/speaker/profile/40/), bossant pour Anaconda et auteur de [pytson](https://www.pyston.org/) = implémentation optimisée de python
- **date de publication** = 2022-04-30 (date du talk) + 2022-05-04 (date du compte-rendu sur LWN)
- **source** = [LWN](https://lwn.net/), news site dédié à linux et le logiciel libre
- **tags** = language>python ; topic>performance ; level>intermediate


**TL;DR** : un intéresant compte-rendu de talk sur les performances python

Python est lent car c'est un langage interprété ? Pas tant que ça : ça n'explique que 10% du ralentissement.

En vrai, python est lent car c'est une langage dynamique !

Par exemple, quand on utilise `print`, le nom peut très bien avoir été overloadé localement, et on paye donc un surcoût de lookup... Et c'est valable aussi pour les attributs des objets (qui sont dynamiques), etc.

Le hic : la plupart du temps, les dev n'utilisent PAS l'aspect dynamique (e.g. ils n'overloadent pas `print`), donc ils payent ce prix sans en utiliser les avantages !

Du coup, son idée principal dans pyston pour accélérer l'exéuction = essayer de détecter que l'utilisateur n'a pas utilisé l'aspect dynamique de python, afin d'éviter les lookups inutiles.

Auters conseils :

- utiliser les slots (pour se passer de l'aspect dynamique)
- ne pas essayer de cacher une méthode (en faisant quelque chose comme `mysqrt = math.sqrt` avant une boucle for), car les intérêts seront de moins en moins bons au fur et à mesure que Python sera de plus en plus fort pour résoudre les attributs

À noter : il parle de deux benchmarks python : [pyperformance](https://pyperformance.readthedocs.io/) = le truc standard, et un benchmark custom qu'il a developpé en Flask pour l'occasion.

Conclusion :

> The overall goal of the new optimizations is to not make Python code pay for dynamic features that it is not using. 
