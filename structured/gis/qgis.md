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

* [Installation](#installation)
   * [Plugins](#plugins)
* [Usage n°1 = import](#usage-n1--import)
   * [Ajouter des tuiles de fond de plan](#ajouter-des-tuiles-de-fond-de-plan)
   * [Comment importer un fichier .osm.pbf ?](#comment-importer-un-fichier-osmpbf-)
      * [Impossible d'importer directement le fichier .osm.pbf](#impossible-dimporter-directement-le-fichier-osmpbf)
      * [Contournement en exportant l'OSM en CSV](#contournement-en-exportant-losm-en-csv)
      * [Comment limiter la conversion CSV à une bbox ?](#comment-limiter-la-conversion-csv-à-une-bbox-)
      * [Importer la couche CSV](#importer-la-couche-csv)
      * [Comment n'afficher le réseau routier que si on est suffisamment zoomé](#comment-nafficher-le-réseau-routier-que-si-on-est-suffisamment-zoomé)
   * [DEPRECATED Comment importer un fichier .osm.pbf ?](#deprecated-comment-importer-un-fichier-osmpbf-)
      * [Comment limiter l'import PBF à une bbox ?](#comment-limiter-limport-pbf-à-une-bbox-)
   * [Comment importer des shapefiles ?](#comment-importer-des-shapefiles-)
   * [Comment importer des features d'une base postgis ?](#comment-importer-des-features-dune-base-postgis-)
      * [Ajout direct de la couche PostGIS](#ajout-direct-de-la-couche-postgis)
      * [Ajout via le gestionnaire de BDD](#ajout-via-le-gestionnaire-de-bdd)
      * [Comment ne pas garder la synchro entre la couche postgis et le serveur](#comment-ne-pas-garder-la-synchro-entre-la-couche-postgis-et-le-serveur)
      * [Comment limiter l'import PostGIS à une bbox ?](#comment-limiter-limport-postgis-à-une-bbox-)
      * [Ajouter le résultat d'une requête comme couche](#ajouter-le-résultat-dune-requête-comme-couche)
* [Usage n°2 = rechercher/sélectionner des features](#usage-n2--recherchersélectionner-des-features)
   * [Retrouver un edge à partir de son id](#retrouver-un-edge-à-partir-de-son-id)
   * [Retrouver tous les edges d'une liste d'ids donnée](#retrouver-tous-les-edges-dune-liste-dids-donnée)
   * [Retrouver tous les edges d'un type particulier](#retrouver-tous-les-edges-dun-type-particulier)
   * [Sélection de features + accès à leurs attributs](#sélection-de-features--accès-à-leurs-attributs)
* [Usage n°3 = attributs des features](#usage-n3--attributs-des-features)
   * [Géométrie des features](#géométrie-des-features)
   * [Consulter les attributs d'un edge](#consulter-les-attributs-dun-edge)
      * [Ouvrir la table](#ouvrir-la-table)
      * [Deux vues : ligne par ligne ou formulaire](#deux-vues--ligne-par-ligne-ou-formulaire)
      * [Ajouter la géométrie des features en tant qu'attribut](#ajouter-la-géométrie-des-features-en-tant-quattribut)
* [Usage n°4 = affichage](#usage-n4--affichage)
   * [N'afficher que les edges avec un certain attribut](#nafficher-que-les-edges-avec-un-certain-attribut)
   * [Rendre une couche transparente](#rendre-une-couche-transparente)
   * [Afficher les noms des edges/nodes sur la carte](#afficher-les-noms-des-edgesnodes-sur-la-carte)
   * [Colorier des features d'une façon particulière](#colorier-des-features-dune-façon-particulière)
   * [Colorier les features sélectionnées d'une façon particulière](#colorier-les-features-sélectionnées-dune-façon-particulière)
* [Usage n°5 = export](#usage-n5--export)
   * [Exporter les osm_id des features sélectionnées](#exporter-les-osm_id-des-features-sélectionnées)
   * [Exporter un geojson des features sélectionnées](#exporter-un-geojson-des-features-sélectionnées)
   * [Exporter une couche PostGIS pour la rendre indépendante de la BDD](#exporter-une-couche-postgis-pour-la-rendre-indépendante-de-la-bdd)
* [Autres notes vrac sur l'UI](#autres-notes-vrac-sur-lui)


# Installation

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

## Plugins

Extensions > Installer/Gérer les extensions

Exemple : le plugin `BoundingBox` pour récupérer la bbox actuellement affichée.

# Usage n°1 = import

## Ajouter des tuiles de fond de plan

Depuis l'explorateur (si nécessaire, pour l'afficher : Vue > Panneaux > Explorateur), il y a un groupe `XYZ Tiles` : on peut ajouter des tuiles.

Sinon, Couches > Gestionnaire des sources de données > XYZ

Une fois les tuiles ajoutées, on peut drag-n-dropper la couche sur la visualization pour les afficher.

URL des couches openstreetmap = `https://tile.openstreetmap.org/{z}/{x}/{y}.png`

## Comment importer un fichier .osm.pbf ?

### Impossible d'importer directement le fichier .osm.pbf

À date de juillet 2023, le driver utilisé pour importer des données OSM (= [OGR](./gdal_ogr.md)) n'est pas capable de construire un index spatial. Du coup, en dehors de petites données (genre Monaco), qgis est inutilisable après import des données :

- le rendu est démesurément lent, donc même si on prend son mal en patience et qu'on attend le chargement, dès qu'on bouge/zoome la carte, tout est à refaire
- la sélection de features est démesurément lente

(c'est visible si on essaye de construire manuellement l'index via _Traitement > Boîte à Outils > Outils généraux pour les vecteurs > Créer un index spatial_ : l'exécution du traitement produit un log indiquant que le driver ne gère pas les index spatiaux)

### Contournement en exportant l'OSM en CSV

Solution = exporter les données en CSV, et importer le fichier CSV :

```
ogr2ogr -f 'CSV' /tmp/paris.csv -lco GEOMETRY=AS_WKT /tmp/Paris.osm.pbf lines
```

À noter :

- pour accélérer le tout premier rendu de la couche, on peut zoomer fortement la carte **avant** d'importer la couche.
- toutes [mes notes sur gdal](./gdal_ogr.md) (et notamment les tags cachés dans `other_tags`) sont applicables au moment de la conversion
- [la doc du driver OSM de gdal](https://gdal.org/drivers/vector/osm.html) indique la signification de `lines` / `points` et consorts :
   > The driver will categorize features into 5 layers :
   > - points : "node" features that have significant tags attached.
   > - lines : "way" features that are recognized as non-area.
   > - multilinestrings : "relation" features that form a multilinestring(type = 'multilinestring' or type = 'route').
   > - multipolygons : "relation" features that form a multipolygon (type = 'multipolygon' or type = 'boundary'), and "way" features that are recognized as area.
   > - other_relations : "relation" features that do not belong to the above 2 layers.
- si je veux construire un graphe routable, je peux donc a priori me contenter de `lines`

### Comment limiter la conversion CSV à une bbox ?

On peut utiliser l'option `-spat xmin ymin xmax ymax` de `ogr2ogr` :

```
ogr2ogr -f 'CSV' /tmp/cropped.csv -lco GEOMETRY=AS_WKT -spat 2.337398547855315 48.851750441390834 2.354516975749249 48.86197299664923  /tmp/Paris.osm.pbf lines
```

### Importer la couche CSV

Une fois nos données OSM converties en CSV, _Couche > Ajouter une couche > Ajouter une couche de texte délimité_ , il faut :

- choisir _SCR de la géométrie_ à **EPSG:4326 — WGS84**
- dans _Paramètres de la couche_, cocher "Index spatial"

En terme de taille de données, avec un extract OSM IDF de 284 Mio, la conversion en CSV prend quelques secondes, l'import sous QGIS prend quelques dizaines de secondes.

### Comment n'afficher le réseau routier que si on est suffisamment zoomé

Si la carte est dézoomée, non seulement il est inutile d'afficher le réseau routier (on n'y verra rien), mais en plus ça ralentit le rendu.

Solution = n'afficher la couche importée qu'à partir d'un certain niveau de zoom :

- clic-droit sur la Couche > Propriétés
- Onglet "Rendu"
- Cocher "visibilité dépendante de l'échelle"
- Choisir par exemple :
    - `Minimium = 1:10000`
    - `Maximum = 1:1`

## DEPRECATED Comment importer un fichier .osm.pbf ?

**ATTENTION** : ces notes sont deprecated, en juillet 2023, j'utilise plutôt une autre façon = conversion en CSV, essentiellement pour pouvoir construire un index spatial sur les données.

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

### Comment limiter l'import PBF à une bbox ?

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

## Comment importer des shapefiles ?

- Menu > Couche > Ajouter une couche > Ajouter une couche vecteur
- Dans les jeux de données, choisir un triplet de fichiers `shp + shx + dbf`

Charger un dbf en standalone : comme les fichiers dbf sont de simples base de données, on peut charger un DBF même sans les géométries associées.

## Comment importer des features d'une base postgis ?

Deux possibilités : soit en ajoutant la couche directement, soit en faisant une requête avec le gestionnaire de base de données, et en ajoutant son résultat comme couche.

Attention : il y a un bug d'affichage où le curseur ne se positionne pas sur le champ password (ça n'empêche pas de le taper).

### Ajout direct de la couche PostGIS

- Menu > Couche > Ajouter une couche > Ajouter des couches PostGIS
- Dans les connexions, créer une connexion vers le serveur avec `Nouveau`
- **important** : la plupart du temps, cocher _"Utiliser la table de métadonnées estimées"_ (détails ci-dessous)
- Cliquer sur Connecter, puis choisir un couple schéma+table dans la liste
- **important** : si besoin, cliquer sur Filtrer pour limiter ce qu'on importe !
- la clause de filtrage est ce qu'on mettrait dans une clause `WHERE` (p.ex. `feature_type = 'this_type'`)
- cliquer sur Tester pour vérifier que la clause est valide, puis valider avec OK
- cliquer sur Ajouter pour ajouter la nouvelle couche contenant les features importées de la base postgis

Les caveats :

- si on n'utilise pas la _table de métadonnées estimées_, le simple fait de lister tous les schémas+tables du serveur est très très long
- mais en utilisant les métadonnées estimées, les statistiques semblent fausses (e.g. le nombre de lignes qui matchent qu'affiche le bouton "Tester")

### Ajout via le gestionnaire de BDD

Il faut préalablement créer la connexion :

- Couche > Gestionnaire des Sources de Données (Ctrl+L)
- PostgreSQL
- Créer la connexion

qgis embarque un client SQL, le "gestionnaire de base de données", qui joue le même genre de rôle que pgadmin.

On peut l'utiliser pour ajouter une couche sur le résultat d'une requête qui comporte une colonne géométrique :

- après avoir cliqué sur _Exécuter_ , cocher _Charger en tant que nouvelle couche_
- nommer la couche, et choisir la couche géométrique (ça n'est pas un champ libre, il faut choisir dans le menu déroulant, qui peut lagger un peu)
- en cliquant sur "Définir le filtre", on peut filtrer encore plus (mais mieux vaut le faire dans le WHERE de la requête, non ?)
- en cliquant sur _Charger_, ça créée la couche (on dirait que ça refait la requête, donc si celle-ci est longue, on l'aura faite deux fois :-/)

### Comment ne pas garder la synchro entre la couche postgis et le serveur

Position du problème : après avoir importé des features depuis une base postgis, qgis semble requêter le serveur régulièrement (e.g. lorsqu'on déplace la carte) pour mettre à jour la couche.

C'est problématique dans le cas de requêtes longues (e.g. quelques minutes).

Pour éviter ça, j'ai pas de silver-bullet mais un contournement = exporter les features de la couche dans un fichier (ce qui ajoute la couche dupliquée, cf. paragraphe dédié), puis supprimer la couche postgis.

C'est pas fi-fou, mais au moins l'affichage est réactif.

### Comment limiter l'import PostGIS à une bbox ?

On peut p.ex. ajouter une clause géométrique (ici, une bbox qui englobe la France sans la Corse) :

```
ST_Intersects(ST_MakeEnvelope(-4.987792, 41.918628, 8.578320, 51.193115, 4326), geom)
```


### Ajouter le résultat d'une requête comme couche

TODO (j'ai l'impression qu'il refait la requête)

# Usage n°2 = rechercher/sélectionner des features

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

## Sélection de features + accès à leurs attributs

**Préambule** = on ne peut sélectionner des features que de la couche actuellement sélectionnée (du coup, la première étape est de choisir une couche dont on veut sélectionner les entités !)

TL;DR : sélectionner des features peut se faire de façon équivalente via la carte ou via la table d'attribut.

Les features sélectionnées apparaissent en jaune.

----

Il y a plein d'outils de sélection (la toolbox permet de les choisir, ou sinon Éditer > Sélection), et notamment :

- clic pour sélectioner une feature
- avec la souris dans un rectangle : Éditer > Sélection > Sélectionner des entités
- sélection par cercle ou polygone (et dans ce dernier cas, clic-droit pour terminer le polygone)
- sélection des features ayant une valeur particulière d'un attribut (F3, cf. ci-dessus)
- sélection des features selon une query SQL (Ctrl+F3, cf. ci-dessus)

Autre raccourci utile = Ctrl+Shift+A pour tout déselectionner.

# Usage n°3 = attributs des features

## Géométrie des features

Au moins pour les coordonées ponctuelles, si on sélectionne une feature avec **Identifier des entités**, on a un champ `Dérivé` qui contient les coordonnées du point _dans le SRS actuel de qgis_ (et non dans le SRS de la source de données).

Pour les linestrings ou les polygons, je n'ai pas encore trouvé mieux qu'ajouter la géométrie de la feature en tant qu'attribut cf. quelques paragraphes plus loin.

(à noter qu'on a à la fois la coordonée ponctuelle du point, et la coordonnée cliquée, proche)

## Consulter les attributs d'un edge

### Ouvrir la table

- Couche > Ouvrir la table d'attributs (F6)
- Couche > Filtrer la table attributaire > Ouvrir la table attributaire des entités sélectionnées (Shift+F6, moyennant le bug ci-dessous)
- Couche > Filtrer la table attributaire > Ouvrir la table attributaire des entités visible (Ctrl+F6, mais le raccourci est sans doute intercepté chez moi car il ne marche pas)
- Ctrl+Shift+I (ou bien Vue > Identifier les entités) permet de cliquer sur un edge pour consulter ses attributs.

Attention que les tags OSM ne sont pas forcément tous chargés dans qgis et peuvent être enfouis dans `other_tags`, cf. [mes notes sur gdal](./gdal_ogr.md).

**BUG** : soit il y a un bug, soit je n'arrive pas à utiliser la table d'attributs :

- si j'essaye de n'afficher que les features sélectionnées dans la table d'attributs, ça foire (ça ne m'affiche rien, ou ça m'affiche un subset des features)
- c'est visible dans le titre de la fenêtre : `Total des entités: 0, Filtrées: 0, Sélectionnées: 6` : il voit bien que j'ai sélectionné 6 features, mais il ne m'en affiche aucune !
- mon contournement = repasser temporairement par _Ne montrer que les entités visibles sur la carte_ puis revenir sur _Ne montrer que les entités sélectionnées_
- après mon contournement, mes features sélectionnées apparissent bien dans la table d'attributs, ce qui est visible dans le titre `Total des entités: 6, Filtrées: 6, Sélectionnées: 6`

### Deux vues : ligne par ligne ou formulaire

Dans le fenêtre d'attributs, on peut basculer entre la vue formulaire et la vue tabulaire avec des boutons en bas à droite de la fenêtre.

### Ajouter la géométrie des features en tant qu'attribut

Utile pour pouvoir copier/coller la géométrie d'une feature.

- depuis la table des attributs, ouvrir calculatrice de champ (Ctrl+I)
- choisir le nom (e.g. `my_geometry`) et définir le type `Texte`
- Dans l'expression, entrer `geom_to_wkt($geometry)`  (profiter de l'autocompletion si besoin)
- cliquer sur ok, chaque feature a une nouvelle colonne avec la géométrie en WKT, p.ex. `Point (1.4989549 43.5485043)`

# Usage n°4 = affichage

## N'afficher que les edges avec un certain attribut

Clic droit sur la couche > Filtrer :

```sql
highway != 'service'
```

## Rendre une couche transparente

Clic-droit sur la couche > Propriétés > Symbologie

Déplier "Rendu de couche" tout en bas.

Régler l'opacité et valider.

## Afficher les noms des edges/nodes sur la carte

On peut modifier le rendu des features pour les labelliser avec... ce qu'on veut :-)

Menu > Couche > "Étiquetage" > étiquette simple

On peut choisir n'importe quel attribut comme label d'un edge (e.g. `osm_id`), qgis se débrouille pour choisir si en fonction du zoom, ça vaut le coup de l'afficher.

On peut également composer des champs avec les opérateurs SQL ; par exemple, pour afficher une concaténation de `osm_id` et du nom de la rue :

```sql
osm_id || ' = ' || name
```

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

# Usage n°5 = export

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

## Exporter une couche PostGIS pour la rendre indépendante de la BDD

(cf. le cas d'usage plus haut = si la requête est longue, comme elle est refaite régulièrement, ça en devient inutilisable)

Le TL;DR est d'exporter la couche (ce qui ajoute automatiquement l'export à la vue courante, et me permet donc de dégager la couche liée à la BDD).

Convertir en layer permanent -> pas bien compris l'intérêt, car je retrouve bien ma couche en chargeant le projet.

Certains formats semblent accepter d'enregistrer non seulement les données, mais également leur format d'affichage (ce qui me permet de ne pas avoir à sauvegarder/charger le style qgis, mais c'est clairement pas complet, p.ex. la transparence n'a pas l'air gérée...)

- AutoCAD DXF
- KML = Keyhole Markup Language
- Mapinfo MIF
- Mapinfo TAB
- Microstation DGN

On peut aussi exporter un fichier de style (en plus des features elles-mêmes) pour retrouver sa couche stylisée à l'identique.

# Autres notes vrac sur l'UI

- molette pour zoomer rapidement / ctrl+molette pour zoomer plus doucement
- drag-and-drop du clic central (molette) pour déplacer la carte
- avec l'outil par défaut clic gauche quelque part sur la carte ne sélectionne PAS la feature, mais centre la carte à l'endroit du clic
- clic droit quelque part sur la carte permet d'accéder aux coordonnées du point cliqué
- Pour sélectionner l'outil "main" : Vue > Centrer la carte
- on peut manipuler la carte au clavier : flêches pour déplacer + PgUp/PgDown pour zoomer :
    - intérêt 1 = ça évite d'avoir à utiliser la souris
    - intérêt 2 = je peux garder l'outil "sélectionner les features" constamment sélectionné pour cliquer
