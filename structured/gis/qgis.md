**C'est quoi ?** Un logiciel permettant de visualiser et manipuler des données SIG.

Contexte = janvier 2022, j'utilise qgis pour visualiser simplement des données au format OSM, sans dépendre d'idEditor, de fonds de carte ou d'autres outils. Mon cahier des charges :

- charger un fichier `.osm.pbf` et visualiser les géométries des nodes/edges
- pouvoir cliquer sur un edge/node pour consulter ses attributs, et notamment son id
- pouvoir retrouver un node (ou un edge) à partir de son id
- pouvoir limiter le chargement des données à une bbox

**Note importante** : qgis peut vite être très gourmand en ressources, et quand le PC commence à freezer, ça peut-être compliqué de réussir à l'arrêter... Du coup, pour garder la possibilité de killer qgis une fois qu'il aura freezé le PC :

```sh
nice -n 19 ionice -c 3 qgis
```

Les docs = [user guide](https://docs.qgis.org/3.22/en/docs/user_manual/) et [trainings](https://docs.qgis.org/3.22/en/docs/training_manual/).


* [Installation de qgis](#installation-de-qgis)
* [Comment importer un fichier .osm.pbf ?](#comment-importer-un-fichier-osmpbf-)
* [Comment importer des shapefiles ?](#comment-importer-des-shapefiles-)
* [Comment limiter l'import à une bbox ?](#comment-limiter-limport-à-une-bbox-)
* [Mes utilisations](#mes-utilisations)
   * [Consulter les attributs d'un edge](#consulter-les-attributs-dun-edge)
   * [Retrouver un edge à partir de son id](#retrouver-un-edge-à-partir-de-son-id)
   * [Retrouver tous les edges d'une liste d'ids donnée](#retrouver-tous-les-edges-dune-liste-dids-donnée)
   * [Retrouver tous les edges d'un type particulier](#retrouver-tous-les-edges-dun-type-particulier)
   * [N'afficher que les edges avec un certain attribut](#nafficher-que-les-edges-avec-un-certain-attribut)
   * [Sélection de features + accès à leurs attributs](#sélection-de-features--accès-à-leurs-attributs)
   * [Afficher les noms des edges/nodes sur la carte](#afficher-les-noms-des-edgesnodes-sur-la-carte)
   * [Exporter les osm_id des features sélectionnées](#exporter-les-osm_id-des-features-sélectionnées)
   * [Exporter un geojson des features sélectionnées](#exporter-un-geojson-des-features-sélectionnées)
   * [Colorier des features d'une façon particulière](#colorier-des-features-dune-façon-particulière)
   * [Colorier les features sélectionnées d'une façon particulière](#colorier-les-features-sélectionnées-dune-façon-particulière)
* [Autres notes vrac sur l'UI](#autres-notes-vrac-sur-lui)

# Installation de qgis

D'après la [doc ubuntu](https://doc.ubuntu-fr.org/qgis) (et [la doc d'install qgis](https://www.qgis.org/fr/site/forusers/alldownloads.html#repositories) aussi FWIW), les versions packagées avec ubuntu sont anciennes.

Du coup, je suis [ce tuto d'installation](https://www.qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu) :

```sh
sudo apt install gnupg software-properties-common  # sur ma machine, les deux paquets étaient déjà installés
wget -qO - https://qgis.org/downloads/qgis-2021.gpg.key | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/qgis-archive.gpg --import
sudo chmod a+r /etc/apt/trusted.gpg.d/qgis-archive.gpg
sudo add-apt-repository "deb https://qgis.org/ubuntu $(lsb_release -c -s) main"
sudo apt update
sudo apt install qgis qgis-plugin-grass  # si besoin, on peut aussi installer qgis-server

qgis --version
# QGIS 3.22.3-Białowieża 'Białowieża' (1628765ec7)
```
# Comment importer un fichier .osm.pbf ?

Le principe va être d'importer un layer ("Couche" en VF) contenant les features du fichier OSM :

- Menu > Couche > Ajouter une couche > Ajouter une couche vecteur
- Source = `/tmp/monaco.osm.pbf`
- Lorsque un fichier `.osm.pbf` est sélectionné, quelques options du driver GDAL deviennent accessibles ; j'ai trouvé utile d'en modifier certaines, cf. [mes notes sur gdal](./gdal_ogr.md) :
    - `CONFIG_FILE` à `/tmp/osmconf.ini` afin de choisir les attributs OSM à charger
    - `USE_CUSTOM_INDEXING` à `Non` si besoin
- Derrière, le layer se charge dans la vue principale
- Le panel des layers est en bas à gauche.

En terme de taille de données, Monaco et Andorre sont chargeables sans souci. Les `lines` de l'Île-de-France passent en terme de RAM sur mon poste à 8 Gio de RAM, par contre le rendu de la carte est trop lent pour que ce soit réellement exploitable.

Comme qgis utilise GDAL/OGR sous le capot, si qgis n'arrive pas à charger un layer OSM, l'utiliation de `ogrinfo` peut aider à débugger en donnant des détails (c'est du vécu, ça m'a permis de comprendre qu'il fallait parfois désactiver le custom indexing).

Si les données sont petites, c'est parfois pas évident de les trouver sur la carte ; on dirait qu'il y a des fonctions pour zoomer sur le layer, mais je n'arrive pas à les faire marcher, du coup je contourne en ouvrant la table des attributs du layer, et en zoomant sur une feature en particulier.

# Comment importer des shapefiles ?

- Menu > Couche > Ajouter une couche > Ajouter une couche vecteur
- Dans les jeux de données, choisir un triplet de fichiers `shp + shx + dbf`

Charger un dbf en standalone : comme les fichiers dbf sont de simples base de données, on peut charger un DBF même sans les géométries associées.

# Comment limiter l'import à une bbox ?

On dirait qu'on ne peut pas faire ça avec qgis, et qu'il faut que je filtre le fichier `.osm.pbf` de façon externe :

```sh
osmium extract --bbox 7.4143945,43.7285819,7.4155760,43.7289873 /tmp/monaco.osm.pbf --output /tmp/extracted.osm.pbf

# si besoin :
man osmium-extract  # attention : la clé de man n'est pas exactement la commande
```

Attention que ça ne marchera sans doute que sur de petits extracts OSM : pour `monaco.osm.pbf` (qui est ridiculement petit : `456 Kio`), osmium reporte une occupation mémoire plus de 1000 fois supérieure à la taille du fichier : `Peak memory used: 645 MBytes`.

Si besoin, la stratégie `simple` nécessite un peu moins de RAM (cf. [la doc](https://docs.osmcode.org/osmium/latest/osmium-extract.html)) :

```sh
osmium extract --strategy simple --bbox 7.4143945,43.7285819,7.4155760,43.7289873 /tmp/monaco.osm.pbf --output /tmp/extracted.osm.pbf
```

J'aurais bien utilisé `ogr2ogr` avec l'option `-spat` pour ça, mais on dirait qu'il ne sait pas créer de fichier OSM, juste les lire : https://gdal.org/drivers/vector/index.html

# Mes utilisations

## Consulter les attributs d'un edge

Ctrl+Shift+I (ou bien Vue > Identifier les entités) permet de cliquer sur un edge pour consulter ses attributs.

Attention que les tags OSM ne sont pas forcément tous chargés dans qgis et peuvent être enfouis dans `other_tags`, cf. [mes notes sur gdal](./gdal_ogr.md).

## Retrouver un edge à partir de son id

F3 (ou bien Éditer > Sélection > Sélectionner les entités par valeur) ouvre un formulaire, dans lequel on peut choisir des features à partir de leurs attributs (e.g. `osm_id`).

On peut alors zoomer sur la feature (ou simplement la faire clignoter ^^).

## Retrouver tous les edges d'une liste d'ids donnée

Ctrl+F3 (ou bien Éditer > Sélection > Sélectionner les entités à l'aide d'une expression) ouvre un formulaire, dans lequel on peut entrer une query de sélection :

```sql
osm_id IN ('177829567', '168104483', '982081599', '176321050', '176336114')
```

## Retrouver tous les edges d'un type particulier

Idem ci-dessus, mais en choisissant une expression plus complexe :

```sql
highway = 'primary' OR highway = 'secondary'
```

## N'afficher que les edges avec un certain attribut

Clic droit sur la couche > Filtrer :

```sql
highway != 'service'
```

## Sélection de features + accès à leurs attributs

TL;DR : sélectionner des features peut se faire de façon équivalente via la carte ou via la table d'attribut, à condition de contourner le bug décrit ci-dessous.

----

F6 (ou bien Clic droit sur la couche > Ouvrir la table d'attributs) ouvre la table d'attributs.

Shift + F6 pour ouvrir la table d'attributs sur les features sélectionnées (moyennant le bug).

Il y a plein d'outils de sélection (la toolbox permet de les choisir, ou sinon Éditer > Sélection), et notamment :

- clic pour sélectioner une feature
- sélection par cercle ou polygone (et dans ce dernier cas, clic-droit pour terminer le polygone)
- sélection des features ayant une valeur particulière d'un attribut (F3, cf. ci-dessus)
- sélection des features selon une query SQL (Ctrl+F3, cf. ci-dessus)

**BUG** : soit il y a un bug, soit je n'arrive pas à utiliser la table d'attributs :

- si j'essaye de n'afficher que les features sélectionnées dans la table d'attributs, ça foire (ça ne m'affiche rien, ou ça m'affiche un subset des features)
- c'est visible dans le titre de la fenêtre : `Total des entités: 0, Filtrées: 0, Sélectionnées: 6` : il voit bien que j'ai sélectionné 6 features, mais il ne m'en affiche aucune !
- mon contournement = repasser temporairement par _Ne montrer que les entités visibles sur la carte_ puis revenir sur _Ne montrer que les entités sélectionnées_
- après mon contournement, mes features sélectionnées apparissent bien dans la table d'attributs, ce qui est visible dans le titre `Total des entités: 6, Filtrées: 6, Sélectionnées: 6`

Autre raccourci utile = Ctrl+Shift+A pour tout déselectionner.

Dans le fenêtre d'attributs, on peut basculer entre la vue formulaire et la vue tabulaire avec des boutons en bas à droite de la fenêtre.

## Afficher les noms des edges/nodes sur la carte

On peut modifier le rendu des features pour les labelliser avec... ce qu'on veut :-)

Menu > Couche > "Étiquetage" > étiquette simple

On peut choisir n'importe quel attribut comme label d'un edge (e.g. `osm_id`), qgis se débrouille pour choisir si en fonction du zoom, ça vaut le coup de l'afficher.

On peut également composer des champs avec les opérateurs SQL ; par exemple, pour afficher une concaténation de `osm_id` et du nom de la rue :

```sql
osm_id || ' = ' || name
```

## Exporter les osm_id des features sélectionnées

**méthode 1** = via la table des attributs : c'est rapide, mais c'est pas fou car j'exporte forcément la géométrie avec :

- ouvrir la table d'attributs sur les features sélectionnées (cf. ci-dessus pour contourner le bug))
- clic droit sur une colonne > organiser les colonnes → tout décocher sauf `osm_id`
- les lignes étant toujours sélectionnées, cliquer sur le bouton "copier dans le presse-papiers" dans la toolbox
- (derrière, si besoin, éditer manuellement le fichier pour dégager manuellement les géométries)

**méthode 2** = en exportant explicitement :
- dans le panel des layers, clic droit sur le layer → Exporter
    - Format > CSV
    - Nom de fichier > `/tmp/pouet.csv`
    - Choisir les attributs à exporter : `osm_id`
    - Géométrie > Type de géométrie > Pas de géométrie (par rapport à la mthode précédente, ça nous permet donc de ne pas se taper les géométries \o/)
- tout en bas de la fenêtre, décocher "ajouter les fichiers sauvegardés à la carte"

(j'ai illustré avec `osm_id`, mais ça marche avec tous les attributs, et même avec plusieurs)

## Exporter un geojson des features sélectionnées

Idem que la méthode 2 ci-dessus, en choisissant le format geojson (les attributs sélectionnés sont également exportées en properties du geojson).

## Colorier des features d'une façon particulière

Exemple d'usage = colorier différemment les routes en fonction de leur importance, pour mieux appréhender les données.

Globalement, pour choisir comment le layer est rendu :

- ouvrir les propriétés du layer (double-cliquer dessus, ou bien clic-droit > Propriétés)
- dans le tab "Symbologie", le combobox tout en haut permet d'indiquer comment rendre le layer

Par défaut, c'est "Symbole unique" qui est retenu = toutes les features du layer sont rendues de la même façon.

Avec "Catégorisé", on peut rendre le layer différemment en fonction d'un attribut, par exemple :

- dans "Valeur", choisir `highway`
- version rapide = cliquer sur "Classer" en bas de la fenêtre pour voir les valeurs possibles et les paramétrer (version laborieuse = on peut aussi les définir un par un à la main)
- en cliquant, on peut configurer la valeur de l'attribut, et la façon dont il est rendu
- si on ajoute un item en laissant la valeur de l'attribut vide, c'est un catchall pour "toutes les autres valeurs"

Avec "Ensemble de règles", je peux définir des règles plus fines sous forme de requête SQL

(ergonomie : si je commence avec "Catégorisé", puis que je switche sur "Ensemble de règles", les règles sont initialisées avec des équivalents de ce qu'il y avait dans "Catégorisé", ce qui me permet de les modifier)

## Colorier les features sélectionnées d'une façon particulière

Si j'utilise la sélection comme un moyen d'afficher un chemin, il peut être utile de le rendre très visible.

Ça se fait en deux étapes :

- _Projet > Propriétés > Général > Couleur de la sélection_ pour choisir un rouge bien flashy
- Clic droit sur la couche + puis _Propriétés > Symbologie > Ensemble de règles_ + ajouter une règle avec le filtre `is_selected() = true` + appliquer.

# Autres notes vrac sur l'UI

- molette pour zoomer rapidement / ctrl+molette pour zoomer plus doucement
- drag-and-drop du clic central (molette) pour déplacer la carte
- avec l'outil par défaut clic gauche quelque part sur la carte ne sélectionne PAS la feature, mais centre la carte à l'endroit du clic
- clic droit quelque part sur la carte permet d'accéder aux coordonnées du point cliqué
