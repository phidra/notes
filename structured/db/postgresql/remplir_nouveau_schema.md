**Contexte** = j'ai un schéma préexistant, il contient plusieurs tables. Je veux créer un nouveau schéma contenant une nouvelle table ; cette nouvelle table contient des données agrégeant plusieurs des tables de l'ancien schéma.

Préalable = test de la query agrégeant les données de l'ancien schéma :

```sql
SELECT
    edge.edge_id,
    ST_AsGeoJson(ST_Transform(edge.geom, 4326)),
    edge.node_start,
    ST_AsGeoJson(ST_Transform(node.geom, 4326))
FROM schema1.edge
JOIN schema1.node ON node.node_id = edge.node_start
LIMIT 2
;
```

Création du schéma, et de la table de destination, vides pour le moment :

```sql
CREATE SCHEMA IF NOT EXISTS another_schema;

CREATE TABLE another_schema.my_new_table
(
    edge_id uuid PRIMARY KEY NOT NULL,
    edge_geom text NOT NULL,
    node_start uuid NOT NULL,
    node_start_geom text NOT NULL
)
;
```

Insertion dans la nouvelle table du nouveau schéma, de données issues de deux tables pré-existantes dans l'ancien schéma :

```sql
INSERT INTO another_schema.my_new_table
(
    edge_id,
    edge_geom,
    node_start,
    node_start_geom
)
SELECT
    edge.edge_id,
    ST_AsGeoJson(ST_Transform(edge.geom, 4326)),
    edge.node_start,
    ST_AsGeoJson(ST_Transform(node.geom, 4326))
FROM schema1.edge
JOIN schema1.node ON node.node_id = edge.node_start
;
```
