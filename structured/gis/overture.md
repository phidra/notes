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


# C'est quoi, ça apporte quoi ?

C'est une base de données géospatiale ; la licence n'est pas très claire (pour personne), mais il semble y avoir deux licence : ODbL et une licence plus permissive = [CDLA](https://cdla.dev/permissive-2-0/).

Il y a [une démo](https://labs.overturemaps.org/) de la data avec des POIs, des routes, et des buildings.

On retrouve des grands noms dans les membres du comité : Tomtom, tous les GAFAM sauf Apple et Google, etc.

Mon avis (à froid) :

- pour un graphe routable, peu de valeur ajoutée par rapport à OSM, vu que toute la data est justement issue d'OSM pour le moment (ça pourrait changer dans le futur)
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

