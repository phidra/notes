* [Vocabulaire](#vocabulaire)
   * [Feature](#feature)
   * [Shapefile](#shapefile)
   * [Géométrie](#géométrie)
   * [Attribut](#attribut)
* [Références](#références)
* [Extraits de la spec shapefile](#extraits-de-la-spec-shapefile)

# Vocabulaire

## Feature

C'est la représentation d'un objet du monde réel.

En pratique, c'est l'agrégation d'une géométrie et d'attributs.

## Shapefile

Au sens strict : fichier `.shp` = fichier stockant uniquement les géométries des features.

Au sens large  : jeux de fichiers stockant les features :

- `myfeatures.shp` = fichier binaire stockant les géométries des features, format décrit dans la spec shapefile
- `myfeatures.dbf` = fichier binaire stockant les attributs des features, au format dBase

Au sens large, il y a également plein d'autres fichiers secondaires, parmi lesquels :

- `features.shx` = "index des géométries" : fichier binaire permettant de retrouver facilement un record donnée dans le fichier .shp

## Géométrie

Un jeu de points, lignes, et aires représentant "quelque chose" (pour les sections : les géométries sont des polylines).

## Attribut

Des propriétés d'une feature, par exemple, l'id d'une section représentée par une polyline.

# Références

- https://gis.stackexchange.com/a/137949   (différence feature / géométrie)
- https://en.wikipedia.org/wiki/Shapefile
- http://desktop.arcgis.com/en/arcmap/10.3/manage-data/shapefiles/what-is-a-shapefile.htm
- https://support.esri.com/en/white-paper/279   (spécification des fichiers shapefiles)
- https://www.esri.com/library/whitepapers/pdfs/shapefile.pdf
- https://en.wikipedia.org/wiki/.dbf


# Extraits de la spec shapefile

```
An ESRI shapefile consists of a main file, an index file, and a dBASE table.
The mainfile is a direct access, variable-record-length file in which each record describes a shapewith a list of its vertices.
In the index file, each record contains the offset of thecorresponding main file record from the beginning of the main file.
The dBASE tablecontains feature attributes with one record per feature.
The one-to-one relationshipbetween geometry and attributes is based on record number.
Attribute records in thedBASE file must be in the same order as records in the main file.
Naming ConventionsAll file names adhere to the 8.3 naming convention.
	The main file, the index file, and the dBASE file have the same prefix.
	The suffix for the main file   is .shp
	The suffix for the index file  is .shx
	The suffix for the dBASE table is .dbf
Examples
	Main file   : counties.shp
	Index file  : counties.shx
	dBASE table : counties.dbf
```
