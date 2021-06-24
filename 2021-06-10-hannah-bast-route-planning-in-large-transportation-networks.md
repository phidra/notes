# ICAPS 2018: Hannah Bast on "Route Planning in Large Transportation Networks: Surprisingly Hard and Surprisingly interesting"

- **url** = https://www.youtube.com/watch?v=B3wKfJAVRkg
- **type** = vidéo
- **auteur** = [Hannah BAST](https://ad.informatik.uni-freiburg.de/staff/bast) ([google scholar](https://scholar.google.com/citations?user=hqSjLE8AAAAJ&hl=en)), notamment co-autrice du survey de 2016 sur le route-planning.
- **date de publication** = talk présenté le 2018-06-28 (vidéo mise en ligne le 2019-06-11)
- **source** = [la chaîne youtube](https://www.youtube.com/channel/UCJhNku53nSvPZc4ez5l9T1Q), de la conf [ICAPS](https://www.icaps-conference.org/) = International Conference on Automated Planning and Scheduling
- **tags** = language> none ; algorithms ; topic>route-planning ; level>advanced

**TL;DR** : intéressant talk d'Hannah BAST sur les techniques de calcul d'itinéraires, avec la même structure que le survey de 2016 = road-network d'abord, public-transit network ensuite.

# Notes vrac

- 02:00 le talk se concentre sur les général purpose algorithmes
- 02:40 relaxer un edge = calculer le coût pour joindre le head-node via cet edge, et mettre à jour le tentative-cost du head-node si cet edge l'améliore.
- 04:30 jusqu'à ce qu'on ait trouvé la target, il faut settle TOUS les nodes. Rigolo : c'est en fait une feature souhaitée pour le one to many (mais un problème pour le 1 to 1)
- 05:30 Elle confirme que le bidirectional peut s'arrêter dès qu'un node est settled (puis il faut prendre le meilleur chemin parmi TOUS les settled nodes, et pas uniquement le meeting node). Divise le search Space par 2.
- 8:30 intéressante présentation de ALT.
- 16:00 contracter les nodes de degré 2 (i.e. sur lequel il n'y a pas de décision de routing à prendre) était un preprocessing classique même avant que les CH soient inventées
- 17:00 La raison pour laquelle on veut contacter les autoroutes en dernier, c'est qu'elles participeront sans doute à beaucoup de plus courts chemins. Si on les contactait en premier, on ajouterait dès le début beaucoup de shortcuts.
- 18:00 idéalement, on contracte en premier les nodes qui n'ajouteront aucun shortcut.
- 19:40 avec un bon orderering, le nombre de shortcuts est censé être petit devant le nombre d'edges originaux.
- 21:00 le search Space est réduit de façon logarithmique avec la taille du graphe
- 23:00 https://movingai.com/GPPC/
- 25:40 marche aussi en one-to-many et many-to-many (deux use cases qui ne marchent pas bien avec les goal-directed techniques)
- 27:00 MAIS pour que les ch soient efficaces, il faut une hiérarchie (peu clair, mais un peu mieux expliqué à 36:00 , puis à 56:30 = la présence d'axes importants, très empruntés par les plus courts chemins, tels que des autoroutes)
- 29:00 transit node routing facile à implémenter avec ch
- 31:00 elle s'intéresse maintenant à PTN
- 32:30 time-expanded model
- 33:10 time-dependent model
- 33:50 frequency based model
- 38:00 elle explique la relaxation des edges dans ces différents modèles
- 41:00 longue explication des transferts patterns
- 47:00 en fait, exploiter la hiérarchie du network, c'est également l'idée qui est derrière l'amélioration des transfer patterns à échelle nationale (qui utilise du clustering)
- 49:00 pas clair : si le graphe est acyclique, Dijkstra n'a plus besoin d'une priority queue (on ne peut pas revenir en arrière dans le temps)
- 54:00 question sur le all-pair-shortest-path-problem (qui la mets un peu mal à l'aise)
- 56:00 le caractère planaire des graphes n'a pas d'intérêt pour les CH (qui marchent donc très bien sur des graphes non planaires). Ce qui est important, c'est la hiérarchie.
