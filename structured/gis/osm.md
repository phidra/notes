OSM = OpenStreetMap

* [Structure de la donnée](#structure-de-la-donnée)
   * [Éléments](#éléments)
   * [Tags](#tags)
   * [Features](#features)
   * [SRTM / HGT = donnée de hauteur](#srtm--hgt--donnée-de-hauteur)
* [Work with the data](#work-with-the-data)
   * [Get the data](#get-the-data)
   * [Data format](#data-format)
* [Tools](#tools)
   * [osmosis](#osmosis)
   * [idEditor](#ideditor)
   * [libosmium](#libosmium)
   * [osmium-tool](#osmium-tool)

# Structure de la donnée

Le wiki est top et bien fourni : https://wiki.openstreetmap.org/wiki/Main_Page

Donnée "referentially complete" = tous les objets référencés par d'autres objets sont présents dans la donnée (pas de pending référence).

## Éléments

3 éléments de base, auxquels ont peut associer des tags ([lien wiki](https://wiki.openstreetmap.org/wiki/Elements)) :

- node (5 milliards en jan 2019)
    - agrégat d'un id + d'une latlon
    - utilisé pour positionner un élément ponctuel sur une carte (e.g. un banc)
    - est l'élément constitutif des ways
- way (540 millions en nov 2018)
    - c'est juste une polyline = liste ordonnée de nodes (min=2 max=2000)
    - cas particulier : premier point = dernier point -> la way est un polygone
- relation
    - liste ordonnée des trois types d'éléments de base pour documenter une... relation (!) entre eux
    - exemples :
        - route = un agrégat de toutes les ways la constituant pourra représenter l'A6
        - turn restriction = un agrégat de 2 ways pourra représenter une prohibition à 2 tronçons
        - multipolygon ) pour représenter une surface à trou
    - la signification de la relation est encodée dans un tag "type"
    - certains éléments jouent un rôle particulier dans la relation, et ont alors un "role" (e.g. from + to pour une turn restriction)

## Tags

https://wiki.openstreetmap.org/wiki/Tags

Portent sur les éléments de base : node, way, relation

Paire key + value qui sont des strings unicode de 255 caractères max

Les clés sont uniques pour un élément donné.

## Features

https://wiki.openstreetmap.org/wiki/Map_Features

Éléments existants physiquement dans le monde réel (naturels ou créés par l'homme), qu'on peut faire exister sur une carte.

Exemples : autoroute, église, lac, plage, piste cyclable, fontaine à eau, caserne de pompiers, etc.

## SRTM / HGT = donnée de hauteur

https://wiki.openstreetmap.org/wiki/SRTM

- donnée librement disponible sur l'élévation (= hauteur) des divers points du globe
- en France, on n'a accès qu'à SRTM3, soit chaque pixel fait environ 90 mètres de côté

https://www.researchgate.net/post/What_is_the_vertical_resolutionaccuracy_of_Global_SRTM_1_arc_second_30_m2

- la précision de la hauteur indiquée est entre 5 et 9 mètres :
- (d'après Clément, il y a des "trous" dans la donnée, et elle nécessite peut-être de l'interpolation)

Le nom du fichier définit sa portée :

> For example, a filename named N26E092.hgt contains elevation data that stretches from 26°N 92°E to 27°N 93°E

Le contenu dépend de la résolution (SRTM1 = très résolu, SRTM3 = 3 fois moins résolu, mais correct tout de même)

Doc :

- https://dds.cr.usgs.gov/srtm/version1/Documentation/QuickStart.txt
- https://dds.cr.usgs.gov/srtm/version1/What_are_these.txt

Donnée : https://dds.cr.usgs.gov/srtm/version1/Eurasia/


# Work with the data

## Get the data

- https://wiki.openstreetmap.org/wiki/Planet.osm
- https://wiki.openstreetmap.org/wiki/Downloading_data
    - la donnée OSM du monde entier est dans Planet.osm
    - C'est gros : 50 Gio en PBF compressé, et 1.2 Tio décompressé
    - les EXTRACTS sont des subsets de la donnée OSM limités à une zone géographique donnée
- https://download.openstreetmap.fr/extracts/europe/france/
    - ce miroir semble permettre de télécharger par pays / région
    - à la différence d'autres, il permet de téléchager uniquement des départements (e.g. [hauts-de-seine](https://download.openstreetmap.fr/extracts/europe/france/ile_de_france/) = 27 MB)
- https://wiki.openstreetmap.org/wiki/Downloading_data#Download_the_data
    - il est possible de télécharger un XML d'une bbox donnée :
        ```
        wget -O muenchen.osm "https://api.openstreetmap.org/api/0.6/map?bbox=11.54,48.14,11.543,48.145"
        ```
    - le format est [OSM XML](https://wiki.openstreetmap.org/wiki/OSM_XML)
    - NOTE : il faut restreindre la fenêtre pour ne pas renvoyer plus de 50000 nodes, sinon on se tape une erreur 400 :
        ```
        https://wiki.openstreetmap.org/wiki/API_v0.6#Retrieving_map_data_by_bounding_box:_GET_.2Fapi.2F0.6.2Fmap
        ```
- http://download.geofabrik.de/europe/france.html
    - geofabrik permet de télécharger par pays / région
        - France entière = 3.6 GB
        - Guyane = 10 MB
        - Île-de-France = 245 MB
    - format de base = XXX.osm.pbf
    - possibilité de récupérer la donnée sous forme de shapefiles ESRI (plus usuelles pour les GIS que PBF) ou XML
- https://download.bbbike.org/osm/bbbike/Paris/
    - bbbike permet de télécharger des grandes villes, dont Paris (PBF de 135 MB)

## Data format

Deux formats mainstream :

- `extract.osm`     -> le format est OSM XML
- `extract.osm.pbf` -> le format est OSM PBF

https://learnosm.org/en/osm-data/file-formats/

- le format de base est XML OSM = un dialecte XML spécifique à OSM
- il est également possible de les exporter en shapefiles, plus commun dans les GIS (avec quelques restrictions)

[Cet article](https://wiki.openstreetmap.org/wiki/PBF_Format) décrit en détail l'utilisation de protocol buffers pour encoder la donnée OSM.

https://paulkernfeld.com/2019/08/05/osm-pbf-performance-tricks.html

- très bon article sur l'encodage des `*.osm.pbf`, présentant les divers moyens de gagner de la place/de la perf
- notamment, en plus des tricks classiques de protocol buffers : encodage delta, compression zlib, fichier par bloc, etc.


# Tools

## osmosis

https://wiki.openstreetmap.org/wiki/Osmosis

https://learnosm.org/en/osm-data/osmosis/

Tool java pour manipuler la donnée OSM :

- conversion vers/to database
- ajout de features
- extraction de features dans une bbox donnée
- ...

## idEditor

https://wiki.openstreetmap.org/wiki/ID

Outil browser-based permettant de MODIFIER la donnée.

C'est quoi les fields ?

- pas clair... peut-être que les fields sont les tags "classiques" pour le type de feature ?
- en effet, dans idEditor, chaque feature semble avoir un type (en haut à gauche du panel "edit feature")
- ce type semble être défini par des tags. En vrac, quelques types :
    - `highway=residential` : https://wiki.openstreetmap.org/wiki/Tag:highway=residential
    - `highway=secondary`   : https://wiki.openstreetmap.org/wiki/Tag:highway=secondary
    - `highway=footway`     : https://wiki.openstreetmap.org/wiki/Tag:highway=footway
    - `landuse=grass`       : https://wiki.openstreetmap.org/wiki/Tag:landuse=grass
    - `amenity=university`  : https://wiki.openstreetmap.org/wiki/Tag:amenity=university
    - `landuse=industrial`  : https://wiki.openstreetmap.org/wiki/Tag:landuse=industrial

Lorsqu'une feature est sélectionnée, on peut consulter les relations à laquelle elle appartient (et quel rôle elle y joue).

## libosmium

- lib C++ header-only de gestion de la donnée OSM
- https://osmcode.org/libosmium/manual.html
- sudo apt-get install libosmium2-{dev,doc}
- sinon, cloner libosmium et protozero (au même niveau), ça marche
- les exemples de libosmium sont TRÈS bien, je peux complètement m'appuyer dessus
- (du coup, je me demande si ça vaut le coup de faire des POCs... EDIT : j'en ai fait une, ça valait le coup pour comprendre comment compiler)

## osmium-tool

https://osmcode.org/osmium-tool/

```sh
osmium fileinfo -e someplace.osm
# [...]
# Bounding box: (2.28477,48.7478,2.31461,48.7605)
# Number of nodes: 10839
# Number of ways: 1556
# Number of relations: 83
# [...]

osmium fileinfo --json someplace.osm
# montre la structure, notamment comment accéder au nombre de nodes

osmium fileinfo -e -g data.count.nodes someplace.osm
# 10839

osmium getid someplace.osm --output-format=xml n7276388553 w779274334 r10612083
# [... le XML du noeud, de la voie, et de la relation ...]
```

(pas avec ma version) on peut faire des extracts dans une bounding box ou dans un polygone geojson

(pas avec ma version) on peut filtrer des nodes/ways/relations selon leurs tags (e.g. pour ne garder que le réseau routier)
