CONTEXTE = je cherche à utiliser `ST_DumpPoints`, ce qui a nécessité de comprendre les [types composites](https://www.postgresql.org/docs/current/rowtypes.html) et les [SRF](https://www.postgresql.org/docs/current/functions-srf.html).

Références :
- https://postgis.net/docs/ST_DumpPoints.html
- https://postgis.net/docs/geometry_dump.html

Notes sur ST_DumpPoints :

Pourquoi le field "path" du geometry_dump est-il un tableau d'entiers à un seul éléments ?
- parce que j'ai appliqué la fonction sur une linestring (pour lequel chaque tableau n'aura qu'un seul élément)
- si je l'applique sur un polygone, chaque tableau aura deux éléments

Au final, `ST_DumpPoints` est à la fois :
- une SRF = à partir d'UNE SEULE way osm, elle renverra PLUSIEURS nodes
- une SRF renvoyant un type composite = ce qu'elle retourne est un tuple à deux éléments : la géométrie d'un point + son index

Et avec ma compréhension toute fraîche des SRF et des composite types, je suis en mesure de comprendre ce type de requêtes :

```sql
WITH subquery AS (SELECT (ST_DumpPoints(ST_Transform(geom, 4326))).* FROM import.osm_ways WHERE osm_id=4224972)
SELECT path[1], ST_X(geom), ST_Y(geom) FROM subquery;

-- path(integer) ; st_x(double precision) ; st_x(double precision)
-- 1             ; 7.41749233325629       ; 43.7296302584965
-- 2             ; 7.41746375096648       ; 43.7296726709265
-- 3             ; 7.41741907542254       ; 43.7297137422521
-- 4             ; 7.41732855086829       ; 43.7297724993933
-- 5             ; 7.41720433106326       ; 43.7298527980257
```
