**C'est quoi ?** Un outil qui importe des données OSM dans une base de données postgis

Semble bien plus rapide qu'osmosis (mais moins qu'osmium).

# Installation

https://imposm.org/docs/imposm3/latest/install.html

Simple à installer : un seul binaire (car en go), par contre uniquement sur x86_64.

```sh
cd /tmp
wget https://github.com/omniscale/imposm3/releases/download/v0.11.0/imposm-0.11.0-linux-x86-64.tar.gz
tar -xvzf imposm-0.11.0-linux-x86-64.tar.gz
cd imposm-0.11.0-linux-x86-64
./imposm version
```

# Utilisation

Préalable = la base de destination doit exister, avec postgis installé :

```sh
# dropdb poc_imposm
PGPASSWORD=mypassword  createdb -h localhost -p 5432 -U myuser "poc_imposm" -e
PGPASSWORD=mypassword  psql -h localhost -p 5432 -d poc_imposm -U myuser -c 'CREATE EXTENSION postgis;' -e
```

Lancement d'imposm :

```sh
./imposm import
-connection postgis://myuser:mypassword@localhost:5432/poc_imposm
-mapping /tmp/mapping_poc_imposm.json
-read /tmp/monaco-latest.osm.pbf
-write
-overwritecache
-limitto mypolygon.geojson
```

Limitation à un polygone geojson :

- utiliser geojson.io pour dessiner facilement le geojson
- on n'est pas limité à une bbox : ça peut être un polygone évolué

Mapping :

- c'est ce qui définit quels sont les ways/nodes/relations à importer (en fonction des tags) :
- https://imposm.org/docs/imposm3/latest/mapping.html
- https://imposm.org/docs/imposm3/latest/tutorial.html
