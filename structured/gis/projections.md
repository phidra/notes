* [Contexte](#contexte)
* [Résumé de ma compréhension du sujet en avril 2021](#résumé-de-ma-compréhension-du-sujet-en-avril-2021)
* [Expérimentations en septembre 2023](#expérimentations-en-septembre-2023)
   * [Conversion depuis 4326 vers 3857 et affichage sur QGIS](#conversion-depuis-4326-vers-3857-et-affichage-sur-qgis)
   * [Jeu de données en 3949](#jeu-de-données-en-3949)
   * [Système de coordonnées utilisé par QGIS](#système-de-coordonnées-utilisé-par-qgis)
   * [Calcul de distances](#calcul-de-distances)

# Contexte

- avril 2021 : j'essaye de mieux comprendre les notions en jeu derrière projection/systèmes de coordonnées/référentiel
- Exemple de phrase que j'aimerais mieux comprendre :
    > Toutes les données que nous avons à disposition voient les coordonnées géographiques encodées dans le système de coordonnées WGS 84 (EPSG:4326)
    >
    > Libre à nous de les transformer ensuite en Web Mercator (EPSG:3857).
    >
    > Quelle différence entre un référentiel, un système de coordonnées, et une projection cartographique ?
- septembre 2023 : je me réintéresse au sujet avec une situation concrète = comprendre les coordonnées utilisées dans un jeu de données ; je fais quelques expérimentations.

# Résumé de ma compréhension du sujet en avril 2021

Note : au vu des connaissances actuelles, le meilleur article est [celui-ci](https://medium.com/cr%C3%A9ation-dune-app-cartographique-avec-firebase-vue/comprendre-les-coordinates-reference-system-crs-b67a88bce63c).

Quel est l'objectif final :

- objectif 1 = comment repérer (=géolocaliser) des objets du monde réel (par exemple un clocher) ; comment indiquer à quelqu'un, de façon précise et non-ambigüe :
    > le clocher se trouve ICI
- objectif 2 = comment représenter le monde réel sur une carte

Pour géolocaliser des points de la terre, il faut les géolocaliser "sur quelque chose", i.e. par rapport à un **référentiel** :

- on ne peut pas les géolocaliser par rapport "au sol", car sinon, tous les points du sol auraient une altitude nulle, quelle que soit leur altitude
- on pourrait les géolocaliser par rapport au niveau de la mer (c'est presque ce qu'on fait), mais reste à définir ce qu'est ce "niveau de la mer" là où il n'y a pas de mer !

Comme c'est lui qui donne la direction du "bas" (par définition de ce qu'est "en bas"), le référentiel idéal serait le géoïde :
- **géoïde** = surface équipotentielle de référence du champ de pesanteur terrestre
- si la Terre était uniformément recouverte d’eau et qu’il n’y avait pas de marée/vent/vague/courant, la surface de la Terre coïnciderait avec le géoïde.
- il sert de zéro de référence pour les mesures précises d'altitude

Mais le géoïde est trop tarabiscoté pour être utilisé comme référence ; on utilise à la place un ellipsoïde

**ellipsoïde** = surface régulière (définie mathématiquement) qui lorsqu'elle est bien choisie (centre, dimensions, orientation...) s'écarte au maximum de quelques dizaines de mètres du géoïde.

Plusieurs ellipsoïdes sont possibles, en fonction des besoins :

- en effet, tous les ellipsoïdes étant des approximations, certains sont plus adaptés à tel endroit de la Terre qu'à tel autre
- la donnée précise d'un ellipsoïde, et son positionnement + orientation par rapport à la Terre est un **datum** :
    > Some of these datums are accurate only in certain locations in the world, as the ellipsoid they’re using is a great fit for the real earth surface in those locations, but may be way way off in other places.
- de nos jours, on utilise principalement l'ellipsoïde et le datum de WGS84 (= du GPS)

une fois qu'on a un référentiel sur lequel géolocaliser des objets, pour exprimer leur position, il faut un système de coordonnées

- par exemple, on définit un méridien d'origine, et on dit qu'on mesure les positions sur l'ellipsoïde avec latitude φ, longitude λ et élévation h
- ces coordonnées utilisant latitude+longitudes sont dites "**géographiques**"

**coordonnées projetées** :

- plutôt que de géolocaliser les objets réels sur l'ellipsoïde, on les géolocalise sur une projection de l'ellipsoïde sur un plan
- dans ce cas, plutôt que lat/long, on les appelle E,N (pour Earthing/Northing), ou bien X,Y
- elles disposent d'un domaine de validité : par exemple, EPSG:3857 est valable entre 85.06°S et 85.06°N
- ça permet par exemple de représenter les objets sur une carte, MAIS les coordonnées projetées ne servent pas uniquement dans le contexte du rendu sur une carte
- en effet : les coordonnées projetées (si elles sont en bijection avec les coordonnées géographiques) permettent elles aussi de géolocaliser les objets de façon non-ambigüe

L'ensemble de ellipsoïde+datum+système de coordonnées est un **système géodésique**, permettant de géolocaliser parfaitement les objets du monde réel.

En théorie, exprimer une position avec des lat/long NE SERT À RIEN si on n'a pas le système géodésique qui va avec :

- les coordonnées sont toujours liées à des hypothèses : quelle forme avons nous choisie pour la Terre et comment nous avons placé cette forme par rapport à la surface de la Terre.
- si on change d’hypothèses, un point change de coordonnées.
- par exemple : dans Le Trésor de Rackham Le Rouge, juste à partir des lat/long, ils se trompent d'endroit où chercher le trésor
- autre exemple : dans postgis, les géométries sont associées au SRID (= identifiant de système géodésique) permettant d'interpréter les coordonées

Mais en pratique, si on reçoit des (lat,long) sans système géodésique, on peut raisonnablement supposer que ce sont des coordonnées GPS.

Les coordonnées GPS utilisent le **système géodésique WGS84**, aussi appelé **EPSG:4326**

Les systèmes géodésiques sont identifiés par leur **SRID** :

- EPSG:4326 = système géodésique du GPS (utilisant WGS84), avec des coordonnées géographiques
- EPSG:3857 = coordonnées projetées utilisées sur le web (aka [Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator_projection), notamment par google/bing/OSM (avec une merdouille 'spherical development of ellipsoidal coordinates')
- EPSG:2154 = RFG93 = Lambert-93 = coordonnées projetées, qui sont les coordonnées officielles pour la France

La France (= **Lambert93**) utilise-t-elle le référentiel WGS84 ?

Réponse courte : non, mais le référentiel utilisé par la France et WGS84 sont compatibles.

Réponse longue :

- le référentiel WGS84 est calqué sur la Terre
- or, à cause de la tectonique des plaques (essentiellement), la position d'un objet réel (par exemple un clocher) par rapport à la Terre... évolue avec le temps !
- dit autrement : sur une période de 100 ans, les corodonnées GPS d'un clocher CHANGENT !
    - (parce que le clocher s'est déplacé à cause de la tectonique des plaques, alors que le référentiel est resté fixe)
- pour limiter cet effet, le référentiel utilisé par Lambert93 est "le même que WGS84", mais qui se déplace en même temps que la plaque tectonique eurasienne (comobile)
- avantage : exprimées en RFG93, les coordonnées de mon clocher seront les mêmes dans 100 ans
    - (le référentiel suit la tectonique de la plaque Europe → si le clocher bouge à cause de la tectonique, le référentiel bougera de la même façon, donc les coordonnées du clocher seront les mêmes)

une **projection**, c'est "simplement" un couple de fonctions `x = f1(φ,λ)` et  `y = f2(φ,λ)` , où :

- `(x,y)` désignent des coordonnées planes
- `φ` la latitude
- `λ` la longitude
- `f1` et `f2` des fonctions qui sont continues partout sur l'ensemble de départ sauf sur un petit nombre de lignes et de points (tels que les pôles).

Il existe donc une infinité de solutions. Les mathématiciens ne se sont pas privés d'en trouver, et on en connaît plus de 200. Différentes projections ont différents avantages/inconvénients :

- projection équivalente  = conserve localement les aires. Cette représentation est préférée dans les atlas pour respecter les surfaces relatives des différents pays
- projection conforme     = conserve localement les angles, donc les formes. La topographie et la géodésie utilisent exclusivement ce type de représentation
- projection aphylactique = elle n'est ni conforme ni équivalente, mais peut être équidistante, c'est-à-dire conserver les distances (sur les méridiens uniquement !).

Une projection ne peut pas être à la fois conforme et équivalente.

Un point du globe étant repéré par sa longitude et sa latitude, une représentation cartographique est caractérisée par l'image des méridiens et des parallèles, c'est-à-dire le canevas de la carte.

google maps / bing maps / OSM représentent leurs cartes en utilisant EPSG:3857 = des coordonnées projetées (parce que la finalité des SIG c'est de représenter des choses sur des cartes ; donc ça fait sens d'utiliser les coordonnées projetées)

Lorsqu'on passe la souris sur une carte (e.g. leaflet, qui utilise donc des coordonnées projetées), comment peut-on avoir en retour des coordonnées géographiques ? Réponse = parce que le software traduit à la volée les coordonnées projetées (sur la carte) en coordonnées géographiques.

Et la cartographie ?
- la carto, c'est représenter le monde réel 3D sur une carte 2D
- en plus de cette notion de projection, être cartographe, c'est faire un GROS travail de sélection d'information, et de stylisation
- La carte géographique n'est pas le territoire. Elle en est tout au plus une représentation ou une “perception”.

# Expérimentations en septembre 2023


**TL;DR** :

- il faut distinguer le SRS utilisé pour stocker les données, et celui utilisé pour représenter les données sur une carte ou dans qgis :
    - dans qgis, le SRS utilisé pour l'affichage peut être modifié dynamiquement
    - il n'est donc pas représentatif du SRS utilisé pour stocker les données visualisées !
- il existe deux familles de coordonnées :
    - coordonnées géographiques (e.g. GPS = WGS84 = EPSG:4326), repérant la position d'une feature avec un couple (lat,lng) qui sont des angles par rapport à méridien/parallèle d'origine sur un sphéroïde
    - coordonnées géométrique aka projetées aka cartésiennes (e.g. EPSG:3857 aka Web Mercator), repérant la position d'une feature avec un couple (x, y) qui sont des distances sur un plan cartésien par rapport à un point d'origine
    - pour afficher quelque chose sur une surface plate (écran d'ordinateur ou carte papier), il faut forcément une projection, ce qui est rendu plus facile avec des coordonnées projetées
- les projections sont imparfaites, si on utilise des coordonnées projetées :
    - selon la projection qu'on utilise, on déformera plus ou moins telle ou telle propriété (distances, angles) des features
    - les distances calculées avec certaines coordonnées projetées (e.g. EPSG:3857) sont d'autant plus fausses qu'on s'éloigne de leur point d'origine
    - NDM : ma compréhension est donc que pour le calcul de distance, mieux vaut sans doute travailler en coordonnées géographiques

----

Situation :

- notre projet renvoie des coordonnées non-GPS (e.g. `x = 259988.8971375063 ; y = 6251687.345071103`)
- j'aimerais comprendre à quoi elles correspondent
- de plus, une fois affichées sous qgis, les mêmes données utilisent des x,y différents (toujours non-GPS)

## Conversion depuis 4326 vers 3857 et affichage sur QGIS

Pour me trouver un exemple concret, grâce à geojson.io, je repère les coordonnées GPS du croisement des rues de Richelieu et Saint-Honoré :

- https://geojson.io/#map=19.51/48.8632282/2.335547
- lat = `2.335519930171472`
- lng = `48.863236875059954`

Grâce à epsg.io, je peux transformer ces coordonnées GPS en coordonnées 3857 = https://epsg.io/transform#s_srs=4326&t_srs=3857&x=2.3355199&y=48.8632369

Je constate plusieurs choses concernant EPSG:3857 (qui s'avère être [Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator_projection), pour mon croisement de rues :

- l'unité est le mètre
- `x = 259988.8971375063`
- `y = 6251687.345071103`

En affichant les données de mon projet sur qgis, je constate que tout est cohérent en 3957 :
- le système de coordonnées de l'affichage qgis (indiqué en bas à droite de qgis) est EPSG:3857
- qgis indique les coordonnées du curseur
- si je déplace le curseur sur le même croisement de rues, le `x` et le `y` sont identiques à celles indiquées par epsg.io

^ je peux donc avoir les coordonnées de ce croisement en GPS ou en 3857.

## Jeu de données en 3949

En regardant le contenu de la base postgis, je vois que la colonne de la table est en 3949 :

```
[...]
geom geometry(Geometry,3949) NOT NULL,
[...]
```

D'après epsg.io, ce système de coordonnées est une projection sur un plan, RGF93 (le nom est `RGF93 v1 / CC49`) : https://epsg.io/3949

cf. https://fr.wikipedia.org/wiki/R%C3%A9seau_g%C3%A9od%C3%A9sique_fran%C3%A7ais

Son unité est le mètre, l'origine des coordonnées (point x=0, y=0) est [au milieu de l'océan atlantique](https://epsg.io/map#srs=3949&x=-0.000000&y=0.000000&z=3&layer=streets)

Attention que ce que la page epsg.io indique comme "Center coordinates" n'est pas l'origine, ça semble plutôt être le centre de la zone couverte par le système de coordonnées. C'est [un point un peu à l'est de La Rochelle](https://epsg.io/map#srs=4326&x=0.260000&y=46.355000&z=7&layer=streets)

## Système de coordonnées utilisé par QGIS

Attention que le SRS (= spatial reference system = système de coordonnées) utilisé par QGIS peut être modifié dynamiquement, il n'est donc pas représentatif des données effectivement stockées dans la source de données !

D'après mes essais, il semble ouvrir un projet vierge en utilisant EPSG:4326 (= le GPS), puis switche sur le SRS des premières données qu'on lui charge.

Comme j'ai tendance à charger les tuiles de fond de plan en premier, et que celles-ci sont en Mercator (EPSG:3857), mes projets ont tendance à être affichés en 3857.

Mais on peut en changer quand on veut (et ça a d'ailleurs pour effet que si on choisit des coordonnées différentes de
celles des tuiles, l'affichage de celles-ci se dégradent un peu).

## Calcul de distances

Sous postgis, le calcul de distance avec `ST_Distance` prendra en compte le type de coordonnées géographiques ou géométriques, [lien](https://postgis.net/docs/en/ST_Distance.html) :

> For geometry types returns the minimum 2D Cartesian (planar) distance between two geometries, in projected units (spatial ref units). \
> For geography types defaults to return the minimum geodesic distance between two geographies in meters, compute on the spheroid determined by the SRID.

[Cette réponse StackOverflow](https://gis.stackexchange.com/questions/242545/how-can-epsg3857-be-in-meters) montre que sur une projection de type 3857, plus on s'éloigne du point de référence, plus le calcul des distances est faux :

> So, while the unit of measurement in EPSG:3857 is indeed meters, as you have correctly spotted, distance measurements become increasingly inaccurate away from the equator. All Mercator-style projections have the same limitation as they move away from their reference point. But, where they are not global, the error can be minimized by appropriate positioning of this point
