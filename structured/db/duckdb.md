**Contexte** : septembre 2023, je m'intéresse à la data overture, distribuée au format parquet ; l'un des outils recommandés par overture est duckdb, je l'utilise pour faire mes essais.

**C'est quoi ?** une database pour bosser avec des data orientées colonnes (de type parquet).

* [Installation](#installation)
* [Lancement](#lancement)
* [Extension spatiale](#extension-spatiale)
* [Hive partitioning](#hive-partitioning)
* [Performances](#performances)
   * [Profiler une requête](#profiler-une-requête)
   * [Comparaison requête locale vs. requête distante](#comparaison-requête-locale-vs-requête-distante)
* [Notes vrac à l'utilisation](#notes-vrac-à-lutilisation)


# Installation

[La page d'installation](https://duckdb.org/docs/archive/0.9.0/installation/) donne le lien permettant de télécharger un binaire précompilé :

```
wget https://github.com/duckdb/duckdb/releases/download/v0.9.0/duckdb_cli-linux-amd64.zip
unzip duckdb_cli-linux-amd64.zip
cp duckdb ~/.local/bin
```

À noter qu'il y a aussi un binding python installable avec pip : toutes les commandes que je passe dans la CLI sont utilisables dans un script python.

# Lancement

```
duckdb -c 'SELECT COUNT(*) FROM ...'
duckdb
duckdb MYFILE
```

# Extension spatiale

duckdb a [une extension spatiale](https://duckdb.org/docs/archive/0.8.1/extensions/spatial) permettant de gérer les objets géospatiaux usuels (point, ligne, polygone et leurs dérivés).

Pour l'utiliser :

```sql
INSTALL spatial;
LOAD spatial;
SELECT ST_GeomFromWKB(geometry) FROM read_parquet("/path/to/OVERTURE/places/type=place/*") LIMIT 5;
-- ┌──────────────────────────────┐
-- │   st_geomfromwkb(geometry)   │
-- │           geometry           │
-- ├──────────────────────────────┤
-- │ POINT (11.912291 55.01179)   │
-- │ POINT (-17.43676 14.72763)   │
-- │ POINT (16.845998 46.853739)  │
-- │ POINT (14.14847 49.30705)    │
-- │ POINT (-86.395182 36.831922) │
-- └──────────────────────────────┘
```

La doc indique que duckdb utilise gdal pour lire tout un tas de formats géospatiaux.

Pour **écrire** un format géospatial (à partir d'une table duckdb), c'est moins clair : d'un côté seul le format geopackage semble supporté en écriture dans la table des drivers, de l'autre une commande est donnée dans la doc pour écrire du geojson en utilisant gdal, y compris en lui passant des options (et laisse à penser que tout ce que gdal sait écrire, duckdb pourra l'exporter) :

```sql
COPY <table> TO 'some/file/path/filename.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON', LAYER_CREATION_OPTIONS 'WRITE_BBOX=YES');
```

# Hive partitioning

C'est quoi ? Le partitionnement d'une grande table selon des clés qui fait que les données sont splittées sur plusieurs fichiers. Ça permet retrouver facilement ses données sans avoir à lire trop de fichiers ; cf. [la doc de duckdb sur hive-partitioning](https://duckdb.org/docs/data/partitioning/hive_partitioning.html) :

> Hive partitioning is a partitioning strategy that is used to split a table into multiple files based on partition keys. The files are organized into folders. Within each folder, the partition key has a value that is determined by the name of the folder.

L'option `hive_partitioning` est automatiquement détectée si besoin.

On peut filtrer les requêtes par partition-key :
```sql
SELECT *
FROM read_parquet('orders/*/*/*.parquet', hive_partitioning=1)
WHERE year=2022 AND month=11;
```

# Performances

J'ai vérifié à l'occasion de mes essais avec la data overture que l'analyse était bien séquentielle : je peux faire des requêtes sur de très gros fichiers (plusieurs dizaines de Gio) sans saturer ma RAM : une requête sur un petit champ de toute la donnée transportation (=63 Gio) ne charge même pas 1 Gio de RAM.

## Profiler une requête

```sql
-- EXPLAIN ANALYZE SELECT DISTINCT sources[1]['dataset'] FROM read_parquet("/path/to/OVERTURE/transportation/*/*");
-- [...]
-- ┌─────────────────────────────────────┐
-- │┌───────────────────────────────────┐│
-- ││         Total Time: 22.30s        ││
-- │└───────────────────────────────────┘│
-- └─────────────────────────────────────┘
-- [...]
```

Je confirme avec un chronomètre à côté :

- que ce `Total Time` est bien le wall-clock time
- que c'est bien le temps qu'il faut pour la même query sans EXPLAIN ANALYZE

## Comparaison requête locale vs. requête distante

Note : avec la CLI, il y a une progressbar qui indique où en est une grosse requête distante.

Du coup j'en profite pour profiler la même requête avec amazon S3 :

```sql
EXPLAIN ANALYZE SELECT DISTINCT sources[1]['dataset'] FROM read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/type=*/*', filename=true, hive_partitioning=1);
-- ││         Total Time: 59.17s        ││
```

La requête est beaucoup plus lente (plus de deux fois plus lente) qu'une requête sur de la data locale. Mais clairement, un facteur 2, c'est pas démesuré non plus !

# Notes vrac à l'utilisation

De façon contre-intuitive, on dirait que l'indexing commence à 1 ? (ce qui serait cohérent avec postgresql)

```sql
SELECT JSON(sources[1]) FROM transportation LIMIT 10;
```

