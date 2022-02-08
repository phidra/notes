# Let’s Use An Automated Theorem Prover To Verify Video Games; I Swear This Is More Fun Than It Sounds

- **url** = https://www.youtube.com/watch?v=Xn6jNHtPpqQ
- **type** = vidéo YouTube
- **auteur** = [Jon MANNING](https://desplesda.net/) = gamedev, conférencier, et auteur de plusieurs livres techniques
- **date de publication** = 2021-02-04
- **source** = [la chaîne YouTube de la LinuxConf Online 2021](https://www.youtube.com/channel/UCciKHCG06rnq31toLTfAiyw) (voir aussi [l'abstract](https://lca2021.linux.org.au/schedule/presentation/54/))
- **tags** = language>none ; topic>stat-solver ; topic>z3-solver ; level>beginner

**TL;DR** : vidéo expliquant comment utilisant Z3 theorem prover pour aider au développement du scénario d'un jeu vidéo ; le sous-titre n'a pas menti, c'était très intéressant !

Le theorem prover est [Z3](https://github.com/Z3Prover/z3), développé par Microsoft.

L'idée est de modéliser les règles qui régissent le scénario, par exemple en explicitant les conditions à réunir pour qu'une branche de scénario soit accessible (e.g. "si Tali a gagné son procès, et qu'il a 4 points de charisme, alors il a le droit de s'engager en politique").

Les exemples sont clairs et intéressants, et donnés avec le jeu Mass Effect.

L'idée est d'établir des "phrases" (NdM des prédicats) qui n'ont pas particulièrement de valeur, mais qui **peuvent** être vraies ou fausses. Ces "phrases" peuvent être des combinaisons (AND, OR, NOT, ⇒, ⇔, etc.) :

- Tali a gagné son procès AND son charisme est supérieur ou égal à 4
- Tali est vivant
- Tali a le droit de s'engager en politique

Certaines de ces phrases peuvent figer la valeur d'autres :

- Tali est vivant = TRUE

Derrière, en modélisant son problème à l'aide de ces phrases, on peut déduire des connaissances sur ce qu'il est possible ou pas de faire (exemple inspirés de la vidéos, mais recréés un peu au pif).

Le solveur z3 permet alors de savoir si toutes les phrases sont **satisfaisables** (et permet de donner un exemple de valeurs qui les satisfait toutes) :

- (Tali est vivant AND Tali a gagné son procès AND son charisme est supérieur ou égal à 4) ⇒ Tali a le droit de s'engager en politique
- Tali a mal parlé à son geolier ⇒ NOT Tali est vivant
- Tali a le droit de s'engager en politique = TRUE

Dans l'exemple ci-dessus, les contraintes sont satisfaisables, avec les variables suivantes :

- TRUE : Tali est vivant
- TRUE : Tali a gagné son procès
- TRUE : son charisme est supérieur ou égal à 4
- FALSE : Tali a mal parlé à son geolier
- TRUE : Tali a le droit de s'engager en politique

Mais si on ajoute une autre contrainte, ça ne marche plus, les contraintes sont unsat = insatisfaisables :

- (Tali est vivant AND Tali a gagné son procès AND son charisme est supérieur ou égal à 4) ⇒ Tali a le droit de s'engager en politique
- Tali a mal parlé à son geolier ⇒ NOT Tali est vivant
- Tali a le droit de s'engager en politique = TRUE
- Tali a mal parlé à son geolier = TRUE

Dans ce cas, z3 est capable d'indiquer le set de valeurs minimales qui sont en contradiction (ici, le fait de mal parler à son geolier zigouillera Tali, qui ne pourra donc plus s'engager en politique).

Un autre usage intéressant de z3 : à partir de règles mathématiques sur des entiers et flottantes, optimiser un critère. Son exemple est illustratif :

- je peux vendre des épées, et je choisis leur prix
- je sais que je vendrais plus d'épées si elles sont bon marché (et je connais la formule liant taux de vente et prix)
- à quel prix dois je vendre mes épées pour maximiser mon revenu total ?

Au final, il parle de trois usages pour z2 :

- Constraint solver
- Optimizer
- Validity checker

Il mentionne deux autres ressources intéressantes :

- [PyExZ3](https://github.com/thomasjball/PyExZ3) = un outil utilisant Z3 pour prouver des choses sur des programmes python
- [Z3 API in Python](https://ericpony.github.io/z3py-tutorial/guide-examples.htm) = un tutoriel d'Eric PONY sur le binding python de z3
