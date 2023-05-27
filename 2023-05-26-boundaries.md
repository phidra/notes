# Boundaries

- **url** = https://www.destroyallsoftware.com/talks/boundaries
- **type** = vidéo
- **auteur** = [Gary BERNHARDT](https://www.destroyallsoftware.com/screencasts), propose un service de screencast pour devs
- **date de publication** = 2012-11-25
- **source** = [site destroyallsoftware](https://www.destroyallsoftware.com/screencasts) où Gary propose ses screencasts (la conf a été enregistrée à [SCNA 2012](https://blog.thesoftwarecraft.com/2012/11/scna.html) = conférence software craftmanship)
- **tags** = language>agnostic ; topic>architecture ; level>intermediate


TL;DR = talk abordant divers sujets tournants autour de l'organisation du code → assimilé architecture. Son message se rapproche de l'archi hexagonale, avec quelques trucs en plus :

- cœur (= hexagone) codé en paradigme fonctionnel
- passage de value comme boundary
- le cœur a des décisions (des `if`) et aucune dépendances, le shell (= adapters) c'est le contraire


Le point important du talk :

> the value is the boundary

Il parle de core-code (équivalent de l'hexagone) vs. shell-code (équivalent des adapters).

16:00 un autre distinguo intéressant (en plus de inside world vs outside world) dans la séparation architecturale :

- le core code n'a pas de dépendance, mais il a beaucoup de chemins possibles = des conditions, des prises de décisions, qu'il faut tester. Comme il n'y a pas de dépendances, ça se prête bien aux tests (que j'appelle) unitaires.
- à l'inverse, ce qu'il appelle le shell-code (la glue qui utilise le domaine) a beaucoup de dépendances (vu qu'il associe plein de trucs entre eux) mais peu de paths possible. Ça se prête bien aux tests d'intégration.

16:10 du coup il en vient à une "archi" avec un code domaine en paradigme fonctionnel, et le monde extérieur qui l'utilise en impératif.

18:20 _The value is the boundary_

Derrière, il montre un exemple concret suivant son approche = un client twitter

21:40 ça ressemble à des schémas d'archi, séparant le domaine et le monde extérieur.

25:00 son cœur étant codé en paradigme fonctionnel, il est facile à tester.

26:00 le shell-code est stateful (ce qui n'est pas le cas du core-code, en paradigme fonctionnel).

29:40 intéressant = comme le shell-code n'a pas de conditions, ne prend pas de décision, l'auteur ne ressent pas un besoin impérieux de le tester.

31:00 ah ben voilà, il dit lui même qu'il y a des accointances entre ce qu'il dit et l'hexagonal architecture.
