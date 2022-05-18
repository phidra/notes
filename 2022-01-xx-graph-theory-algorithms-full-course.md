# Algorithms Course - Graph Theory Tutorial from a Google Engineer

- **url** = https://www.youtube.com/watch?v=09_LlHjoEiY
- **type** = vidéo YouTube
- **auteur** = [William FISET](https://www.williamfiset.com/), ingénieur chez Google qui semble orienté "contenu éducationnel" : [son github](https://github.com/williamfiset), [sa chaîne youtube perso](https://www.youtube.com/channel/UCD8yeTczadqdARzQUp29PJw), [son linkedin](https://www.linkedin.com/in/williamfiset)
- **date de publication** = 2019-10-09 (mais c'est la date de la grosse vidéo de 7h englobante : chaque vidéo était publiée plus tôt, e.g. [celle-ci](https://www.youtube.com/watch?v=eQA-m22wjTQ&list=PLDV1Zeh2NRsDGO4--qE8yH72HFL1Km93P&index=3) en mars 2018)
- **source** = [la chaîne YouTube de freeCodeCamp.org](https://www.youtube.com/channel/UC8butISFwT-Wl7EV0hUK0BQ), mais [la chaîne YouTube de William FISET](https://www.youtube.com/c/WilliamFiset-videos) a l'air aussi très intéressante !
- **tags** = language>none ; topic>graph ; topic>graph-theory ; topic>maths ; level>beginner


```diff
@@ WIP : je n'ai pas fini de regarder toute la vidéo + ces notes sont en cours de rédaction... @@
```

**TL;DR** : Notes très brutes sur la série de vidéos de William FISET sur la théorie des graphes.

**CONTEXTE + REPRISE** : en gros, le travail est en pause, mais n'est pas terminé, ces notes sont incomplètes :

- en janvier 2022, j'ai commencé à regarder cette excellente série de vidéos
- j'ai pris des notes très brutes sur ce que j'avais déjà visionné, mais il me reste encore plus de la moitié de la vidéo à regarder
- j'ai converti une grande partie des notes très brutes en notes un-peu-moins-brutes markdown, ci-dessous
- **REPRISE** = si besoin, relire ces notes pour me remettre le sujet en tête
- **REPRISE** = convertir le reste des notes très brutes en notes un-peu-moins-brutes markdown (cf. les `TODO` et autre `REPRENDRE LES NOTES ICI` ci-dessous)
- **REPRISE** = poursuivre le travail d'annotation + pérennisation des notes sur le reste de la vidéo (l'endroit exact où reprendre est indiqué plus bas)

* [Algorithms Course - Graph Theory Tutorial from a Google Engineer](#algorithms-course---graph-theory-tutorial-from-a-google-engineer)
   * [Topological sort](#topological-sort)
   * [Shortest Path problems = SSSP + APSP](#shortest-path-problems--sssp--apsp)
      * [SSSP sur DAG](#sssp-sur-dag)
      * [Dijkstra pour SSSP](#dijkstra-pour-sssp)
      * [Bellman-Ford pour SSSP](#bellman-ford-pour-sssp)
      * [Mes réflexions sur Bellman-Ford](#mes-réflexions-sur-bellman-ford)
         * [Approche en programmation dynamique](#approche-en-programmation-dynamique)
         * [Implémentation conceptuelle de la relation de récurrence](#implémentation-conceptuelle-de-la-relation-de-récurrence)
         * [Autre façon d'aborder l'algo conceptuel = avec les mains](#autre-façon-daborder-lalgo-conceptuel--avec-les-mains)
         * [Implémentation réelle = pourquoi n'utilise-t-on pas V tableaux de d[k] mais un seul](#implémentation-réelle--pourquoi-nutilise-t-on-pas-v-tableaux-de-dk-mais-un-seul)
         * [Notes à trier](#notes-à-trier)
      * [Floyd-Warshall pour APSP](#floyd-warshall-pour-apsp)

Dans ces notes, j'essaye de mettre **en gras** les notions importantes (mais je ne reporte pas systématiquement leur définition).

J'ai essayé de découper le contenu en titres, mais du contenu intéressant peut être disséminé ailleurs :  ce n'est pas parce qu'il n'existe pas de titre pour une notion qu'elle n'est pas présente quelque part dans ces notes.

Note : pour les termes de théorie des graphes, ces deux ressources peuvent également être intéressantes :

- en français : [lexique de la théorie des graphes](https://fr.wikipedia.org/wiki/Lexique_de_la_th%C3%A9orie_des_graphes)
- en anglais : [glossary of graph theory](https://en.wikipedia.org/wiki/Glossary_of_graph_theory)


**Colouring** = attribuer un id à un node = labeliser un node.


## Topological sort

**Topological sort a.k.a. topsort** = construire le graphe de dépendance = ordonner les nœuds du graphe de sorte qu'un nœud apparaît systématiquement _après_ ses dépendances.

Un topsort n'est possible que si le graphe orienté n'a pas de dépendance circulaire (en d'autres termes, un topsort n'est possible que sur un DAG)

Algo pour trouver un topsort sun un DAG (plusieurs résultats valides possibles) :

- on va construire le tri topologique (inversé) = une liste ordonnée des nœuds ; à l'initialisation, celle-ci est vide.
- répéter les étapes ci-dessous jusqu'à avoir visité tous les nœuds
- choisir un nœud non visité au hasard, et faire un DFS depuis celui ci
- à chaque fois que le DFS backtracke, on append le nœud au tri topologique inversé
-  à la fin on inverse le tri topololique inversé, pour récupérer le tri topologique dans le bon ordre

Le topsort étant basé sur un DFS, il s'exécute donc en `O(V+E)`.

## Shortest Path problems = SSSP + APSP

**SSSP = Single Source Shortest Path problem** = trouver les plus courts chemins depuis un _unique_ nœud source vers _tous_ les autres nœuds du graphe.

**APSP = All Pairs Shortest Path problem** = trouver les plus courts chemins entre _chaque_ paire de nœuds du graphe.

### SSSP sur DAG

01:11:00 sur un DAG, le SSSP peut être résolu efficacement en `O(V+E)` :

- initialisation = on maintient un tableau associant à chaque nœud sa tentative distance depuis le nœud source, initiliasée à `0` pour le nœud source, et à `+∞` partout ailleurs
- faire un topsort
- traiter les nœuds séquentiellement dans l'ordre du topsort
- relaxer les edges de chaque nœud (i.e. mettre à jour les tentative distances de chaque nœud)
- c'est tout (en effet, le DAG garantit qu'il n'y a pas de meilleure distance que celle trouvée)
- en sortie, on a les plus courtes distances entre un nœud source et tous les autres nœuds

01:15:40 rechercher le longest path est un problème NP-hard dans le cas général, mais `O(V+E)` sur les DAG. En effet, en multipliant les poids par `-1`, on se ramène à un SSSP.

### Dijkstra pour SSSP

01:19:30 **Dijkstra** est un bon algo pour le SSSP, mais il est limité aux graphes dont les edges ont des poids _POSITIFS_ uniquement.

01:34:45 insérer plusieurs fois un même node dans la priority queue (ce qui arrive quand on veut améliorer sa tentative distance) est la façon _LAZY_ de faire. La façon _eager_ utilise une prority queue particulière, qui dispose d'une opération `DecreaseKey` logarithmique : **Indexed Priority Queue (IPQ)** (j'ai pas regardé en détail, mais William FISET a [une vidéo dédiée au sujet](https://www.youtube.com/watch?v=jND_WJ8r7FE)).

01:39:50 Dijkstra avec **D-ary heaps**. Un D-ary heap est une généralisation d'un heap "classique" (qui est un 2-heap, du coup) : c'est un heap où chaque nœud a `D` fils.

Il y a donc moins de niveaux dans l'arbre, mais plus de nœuds par niveau : c'est donc une data structure qui accélére les opérations `DecreaseKey` (puisque faire remonter/descendre des nœuds a moins de niveaux à traverser) au détriment des opérations `GetMinElement` (qui doit rechercher le fils le plus petit de chaque niveau).

Les D-ary heaps sont donc intéressants pour un Dijkstra sur un graphe dense, puisque dans ceux-ci, un nœud sera accessible par beaucoup de chemins différents, et qu'on va donc souvent mettre à jour la tentative distance d'un nœud. D'après la vidéo, la bonne valeur de `D` à choisir pour son heap est `E/V`.

01:42:30 **Fibonacci heap** = asymptotiquement le meilleur, mais comme il a de lourdes constante (sans parler d'une implémentation complexe), il n'est adapté qu'à de très gros graphes.

Dijkstra = algorithme greedy car il sélectionne à chaque itération le nœud le plus prometteur.

### Bellman-Ford pour SSSP

01:50:45 **Bellman-Ford** = algo Dynamic programming. Complexité nettement moins bonne que Dijkstra puisque Bellman-Ford est `O(EV)`. Son intérêt = il autorise les poids négatifs, et permet de détecter les cycles négatifs (i.e. les cycles dont la somme des poids est négative).

Comment fonctionne l'algo :

- on maintient un tableau associant à chaque nœud sa tentative distance depuis le nœud source (initialisé à `+∞` partout, sauf pour le nœud source initialisé à `0`)
- pour trouver tous les shortests paths, on relaxe tous les edges du graphe, et ce `V` fois
- (optimisation pour le SSSP uniquement, sans s'intéresser aux cycles négatifs = on peut s'arrêter dès qu'une passe de relaxation de tous les edges du graphe n'améliore plus rien)
- à ce stade, s'il n'existe pas de cycle négatifs, les distances des nœuds ne peuvent plus être améliorés
- pour détecter les cycles négatifs, on relance l'algorithme (entièrement : avec les `V` passes) une deuxième fois : les nœuds améliorés par ce deuxième run sont atteignables par un négative cycle → leur distance minimale depuis la source est donc `-∞`, vu qu'on peut emprunter autant de fois que nécessaire le cycle négatif avant de les rejoindre.

Note : si on laisse de côté les considérations d'optimisations et qu'on se concentre sur la correctness de l'algo, il n'est pas nécessaire d'imposer un ordre particulier pour relaxer les edges : comme on les relaxe `V` fois, on est sûr de trouver le bon chemin, même si on a relaxé dans le pire ordre.

### Mes réflexions sur Bellman-Ford

Sous ce paragraphe, il n'y a que des notes à moi (i.e. elles sont moins fiables que les informations directement issues de la vidéo).

#### Approche en programmation dynamique

Même si ça n'est pas limpide à la lecture de l'algo, Bellman-Ford est un algo de programmation dynamique, la [page wikipedia](https://fr.wikipedia.org/wiki/Algorithme_de_Bellman-Ford) donne plus d'infos, et notamment la relation de récurrence :

- PROBLÈME : SSSP = à partir d'un nœud SOURCE `s` fixé, calculer pour CHAQUE nœud du graphe target `t` le coût du plus court chemin
- NOTATION : `d[t,k]` est la plus courte distance du nœud source `s` à `t` avec un chemin qui contient au plus `k` arcs
- INITIALISATION : `d[t,0] = +∞` pour `t ≠ s`  et `d[s,0] = 0`
- pour un `k` quelconque,  `d[t,k]` est la plus petite de ces deux valeurs :
   1. `d[t,k-1]`
   2. `min` sur tous les arcs `(u,t)` de `d[u,k-1] + poids(u,t)`

Le permier sous-cas (où le min est `d[t,k-1]`) signifie que le PCC vers `t` trouvé jusqu'ici, en utilisant `k-1` edges (ou moins) reste le plus intéressant. Dit autrement, utiliser `k` edges ne nous permet pas d'améliorer le meilleur PCC jusqu'ici. À l'inverse, le second cas correspond au fait qu'en utilisant `k` edges au lieu de `k-1` edges (ou moins), on arrive à améliorer le meilleur PCC jusqu'ici.

Comme le plus court chemin a _au plus_ `V-1` edges (en effet, un PCC à `V` edges n'est pas possible, sans quoi ce serait un cycle), quand on a itéré `V-1` fois (i.e. quand `k = V-1`) on peut arrêter l'algo. Dit autrement, la plus courte distance du nœud `s` au nœud `t` est au pire `d[t, V-1]`.

En résumé : la relation de récurrence porte sur le nombre d'edges dans le PCC. Si en incrémentant `k`, il existe un chemin vers `t` avec `k` edges encore plus court qu'avec seulement `k-1` edges, alors c'est notre nouveau PCC vers `t`.

#### Implémentation conceptuelle de la relation de récurrence

ATTENTION : l'implémentation ci-dessous est CONCEPTUELLE, et ne sert qu'à comprendre la relation de récurrence et ses implications. En  vrai, on implémente Bellman-Ford un peu différemment...

- Initialisation (`k=0`) :
   - On alloue `V` tableaux de nœuds `d[k]` de `0` à `V-1`, chacun va stocker les les plus courtes distances de `s` à chacun des nœuds, en n'utilisant que `k` edges.
   - On initialise l'algo en remplissant le premier tableau `d[0]` : c'est le tableau des plus courtes distances depuis `s` vers tous les nœuds `t` en empruntant `0` edges (ou moins :-P). Fort logiquement, il contient des `+∞` pour toutes les cellules, sauf pour la cellule de `s` qui contient `0`.
- Première itération (`k=1`) :
   - On cherche à calculer la valeur de toutes les cellules de `d[1]` (i.e. les plus courtes distances depuis `s` en empruntant `0` ou `1` edges), sachant qu'on connaît les plus courtes distances vers tous les nœuds en empruntant `0` edges.
   - Pour cela, on va ajouter un edge supplémentaire à tous les nœuds, et voir si on améliore le coût d'arrivée à la target.
   - On itère sur tous les edges `(u,v)` du graphe, et on regarde si on ne rejoindrait pas `v` plus vite qu'auparavant si on ajoutait cet edge supplémentaires à la plus courte distance vers `u` qui empruntait `0` edges.
   - Dit autrement : compte-tenu de la valeur de `u` et de `v` dans `d[0]`, ne serait-il pas plus rapide de rejoindre `v` via `u` ?
   - Dit encore autrement, on regarde si `d[u,0] + poids(u,v)` (ce qui est donc un chemin vers `v` à `1` edge) ne serait pas moins coûteux que `d[v,0]` (qui était un chemin vers `v` à `0` edges) ? (si oui, on a réussi à améliorer notre meilleure distance de `s` à `v` jusqu'ici ; si non, cette meilleure distance reste la même à `k` edges qu'à `k-1` edges du point de vue de l'edge `(u,v)`)
   - En faisant ça pour tous les edges du graphe, on est capable de calculer toutes les cellules de `d[1]` = les plus courtes distances depuis `s` vers tous les nœuds `t` qui empruntent exactement `1` edge.
- Quels sont les edges améliorés par cette première itération :
   - d'une façon générale, on ne peut pas améliorer les `v` pour les edges `(u,v)` qui partent d'un `u` dont la distance est `+∞` : en effet, il n'est PAS plus intéressant d'emprunter l'edge supplémentaire `(u,v)` vu que `d[u]` est infini à la base
   - à l'issue de cette première itération, on ne pourra donc PAS améliorer la distance de `v` si `d[u,0] = +∞`
   - donc les seuls edges qui peuvent améliorer `v` sont ceux pour lesquels `d[u,0] ≠ +∞`... C'est à dire les edges en provenance du nœud source `s`, car seul `s` vérifie cette inégalité !
   - on vient simplement de traduire mathématiquement le fait que "_si on se restreint à des chemins qui n'ont qu'un seul edge, alors les seuls nœuds dont on peut calculer la plus courte distance depuis `s` sont les voisins immédiats de `s`_" :-D
- Seconde itération (`k=2`) :
   - on commence une nouvelle itération : on cherche à calculer `d[2]` = les plus courtes distances depuis `s` en empruntant `1` ou `2` edges au max... sachant qu'on connaît déjà les plus courtes distances depuis `s` vers chaque node en empruntant exactement `1` edge
   - Pour cela, on va ajouter un edge supplémentaire à tous les nœuds (ce qui fera des chemins plus longs d'un edge), et voir si on améliore le coût d'arrivée à la target.
   - pour ce faire, on va relaxer chaque edge en utilisant la distance de `u` limitée à `1` edge, et en lui ajoutant l'edge `(u,v)` : le chemin ainsi calculé sera un chemin de `s` vers `v` empruntant `2` edges, et on regardera si ce nouveau chemin à `2` edges vers `v` était plus court que le meilleur jusqu'ici à `1` edge.
   - quels seront les edges améliorés ? Ici aussi, pour la plupart des edges, la "distance de `u` limitée à `1` edge" vaut `+∞` et il ne sera donc jamais plus intéressant d'emprunter l'edge `(u,v)` pour rejoindre `v`...
   - les seuls edges qui pourraient améliorer le coût de `v` sont ceux pour lesquels `d[u,1] ≠ +∞`... C'est à dire les edges qui partent d'un nœud voisin du nœud source `s` (i.e. un nœud qui avait été amélioré lors de la première itération), donc les nœuds qui se situent à deux edges du nœud source (au plus)
- Troisième itération (`k=3`) :
   - on fait pareil : on connaît les plus courts chemins empruntant `2` edges (a.k.a `d[2]`), et on ajoute un edge à tous les nœuds pour voir si les chemins à `3` edges sont plus courts.
- etc.

CONCLUSION IMPORTANTE : conceptuellement, on voit que les itérations de Bellman-Ford "propagent" la mise à jour des plus courtes distances edge après edge depuis la source, en utilisant de plus en plus d'edges à chaque itération.

NOTE : quel lien entre la relation de récurrence et cette implémentation conceptuelle, et notamment pourquoi l'implémentation itère sur tous les edges alors que la relation de récurrence de le mentionne pas ? Dans la relation de récurrence, on calculait, pour chaque node `t` le `min` suivant :

> `min` sur tous les arcs `(u,t)` de `d[u,k-1] + poids(u,t)`

Stricto-sensu, cette ligne revient pour chaque node `t` à itérer sur tous les edges qui ont `t` comme nœud target. Il faudrait donc faire quelque chose comme :

```python
for t in graph.nodes():  # pour chaque node t ...
    #...on itère sur tous les edges qui ont t comme target :
    for edge in [e for e in graph.edges() if e.target == t]:
        # relaxer l'edge et regarder si on améliore la plus courte distance vers t
```

Mais en pratique, cette double boucle peut-être remplacée par une simple boucle sur tous les edges, qui est équivalente :

```python
# on itère sur tous les edges, ce qui couvrira tous les edges qui ont t comme target, pour chaque node t :
for edge in graph.edges():
    t = edge.target
     # relaxer l'edge et regarder si on améliore la plus courte distance vers t
```

#### Autre façon d'aborder l'algo conceptuel = avec les mains

Avec les mains (deux exemples concrets sont donnés plus bas) :

- on va faire plusieurs passes, pour propager les chemins depuis le nœud source
- pour cela, à chaque passe, on relaxe tous les edges dont le nœud source a déjà été atteint depuis la source
- la première passe ne relaxe que les edges voisins du nœud source ; les passes suivantes relaxent des edges de plus en plus loin du nœud source
- critère d'arrêt : quand on a suffisamment propagé de sorte que tous les nœuds ait été atteints, continuer à faire des passes n'améliorera plus rien → on peut arrêter l'algo

Quelques points d'attention :

- ici aussi, l'implémentation décrite a un but pédagogique : en vrai, on implémente Bellman-Ford un peu différemment...
- pour que ce soit plus intuitif, j'ai introduit une différence avec l'implémentation conceptuelle de la relation de récurrence présentée au paragraphe précédent :
    - dans l'implémentation conceptuelle précédente, on relaxait tous les edges à chaque passe, ce qui était sans effet pour les edges dont le nœud d'origine n'était pas encore atteint
    - dans l'implémentation avec les mains, on ne relaxe que les edges dont le nœud d'origine a déjà été atteint (i.e. le score du nœud d'origine n'est pas `+∞`) ; en d'autres termes, on n'effectue pas les opérations sans effets
    - (pour moi, on pourrait même pousser le concept plus loin, et se limiter à ne relaxer que les edges dont le nœud d'origine a été amélioré à l'itération précédente)
- contrairement à Dijkstra, on ne "choisit" pas les edges à relaxer, chaque passe les relaxe tous (ou de façon équivalente : ne relaxe que ceux dont le nœud source a été atteint depuis la source), ce qui a des avantages et des inconvénients :
   - inconvénient = contrairement à Dijkstra, ça n'est qu'à la fin de l'algo qu'on est sûr qu'un nœud a sa distance définitive (alors que dans Dijkstra, un nœud settle ne sera plus amélioré)
   - avantage = comme on relaxe tous les edges sans se poser de question, les poids négatifs n'empêchent pas l'algo de trouver les plus courts chemins (tant qu'il n'y a pas de cycle négatif), alors que dans Dijkstra, on utilise une priority-queue en extrayant le nœud min car on a la certitude que tous les nœuds qui sont moins bons que lui LE RESTERONT, car chaque edge non-encore relaxé ne fera qu'AUGMENTER le score d'un nœud. Si cette propriété n'est plus vrai parce qu'un edge a un poinds négatif, Dijkstra devient faux.
- dans le cas général, on est obligé de faire `V-1` passes, pour s'assurer d'avoir propagé les scores le long de tout le plus court chemin, qui peut avoir jusqu'à `V-1` edges. Illustration avec un cas concret, où on veut calculer le PCC de `S` à `T` :
   - note : pour simplifier les explications ci-dessous, on suppose que l'edge `TC` est à sens unique de `C` vers `T` : une fois arrivé à `T`, on ne peut aller nulle-part
   ```
               10
   S------------------------T
    \                      /
     \ 1                  / 1
      \                  /
       A-------B--------C
           1        1
   ```
   - la première passe relaxe `SA` et `ST` : dès la première passe, on a trouvé UN chemin vers `T`, de poids `10` (qui est sous-optimal)
   - la seconde passe relaxe `AB` (stricto sensu, elle relaxe de nouveau `SA` et `ST` également, mais pour simplifier l'explication, je me cantonne à mentionner la relaxation des edges qui améliorent le score de leur nœud de destination)
   - la troisième passe relaxe `BC`
   - la quatrième passe relaxe `CT` et améliore le score de `T` → ce n'est qu'à cette quatrième passe qu'on trouve le chemin optimal vers `T`, de poids `4`.
   - on voit que dans la pire situation où le PCC emprunte tous les nodes du graphe, il nous faudra faire `V-1` passes pour le trouver (sans quoi, on ne pourra avoir qu'un chemin sous-optimal, voire pas de chemin du tout dans le cas du path-graph de `S` vers `T`) : il faut laisser à l'algo le temps de propager suffisamment loin pour trouver le PCC
- contrairementà Dijkstra, il est parfois indispensable de relaxer plusieurs fois un edge :
   - en effet, à chaque fois que son nœud d'origine a été amélioré, il faut relaxer de nouveau l'edge pour "propager" cette amélioration à son nœud de destination (c'est une conséquence du point précédent, en fait)
   - pour l'illustrer, on modifie un poil notre graphe précédent (ici aussi `CD` est à sens unique vers `D`, et on se cantonne à mentionner les edges qui améliorent leur nœud de destination) :
   ```
               10                1
   S------------------------D----------T
    \                      /
     \ 1                  / 1
      \                  /
       A-------B--------C
           1        1
   ```
   - la première passe relaxe `SD` et `SA`
   - la seconde passe relaxe `AB` et `DT` → on a trouvé UN chemin vers `T`, de poids `11` (qui est sous-optimal).
   - la troisième passe relaxe `BC`.
   - la quatrième passe relaxe `CD` et améliore le score de `D`.
   - la cinquième passe RELAXE DE NOUVEAU `DT` et améliore le score de `T` → on a enfin trouvé le chemin optimal vers `T` de poids `5`.
   - on voit que dès que le score d'un nœud `N` a été amélioré lors d'une passe, il faut relaxer de nouveau tous les out-edges de `N` à la passe suivante : il est donc parfois indispensable de relaxer plusieurs fois un même edge.

#### Implémentation réelle = pourquoi n'utilise-t-on pas `V` tableaux de `d[k]` mais un seul

La principale différence par rapport à l'implémentation conceptuelle, c'est qu'on n'utilise pas `V` tableaux `d[k]`, mais un seul :

```python
# initialisation de d à +∞ partout, et à 0 pour s
for k in range(graph.num_nodes() - 1):  # on fait V-1 passes
    for edge in graph.edges():
        # on relaxe l'edge :
        candidate_score = d[edge.origin] + edge.weight
        if d[edge.target] > candidate_score:
            d[edge.target] = candidate_score
```

Note : tout comme avec Dijkstra, si c'est le chemin complet qui nous intéresse (et non uniquement son coût), on peut maintenir un tableau des prédécesseurs.

Vu l'implémentation conceptuelle, et l'implémentation réelle, la complexité est en `O(V*E)`.

Pourquoi n'utiliser qu'un tableau `d` ?


On dirait que networkx utilise une variante : 

https://en.wikipedia.org/wiki/Shortest_Path_Faster_Algorithm

TODO


#### Notes à trier

TODO TODO = si je résume ce qu'il faut que je précise ici, ça donne :

- comment on implémenterait l'algo pour appliquer la relation de récurrence (et notamment, le fait que le MIN revient à itérer sur tous les edges) ; à ce stade, il faudrait DEUX tableaux de distances : d[k] et d[k-1]
- le fait qu'on utilise qu'un seul d[k]
- explications avec les mains sur un graphe
- notamment, mon histoire du fait que selon l'ordre dans lequel on itère sur les edges, il faut plus ou moins de passes (pour illustrer ça, un path-graph est parfait)
- dire que la page wikipedia mentionne des optimisations, qui concernent surtout l'ordre selon lequel on traite les nodes (et again : vu mon illustration sur un path-graph, ça fait sens)

REPRENDRE LES NOTES ICI
REPRENDRE LES NOTES ICI
REPRENDRE LES NOTES ICI
REPRENDRE LES NOTES ICI
REPRENDRE LES NOTES ICI

NDM : ici, il faut que j'explique pourquoi ça marche, et pourquoi il faut `V` passes...

TODO = expliquer la relation de récurrence de l'algo, et expliquer le lien avec l'algo (et notamment le fait que pour respecter exactement la relation de récurrence, on pourrait plutôt maintenir DEUX tableaux des tentatives distances : l'un pour le `k` courant, l'autre pour `k-1`, mais qu'en pratique, on peut commencer à mettre à jour celui de `k-1` et l'utiliser pour la comparaison.


Ndm L'idée derrière est qu'en relaxant (rappel : en relaxant un edge, on met à jour la distance de son targetnode, en l'améliorant) tous les edges du graphe autant de fois qu'il y a de nœuds sur le plus court chemin, on est sûr de le trouver. Si le plus court chemin nécessité de passer par TOUS les vertex du graphe, on aboutit au pire temps = O(EV)

Question = pourquoi doit on lancer l'algo `V` fois ?

TODO = expliquer que si on relaxe les edges dans un mauvais ordre, on n'aura pas forcément le plus court chemin.

TODO = donner un graphe exemple, avec un edge de source à une target, et un autre chemin avec plein d'edges, et montrer que selon l'ordre qu'on prend pour relaxer les edges, on trouve ou non le PCC sur le premier run. À l'inverse, si on relaxe les edges `V` fois, même dans le pire cas, on trouve le PCC.

QUESTION = pourquoi doit on RELANCER l'algo `V` fois ?

Même principe : dans le pire cas, le negative cycle nécessite de passer par TOUS les edges du graphe pour avoir un poids négatif. Du coup, si on a pas de bol sur l'ordre selon lequel on itère sur les edges, i.e. si on est dans le pire cas, on aura besoin de `V` passes pour emprunter le cycle négatif en entier dans le bon ordre (et donc améliorer encore le poids d'une tentative distance).

NOTE a posteriori : si on veut juste savoir s'il y a au moins un cycle de poids négatifs, on peut ne relancer l'algo qu'avec une seule itération. Si on veut connaître tous les nœuds de poids `-∞`, il faut relancer l'algo avec `V` itérations.

TODO = illustrer

### Floyd-Warshall pour APSP

02:05:20 Floyd-Warshall pour APSP. Dynamic programming

Peut détecter les négative cycles.

Ndm, ça peut m'intéresser pour trouver les nœuds les plus importants (en terme de PCC) sur un graphe, et ainsi calculer un bon ordering ?
O(V3) , donc sans doute pas applicable sur des graphes Europe...

Meilleure data structure pour FW = ADJACENCY MATRIX

L'idée est de construire tous les PCC entre deux nœuds quelconques, dont le chemin n'est constitué que du node 0. Puis, que des nodes 0 et 1. Puis que des nodes 0, 1, et 2. Quand on en arrive à ceux utilisant les nodes de 0 à n-1, on a trouvé TOUS les PCC possibles

La relation de récurrence DP est donnée à 02:12:00 , et Grosso modo, le meilleur chemin entre les nœuds i et j utilisant k nœuds intermédiaires, c'est le min de :
---> Soit le meilleur chemin entre les nœuds i et j utilisant k-1 nœuds intermédiaires
---> Soit la somme du meilleur chemin entre les nœuds i et k utilisant k-1 nœuds intermédiaires, et du meilleur chemin entre les nœuds  k et jutilisant k-1 nœuds intermédiaires (en gros, on regarde si en s'autorisant maintenant à passer par k, on n'irait pas par hasard plus vite que quand on était limité aux k-1 premiers nœuds)
Et on réutilise donc à l'étape k les résultats calculés à l'étape k-1

Note : attention à ne pas sommer un poids infini avec autre chose (intéger overflow).

L'initialisation suppose qu'on n'a que des chemins directs (de taille 1 edge) entre chaque paire de nœud.

On dirait qu'à l'inverse de Dijkstra, pour reconstruire les chemins, on ne stocke pas le prédécesseur d'un node, mais le next node d'une paire de nœud : initialement, le next node d'une paire i,j est j (vu qu'initialement, les PCC n'ont qu'un seul edge), et dans la boucle principale de l'algorithme, on remplace le next de i,j par le next de i,k si c'est pertinent de passer par k

Après exécution de Floyd-Warshall, on peut faire un traitement (optionnel si on sait qu'il n'y a pas d'edges de poids négatifs) pour détecter les cycles négatifs. Même principe que pour BF : on run l'algo une deuxième fois (entièrement) et les paires de nœuds améliorées voient leur poids settés à moins infini







02:29:15 = bridges et articulation points

Bridge = si on remove edge,le nombre de connected component augmente
Idem pour articulation points
Ce sont des "weak points" dans un graphe.

02:33:0 explication de l'algo pour trouver des bridges dans un graphe undirected, conceptuellement : 
D'abord, on fait un dfs, en transformant les edges en edges orientés (par le dfs qu'on fait), et attribuant un id à chaque nœud, qui incrémente (en commençant à zéro) pour chaque nouveau nœud rencontré.
Puis, on réitère sur chaque nœud de notre dfs, et on attribue à chacun la lowlink value = id du plus petit nœud atteignable depuis ce nœud (inclus)
Enfin, les bridges sont les edges qui vont d'un lowlink de départ à un lowlink d'arrivée SUPÉRIEUR

Ndm : on dirait que la clé est dans la transformation du graphe undirected en graphe orienté : en traversant les bridges, on transforme ainsi le graphe fortement connexe en graphe "simplement" connexe, mais fortement disjoint. Du coup, ça impacte le lowlink de tous les nœuds derrière le bridge, vu qu'on ne peut plus atteindre la composante fortement connexe de départ depuis la composante fortement connexe d'arrivée, vu que le bridge a été orienté en sens unique de la première vers la seconde.
Du coup, le lowlink de la composante en aval du bridge sera forcément plus grand que celui de la première.
(Si ça n'est pas le cas, c'est qu'il existe un autre edge pour REVENIR dans la première composante, et que donc l'edge n'était pas un bridge)

Cette description (avec un dfs initial pour attribuer les ids + autant de dfs que de nodes pour trouver le plus petit node atteignable, donc en O de V*(V+E)...) n'est que conceptuelle, car pour implémenter efficacement, on ne fait qu'un seul dfs qui fait tout d'un coup.

Ndm : j'avais l'impression d'avoir une bonne idée pour implémenter l'algo, mais la vidéo ( ou cet article : https://cp-algorithms.com/graph/bridge-searching.html) ne suive pas mon idée...
Ah, en fait, avec mon idée (de maintenir un set des nœuds courants, auquel on attribue le plus petit lowlink possible dès qu'on essaye de relaxer un edge vers un node déjà visité) est incomplète : pour être complet, il faut Maintenir non pas uniquement le set, mais tout le tree. Donc : retenir le parent d'un node.

Articulation points : soit une extrémité d'un bridge, va soit les deux sont des articulation points. Mais ça n'est pas suffisant : il peut en exister sans bridge !

Du coup, pour les détecter, Grosso modo on détecte les cycles dans le dfs (en détectant qu'on essaye d'étudier une lowlink value à un node égale au node lui même : en gros, on est "revenu" au node). Il y a une merdouille avec le premier Noeud du dfs.

02:57:30 Tarjan pour détecter les strongly connected components (SCC)

Ici aussi, on utilise la lowlink value = plus petit id (incluant le nœud lui même) atteignable par un node dans un dfs (ndm : dans un dfs qui oriente les edges)

Comme ça dépend pas mal du premier dfs conceptuel (celui qui attribue un id aux nodes), Tarjan a une modif : il maintient une stack de nodes utilisable pour trouver le plus petit lowlink (et chaque stack frame contient les nodes d'une SCC en cours). On ne peut diminuer la lowlink value d'un node QUE vers la value d'un node dans la stack.

Tout comme pour la détection des bridges, l'algo décrit plus haut est conceptuel, mais l'implémentation réelle fait tout en un seul dfs

03:03:00 résumé de Tarjan que je trouve clair.

03:04:30 très bonne animation expliquant Tarjan

Ndm : il parle souvent de DFS callback , pour désigner le fait de faire une action quand le dfs a fini d'explorer tout un sous arbre et qu'il faut backtracker. Ici, la callback update la lowlink value au min de la nôtre, et du node d'où l'on vient. Ça permet que quand le dfs retombe sur un node déjà visité (il a bouclé, quoi), on rétro-propage récursivement  sa faible lowlink à tous le sous graphe.

Quand la lowlink value minimale qu'on rétro-propage atteint le node de même id que cette lowlink, on a fini de traiter la SCC, on poppe tous ses nœuds de la stack frame autorisée.

Reprendre à 03:20:10 travelling salesman problème
