* [Généralités postgis](#généralités-postgis)
* [Howtos](#howtos)
* [Exemples en vrac](#exemples-en-vrac)
   * [geojson des features dans une bbox](#geojson-des-features-dans-une-bbox)
   * [features dans une bbox avec l'operator &amp;&amp;](#features-dans-une-bbox-avec-loperator-)
   * [les 15 features les plus proches d'un point](#les-15-features-les-plus-proches-dun-point)

# Généralités postgis

http://www.postgis.fr/chrome/site/docs/workshop-foss4g/doc/introduction.html

postgis = postgres + 3 choses ajoutées :

- types de données (point, linestring, etc.)
- index spatial = bounding box ( https://postgis.net/workshops/postgis-intro/indexing.html )
- fonctions spatiales

Ce qu'on peut faire avec des géométries :
- `ST_GeometryType` : pour savoir s'il s'agit d'un point, polyline, etc.
- `ST_AsText` : pour afficher la géométrie au format texte
- `ST_X` / `ST_Y` : sur des points
- `ST_Length` / `ST_StartPoint` : sur des polylines
- `ST_Area` / `ST_Perimeter` : sur des polygones

Les géométries sont stockées dans un format propre à PostGIS, on peut faire des conversions à l'entrée et à la sortie :

- Well-Known Text (WKT)
- geojson
- SVG
- etc.

Relations spatiales, qui peuvent être utilisées dans une clause `WHERE` :
- `ST_Equals`
- `ST_Contains` / `ST_Within` / `ST_Intersects`
- `ST_Distance`

**ATTENTION** : les notions de contenance ne s'appliquent pas à la frontière des polygones ([lien](http://lin-ear-th-inking.blogspot.com/2007/06/subtleties-of-ogc-covers-spatial.html))

On peut faire des jointures sur les relations spatiales \o/

Notamment, pour requêter par proximité, voir l'opérateur <-> utilisé dans [cet exemple](https://postgis.net/docs/geometry_distance_knn.html)

Opérateurs sur les géométries :

- `&&` si A intersecte B      : https://postgis.net/docs/overlaps_geometry_box2df.html
- `~ ` si A contient B        : https://postgis.net/docs/contains_geometry_box2df.html
- `@ ` si A est contenu par B : https://postgis.net/docs/is_contained_box2df_geometry.html

note : pour une polyline, il y a une différence entre :

- son centroïde (pas nécessairement sur la polyline) : https://postgis.net/docs/ST_Centroid.html
- un point sur la polyline (éventuellement au milieu) : https://postgis.net/docs/ST_LineInterpolatePoint.html

# Howtos

Limiter une requête à une zone géographique donnée = `ST_Within` ou en alternative, utiliser l'operator `&&` (cf. exemples ci-dessous)

Construire un geojson = `ST_AsGeoJson` (cf. exemples ci-dessous)

Récupérer les géométries au format texte (WKT = Well-Known Test) plutôt que l'encodage postgis : `ST_AsText`

Transformer une géométrie dans un SRID différent : `ST_Transform(geom, 4326)`

Calculer une similarité entre deux polylines : [ST_HausdorffDistance](https://postgis.net/docs/ST_HausdorffDistance.html) ou [ST_FrechetDistance](ST_FrechetDistance) (qui nécessite un postgis plus récent)

# Exemples en vrac

**Attention** : rester critiques sur ces exemples, certaines requêtes sont anciennes et pas les plus efficaces.

## geojson des features dans une bbox

Avec `ST_Within` pour tester l'appartenance à la bbox, `ST_Collect` et `ST_AsGeoJson` pour récupérer le geojson :

```sql
SELECT ST_AsGeoJson(ST_Collect(geom))
FROM my_super_schema.my_super_table
WHERE ST_Within(geom, ST_Envelope(ST_GeomFromText('LINESTRING(3.010598 50.669919, 3.113300 50.586288)', 4326)))
;
```

**EDIT** : une ancienne version, deprecated, car 10 fois plus lente = en chaînant des requêtes, en filtrant avec `ST_Within`, et en construisant un geojson :

```sql
WITH

-- STEP 1 = on construit une bbox à partir des deux points, en utilisant ST_Envelope :
bbox AS (SELECT ST_Envelope(ST_GeomFromText('LINESTRING(3.010598 50.669919, 3.113300 50.586288)', 4326)) AS envelope),

-- STEP 2 = on filtre les sections dont la géométrie est contenue dans la bbox :
sections AS (
    SELECT section.id, section.name, section.geom
    FROM my_super_schema.my_super_table AS section
    WHERE (
        section.name ~~* '%périphérique%'
        AND
        ST_Within(section.geom, (SELECT envelope FROM bbox))
    )
)

-- STEP 3 = on fusionne les géométries, et on construit le geojson :
SELECT ST_AsGeoJson(ST_Multi(ST_Union(geom))) FROM sections
;
```

## features dans une bbox avec l'operator &&

Si la table `nodes` contient des points, je peux vérifier s'ils sont dans une bbox donnée :

```sql
SELECT id, tags->'height', ST_X(geom), ST_Y(geom)
FROM nodes
WHERE geom && ST_MakeEnvelope(
    -0.580191, -- west longitude
    44.837136, -- south latitude
    -0.578437, -- east longitude
    44.838377, -- north latitude
    4326);
```


## les 15 features les plus proches d'un point

Création de l'index :

```sql
CREATE INDEX idx_geometry ON mypolylines USING gist (col_geometry);
```

Requête des 15 polylines les plus proches d'un point :

```sql
-- the point :
WITH point AS (SELECT ST_SetSRID(ST_MakePoint(2.342360, 48.863653),4326) AS geom)

-- compute 15 closest sections to the point :
SELECT
    edge.id AS id,
    ST_DistanceSphere(edge.geom, point.geom) AS distance
FROM my_super_schema.my_super_table AS edge, point
ORDER BY edge.geom <-> (SELECT geom FROM point)
LIMIT 15;
```
