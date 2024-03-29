# Graph Theory — Sarada HERKE

- **url** = https://www.youtube.com/c/SaradaHerke
- **type** = chaîne YouTube
- **auteur** = Sarada HERKE = une docteur en maths ([sa page à l'université de Queensland en Australie](https://smp.uq.edu.au/profile/270/sara-herke), [son profil ResearchGate](https://www.researchgate.net/profile/Sarada-Herke), [son twitter](https://twitter.com/spoonfulofmaths), ...)
- **date de publication** = N/A la chaîne est mise à jour au fur et à mesure des vidéos...
- **source** = [la chaîne YouTube](https://www.youtube.com/c/SaradaHerke)
- **tags** = language>none ; topic>graph ; topic>graph-theory ; topic>maths ; level>beginner

**TL;DR** : Notes très brutes sur la série de vidéos de Sarada HERKE sur la théorie des graphes.

Les notions abordées restent introductives, mais sont définies plutôt formellement.

Dans ces notes, j'essaye de mettre **en gras** les notions importantes (mais je ne reporte pas systématiquement leur définition).

J'ai mais certains titre de vidéos en chapitre, mais pas tous : ce n'est pas parce qu'il n'existe pas de titre pour une notion qu'elle n'est pas présente quelque part dans ces notes.

Note : pour gagner du temps, je ne regarde pas les vidéos qui se contentent de prouver les théorèmes ou les résultats, mais elles ont l'air super super intéressantes du point de vue mathématique !

Note bis : pour les termes de théorie des graphes, ces deux ressources peuvent également être intéressantes :

- en français : [lexique de la théorie des graphes](https://fr.wikipedia.org/wiki/Lexique_de_la_th%C3%A9orie_des_graphes)
- en anglais : [glossary of graph theory](https://en.wikipedia.org/wiki/Glossary_of_graph_theory)

* [Graph Theory — Sarada HERKE](#graph-theory--sarada-herke)
   * [Notes sur les quelques premières vidéos](#notes-sur-les-quelques-premières-vidéos)
   * [Graph Theory: 12. Spanning and Induced Subgraphs](#graph-theory-12-spanning-and-induced-subgraphs)
   * [Graph Theory: 16. Walks Trails and Paths](#graph-theory-16-walks-trails-and-paths)
   * [Graph Theory: 17. Distance Between Vertices and Connected Components](#graph-theory-17-distance-between-vertices-and-connected-components)
   * [Graph Theory: 18. Every Walk Contains a Path](#graph-theory-18-every-walk-contains-a-path)
   * [Graph Theory: 19. Graph is Bipartite iff No Odd Cycle](#graph-theory-19-graph-is-bipartite-iff-no-odd-cycle)
   * [Graph Theory: 20. Edge Weighted Shortest Path Problem](#graph-theory-20-edge-weighted-shortest-path-problem)
   * [Graph Theory: 21. Dijkstra's Algorithm](#graph-theory-21-dijkstras-algorithm)
   * [Graph Theory: 23. Euler Trails and Euler Tours](#graph-theory-23-euler-trails-and-euler-tours)
   * [Graph Theory: 24. Euler Trail iff 0 or 2 Vertices of Odd Degree](#graph-theory-24-euler-trail-iff-0-or-2-vertices-of-odd-degree)
   * [Graph Theory: 25. Graph Decompositions](#graph-theory-25-graph-decompositions)
   * [Graph Theory: 26. Cycle Decomposition iff All Vertices Have Even Degree](#graph-theory-26-cycle-decomposition-iff-all-vertices-have-even-degree)
   * [Graph Theory: 27. Hamiltonian Graphs and Problem Set](#graph-theory-27-hamiltonian-graphs-and-problem-set)
   * [Graph Theory: 29. Lovasz Conjecture on Hamilton Paths](#graph-theory-29-lovasz-conjecture-on-hamilton-paths)
   * [Graph Theory: 28. Hamiltonian Graph Problems](#graph-theory-28-hamiltonian-graph-problems)
   * [Graph Theory: 34. Bridge edges](#graph-theory-34-bridge-edges)
   * [Graph Theory: 36. Definition of a Tree](#graph-theory-36-definition-of-a-tree)
   * [Graph Theory: 39. Types of Trees](#graph-theory-39-types-of-trees)
   * [Graph Theory: 40. Cayley's Formula and Prufer Seqences part](#graph-theory-40-cayleys-formula-and-prufer-seqences-part)
   * [Graph Theory: 42. Degree Sequences and Graphical Sequences](#graph-theory-42-degree-sequences-and-graphical-sequences)
   * [Graph Theory: 43. Havel-Hakimi Theorem on Graphical Sequences](#graph-theory-43-havel-hakimi-theorem-on-graphical-sequences)
   * [Graph Theory: 44. Degree Sequence of a Tree](#graph-theory-44-degree-sequence-of-a-tree)
   * [Graph Theory: 45. Specific Degrees in a Tree](#graph-theory-45-specific-degrees-in-a-tree)
   * [Graph Theory: 46. Relation Between Minimun Degree and Subtrees](#graph-theory-46-relation-between-minimun-degree-and-subtrees)
   * [Graph Theory: 47. Subgraphs of Regular Graphs](#graph-theory-47-subgraphs-of-regular-graphs)
   * [Graph Theory: 48. Complement of a Graph](#graph-theory-48-complement-of-a-graph)
   * [Graph Theory: 49. Cartesian Product of Graphs](#graph-theory-49-cartesian-product-of-graphs)
   * [Graph Theory: 50. Maximum vs Maximal](#graph-theory-50-maximum-vs-maximal)
   * [Graph Theory: 51. Eccentricity, Radius &amp; Diameter](#graph-theory-51-eccentricity-radius--diameter)
   * [Graph Theory: 52. Radius and Diameter Examples](#graph-theory-52-radius-and-diameter-examples)
   * [Graph Theory: 53. Cut-Vertices](#graph-theory-53-cut-vertices)
   * [Graph Theory: 54. Number of Cut-Vertices](#graph-theory-54-number-of-cut-vertices)
   * [Graph Theory: 55. Bridges and Blocks](#graph-theory-55-bridges-and-blocks)
   * [Graph Theory: 56. Central Vertices are in a Single Block](#graph-theory-56-central-vertices-are-in-a-single-block)
   * [Graph Theory: 57. Planar Graphs](#graph-theory-57-planar-graphs)
   * [Graph Theory: 58. Euler's Formula for Plane Graphs](#graph-theory-58-eulers-formula-for-plane-graphs)
   * [Graph Theory: 59. Maximal Planar Graphs](#graph-theory-59-maximal-planar-graphs)
   * [Graph Theory: 60. Non Planar Graphs](#graph-theory-60-non-planar-graphs)
   * [Graph Theory: 61. Characterization of Planar Graphs](#graph-theory-61-characterization-of-planar-graphs)
   * [Graph Theory: 62. Graph Minors and Wagner's Theorem](#graph-theory-62-graph-minors-and-wagners-theorem)
   * [Graph Theory: 63. Petersen Graph is Non-Planar](#graph-theory-63-petersen-graph-is-non-planar)
   * [Graph Theory: 64. Vertex Colouring](#graph-theory-64-vertex-colouring)
      * [Coloration et scheduling](#coloration-et-scheduling)
   * [Graph Theory: 65. 2-Chromatic Graphs](#graph-theory-65-2-chromatic-graphs)
   * [Graph Theory: 66. Basic Bound on the Chromatic Number](#graph-theory-66-basic-bound-on-the-chromatic-number)

## Notes sur les quelques premières vidéos

Les appellations de certains graphes particuliers :
- `Kx` = graphe complet d'ordre x
- `Kx,y` = graphe complet bipartite à x et y éléments
- `K1,x` = cas particulier, ça revient à un star-graph
- `Px` = path-graph d'ordre x = il y a x vertex formant un chemin linéaire.

Notations :

- `|E|` = le nombre d'edges du graphe est habituellement noté `m`
- `|V|` = le nombre de vertices du graphe est habituellement noté `n`

**adjacent vertices** = vertex qui sont reliés par un edge.

**Graphe bipartite** = les nœuds sont partitionnables en deux sets indépendants, de sorte que tous les edges du graphe aient un nœud dans le premier set et l'autre dans le deuxième.

Pour savoir si un graphe est bipartite, on le traverse en coloriant les vertex de deux couleurs, en alternant entre les couleurs. Si on peut le faire, le graphe est bipartite. Si quand on essaye de faire ça, on doit colorier un nœud des deux couleurs en même temps, le graphe n'est pas bipartite.

**Complete Bipartite graph** = chaque vertex d'un set est voisin de TOUS les vertex de l'autre set.

Théorème : dans un graphe non-orienté, la somme des degrés est toujours 2 x le nombre d'edges (logique : un edge relie toujours deux nœuds)

Corollaire : il y a un nombre pair de nœuds de degrés impairs (sinon on ne peut pas respecter la contrainte du théorème précédent).

**Connected / disconnected**

**Degree**

**k-regular graph** = tous les vertex ont le même degré `k`

`δ(G)` = le degré minimal d'un graphe (i.e. il n'existe pas de nœud dans le graphe ayant moins d'edges que `δ(G)`)

`Δ(G)` = le degré maximal d'un graphe

[sage](https://www.sagemath.org/) est un outil permettant de travailler avec des graphes ?

**Adjacency Matrix** (et la moins connue **incidence Matrix**)

**Isomorphic graphs** = les nœuds sont en bijection + les edges sont exactement préservés par la bijection (il n'y a pas plus d'edges, et pas moins d'edges)

Note : globalement, c'est une question difficile de trouver la raison pour laquelle deux graphes ne sont PAS isomorphic.

## Graph Theory: 12. Spanning and Induced Subgraphs

Ces deux graphes sont tous les deux des subgraphs (de types différents) d'un graphe initial `G` :
- **Spanning subgraph** = mêmes vertexs que `G`, mais possiblement moins d'edges.
- **Induced subgraph** = obtenu à partir de `G` en supprimant des vertex (et leurs edges), pour ne garder que les vertex qui sont dans un set défini (et les edges entre ces vertex "survivants")

**Graph order** = nombre de vertex dans le graphe.

**Graph size** = nombre d'edges dans le graphe.

## Graph Theory: 16. Walks Trails and Paths

**walk** = séquence de vertex adjacents (on peut "marcher" le long du walk). Il n'y a pas obligation que les vertices ou les edges soient uniques (on peut repasser par les mêmes vertices/edges) ni d'emprunter tous les vertices/edges du graphe. Dans un multigraphe, il faut préciser une suite de vertices ET d'edges : v₁e₁v₂e₂...exvx. À l'inverse, si le graphe est simple, il suffit de préciser la liste de vertices (car il n'y alors pas d'ambiguïté sur les edges empruntés).

**trail** = walk où les edges sont uniques (on ne réemprunte pas deux fois le même edge) ; toujours pas d'obligation que les vertices soient uniques, ni d'emprunter tous les vertices/edges.

**path** = walk où edges ET les vertices sont uniques (on ne réemprunte ni les mêmes edges, ni les mêmes vertices) ; pas d'obligation ni d'emprunter tous les vertices/edges, ni de "finir" sur le même vertice que celui par lequel on a commencé.

**closed walk** = un walk qui finit sur le même vertex que celui par lequel on a commencé.

**cycle** = closed path (i.e. séquence de vertices/edges uniques, qui revient sur le vertex initial).

## Graph Theory: 17. Distance Between Vertices and Connected Components

**Distance between nodes** = length of shortest path between nodes (par convention, la distance est infinie si les deux nœuds sont sur des composantes connexes différentes).

**Connected graph** = quelque soit la paire de nœuds qu'on considère dans le graphe, leur distance n'est pas infinie.

**Connected component** = maximal connected subgraph.

Maximal = si on ajoute un autre node du graphe original à ce subgraph, le subgraph devient disconnected.

## Graph Theory: 18. Every Walk Contains a Path

Un xy-walk contient obligatoirement un xy-path

## Graph Theory: 19. Graph is Bipartite iff No Odd Cycle

Condition nécessaire et suffisante pour qu'un graphe soit bipartite = il ne contient pas de cycle impair (i.e. pas de cycle dont le nombre de nœuds est impair).

## Graph Theory: 20. Edge Weighted Shortest Path Problem

**Weighted graph** = on peut associer un poids à chaque edge.

Dit autrement : il existe une fonction (la _weight function_) définie sur `E(G)` à valeurs dans `ℝ`.

## Graph Theory: 21. Dijkstra's Algorithm

Ce qui concerne Dijkstra est dans ces deux vidéos :
- Graph Theory: 21. Dijkstra's Algorithm
- Graph Theory: 22. Dijkstra Algorithm Examples

## Graph Theory: 23. Euler Trails and Euler Tours

**Euler trail** = trail qui emprunte TOUS les edges du graphe (et par définition d'un trail, il ne les emprunte qu'une seule fois).

**Euler tour** = _closed Euler trail_ = Euler tour qui revient sur le vertex initial.

**Eulerian graph** = graphe qui contient un euler tour.

Moyen mnémotechnique (pour ne pas confondre Eulérien et Hamiltonien) = le problème fondateur de la théorie des graphes = les 7 ponts de Königsberg a été résolu par **EULER** en utilisant les propriétés d'un Euler tour (car il fallait passer par tous les 7 ponts une et une seule fois.

## Graph Theory: 24. Euler Trail iff 0 or 2 Vertices of Odd Degree

Condition nécessaire et suffisante pour qu'il existe un Euler trail dans un graphe = il y a exactement 0 ou 2 nœuds de degré impair dans le graphe.

C'est ce qui a été utilisé par Euler pour résoudre le problème des 7 ponts de Königsberg.

À noter que si on veut un Euler TOUR (i.e. si on veut finir sur le même vertex qu'on a commencé), il faut exactement 0 nœud de degré impair.

## Graph Theory: 25. Graph Decompositions

**Graph decomposition** = partitionner le graphe en une famille de edge-disjoint subgraphs, i.e. si on fait l'union des edges de tous les subgraph, on retombe sur les edges du graphe original.

À noter que les différents subgraphs ont le droit de partager des vertices (c'est même inévitable), ce sont les *edges* qui ne doivent se trouver que dans un seul subgraph.

Quelques décompositions particulières : cycle-decompositions, path-decompositions (et il existe a toujours une trivial path décomposition, en `m` chemins triviaux ne contenant qu'un edge), hamiltonian decomposition (en cycles hamiltoniens) ...

## Graph Theory: 26. Cycle Decomposition iff All Vertices Have Even Degree

**even graph** = tous les nœuds du graphe sont de degré pair.

Condition nécessaire et suffisante pour qu'un graphe soit décomposable en cycles = le graph est pair.

En effet, chaque cycle passant par un vertex "utilise" deux edges du vertex, donc s'il y a une décomposition en cycles, tous les vertex sont pairs.

Inversement, si tous les vertex sont pairs, on peut extraire des cycles à répétition, on est sûr qu'on ne sera jamais coincé : si on peut "entrer" dans un vertex, on pourra en "ressortir".

## Graph Theory: 27. Hamiltonian Graphs and Problem Set

**Hamiltonian path** = path empruntant tous les VERTEX (par opposition au Euler trail, qui emprunte tous les EDGES).

**Hamiltonian graph** = graphe qui contient un cycle hamiltonien.

La vidéo 27 présente quelques problèmes avec les graphes Hamiltonian, et leur solution est donnée dans la vidéo 28.

## Graph Theory: 29. Lovasz Conjecture on Hamilton Paths

Note : à la différence des graphes eulériens (pour lesquels on a le théorème d'Euler), savoir caractériser un graphe hamiltonien n'est pas facile...

Conjecture de lovasz = tous les graphes connexes finis qui sont vertex-transitive ont un Hamiltonian path.

**Automorphisme** = il existe un isomorphisme d'un graphe sur lui même (les vertex sont similaires à d'autres vertex au sein du même graphe). Par exemple, `Kn,n` est automorphic, puisque les deux sets jouent un rôle équivalent.

**Vertex transitive graph** = TOUS les vertex du graph sont similaires entre eux. Exemples : Kn, K(n,n), les graphes cycles, ... Contre-exemple : path graph, car les deux extrémités jouent un rôle différents des autres nœuds. Plus généralement, un vertex transitive graph est forcément régulier (ce qui n'est pas le cas du path graph)

## Graph Theory: 28. Hamiltonian Graph Problems

De 28 à 33, plusieurs problèmes et leurs solutions concernant les graphes Hamiltonian

## Graph Theory: 34. Bridge edges

Point sur la notation :

- `G — S` = le graphe `G` auquel on retire les VERTEX du set `S` (i.e. c'est le graphe induit par le set de nœuds restants).
- `G \ E` = le graphe `G` auquel on retire les EDGES du set `E`

**Bridge-edge** = un edge est un bridge-edge quand, si on le retire du graphe, celui-ci n'est plus connecté. Ou plus précisément, si on retire le bridge-edge, on incrémente le nombre de ses composantes connexes.

Condition nécessaire et suffisante pour qu'un edge soit un bridge-edge = il n'est pas sur un cycle du graphe.

## Graph Theory: 36. Definition of a Tree

**Tree** = graphe connexe sans cycle.

Note : il y a plusieurs autres définitions équvialentes, données dans la vidéo 38. En résumé, si un graphe vérifie deux quelconques des trois propriétés suivantes, c'est un tree :
- connected
- acyclic
- `m = n-1`

**Forest** = graphe sans cycle (et il peut être disjoint) = chacune de ses composantes connexes est un tree.

Condition nécessaire et suffisante pour qu'un graphe soit un tree = tous ses edges sont des bridges.

**Formule de Cayley** = il y a `n^(n-2)` trees possibles à partir de `n` vertex. Dans cette formule, on a supposé que tous les vertices sont labellisés (et du coup, certains des trees comptés par cette formule sont isomorphes entre eux !).

**leaf** : dans un tree, un vertex de degré `1` est une leaf.

**Spanning tree** = spanning subgraph that is a tree. Il en existe au moins un pour tout graphe connexe.

Note rigolote : comme on toujours a la relation `m=n-1` pour les trees, pour transformer un graph non-tree en spanning-tree, on peut savoir à l'avance le nombre d'edges qu'il faudra lui retirer : c'est `m-n+1` :-)

Corrolaire : tout graphe connexe vérifie `m >= n-1` (le cas où `m = n-1` est le cas où le graphe connexe est un tree ; derrière, on ne peut que rajouter des edges).

## Graph Theory: 39. Types of Trees

**Caterpillar graph** = arbre dans lequel tous les sommets sont à la distance `1` d'un chemin central.

## Graph Theory: 40. Cayley's Formula and Prufer Seqences part

Cette vidéo et la suivante (41) prouvent la formule de Cayley. Je ne les ai pas regardées, mais elles sont sans doute très intéressantes du point de vue mathématique !

## Graph Theory: 42. Degree Sequences and Graphical Sequences

**degree sequence** = liste des degrés des vertex du graphe. Selon les sources, soit elle est ordonnée par ordre décroissant, soit peu importe son ordre (au final, ça ne change pas grand chose).

**graphical sequence** = à partir d'une séquence d'entiers non négatifs quelconque donnée, trouver un graphe qui a cette séquence comme degree sequence est difficile. Si un tel graphe existe, la séquence d'entiers est dite "graphical séquence".

Une graphical séquence peut conduire à PLUSIEURS graphes différents.

Note : [la page wikipedia](https://en.wikipedia.org/wiki/Degree_(graph_theory)#Degree_sequence) parle d'une séquence décroissante uniquement ; en pratique ça change pas grand chose.

Apparemment, le fait de trouver un graphe à partir d'une degree sequence s'appelle le [problème de réalisation de graphe](https://fr.m.wikipedia.org/wiki/Probl%C3%A8me_de_r%C3%A9alisation_de_graphe) (en anglais : [graph realization problem](https://en.wikipedia.org/wiki/Graph_realization_problem)).

## Graph Theory: 43. Havel-Hakimi Theorem on Graphical Sequences

Le théorème de [Havel-Hakimi](https://en.wikipedia.org/wiki/Havel%E2%80%93Hakimi_algorithm) (assez rigolo, qui donne aussi un algo pour construire le graphe) donne une condition nécessaire et suffisante pour qu'une séquence soit graphical.

## Graph Theory: 44. Degree Sequence of a Tree

Une degree sequence est celle d'un tree, si la somme des degrés dans la séquence est `2n-2`.

## Graph Theory: 45. Specific Degrees in a Tree

Formule pour trouver le nombre de nœuds de degré `k` dans un tree.

## Graph Theory: 46. Relation Between Minimun Degree and Subtrees

En gros, étant donné un tree `T` ayant `k` nœuds, tout graphe `G` dont le plus petit degré est au moins `k-1` possède le tree `T` comme subgraph.

## Graph Theory: 47. Subgraphs of Regular Graphs

Tout graphe de degré max `r` est un sous-graphe du graphe `r-regular`.

## Graph Theory: 48. Complement of a Graph

**Complement of a graph G** (en français : [https://fr.wikipedia.org/wiki/Graphe_compl%C3%A9mentaire](graphe complémentaire)) = a les mêmes nœuds que `G`, mais a des edges "opposés" (i.e. il a les edges qui manquent à `G` pour que `G` soit un graphe complet).

Le nombre d'edges du complémentaire est "2 parmi n" moins `m` : en effet, le nombre d'edges du graphe complet `Kn` est "2 parmi n", et `m` est le nombre d'edges de `G`.

**self-complement** (en français : [graphe auto-complémentaire](https://fr.wikipedia.org/wiki/Graphe_auto-compl%C3%A9mentaire)) = `G` et son complémentaire sont isomorphes.
Self-complement = G et son complémentaire sont isomoorphiques.

Condition nécessaire et suffisante pour qu'il existe un graphe auto-complémentaire d'ordre `n` = l'ordre `n` vaut 0 ou 1 modulo 4.

(dit autrement : il n'existe pas des graphes auto-complémentaires d'ordre `n` pour tous les `n` !)

## Graph Theory: 49. Cartesian Product of Graphs

**Cartesian product** of graphs : en gros, on prend toutes les combinaisons possibles des nœuds, et on leur ajoute des edges s'ils existent dans l'un des deux graphes.

Exemple intéressant = le plan euclidien est le produit cartésien de `ℝxℝ`, puisque chaque point est un 2-tuple d'éléments de `ℝ`.

La vidéo donne la définition formelle, et donne l'exemple du produit cartésien de `K₂` et `P₄` :

```
K₂ :
  x
  |
  x

P₄ (=le path-graph d'ordre 4) :
  x---x---x---x


K₂ x P₄ :

  x---x---x---x
  |   |   |   |
  x---x---x---x
```

## Graph Theory: 50. Maximum vs Maximal

**Maximum** = on maximise une quantité de quelque chose (i.e. on ne peut pas trouver de XXX plus grand).

**Maximal** = on maximise une relation d'inclusion (i.e. on ne peut pas inclure dans XXX un élément de plus).

**Independent set in a graph G** = set de vertices tel qu'aucune paire de vertex du set ne soient adjacents. Le terme français semble être **un stable** ([lien](https://fr.wikipedia.org/wiki/Stable_(th%C3%A9orie_des_graphes))).

**Maximal independent set** = on ne peut pas inclure un nouveau vertex au set (car si on le fait, le set n'est plus indépendant).

**Maximum independent set** = c'est l'independent set le plus grand (i.e. qui a le plus grand cardinal, qui contient le plus de vertex) qu'on puisse trouver.

Un independent set peut être maximxal sans être maximum, on peut l'illustrer avec le graphe suivant :

```
  A---B
  |   |
  C---D
      |
      E
```

- l'independent-set `{A,D}` est maximal (je ne peux pas lui ajouter d'autre vertex, sans quoi il ne sera plus indépendant) mais n'est pas maximum (il existe d'autres independent-sets plus grands).
- l'independent-set `{B,C,E}` est maximal ET maximum (c'est le plus grand independent-set de ce graphe)
- l'independent-set `{B,C}` n'est pas maximal (en effet, je pourrais lui ajouter le vertex `E`, ce serait encore un independent-set), et a fortiori encore moins maximum

Autre exemple illustrant la différence entre maximal et maximum = les composantes connexes : par définition, une composante connexe est maximale (i.e. c'est un sous-graphe auquel on ne peut pas ajouter de vertex, sans que ce sous-graphe devienne disjoint). Par contre, il n'existe qu'une seule composante connexe maximum = la composante "principale".

## Graph Theory: 51. Eccentricity, Radius & Diameter


**Distance between u and v** = length of shortest path between `u` and `v`.

Le graphe muni de la distance est un espace métrique : la distance entre deux nœuds est une distance [au sens mathématique du terme](https://fr.wikipedia.org/wiki/Distance_(math%C3%A9matiques)), et elle vérifie donc les 3 propriétés habituelles d'une distance :

- symétrie : `d(u,v) = d(v,u)`
- séparation : `d(u,v) = 0  ⇔  u = v`
- inégalité triangulaire : `d(u,v) ≤ d(u,w) + d(w,v)`

**Eccentricity (excentricité) d'un vertex u** = max des distances entre le nœud et tous les autres nœuds.

Note : si un vertex a une excentricité de 1, c'est qu'il est adjacent à tous les autres vertex du graphe (star graph). Plus généralement : petite excentricité ⇒ vertex proche de tous les autres nœuds du graphe.

**Diameter du graphe** = la plus grande excentricité parmi tous les nœuds du graphe.

**Radius du graphe** = la plus petite excentricité parmi tous les nœuds du graphe.

**Peripheral vertex** = vertex dont l'excentricité est égale au diamètre.

**Periphery of the graph** = ensemble de tous les peripheral vertex.

**Central vertex** = vertex dont l'excentricité est égale au rayon.

**Center of the graph** = ensemble de tous les central vertex.

Ces concepts sont tous liés à la notion de distance, mais ne s'appliquent pas aux mêmes objets :

- distance = s'applique à une PAIRE de vertex
- eccentricity = s'applique à un SEUL vertex
- diameter/radius = s'applique à tout le GRAPHE

Notations : `cen(g)` , `dia(g)`, `rad(g)`, ...

Théorème = le diamètre est supérieur ou égal à R et inférieur ou égal à 2R : `rad(G) ≤ dia(G) ≤ 2.rad(G)`

## Graph Theory: 52. Radius and Diameter Examples

Théorème = si G est disconnected, son complémentaire (noté "G barre" = G avec une barre au dessus) est connexe et de diamètre inférieur ou égal à 2.

(la preuve est assez logique : en gros, si G est déconnecté, alors il y aura des edegs entre tous les nœuds de chaque composantes connexes, et on pourra aller partout en au max deux edges)

Théorème = tout graphe G est le centre d'un graphe connecté H. La façon dont il est construit est assez rigolote :

- on ajoute deux vertex u1 et u2, chacun est adjacent à tous les vertices de G.
- on ajoute deux vertex v1 et v2, chacun n'est adjacent qu'à u1 (resp. u2)
- avec un tel graphe, par construction :
   - tous les nœuds de G sont au max à 2 de v1 ou v2 (car ils sont tous connectés à u1/u2) et au moins à 2 de v1/v2
   - u1 est à 3 de v2 et v1 est à 4 de v2 ⇒ 2 est bien la plus petite excentricité de H aka le radius de H
   - ⇒ tous les nœuds de G ont bien une excentricité de 2, égale au radius de H, donc G est le centre de H, CQFD

## Graph Theory: 53. Cut-Vertices

**cut-vertex** = v est cut-vertex de G si `G—v` (G privé de v) a plus de composantes connexes que G.

Notation : `C(G)` (ou parfois `K(G)`) est le nombre de composantes connexes de G.

Caractérisation des cut-vertices v = il existe deux vertex u et w pour lesquels v est sur tous les uw-path. (il existe l'équivalent pour les bridges)

## Graph Theory: 54. Number of Cut-Vertices

Ici, on donne quelques limites sur le nombre de cut-vertices que peut contenir un graphe.

Les cas extrêmes :
- `Kn` qui a 0 cut-vertex (en effet, si le graphe est complet, on peut lui retirer des vertices, il restera connexe)
- `Pn` qui en a `n-2` (en effet, sur un path-graph, à l'exception des deux extrêmités, tout vertex relie deux moitiés du graphe)

Théorème : tout graphe connecté non trivial a au moins deux nodes si ne sont pas des cut-vertices.

Plus généralement, les peripheral vertices ne peuvent pas être des cut-vertices.

## Graph Theory: 55. Bridges and Blocks

**bridge-edge** : `e` est un bridge si `C(G—e) > C(G)` ; aka retirer `e` augmente le nombre de composantes connexes.

En fait, on a même `C(G—e) = C(G) + 1` ; c'est une différence avec les cut-vertices où en retirant un seul nœud, on peut séparer le graphe en plusieurs morceaux : en retirant un bridge, on ne peut "que" séparer le graphe en 2.

Caractérisation des bridges e (similaire à la caractérisation des cut-vertices) = il existe deux vertex u et w pour lesquels e est sur tous les uw-path.

Condition nécessaire et suffisante pour que `e` soit un bridge : il n'est sur aucun cycle du graphe.

Si uv est un bridge et que `deg(u) >= 2`, alors u est un cut-vertex ; c'est logique graphiquement, de base les deux nœuds d'un bridge sont des cut-vertices, et ce n'est que si l'un des deux sous-graphe est vide que ce n'est pas le cas.

**2-connected graph = non-separable graph** : un graphe connecté non-trivial est dit non-separable (ou 2-connected) s'il n'a pas de cut-vertices ; dit autrement, tous ses nœuds sont connectés par au moins deux chemins, donc on peut lui retirer n'importe quel vertex, on ne déconnectera jamais le graphe.

NdM : je trouve la notation "2-connected" plus intuitive.

**Block** = maximal 2-connected subgraph. NdM : c'est un peu l'équivalent d'une composante connexe, mais pour la 2-connexité.

Théorème : si G a au moins un cut-vertex, alors au moins deux blocks de G contiennent un cut-vertex de G (ils sont appelés **end-blocks**).

## Graph Theory: 56. Central Vertices are in a Single Block

Les central vertices appartiennent tous au même block (démontré par l'absurde).

## Graph Theory: 57. Planar Graphs

**Planar graph** (en français : graphe planaire) = can be drawn on the plane without edge crossing.

**Plane graph** (en français : graphe plan, on dirait) = has been drawn without edge crossing : un plane graph est "un dessin" d'un planar graph, parmi d'autres possibles.

**Exterior region** = un plane graph divise le plan en régions, dont une _unbounded_ appelée the exterior region (en français, la région externe ?).

Comme ils n'ont pas de cycle (par définition), les arbres ne définissent qu'une seule région = la _unbounded region_.

**Boundary of a region in a plane graph** = le set de vertices et d'edges qui délimitent la région.

Deux plane graphs DIFFÉRENTS peuvent être isomoorphiques. Dit autrement, il y a plusieurs façons différentes de dessiner un même graphe sur le plan.

K₃,₃ et K₅ ne sont pas planar.

## Graph Theory: 58. Euler's Formula for Plane Graphs

Théorème = pour tout connected plane graph, on a : `n-m+r = 2` (où `r` = le nombre de régions du plane graph)

Et même : le théorème se généralise pour les graphes non-connectés en : `n-m+r = 1 + C(G)`

Note : tout polyèdre est planar (en il suffit de le "poser" sur une de ses faces, et "d'étirer" jusqu'à ce que toutes les faces du polyèdre soient visibles vues de dessus), du coup on retrouve la célèbre formule `V-E+F = 2`.

## Graph Theory: 59. Maximal Planar Graphs

Combien d'edges au maximum peut-on ajouter à un plane graph, sans finir par le rendre non-planaire ?

**Maximal plane graph** : le graphe résultant de l'ajout du max possible d'edges est le maximal plane graph.

**Triangulation** : synonyme pour maximal plane graph, car dans celui-ci, chacune de ses régions est un triangle.

Le nombre d'edges finaux d'un maximal plane graph ne dépend que de son ordre : `m = 3n - 6` (valable pour les cas non-triviaux, donc si `n ≥ 3`). Ça se démontre assez simplement en 1. notant que les régions sont forcément délimitées par des triangles, 2. comptant les edges de chaque région et utilisant la formule d'Euler, et 3. faisant attention qu'on a compté certains edges deux fois.

Corollaire : le nombre max d'edges d'un plane graph quelconque, pas forcément maximal, est `m ≤ 3n - 6` (et ce max est atteint si le graphe est un maximal plane graph).

## Graph Theory: 60. Non Planar Graphs

K₃,₃ et K₅ ne sont pas planaires.

Dans un graphe planaire sans triangle, on a : `m ≤ 2n-4` (la preuve suit le même principe que pour la formule avec les graphes triangulaires juste au dessus).

Les démonstrations de non planéités pour ces deux graphes en particulier (dont on connaît `n` et `m`) font intervenir les relations entre `n` et `m` qu'on a trouvées précédemment.

Obviously, si un graphe G a K₃,₃ ou K₅ comme subgraph, alors G n'est pas planaire (sinon, ça voudrait dire qu'on pourrait dessiner le grand graphe sur un plan, puis effacer les edges en trop par rapport à K₃,₃ ou K₅, et donc ça voudrait dire qu'on peut dessiner K₃,₃ ou K₅ sur un plan).

## Graph Theory: 61. Characterization of Planar Graphs

On généralise la dernière phrase de la vidéo 60 : si un graphe G est construit en "ajoutant" un vertex sur un edge de K₅ (pour le splitter en deux demi-edges), alors il ne peut pas être planaire non plus, sans quoi on pourrait dessiner K₅ sur un plan, en dessinant G sur un plan, puis en supprimant ce vertex.

**Elementary Subdivision of graph G** = graphe H obtenu en ajoutant un vertex au milieu d'un edge de G (i.e. obtenu en coupant un edge en deux-edges).

Comme plus haut, toute subdivision H d'un graphe G est planar si et seulement si G était planar.

**Subdivision of graph G** = un graphe H obtenu après une séquence de zéro ou plusieurs subdivisions élémentaires.

Du coup, si un graphe contient une subdivision de K₅ (ou K₃,₃), il n'est pas planaire.

**Théorème de Kuratowski** = le fait pour un graphe G de contenir une subdivision de K₅ ou K₃,₃ est une condition nécessaire et suffisante pour que le graphe ne soit pas planaire.

Dit autrement : les seuls graphes qui ne sont _pas_ planaires sont ceux qui contiennent une subdivision  de K₅ ou K₃,₃.

## Graph Theory: 62. Graph Minors and Wagner's Theorem

**Edge contraction** = fusionner les deux nodes de l'edge, en supprimant l'edge qui les relie.

(NdM : obviously, cette notion n'a pas de rapport avec la contraction d'un node utilisé dans les Contraction Hierarchies)

Note : l'edge contraction est à ne pas confondre avec l'edge deletion (dans le second cas, on ne fusionne pas les nœuds).

À noter que même si on contracte un edge d'un graphe simple, la contraction peut produire un multigraphe. En fonction des situations, on pourra parfois conserver ce multigraphe, ou au contraire simplifier les edges parallèles en un seul edge pour retomber sur un graphe simple.

Notation après avoir contracté l'edge `e` sur le graphe G = `G/e`.

(à ne pas confondre avec la notation `G\e` où on a _supprimé_ l'edge `e`)

**Graph minor** = un graphe obtenu par applications successives de :

- deleting an edge
- deleting an isolated vertex
- contracting an edge

G est toujours un mineur de lui-même.

Par extension, on appelle aussi "mineur de G" tout graphe H qui est isomorphe à un mineur de G.

**Théorème de Wagner** = condition nécessaire et suffisante pour qu'une graphe soit planaire : il n'a ni K₃,₃ ni K₅ comme mineurs.

Note : c'est un théorème proche mais différent du théorème de Kuratowski (Kuratowski : subdivision, Wagner : mineur) ; en fonction des situations l'un sera plus pratique que l'autre à appliquer, ou le contraire.

Une subdivision d'un graphe H peut toujours être ramenée à un mineur de H (en contractant les edges splittés par la subdivision), mais l'inverse n'est pas vrai.

## Graph Theory: 63. Petersen Graph is Non-Planar

Petersen graph = graphe célèbre car c'est un exemple ou un contre-exemple de plein de trucs en théorie des graphes.

La vidéo est l'application de chacun des deux théorèmes (Kuratowski et Wagner) pour prouver que le Petersen graph n'est pas planar

## Graph Theory: 64. Vertex Colouring

**(proper) vertex-colouring of G** : c'est un labelling (i.e. une fonction de `V(G)` vers les `k` premiers entiers, entiers qu'on appelle alors couleurs) tels que deux vertex adjacents ont une couleur différente.

Une coloration qui n'est pas propre est un labelling qui ne vérifie pas la priorité. La plupart du temps, quand on parle de "coloration" on parle de "proper coloration".

**k-colouring** : une coloration d'un graphe G utilisant `k` couleurs.

**graph k-colourable** : un graphe est k-colourable s'il existe une proper k-colouring pour ce graphe.

**chromatic number** (nombre chromatique) = le plus petit `k` tel que G est k-colourable, i.e. le nombre minimal de couleurs avec lequel on peut colorier le graphe.

Notation : nombre chromatique = `χ(G)`

**graphe k-chromatic** = graphe dont le nombre chromatique est `k`.

Quelques propriétés :

- obviously, `χ(G)` est borné par `n`
- `χ(G) == 1` si et seulement si le graphe n'a pas d'edges (en effet, dès qu'il existe un edge, il faut au moins deux couleurs pour ses deux nœuds)
- `χ(Cn)` (nombre chromatique du graphe cycle d'ordre n) vaut 2 si n est pair et 3 si n est impair.
- `χ(Kn)` (nombre chromatique du graphe complet d'ordre n) vaut n
- Si H est un sous-graphe de G alors `χ(G) ≥ χ(H)` (c'est logique : si on ne peut pas colorier le sous-graphe H en moins de k couleurs, il faudra au moins k couleurs pour colorier G)

**colour classes** : un k-colouring partitionne les vertex en k independent-sets, qui sont appelés colour classes.

### Coloration et scheduling

Les colorations sont bien adaptées à des problème où on doit scheduler des trucs (i.e. affecter des ressources à des slots).

L'exemple qui suit n'est pas dans la vidéo, mais illustre bien le point.

Position du problème = je suis le directeur d'une université, on est mi-août, et je dois préparer l'emploi du temps pour la rentrée prochaine :

- mes étudiants ont des heures de cours auxquels ils doivent assister
- en fonction de leur programme à la carte, tous mes étudiants n'assistent pas aux mêmes cours
- j'ai à ma disposition 35 créneaux possibles pour placer les heures de cours (7 créneaux d'une heure par journée, et 5 journées par semaine)
- pour un créneau horaire donné (e.g. le lundi de 10h à 11h), il est tout à fait autorisé que PLUSIEURS cours soient dispensés en même temps, à condition qu'aucun étudiant ne doive assister à plusieurs de ces cours
- dit autrement : quand il y a deux cours simultanés sur le même créneau horaire, aucun étudiant ne doit être inscrit à ces deux cours
- question 1 : est-il possible d'affecter les cours aux créneaux horaires, de sorte que chaque étudiant ait la possibilité d'assister à tous ses cours ?
- question 2 : si oui, comment affecter optimalement les cours aux créneaux horaires, de façon à terminer la semaine le plus tôt possible ? (i.e. à minimiser le nombre de créneaux horaires dans lesquels il y aura des cours à dispenser)

Il y a trois notions dans notre problème :

- une heure de cours à dispenser
- l'un des 35 créneaux horaires possibles
- les contraintes apportées par les étudiants = les cours auxquels doit assister chaque étudiant

On va modéliser tout ça par un graphe où :

- chaque vertex représente une heure de cours à dispenser
- chaque edge entre deux nœuds représente le fait qu'un étudiant (peu importe lequel) doit assister à ces deux heures de cours

Alors, il suffit de trouver une proper coloration de ce graphe !

Chaque couleur représente l'un des 35 créneaux horaires disponibles : pour connaître les cours qui peuvent être dispensés en même temps sur un créneau horaire donné (i.e. une couleur donnée), il suffit de lister les vertex de cette couleur.

En effet, comme les vertex sont de la même couleur, ça veut dire qu'il n'y a aucun edge les reliant, et donc que pour tous ces vertex de même couleur (i.e. tous ces cours qui seront affectés sur le même créneau horaire), aucun étudiant ne doit assister à plusieurs de ces cours.

Réponse à la question 2 : pour terminer la semaine au plus tôt possible, il faut trouver la coloration minimale (i.e. celle où le nombre de couleurs est le nombre chromatique), car dans ce cas, on utilisera le moins de couleurs possibles (i.e. le moins de de créneaux horaires possibles).

Réponse à la question 1 : pour savoir s'il est physique possible de satisfaire tout le monde, il faut regarder si le nombre chromatique du graphe dépasse 35 : si oui, c'est que pour satisfaire tout le monde, on a besoin de plus de créneaux horaires que ceux à notre disposition, et certains étudiants ne pourront alors pas assister à tous leurs cours.

Si on prend un peu de recul, d'une façon générale, trouver une coloration permet de trouver une réparation des vertex en groupes, en s'assurant qu'au sein d'un même groupe, les vertex n'auront pas de liens entre eux.

## Graph Theory: 65. 2-Chromatic Graphs

Quels sont les graphes 2-chromatiques ?

Les trees sont 2-chromatiques ; en effet, chaque niveau du tree est d'une des deux couleurs.

L'inverse n'est pas vrai : C₄ (graphe cycle d'ordre 4) est 2-chromatique, mais n'est pas un tree.

Condition nécessaire et suffisante pour qu'un graphe soit 2-chromatique = le graphe est bipartite.

Ndm : j'avais pas fait attention mais tout tree est bipartite :-)

## Graph Theory: 66. Basic Bound on the Chromatic Number

Théorème : `χ(G) ≤ Δ(G) + 1`

Théorème de Brooks = les deux seuls cas où l'égalité est atteinte sont Kn et Cn = les graphes cycles d'ordre impair. Dans tous les autres cas, on a `χ(G) ≤ Δ(G)`

La formulation [sur wikipedia](https://fr.wikipedia.org/wiki/Th%C3%A9or%C3%A8me_de_Brooks) :

> Dans tout graphe connexe non orienté G de degré maximal `Δ`, le nombre chromatique `χ(G)` vérifie `χ(G) ≤ Δ`, sauf si G est un graphe complet ou un cycle de longueur impaire, auquel cas `χ(G) = Δ + 1`.
