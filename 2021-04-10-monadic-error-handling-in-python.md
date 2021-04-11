# Exception handling tips in Python + Monadic error handling in Python

- **url** = [url part1](https://www.youtube.com/watch?v=ZsvftkbbrR0) + [url part2](https://www.youtube.com/watch?v=J-HWmoTKhC8)
- **type** = vidéo
- **auteur** = [ArjanCodes](https://www.youtube.com/channel/UCVhQ2NnY5Rskt6UjCUkJ_DA), _university lecturer in computer science, with more than 20 years of experience in software development and design_
- **date de publication** = 2021-03-26 + 2021-04-02
- **source** = [sa chaîne youtube](https://www.youtube.com/channel/UCVhQ2NnY5Rskt6UjCUkJ_DA)
- **tags** = language>python ; topic>error-handling ; base-concepts

**TL;DR** : vidéo en deux parties sur la gestion d'erreurs avec exception, puis avec monades


Inconvénients des exceptions :
- "deuxième" control-flow du programme (beaucoup moins explicite, en plus)
- du coup, ressource leak si on fait pas attention à ce second control-flow
- couplage entre du code de haut-niveau et une exception de bas-niveau
- on ne peut jamais être sûr, à la lecture du code, qu'on a traité toutes les exceptions possibles

Alternatives à l'utilisation des exceptions :
- sortir brutalement (`sys.exit()`)
- retourner un code d'erreur (et passer le "vrai" résultat en paramètre à muter ?)
- retourner une sentinel-value (e.g. l'entier `-1`, pour la fonction racine-carrée, indique qu'il y a eu une erreur)
- passer une callback `onerror`
- gestion d'erreur "à la go" = retourner un doublet valeur + errorcode

Bonne pratique = n'attraper que les erreurs dont on sait quoi faire, et laisser passer les autres pour qu'un gestionnaire de plus haut-niveau sache quoi en faire.

Explication simple mais claire de la gestion d'erreur avec monade : 1. on renvoie un objet qui est soit un succès, soit un failure 2. chaque fonction sait gérer ce type d'objet, et sait effectuer son traitement s'il est un succès, ou propager l'erreur sinon.

Parallèle avec les "tracks" = _Railway oriented programming_

Il donne ses exemples avec la lib `returns` : [lien](https://github.com/dry-python/returns)

Avantages :
- fini le hidden control flow
- du coup, pas de ressource leak
- success ET failure sont gérés explicitement par toutes les fonctions

