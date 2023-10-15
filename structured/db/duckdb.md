**Contexte** : septembre 2023, je m'intéresse à la data overture, distribuée au format parquet ; l'un des outils recommandés par overture est duckdb, je l'utilise pour faire mes essais.

**C'est quoi ?** une database pour bosser avec des data orientées colonnes (de type parquet).

* [Installation](#installation)
* [Extension spatiale](#extension-spatiale)
* [Hive partitioning](#hive-partitioning)

# Installation

[La page d'installation](https://duckdb.org/docs/archive/0.9.0/installation/) donne le lien permettant de télécharger un binaire précompilé :

```
wget https://github.com/duckdb/duckdb/releases/download/v0.9.0/duckdb_cli-linux-amd64.zip
unzip duckdb_cli-linux-amd64.zip 
cp duckdb ~/.local/bin 
```

À noter qu'il y a aussi un binding python installable avec pip : toutes les commandes que je passe dans la CLI sont utilisables dans un script python.

# Extension spatiale

duckdb a [une extension spatiale](https://duckdb.org/docs/archive/0.8.1/extensions/spatial) permettant de gérer les objets géospatiaux usuels (point, ligne, polygone et leurs dérivés).

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
