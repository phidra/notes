Contexte = janvier 2022, j'essaye de charger des données `.osm.pbf` dans qgis, ce qui m'amène à m'intéresser à GDAL/OGR.

* [Commandes utiles](#commandes-utiles)
* [C'est quoi GDAL ? OGR ?](#cest-quoi-gdal--ogr-)
* [Utilisation](#utilisation)
* [Configuration](#configuration)
   * [Fichier osmconf.ini](#fichier-osmconfini)
   * [Chargement des tags OSM](#chargement-des-tags-osm)
   * [Custom indexing](#custom-indexing)
   * [OSM_MAX_TMPFILE_SIZE](#osm_max_tmpfile_size)

# Commandes utiles

- trouver une way particulière à partir de son `osm_id` (en l'occurence [la way d'id 176577460](https://www.openstreetmap.org/way/176577460)) :
    ```sh
    ogrinfo -ro -where "osm_id = '176577460'" /tmp/monaco.osm.pbf lines
    ```
- récupérer un geojson avec la géométrie d'une section particulière :
    ```sh
    ogr2ogr -where "osm_id = '176577460'" -f 'GeoJSON' dest.geojson /tmp/monaco.osm.pbf lines
    ```
- récupérer un geojson avec la géométrie de _plusieurs_ sections :
    ```sh
    ogr2ogr -where "osm_id IN ('176577460', '166149571', '318654737', '158189815')" -f 'GeoJSON' dest.geojson /tmp/monaco.osm.pbf lines
    ```
- limiter l'import à une bbox (fonctionne pour `ogrinfo` et `ogr2ogr`) :
    ```sh
    ogrinfo -ro -spat 7.4143945 43.7285819 7.4155760 43.7289873 /tmp/monaco.osm.pbf lines
    ```

# C'est quoi GDAL ? OGR ?

https://live.osgeo.org/en/overview/gdal_overview.html

En gros, GDAL est une librairie permettant de charger BEAUCOUP de format cartographiques, et de les stocker dans un format unifié (utilisable derrière par des applications carto ; et justement utilisé par beaucoup de SIG comme QGIS ou GRASS).

OGR est la sous-partie de GDAL qui sait gérer les formats vectoriels (comme les fichiers `.osm.pbf`), par opposition aux formats RASTER.

La librairie s'accompagne de binaires utilitaires, notamment :
- `ogrinfo` pour analyser une source de données (e.g. fichier `.osm.pbf`), ou en extraire les infos d'une feature particulière (e.g. consulter les tags et la géométrie d'une way OSM)
- `ogr2ogr` pour convertir des données d'un format à un autre (e.g. convertir un fichier `.osm.pbf` en geojson, ou en dump postgis)

# Utilisation

```sh
# sample data :
wget -O /tmp/monaco.osm.pbf https://download.geofabrik.de/europe/monaco-latest.osm.pbf

ogrinfo /tmp/monaco.osm.pbf
# INFO: Open of `/tmp/monaco.osm.pbf'
# 	  using driver `OSM' successful.
# 1: points (Point)
# 2: lines (Line String)
# 3: multilinestrings (Multi Line String)
# 4: multipolygons (Multi Polygon)
# 5: other_relations (Geometry Collection)

ogrinfo /tmp/monaco.osm.pbf lines
# toutes les infos du layer 'lines' (il y en a beaucoup)
```

Ci-dessous, les premières features retournées par :

```sh
ogrinfo monaco.osm.pbf lines
```

<details>
  <summary>Click to expand!</summary>

```
INFO: Open of `monaco.osm.pbf'
      using driver `OSM' successful.

Layer name: lines
Geometry: Line String
Feature Count: -1
Extent: (7.409205, 43.723350) - (7.448637, 43.751690)
Layer SRS WKT:
GEOGCRS["WGS 84",
    DATUM["World Geodetic System 1984",
        ELLIPSOID["WGS 84",6378137,298.257223563,
            LENGTHUNIT["metre",1]]],
    PRIMEM["Greenwich",0,
        ANGLEUNIT["degree",0.0174532925199433]],
    CS[ellipsoidal,2],
        AXIS["geodetic latitude (Lat)",north,
            ORDER[1],
            ANGLEUNIT["degree",0.0174532925199433]],
        AXIS["geodetic longitude (Lon)",east,
            ORDER[2],
            ANGLEUNIT["degree",0.0174532925199433]],
    ID["EPSG",4326]]
Data axis to CRS axis mapping: 2,1
osm_id: String (0.0)
name: String (0.0)
highway: String (0.0)
waterway: String (0.0)
aerialway: String (0.0)
barrier: String (0.0)
man_made: String (0.0)
z_order: Integer (0.0)
other_tags: String (0.0)
OGRFeature(lines):4097656
  osm_id (String) = 4097656
  name (String) = Avenue Princesse Alice
  highway (String) = primary
  z_order (Integer) = 7
  other_tags (String) = "lanes"=>"2","lit"=>"yes","maxspeed"=>"30","surface"=>"asphalt"
  LINESTRING (7.4259518 43.7389494,7.4258602 43.7389997,7.4257964 43.739037,7.4257682 43.7390601,7.425743 43.7390849,7.4257083 43.7391243,7.4256818 43.7391636,7.4256611 43.7392238,7.4256536 43.7392768,7.4256563 43.7393298,7.4256802 43.7393746,7.4257083 43.7394273,7.4257117 43.7394671,7.4256947 43.7395025,7.4256666 43.7395255,7.4256203 43.7395471,7.4251395 43.7397104)

OGRFeature(lines):4098197
  osm_id (String) = 4098197
  name (String) = Boulevard d'Italie
  highway (String) = primary
  z_order (Integer) = 7
  other_tags (String) = "lanes"=>"2","lit"=>"yes"
  LINESTRING (7.4301052 43.7459324,7.4302348 43.7461404,7.4302983 43.7462114,7.4305681 43.7464179,7.4312044 43.7467952,7.4313819 43.7469277,7.4315142 43.7470242,7.4316736 43.7471634,7.4317295 43.7472372,7.4317478 43.7472774,7.4317564 43.7473241,7.4317564 43.7473696,7.4317425 43.7474438,7.4317323 43.7475177,7.4317353 43.7475492,7.4317796 43.7476591,7.4318459 43.7477366,7.4319666 43.7477967,7.4320693 43.7478296,7.4321917 43.7478497)

OGRFeature(lines):4224972
  osm_id (String) = 4224972
  name (String) = Avenue des Papalins
  highway (String) = residential
  z_order (Integer) = 3
  other_tags (String) = "lit"=>"yes","oneway"=>"yes","surface"=>"asphalt","smoothness"=>"excellent"
  LINESTRING (7.4174924 43.7296303,7.4174638 43.7296727,7.4174191 43.7297138,7.4173286 43.7297725,7.4172044 43.7298528)

[...]
```

</details>

# Configuration

La doc du driver GDAL permettant d'important des données OSM est : https://gdal.org/drivers/vector/osm.html

## Fichier `osmconf.ini`

L'import des features d'un fichier OSM est configuré par un fichier `osmconf.ini` (celui par défaut est : `/usr/share/gdal/osmconf.ini`).

Pour le customiser :

```sh
cp /usr/share/gdal/osmconf.ini /tmp/osmconf.ini
vim /tmp/osmconf.ini  # e.g. add a specific OSM tag to 'attributes'
OSM_CONFIG_FILE=/tmp/osmconf.ini ogrinfo -ro -where "osm_id = '176577460'" /tmp/monaco.osm.pbf lines
```

## Chargement des tags OSM

Par défaut, tous les tags OSM ne sont **PAS** chargés : ogr suit un fonctionnement _opt-in_ où seuls les tags explicitement mentionnés dans le fichier de conf sont chargés comme attributs, [source1](https://gdal.org/drivers/vector/osm.html#configuration), [source2](https://gdal.org/drivers/vector/osm.html#other-tags-field) :

> In the data folder of the GDAL distribution, you can find a osmconf.ini file that can be customized to fit your needs. \
> You can also define an alternate path with the OSM_CONFIG_FILE configuration option. \
> The customization is essentially which OSM attributes and keys should be translated into OGR layer fields. \
> Fields can be computed with SQL expressions (evaluated by SQLite engine) from other fields/tags. For example to compute the z_order attribute. \
> [...] \
> When keys are not strictly identified in the osmconf.ini file, the key/value pair is appended in a “other_tags” field, with a syntax compatible with the PostgreSQL HSTORE type.

Par exemple pour la section `[lines]`, les attributs chargés par défaut sont :

```ini
[lines]
# [...]
# keys to report as OGR fields
attributes=name,highway,waterway,aerialway,barrier,man_made
# [...]
```

Tous les tags qui ne sont pas explicitement mentionnés dans `attributes` se retrouvent packés dans un attribut fourre-tout = `other_tags`, au [format hstore](https://www.postgresql.org/docs/14/hstore.html).

Du coup, je peux éditer mon `osmconf.ini` pour ajouter d'autres tags à charger :

```ini
[lines]
attributes=name,highway,waterway,aerialway,barrier,man_made,surface,smoothness,oneway
```

## Custom indexing

Sur certains fichiers `.osm.pbf`, j'ai dû désactiver le custom indexing pour pouvoir charger mes données :

```sh
OSM_USE_CUSTOM_INDEXING=NO ogrinfo -ro -where "osm_id = '176577460'" /tmp/monaco.osm.pbf lines
```

(c'était la raison qui faisait que qgis ne chargeait pas le layer, mais ça a été beaucoup plus facile à comprendre avec `ogr2info`)

## OSM_MAX_TMPFILE_SIZE

Ceci peut être un paramètre intéressant à tuner pour optimiser (et éviter de dumper sur disque), [source](https://gdal.org/drivers/vector/osm.html#vector-osm) :

> The driver will use an internal SQLite database to resolve geometries. \
> If that database remains under 100 MB it will reside in RAM. \
> [...] \
> The 100 MB default threshold can be adjusted with the OSM_MAX_TMPFILE_SIZE configuration option (value in MB).
