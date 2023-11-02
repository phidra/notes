**Contexte** : septembre 2023, je m'intéresse à la data overture.

* [C'est quoi, ça apporte quoi ?](#cest-quoi-ça-apporte-quoi-)
* [Quel lien avec OSM ?](#quel-lien-avec-osm-)
   * [ce qu'en dit overture](#ce-quen-dit-overture)
   * [ce qu'en dit OSM officiellement](#ce-quen-dit-osm-officiellement)
   * [ce qu'en pense la communauté OSM française](#ce-quen-pense-la-communauté-osm-française)
   * [ce qu'en pense decryptageo](#ce-quen-pense-decryptageo)
* [Téléchargement de la data](#téléchargement-de-la-data)
* [Organisation de la donnée](#organisation-de-la-donnée)
   * [transportation](#transportation)
      * [Connectors](#connectors)
      * [Segments](#segments)
      * [Connections entre les segments](#connections-entre-les-segments)
   * [places](#places)
   * [buildings](#buildings)
   * [admins](#admins)
      * [administrative boundary](#administrative-boundary)
      * [locality](#locality)
* [Analyse avec duckdb](#analyse-avec-duckdb)
   * [En résumé](#en-résumé)
   * [Commun à tous les thèmes](#commun-à-tous-les-thèmes)
   * [transportation](#transportation-1)
      * [Volume des données](#volume-des-données)
      * [Source des données](#source-des-données)
      * [Schéma et samples de lignes](#schéma-et-samples-de-lignes)
   * [admin](#admin)
      * [Volume des données](#volume-des-données-1)
      * [Source des données](#source-des-données-1)
      * [Schéma et samples de lignes](#schéma-et-samples-de-lignes-1)
   * [places](#places-1)
      * [Volume des données](#volume-des-données-2)
      * [Source des données](#source-des-données-2)
      * [Schéma et samples de lignes](#schéma-et-samples-de-lignes-2)
   * [buildings](#buildings-1)
      * [Volume des données](#volume-des-données-3)
      * [Source des données](#source-des-données-3)
      * [Schéma et samples de lignes](#schéma-et-samples-de-lignes-3)


# C'est quoi, ça apporte quoi ?

C'est une base de données géospatiale ; la licence n'est pas très claire (pour personne), mais il semble y avoir deux licence : ODbL et une licence plus permissive = [CDLA](https://cdla.dev/permissive-2-0/).

Il y a [une démo](https://labs.overturemaps.org/) de la data avec des POIs, des routes, et des buildings.

On retrouve des grands noms dans les membres du comité : Tomtom, tous les GAFAM sauf Apple et Google, etc.

Mon avis (à froid) :

- pour un graphe routable, peu de valeur ajoutée par rapport à OSM, vu que toute la data est justement issue d'OSM pour le moment = septembre 2023 (ça pourrait changer dans le futur)
- possible intérêt sur le fait que la donnée est "raffinée" (cf. daylight)
- possible intérêt sur le fait que la donnée suit un schéma connu (par rapport à la data OSM brute dont les tags sont un joyeux foutoir)
- intérêt probable sur l'agrégation d'autres sources de données qu'OSM, notamment sur les POIs (qui semblent être l'un des points noirs d'OSM)

----

Quelques extraits de [leur page d'accueil](https://overturemaps.org/).

Public visé :

> Overture is for developers who build map services or use geospatial data.

Ce qu'apporte Overture :

- Solution pour agréger différentes sources de données :
    > Sourcing and curating high-quality, up-to-date, and comprehensive map data from disparate sources is difficult and expensive.
    >
    > Overture aims to incorporate map data from multiple sources including Overture Members, civic organizations, and open data sources.
- Solution pour n'utiliser qu'un unique identifier pour représenter une feature du monde réel :
    > Multiple datasets reference the same real-world entities using their own conventions and vocabulary, making them difficult to merge and combine.
    >
    > Overture Maps will simplify interoperability by providing a system that links entities from different data sets to the same real-world entities.
- Solution pour avoir une data à la qualité vérifiée :
    > Map data is vulnerable to errors and inconsistencies.
    >
    > Overture Maps data will undergo validation checks to detect map errors, breakage, and vandalism to help ensure that map data can be used in production systems.
- Solution pour avoir une data à la structure clairement définie :
    > Open map data can lack the structure needed to easily build map products.
    >
    > Overture will define and drive adoption of a common, well-structured, and documented data schema to create an easy-to-use ecosystem of map data.

# Quel lien avec OSM ?

## ce qu'en dit overture

https://overturemaps.org/resources/faq/

> What is the relationship between Overture and OpenStreetMap?
>
> Overture is a data-centric map project, not a community of individual map editors. Therefore, Overture is intended to be complementary to OSM. We combine OSM with other sources to produce new open map data sets. Overture data will be available for use by the OpenStreetMap community under compatible open data licenses. Overture members are encouraged to contribute to OSM directly.

^ overture **UTILISE** la data OSM (et pour moi, c'est pas clair comment, vu l'ODbL !) ; ils se présentent comme complémentaires à OSM.

## ce qu'en dit OSM officiellement

https://blog.openstreetmap.org/2022/12/22/views-from-the-openstreetmap-foundation-on-the-launch-of-overture/

En gros, ils :

- reconnaissent l'intérêt d'overture (_The technical problems that Overture is addressing, such as quality checks, data integration, and alignment to schemas, are valuable for any map data provider._)
- encouragent un modèle ouvert
- trouvent qu'overture n'est pas encore très clair (je suis d'accord)
- pensent qu'une data autoamtique n'est pas viable, i.e. qu'un modèle communautaire est inévitable
- demandent à être inclus dans la boucle des discussions stratégiques plutôt que de créer un compétiteur

## ce qu'en pense la communauté OSM française

https://forum.openstreetmap.fr/t/overture-maps-parlons-en/11616

Très long thread, mais très intéressant, il part d'une réunion de la communauté, et enchaîne avec une discussion :

- les [notes de la réunion](https://forum.openstreetmap.fr/t/overture-maps-parlons-en/11616/18) :
    - 22 présents, dont une collègue ;-)
    - la discussion porte sur les réactions à avoir dans la communauté OSM
- beaucoup de questionnements par rapport à la licence des données overture vis-à-vis de l'ODbL
- seul intérêt aux yeux de certains = les POIs (point faible d'OSM, apparemment)
- avis partagés sur la qualité desdits POIs ; exemples :
    > C’est vraiment un fourre-tout ce jeu de données. Ca mélange des POI, des noms de places, des arrêts de tram… La géoloc est pas trop dégueu, mais il manque des commerces essentiels. Par contre, on a des POI de cabinets privés, d’autoentrepreneurs, d’assocs qu’on ne voit pas depuis la rue.
    >
    > Du grand n’importe quoi oui ! J’ai regardé dans mon quartier et ma campagne: positionnement vraiment approximatif, des commerces fermés, qui ont changé de nom, de nature. Bref, quantité, mais pas de qualité…
    >
    > ce n’est pas du tout une source utilisable pour OSM ; Et ce n’est pas non plus un digne remplaçant d’OSM pour les POIs : pas plus d’exhaustivité pas plus d’actualité …et trop de géolocs incompatibles avec un usage grande échelle (position au numéro, au bâtiment…)
    >
    > J’ai regardé par chez moi : ni fait ni à faire. Un peu le même constat, mais c’est la proportion de doublons, de vieilleries, de magasins repris avec du coup un doublon et deux emplacements, d’incomplétion qui est impressionnant.
    >
    > Je ne serais peut-être pas aussi catégorique. Tout n’est pas à jeter. Je viens d’extraire une petite zone (ville de Lons-le-Saunier) du jeu de donnée « Places » monde (merci DuckDB). (...) la géolocalisation est plutôt bonne. À explorer.
    >
    > Je suis exactement en train de faire la même chose France entière. Et j’ai le même constat que toi. Attestation à ne pas dénigrer tout ça trop vite.
    >
    > Après avoir exploré assez succinctement les données avec cet outil 3, ainsi que celui-ci 2 déjà cité plus haut, je suis assez d’accord qu’il y des choses exploitables en particulier sur les PoI (« places ») et qui pourraient être ajoutées dans OSM.
- quelques snippets de code (e.g. [conversion des places d'overture en gpkg](https://forum.openstreetmap.fr/t/overture-maps-parlons-en/11616/59))

## ce qu'en pense decryptageo

(https://decryptageo.fr/ = site d'actualités géographiques)

https://decryptageo.fr/overture-maps-carte-futur-crise-dadolescence-dopenstreetmap/

Résumé en demi-teinte pour OSM : overture acte la data OSM est la base de toute donnée carto, mais montre qu'OSM seul n'a pas pu répondre à tous les enjeux.

Quelques extraits :

> (Overture) adoube définitivement OpenStreetMap comme le socle commun de la carte du futur.

> Overture vise à l’évidence à rivaliser avec l’omniprésent Google Maps et avec Apple Plan.

> Or Overture est pensé comme un OSM enrichi. TomTom adjoindra ainsi ses nombreuses données collectées via les capteurs de l’IoT et en premier lieu ceux des nombreuses voitures équipées de sa solution, notamment auprès de ses clients Stellantis, Volkswagen ou encore Uber.

> L’absence dans l’alliance de la fondation OpenStreetMap — qui a été mise devant le fait accompli — acte sa relégation comme simple fournisseur d’une base de données upstream. Désormais, le monde entier utilisera en effet les données OSM mais ce sera via sa vitrine grand public, Overture, un écosystème contrôlé directement par les membres du consortium. (...) Trouver sa nouvelle place s’avérera désormais une tâche délicate pour OpenStreetMap pour qui le rapport de force s’est inversé.

> La majorité des acteurs Européens du numérique qui a jusqu’à présent raté le train d’OpenStreetMap serait tout aussi bien inspirée de repenser sa stratégie liée aux données géographiques.

# Téléchargement de la data

La donnée est dispo sur AWS ou AZURE.

Tout est expliqué [ici](https://overturemaps.org/download/) et [là](https://github.com/OvertureMaps/data/blob/main/README.md#5-download-the-parquet-files) pour des commandes concrètes.

La dernière release en date (qui est aussi la première) est : `Overture 2023-07-26-alpha.0`

Même si on n'est pas obligés de downloader la data en local pour bosser avec, on peut le faire :


```
pipx install awscli

mkdir admins
aws s3 cp --region us-west-2 --no-sign-request --recursive "s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins" admins

mkdir places
aws s3 cp --region us-west-2 --no-sign-request --recursive s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places places

mkdir transportation
aws s3 cp --region us-west-2 --no-sign-request --recursive s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation transportation

mkdir buildings
aws s3 cp --region us-west-2 --no-sign-request --recursive s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings buildings
```

**ATTENTION** : la data est volumineuse :

- 657 Mio pour les admins
- 8.1 Gio pour les places
- 82 Gio pour les transportation
- 111 Gio pour les buildings

**ATTENTION** : la taille du fichier final évolue au cours du téléchargement → il ne faut pas se fier aux tailles indiquées au début, ça va grossir (multiplié par 10, même, dans le cas des transportations).

# Organisation de la donnée

Les données sont groupées en 4 catégories (appelées `theme`) :

- `places` = les POIs
- `buildings` = les bâtiments
- `transportation` = le réseau routier
- `admin` = les entités administratives et leurs frontières

La donnée est au format parquet, elle suit un schéma documenté = https://docs.overturemaps.org/

Tous les objets overture ont certaines propriétés globales :

- `type` (forcément parmi `["connector","building","segment","locality","administrativeBoundary","place"]`)
- `theme` (forcément parmi `["admins","buildings","places","transportation"]`)
- `sources` qui indique d'où proviennent les données
- `version`
- `etc.

## transportation

Les données routières sont des polylines appelées `segments` connectées par des nœuds appels `connectors` :

### Connectors

https://docs.overturemaps.org/reference/transportation/connector

> Connectors help define the topology of the transportation network by defining points of physical connection between two or more Segments.
>
> Apart from their point geometry and the core properties required for all Overture features, connectors do not have any other properties.
>
> Connectors create physical connections between segments. Connectors are compatible with GeoJSON Point features.

^ les connectors sont des noeuds de jonction entre des edges

> Connector's geometry which MUST be a Point as defined by GeoJSON

^ les connectors ont forcément une géométrie de type point.

Les connectors n'ont pas de propriétés spéciales aux connectors ils n'ont que les propriétés globales à tout objet overture.

À l'analyse du contenu des objets des fichiers `transportation`, on dirait que les connectors ont les champs `road`, `subtype` et `connectors` vides (ce qui est logique, vu que ces champs sont pertinents pour les segments).

### Segments

https://docs.overturemaps.org/reference/transportation/segment/


> Segments are paths which can be traveled by people or things. Segment geometry represents the center-line of a path which may be traveled.
>
> Segment properties describe both the physical attributes (e.g. road surface and width) and non-physical attributes (e.g. access restriction rules) of that path.

^ les segments sont les edges du graphe, avec leurs propriétés.

En plus des properties overture, si le segment est de type `road`, il gagne une propriété `road` supplémentaire qui décrit la route :

- le type de route
- son nom (ce qui a l'air d'être un objet assez complexe)
- le type de surface
- des booléens qui caractérisent la route (e.g. isBridge, isPrivate)
- pour chaque lane de la route, la direction et les restrictions qui s'appliquent

Au global pour toute la route, les restrictions qui s'appliquent :

- speedLimit
- accès autorisé ou interdit
- turn restrictions = _Turn restriction from this segment to other segments._ (avec la raison, les contraintes = s'il faut emprunter des segments pour que la restriction s'applique, etc.)

### Connections entre les segments

> Two Segments are physically connected if a common Connector is referenced from the connectors property of both Segments.
>
> Where this occurs, both Segment geometries must contain the common Connector's coordinates.
>
> A physical connection between Segments indicates that it may be possible to travel from one segment to the next at the connected location,
> provided the Segment properties do not indicate a restriction, such as a turn restriction, which would prevent the transition in a particular case.
>
> Conversely, two segments are not physically connected—even if their geometries intersect—if they do not share a Connector in common.

^ en résumé, tout est assez logique :

- chaque segment a une liste de connectors
- si un même connector apparaît dans deux segments différents, alors ils sont considérés comme connectés (et dans ce cas, le point du connector doit être présent dans les deux polylines des segments)
- ça n'est pas parce que deux segments sont connectés qu'on a automatiquement le droit d'emprunter la connexion (e.g. sens interdit pour les voitures)
- et quoi qu'il arrive si deux segments ne sont pas connectés, alors on ne peut physiquement pas passer de l'un à l'autre

Types de segments = actuellement, il y a 3 types de segments, dont deux pas encore prêts  :

- `road` = _A road segment represents a section of any kind of road, street, or path, including a dedicated path for walking or cycling (but excluding a railway)._
- `railway` (work-in-progress)
- `waterway` (work-in-progress)

## places

https://docs.overturemaps.org/reference/places/place

> A place is a business or point of interest within the world.

^ les _places_ sont les POI.

> Place's geometry which MUST be a Point as defined by GeoJSON schema.

^ la géométrie est obligatoirement de type point.

En plus des propriétés overture, les places ont pas mal de propriétés propres :

- `names` = le nom (ce qui a l'air d'être, tout comme pour les routes, un objet assez complexe, loin d'être trivial...)
- `catégorie` = le seule champ obligatoire (la liste des catégories possibles est en TODO)
- `confidence` = entre 0 et 1, plus c'est haut, plus on est confiant
- `addresses` = les adresses du POI
- `brand` = la marque du POI
- `websites` / `socials` / `phone` / `email` = les sites et coordonnées du POI

## buildings

https://docs.overturemaps.org/reference/buildings/building/

> Buildings are human-made structures with roofs or interior spaces that are permanently or semi-permanently in one place

La géométrie est obligatoirement un polygone.

En plus des properties overture, les buildings ont :

- `names`
- `height`
- `numFloors`
- `class` = le type de building

## admins

Overture sépare les données admin en deux : les frontières et les zones délimitées par les frontières.

### administrative boundary

https://docs.overturemaps.org/reference/admins/administrativeBoundary

> Administrative Boundaries are borders surrounding Administrative Localities.

^ c'est une frontière administrative, i.e. la délimitation d'une zone.

La géométrie est obligatoirement une polyline.

Il n'y a pas de propriété particulière en dehors des propriétés communes à tous les objets overture.

### locality

https://docs.overturemaps.org/reference/admins/locality

> Localities are named, and typically populated, areas.

^ les localities sont des zones.

Leur géométrie est soit un Point, soit un Polygon, soit un MultiPolygon.

Il y en a de deux types :

- `administrativeLocality` = _represent countries and hierarchical subdivisions of countries._
- `namedLocality` = _named areas which are not formally part of the hierarchical subdivision of a country_

Et un autre champ `type` décrit la locality qui peut être `["country","county","state","region","province","district","city","town","village","hamlet","borough","suburb","neighborhood","municipality"]`.

Les `administrativeLocality` contiennent des propriétés en plus, notamment `adminLevel` pour positionner la localité dans la hiérarchie (est-ce une région ? un département ? une commune ?)


# Analyse avec duckdb

## En résumé

**transportation** : toute la donnée transportation provient d'OSM, il y a 294 millions de sections, et 330 millions de connectors (soit 624 millions d'objets dans le graphe en tout !)

**admin** : on a 99k admins, avec une écrasante majorité de frontières administratives et quelques localities... On dirait que les sources principales sont Tomtom et ESRI.

**buildings** : il y a 785 millions de buildings, et à part quelques sources locales à certaines grandes villes, on est sur le triplet :

- ESRI
- OSM
- Microsoft ML Buildings

**places** : 59 millions de POIs, de sources variées (certains POIs ont 175 sources !), et notamment :

- des POI msft (que je suppose être microsoft)
- des POI meta (que je suppose être facebook)

## Commun à tous les thèmes

L'analyse ci-dessous est faite en septembre 2023.

Comme overture agrège des données de plusieurs sources différentes, chaque feature est associée à sa source.

Je n'indique que les requêtes à destination des fichiers sur AWS, mais pour analyser la data téléchargée en local, il suffit de passer le chemin vers les fichiers locaux :

```sql
SELECT DISTINCT len(sources) FROM read_parquet("/path/to/OVERTURE/places/*/*");
```

## transportation

### Volume des données

624 millions d'objets (!)

294 millions de sections, et 330 millions de connectors.

```sql
SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │    624933749 │
-- └──────────────┘

SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/type=segment/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │    294327662 │
-- └──────────────┘

SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/type=connector/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │    330606087 │
-- └──────────────┘
```

### Source des données

TL;DR = toute la donnée transportation provient d'OSM.

Connaître le nombre de sources pour la donnée transportation :

```sql
SELECT DISTINCT len(sources) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ len(sources) │
-- │    int64     │
-- ├──────────────┤
-- │            1 │
-- └──────────────┘
```

^ toute la donnée transportation (aussi bien segment que connector) a une seule source de données ; une deuxième requête montre que tout vient d'OSM :

```sql
SELECT DISTINCT sources[1]['dataset'] FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/*/*', filename=true, hive_partitioning=1);
-- ┌───────────────────────┐
-- │ sources[1]['dataset'] │
-- │       varchar[]       │
-- ├───────────────────────┤
-- │ [OpenStreetMap]       │
-- └───────────────────────┘
```

### Schéma et samples de lignes

Analyse des schémas overture + 5 premières lignes :

```sql
DESCRIBE SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/*/*', filename=true, hive_partitioning=1);
-- ┌─────────────┬────────────────────────────────────────────────────────────┬─────────┬─────────┬─────────┬─────────┐
-- │ column_name │                        column_type                         │  null   │   key   │ default │  extra  │
-- │   varchar   │                          varchar                           │ varchar │ varchar │ varchar │ varchar │
-- ├─────────────┼────────────────────────────────────────────────────────────┼─────────┼─────────┼─────────┼─────────┤
-- │ id          │ VARCHAR                                                    │ YES     │         │         │         │
-- │ updatetime  │ TIMESTAMP                                                  │ YES     │         │         │         │
-- │ version     │ INTEGER                                                    │ YES     │         │         │         │
-- │ level       │ INTEGER                                                    │ YES     │         │         │         │
-- │ subtype     │ VARCHAR                                                    │ YES     │         │         │         │
-- │ connectors  │ VARCHAR[]                                                  │ YES     │         │         │         │
-- │ road        │ VARCHAR                                                    │ YES     │         │         │         │
-- │ sources     │ MAP(VARCHAR, VARCHAR)[]                                    │ YES     │         │         │         │
-- │ bbox        │ STRUCT(minx DOUBLE, maxx DOUBLE, miny DOUBLE, maxy DOUBLE) │ YES     │         │         │         │
-- │ geometry    │ BLOB                                                       │ YES     │         │         │         │
-- │ filename    │ VARCHAR                                                    │ YES     │         │         │         │
-- │ theme       │ VARCHAR                                                    │ YES     │         │         │         │
-- │ type        │ VARCHAR                                                    │ YES     │         │         │         │
-- ├─────────────┴────────────────────────────────────────────────────────────┴─────────┴─────────┴─────────┴─────────┤
-- │ 13 rows                                                                                                6 columns │
-- └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/*/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌──────────────────────┬──────────────────────┬─────────┬───────┬─────────┬───┬──────────────────────┬──────────────────────┬──────────────────────┬────────────────┬───────────┐
-- │          id          │      updatetime      │ version │ level │ subtype │ … │         bbox         │       geometry       │       filename       │     theme      │   type    │
-- │       varchar        │      timestamp       │  int32  │ int32 │ varchar │   │ struct(minx double…  │         blob         │       varchar        │    varchar     │  varchar  │
-- ├──────────────────────┼──────────────────────┼─────────┼───────┼─────────┼───┼──────────────────────┼──────────────────────┼──────────────────────┼────────────────┼───────────┤
-- │ connector.8f3da556…  │ 2023-07-24 00:27:1…  │       0 │       │         │ … │ {'minx': 75.935448…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ transportation │ connector │
-- │ connector.8f2ba010…  │ 2023-07-24 00:27:1…  │       0 │       │         │ … │ {'minx': -72.21874…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ transportation │ connector │
-- │ connector.8f1ea006…  │ 2023-07-24 00:27:1…  │       0 │       │         │ … │ {'minx': 12.223394…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ transportation │ connector │
-- │ connector.8f64ab44…  │ 2023-07-24 00:27:1…  │       0 │       │         │ … │ {'minx': 95.749541…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ transportation │ connector │
-- │ connector.8f489a41…  │ 2023-07-24 00:27:1…  │       0 │       │         │ … │ {'minx': -96.57796…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ transportation │ connector │
-- ├──────────────────────┴──────────────────────┴─────────┴───────┴─────────┴───┴──────────────────────┴──────────────────────┴──────────────────────┴────────────────┴───────────┤
-- │ 5 rows                                                                                                                                                  13 columns (10 shown) │
-- └───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

SELECT ST_GeomFromWKB(geometry) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/type=connector/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌────────────────────────────────┐
-- │    st_geomfromwkb(geometry)    │
-- │            geometry            │
-- ├────────────────────────────────┤
-- │ POINT (75.935448 28.926191)    │
-- │ POINT (-72.2187488 48.5193487) │
-- │ POINT (12.2233941 44.9207945)  │
-- │ POINT (95.7495414 16.4539188)  │
-- │ POINT (-96.57796 31.165204)    │
-- └────────────────────────────────┘

SELECT ST_GeomFromWKB(geometry) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/type=segment/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
-- │                                                                                                       st_geomfromwkb(geometry)                                                                                                             │
-- │                                                                                                               geometry                                                                                                                     │
-- ├────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
-- │ LINESTRING (-107.5873493 37.234346, -107.5878495 37.2342104, -107.5884059 37.2340581, -107.5889411 37.2339061, -107.5898932 37.233646, -107.5907824 37.2333979, -107.5912304 37.2332708, -107.591516 37.2331896, -107.5931535 37.2327326)  │
-- │ LINESTRING (-104.3583375 50.4686508, -104.356578 50.4685005, -104.356121 50.4683011, -104.3562581 50.4675626, -104.3564493 50.4665338, -104.3563141 50.4660475, -104.3548185 50.4656323, -104.353634 50.4652143…                           │
-- │ LINESTRING (-76.5928227 39.1155801, -76.5924291 39.113732, -76.592085 39.1121165, -76.5920123 39.1117751, -76.5919797 39.111622, -76.5914504 39.1091364)                                                                                   │
-- │ LINESTRING (16.8155591 52.3966511, 16.8155481 52.3966656, 16.8155317 52.3966846)                                                                                                                                                           │
-- │ LINESTRING (7.9296541 52.8704244, 7.9332905 52.8688364, 7.9372954 52.867085, 7.9378862 52.8669101, 7.9384745 52.8667878, 7.938908 52.8667111, 7.9397821 52.8665565, 7.9404575 52.86645, 7.9407112 52.86641…                                │
-- └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## admin

### Volume des données

On a 99k admins, avec une écrasante majorité de frontières administratives et quelques localities...


```sql
SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │        99403 │
-- └──────────────┘

SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/type=administrativeBoundary/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │        96455 │
-- └──────────────┘

SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/type=locality/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │         2948 │
-- └──────────────┘
```


### Source des données

TL;DR : Tomtom et ESRI (= l'entreprise qui édite arcgis) semblent être les contributeurs principaux des admins.

```sql
SELECT DISTINCT len(sources) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ len(sources) │
-- │    int64     │
-- ├──────────────┤
-- │            2 │
-- │            1 │
-- │           41 │
-- │           40 │
-- └──────────────┘
```

^ soit une ou deux sources, soit... 41 !

```sql
SELECT DISTINCT sources[1]['dataset'] FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/*/*', filename=true, hive_partitioning=1) WHERE len(sources) = 1;
-- ┌───────────────────────┐
-- │ sources[1]['dataset'] │
-- │       varchar[]       │
-- ├───────────────────────┤
-- │ [TomTom]              │
-- └───────────────────────┘
```

^ quand il n'y a qu'une source, c'est Tomtom

```sql
SELECT DISTINCT (sources[1]['dataset'], sources[2]['dataset']) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/*/*', filename=true, hive_partitioning=1) WHERE len(sources) = 2;
-- ┌────────────────────────────────────────────────────────┐
-- │ main.row(sources[1]['dataset'], sources[2]['dataset']) │
-- │             struct( varchar[],  varchar[])             │
-- ├────────────────────────────────────────────────────────┤
-- │ {'': [TomTom], '': [Esri Community Maps]}              │
-- └────────────────────────────────────────────────────────┘
```

^ quand il y a deux sources, ce sont Tomtom et ESRI.

Pas facile (avec mon niveau de connaissance actuelle sur le nesting duckdb) d'analyser les cas 40 et 41, mais on dirait à l'oeil que les deux seuls providers d'admins sont Tomtom et ESRI...

### Schéma et samples de lignes

```sql
DESCRIBE SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/type=*/*', filename=true, hive_partitioning=1);
-- ┌──────────────────────┬────────────────────────────────────────────────────────────┬─────────┬─────────┬─────────┬─────────┐
-- │     column_name      │                        column_type                         │  null   │   key   │ default │  extra  │
-- │       varchar        │                          varchar                           │ varchar │ varchar │ varchar │ varchar │
-- ├──────────────────────┼────────────────────────────────────────────────────────────┼─────────┼─────────┼─────────┼─────────┤
-- │ id                   │ VARCHAR                                                    │ YES     │         │         │         │
-- │ updatetime           │ VARCHAR                                                    │ YES     │         │         │         │
-- │ version              │ INTEGER                                                    │ YES     │         │         │         │
-- │ names                │ MAP(VARCHAR, MAP(VARCHAR, VARCHAR)[])                      │ YES     │         │         │         │
-- │ adminlevel           │ INTEGER                                                    │ YES     │         │         │         │
-- │ maritime             │ VARCHAR                                                    │ YES     │         │         │         │
-- │ subtype              │ VARCHAR                                                    │ YES     │         │         │         │
-- │ localitytype         │ VARCHAR                                                    │ YES     │         │         │         │
-- │ context              │ VARCHAR                                                    │ YES     │         │         │         │
-- │ isocountrycodealpha2 │ VARCHAR                                                    │ YES     │         │         │         │
-- │ isosubcountrycode    │ VARCHAR                                                    │ YES     │         │         │         │
-- │ defaultlanugage      │ VARCHAR                                                    │ YES     │         │         │         │
-- │ drivingside          │ VARCHAR                                                    │ YES     │         │         │         │
-- │ sources              │ MAP(VARCHAR, VARCHAR)[]                                    │ YES     │         │         │         │
-- │ bbox                 │ STRUCT(minx DOUBLE, maxx DOUBLE, miny DOUBLE, maxy DOUBLE) │ YES     │         │         │         │
-- │ geometry             │ BLOB                                                       │ YES     │         │         │         │
-- │ filename             │ VARCHAR                                                    │ YES     │         │         │         │
-- │ theme                │ VARCHAR                                                    │ YES     │         │         │         │
-- │ type                 │ VARCHAR                                                    │ YES     │         │         │         │
-- ├──────────────────────┴────────────────────────────────────────────────────────────┴─────────┴─────────┴─────────┴─────────┤
-- │ 19 rows                                                                                                         6 columns │
-- └───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/type=*/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌──────────────────────┬──────────────────────┬─────────┬──────────────────────┬───┬──────────────────────┬──────────────────────┬──────────────────────┬─────────┬──────────────────────┐
-- │          id          │      updatetime      │ version │        names         │ … │         bbox         │       geometry       │       filename       │  theme  │         type         │
-- │       varchar        │       varchar        │  int32  │ map(varchar, map(v…  │   │ struct(minx double…  │         blob         │       varchar        │ varchar │       varchar        │
-- ├──────────────────────┼──────────────────────┼─────────┼──────────────────────┼───┼──────────────────────┼──────────────────────┼──────────────────────┼─────────┼──────────────────────┤
-- │ 83186efffffffff110…  │ 2023-05-28T00:18:0…  │       0 │                      │ … │ {'minx': -0.702928…  │ \x01\x02\x00\x00\x…  │ s3://overturemaps-…  │ admins  │ administrativeBoun…  │
-- │ 85186a93fffffff110…  │ 2023-05-28T00:18:0…  │       0 │                      │ … │ {'minx': -0.681400…  │ \x01\x02\x00\x00\x…  │ s3://overturemaps-…  │ admins  │ administrativeBoun…  │
-- │ 83186efffffffff110…  │ 2023-05-28T00:18:0…  │       0 │                      │ … │ {'minx': -0.689162…  │ \x01\x02\x00\x00\x…  │ s3://overturemaps-…  │ admins  │ administrativeBoun…  │
-- │ 85186a93fffffff110…  │ 2023-05-28T00:18:0…  │       0 │                      │ … │ {'minx': -0.657948…  │ \x01\x02\x00\x00\x…  │ s3://overturemaps-…  │ admins  │ administrativeBoun…  │
-- │ 85186a93fffffff110…  │ 2023-05-28T00:18:0…  │       0 │                      │ … │ {'minx': -0.681400…  │ \x01\x02\x00\x00\x…  │ s3://overturemaps-…  │ admins  │ administrativeBoun…  │
-- ├──────────────────────┴──────────────────────┴─────────┴──────────────────────┴───┴──────────────────────┴──────────────────────┴──────────────────────┴─────────┴──────────────────────┤
-- │ 5 rows                                                                                                                                                            19 columns (9 shown) │
-- └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## places

### Volume des données

59 millions de POIs

```sql
SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │     59175720 │
-- └──────────────┘
```

### Source des données

```sql
SELECT DISTINCT len(sources) AS nb_sources FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/*/*', filename=true, hive_partitioning=1) ORDER BY nb_sources;
```

^ Les patterns de nombre de sources pour chaque features sont assez variés : il y en a 87 en tout.

Ça va de 1 (i.e. certaines places ont une unique source) à 175, i.e. certaines places ont 175 sources (!)

Notamment, il y a des sources variées :

- des POI msft (que je suppose être microsoft)
- des POI meta (que je suppose être facebook)

### Schéma et samples de lignes


```sql
DESCRIBE SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/*/*', filename=true, hive_partitioning=1);
-- ┌─────────────┬─────────────────────────────────────────────────────────────────────────┬─────────┬─────────┬─────────┬─────────┐
-- │ column_name │                               column_type                               │  null   │   key   │ default │  extra  │
-- │   varchar   │                                 varchar                                 │ varchar │ varchar │ varchar │ varchar │
-- ├─────────────┼─────────────────────────────────────────────────────────────────────────┼─────────┼─────────┼─────────┼─────────┤
-- │ id          │ VARCHAR                                                                 │ YES     │         │         │         │
-- │ updatetime  │ VARCHAR                                                                 │ YES     │         │         │         │
-- │ version     │ INTEGER                                                                 │ YES     │         │         │         │
-- │ names       │ MAP(VARCHAR, MAP(VARCHAR, VARCHAR)[])                                   │ YES     │         │         │         │
-- │ categories  │ STRUCT(main VARCHAR, alternate VARCHAR[])                               │ YES     │         │         │         │
-- │ confidence  │ DOUBLE                                                                  │ YES     │         │         │         │
-- │ websites    │ VARCHAR[]                                                               │ YES     │         │         │         │
-- │ socials     │ VARCHAR[]                                                               │ YES     │         │         │         │
-- │ emails      │ VARCHAR[]                                                               │ YES     │         │         │         │
-- │ phones      │ VARCHAR[]                                                               │ YES     │         │         │         │
-- │ brand       │ STRUCT("names" MAP(VARCHAR, MAP(VARCHAR, VARCHAR)[]), wikidata VARCHAR) │ YES     │         │         │         │
-- │ addresses   │ MAP(VARCHAR, VARCHAR)[]                                                 │ YES     │         │         │         │
-- │ sources     │ MAP(VARCHAR, VARCHAR)[]                                                 │ YES     │         │         │         │
-- │ bbox        │ STRUCT(minx DOUBLE, maxx DOUBLE, miny DOUBLE, maxy DOUBLE)              │ YES     │         │         │         │
-- │ geometry    │ BLOB                                                                    │ YES     │         │         │         │
-- │ filename    │ VARCHAR                                                                 │ YES     │         │         │         │
-- │ theme       │ VARCHAR                                                                 │ YES     │         │         │         │
-- │ type        │ VARCHAR                                                                 │ YES     │         │         │         │
-- ├─────────────┴─────────────────────────────────────────────────────────────────────────┴─────────┴─────────┴─────────┴─────────┤
-- │ 18 rows                                                                                                             6 columns │
-- └───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=place/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌──────────────────────┬──────────────────────┬─────────┬──────────────────────┬──────────────────────┬─────────────────────┬───┬──────────────────────┬──────────────────────┬──────────────────────┬─────────┬─────────┐
-- │          id          │      updatetime      │ version │        names         │      categories      │     confidence      │ … │         bbox         │       geometry       │       filename       │  theme  │  type   │
-- │       varchar        │       varchar        │  int32  │ map(varchar, map(v…  │ struct(main varcha…  │       double        │   │ struct(minx double…  │         blob         │       varchar        │ varchar │ varchar │
-- ├──────────────────────┼──────────────────────┼─────────┼──────────────────────┼──────────────────────┼─────────────────────┼───┼──────────────────────┼──────────────────────┼──────────────────────┼─────────┼─────────┤
-- │ tmp_9EF32E0C9C03C9…  │ 2023-07-24T00:00:0…  │       0 │ {common=[{value=Br…  │ {'main': veterinar…  │  0.5989174246788025 │ … │ {'minx': -4.175979…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ places  │ place   │
-- │ tmp_FBB8618B19BE2F…  │ 2023-07-24T00:00:0…  │       0 │ {common=[{value=Tr…  │ {'main': park, 'al…  │  0.9108787178993225 │ … │ {'minx': -58.53305…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ places  │ place   │
-- │ tmp_081A89B04D0E03…  │ 2023-07-24T00:00:0…  │       0 │ {common=[{value=St…  │ {'main': beauty_sa…  │  0.9628990292549133 │ … │ {'minx': -46.59369…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ places  │ place   │
-- │ tmp_45CF77B709A680…  │ 2023-07-24T00:00:0…  │       0 │ {common=[{value=เต้…  │ {'main': thai_rest…  │ 0.47563284635543823 │ … │ {'minx': 99.555783…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ places  │ place   │
-- │ tmp_9EE93E03D9BFDB…  │ 2023-07-24T00:00:0…  │       0 │ {common=[{value=ร้า…  │ {'main': community…  │  0.5783658027648926 │ … │ {'minx': 98.759051…  │ \x01\x01\x00\x00\x…  │ s3://overturemaps-…  │ places  │ place   │
-- ├──────────────────────┴──────────────────────┴─────────┴──────────────────────┴──────────────────────┴─────────────────────┴───┴──────────────────────┴──────────────────────┴──────────────────────┴─────────┴─────────┤
-- │ 5 rows                                                                                                                                                                                           18 columns (11 shown) │
-- └────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## buildings

### Volume des données

785 millions de buildings, quand même !!!

```sql
SELECT COUNT(*) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ count_star() │
-- │    int64     │
-- ├──────────────┤
-- │    785524851 │
-- └──────────────┘
```

### Source des données

**TL;DR** : à part quelques sources locales à certaines grandes villes, on est sur le triplet :

- ESRI
- OSM
- Microsoft ML Buildings

```sql
SELECT DISTINCT len(sources) FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/*/*', filename=true, hive_partitioning=1);
-- ┌──────────────┐
-- │ len(sources) │
-- │    int64     │
-- ├──────────────┤
-- │            2 │
-- │            1 │
-- └──────────────┘
```

^ chaque building a une ou deux sources.

Sur les sources à proprement parler, je ne reporte pas les commandes, mais l'analyse sur mon poste des sources des buildings montrait des sources assez diverses :

```sql
SELECT DISTINCT sources[1]['dataset'] FROM read_parquet("/path/to/local/OVERTURE/buildings/*/*") WHERE len(sources) = 1;
-- ┌────────────────────────────────────────────────────────────────────────┐
-- │                         sources[1]['dataset']                          │
-- │                               varchar[]                                │
-- ├────────────────────────────────────────────────────────────────────────┤
-- │ [Esri Buildings | Denver Regional Council of Governments 2D Buildings] │
-- │ [Miami-Dade County Open Data 3D Buildings]                             │
-- │ [OpenStreetMap]                                                        │
-- │ [Microsoft ML Buildings]                                               │
-- │ [Portland Building Footprint 2D Buildings]                             │
-- │ [Washington DC Open Data 3D Buildings]                                 │
-- │ [NYC Open Data 3D Buildings]                                           │
-- │ [Austin Building Footprints Year 2013 2D Buildings]                    │
-- │ [City of Cambridge, MA Open Data 3D Buildings]                         │
-- │ [Denver Regional Council of Governments 2D Buildings]                  │
-- │ [Boston BPDA 3D Buildings]                                             │
-- │ [Esri Community Maps]                                                  │
-- │ [Esri Buildings | Austin Building Footprints Year 2013 2D Buildings]   │
-- ├────────────────────────────────────────────────────────────────────────┤
-- │                                13 rows                                 │
-- └────────────────────────────────────────────────────────────────────────┘

SELECT DISTINCT (sources[1]['dataset'], sources[2]['dataset']) FROM read_parquet("/path/to/local/OVERTURE/buildings/*/*") WHERE len(sources) = 2;
-- ┌────────────────────────────────────────────────────────┐
-- │ main.row(sources[1]['dataset'], sources[2]['dataset']) │
-- │             struct( varchar[],  varchar[])             │
-- ├────────────────────────────────────────────────────────┤
-- │ {'': [USGS Lidar], '': [Microsoft ML Buildings]}       │
-- │ {'': [USGS Lidar], '': [OpenStreetMap]}                │
-- └────────────────────────────────────────────────────────┘
```

### Schéma et samples de lignes

```sql
DESCRIBE SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/*/*', filename=true, hive_partitioning=1);
-- ┌─────────────┬────────────────────────────────────────────────────────────┬─────────┬─────────┬─────────┬─────────┐
-- │ column_name │                        column_type                         │  null   │   key   │ default │  extra  │
-- │   varchar   │                          varchar                           │ varchar │ varchar │ varchar │ varchar │
-- ├─────────────┼────────────────────────────────────────────────────────────┼─────────┼─────────┼─────────┼─────────┤
-- │ id          │ VARCHAR                                                    │ YES     │         │         │         │
-- │ updatetime  │ VARCHAR                                                    │ YES     │         │         │         │
-- │ version     │ INTEGER                                                    │ YES     │         │         │         │
-- │ names       │ MAP(VARCHAR, MAP(VARCHAR, VARCHAR)[])                      │ YES     │         │         │         │
-- │ level       │ INTEGER                                                    │ YES     │         │         │         │
-- │ height      │ DOUBLE                                                     │ YES     │         │         │         │
-- │ numfloors   │ INTEGER                                                    │ YES     │         │         │         │
-- │ class       │ VARCHAR                                                    │ YES     │         │         │         │
-- │ sources     │ MAP(VARCHAR, VARCHAR)[]                                    │ YES     │         │         │         │
-- │ bbox        │ STRUCT(minx DOUBLE, maxx DOUBLE, miny DOUBLE, maxy DOUBLE) │ YES     │         │         │         │
-- │ geometry    │ BLOB                                                       │ YES     │         │         │         │
-- │ filename    │ VARCHAR                                                    │ YES     │         │         │         │
-- │ theme       │ VARCHAR                                                    │ YES     │         │         │         │
-- │ type        │ VARCHAR                                                    │ YES     │         │         │         │
-- ├─────────────┴────────────────────────────────────────────────────────────┴─────────┴─────────┴─────────┴─────────┤
-- │ 14 rows                                                                                                6 columns │
-- └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

SELECT * FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/*/*', filename=true, hive_partitioning=1) LIMIT 5;
-- ┌──────────────────────┬──────────────────────┬─────────┬──────────────────────┬───────┬────────┬───┬──────────────────────┬──────────────────────┬──────────────────────┬───────────┬──────────┐
-- │          id          │      updatetime      │ version │        names         │ level │ height │ … │         bbox         │       geometry       │       filename       │   theme   │   type   │
-- │       varchar        │       varchar        │  int32  │ map(varchar, map(v…  │ int32 │ double │   │ struct(minx double…  │         blob         │       varchar        │  varchar  │ varchar  │
-- ├──────────────────────┼──────────────────────┼─────────┼──────────────────────┼───────┼────────┼───┼──────────────────────┼──────────────────────┼──────────────────────┼───────────┼──────────┤
-- │ tmp_77373230333237…  │ 2019-08-30T18:46:2…  │       0 │ {}                   │       │        │ … │ {'minx': 7.8592489…  │ \x01\x03\x00\x00\x…  │ s3://overturemaps-…  │ buildings │ building │
-- │ tmp_77323833303735…  │ 2014-05-20T08:11:5…  │       0 │ {}                   │       │        │ … │ {'minx': 2.6939908…  │ \x01\x03\x00\x00\x…  │ s3://overturemaps-…  │ buildings │ building │
-- │ tmp_38343239323138…  │ 2023-07-01T07:00:0…  │       0 │ {}                   │       │        │ … │ {'minx': -95.77524…  │ \x01\x03\x00\x00\x…  │ s3://overturemaps-…  │ buildings │ building │
-- │ tmp_77363730303736…  │ 2019-02-12T22:31:1…  │       0 │ {}                   │       │        │ … │ {'minx': 103.14134…  │ \x01\x03\x00\x00\x…  │ s3://overturemaps-…  │ buildings │ building │
-- │ tmp_77313039353835…  │ 2022-09-19T08:45:2…  │       0 │ {}                   │       │        │ … │ {'minx': 111.90019…  │ \x01\x03\x00\x00\x…  │ s3://overturemaps-…  │ buildings │ building │
-- ├──────────────────────┴──────────────────────┴─────────┴──────────────────────┴───────┴────────┴───┴──────────────────────┴──────────────────────┴──────────────────────┴───────────┴──────────┤
-- │ 5 rows                                                                                                                                                                  14 columns (11 shown) │
-- └───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
