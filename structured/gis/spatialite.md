spatialite est à sqlite ce que postgis est à postgresql

* [Généralités](#généralités)
* [Fonctionnement interne](#fonctionnement-interne)
* [Les CRS des géométries](#les-crs-des-géométries)


# Généralités

Certaines fonctions sont iso-postGIS, e.g. `ST_Transform` ([lien vers la doc](http://www.gaia-gis.it/gaia-sins/spatialite-sql-latest.html#p15)).

Il y a un vieux tutorial non-maintenu, mais qui introduit pas mal de choses : http://www.gaia-gis.it/gaia-sins/spatialite-cookbook/index.html

C'est une extension de sqlite, il faut donc la loader (notamment dans dBeaver) :

```sql
SELECT load_extension('mod_spatialite');
```

La liste des fonctions est [documentée ici](http://www.gaia-gis.it/gaia-sins/spatialite-sql-latest.html).


# Fonctionnement interne

Tout comme postgis, spatialite permet d'ajouter des colonnes géoémtriques aux tables.

Tout est géré comme une surcouche, il faut donc parfois appeler les commandes de bookkeeping ; p.ex. à la création d'une table, ça se fait en deux temps : 1. créer la table sans la colonne géométrique, et 2. lui adjoindre la colonne géométrique.

Dans une base spatialite, il y a du coup des tables de bookkeeping :

- `geometry_columns` qui stocke les informations sur les colonnes géométriques des différentes tables
- `spatial_ref_sys` (et `spatial_ref_sys_aux`) qui stockent les infos sur les différents CRS que spatialite connaît

Les projections sont effectuées avec la librairie [proj](https://docs.rs/proj/latest/proj/).

# Les CRS des géométries

```sql
-- spatialite connaît 6215 CRS :
SELECT COUNT(*) FROM spatial_ref_sys srs;

-- parmi ceux-ci, seuls 51 n'émanent pas d'EPSG (moins de 1%) :
SELECT COUNT(*) FROM spatial_ref_sys srs WHERE auth_name = 'epsg';
-- 6164

-- Les autres auth sont :
--     NONE (2 CRS : undefined cartesian + undefined geographic)
--     gfoss.it (4 CRS en Italie)
--     mj10777.de (tous les autres, ça semble être en allemagne)
```

On peut interroger le CRS d'une colonne géométrique d'une table :

```sql
SELECT * FROM geometry_columns gc  WHERE f_table_name = 'my_table' AND f_geometry_column = 'geom'
-- f_table_name|f_geometry_column|geometry_type|coord_dimension|srid|spatial_index_enabled|
-- ------------+-----------------+-------------+---------------+----+---------------------+
-- my_table    |geom             |            0|              2|3949|                    1|

-- ^ la colonne 'geom' de la table 'my_table' utilise le CRS de SRID 3949


SELECT * FROM spatial_ref_sys srs WHERE srid = 3949
-- srid|auth_name|auth_srid|ref_sys_name|
-- ----+---------+---------+------------+
-- 3949|epsg     |     3949|RGF93 / CC49|

-- ^ le CRS de SRID 3949 est RFG93 = EPSG:3949


SELECT * FROM spatial_ref_sys_aux WHERE srid = 3949
-- srid|is_geographic|has_flipped_axes|spheroid|prime_meridian|datum                          |projection                 |unit |axis_1_name|axis_1_orientation|axis_2_name|axis_2_orientation|
-- ----+-------------+----------------+--------+--------------+-------------------------------+---------------------------+-----+-----------+------------------+-----------+------------------+
-- 3949|            0|               0|GRS 1980|Greenwich     |Reseau_Geodesique_Francais_1993|Lambert_Conformal_Conic_2SP|metre|Easting    |East              |Northing   |North             |

-- ^ d'autes infos complémentaires sur ce CRS
-- notamment, il utilise des coordonnées projetées (is_geographic=0)

```

Par simplicité, il y a une **view** `spatial_ref_sys_all` (qui n'est donc pas une table) qui agrège les données de `spatial_ref_sys` et `spatial_ref_sys_aux` :

```sql
SELECT * FROM spatial_ref_sys_all WHERE srid = 3949
```
