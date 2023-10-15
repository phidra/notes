# Parquet

**C'est quoi ?** Un format de fichier orienté colonnes ; le format est indépendant du sujet GIS, et sert de base au format GIS = geoparquet.

De ce que j'en comprends, ça permet des optimisations lorsqu'on bosse avec des énormes jeux de données, surtout si celles-ci ne sont pas aussi bien structurées qu'en SQL.

Par exemple, si on doit travailler sur un subset de colonnes, ça permet d'ignorer le reste des colonnes, et donc des portions entières du fichier ; alors que dans un format orienté ligne, même si une unique colonne nous intéresse, il faut "toucher" à chaque ligne.

# Geoparquet

https://geoparquet.org/

**C'est quoi ?** Un futur standard de l'[Open Geospatial Consortium (OGC)](https://ogc.org/) pour gérer des types géospatiaux (`Point`, `Line`, `Polygon`) à parquet.

De ce que j'en comprends, geoparquet est à parquet ce que postGIS est à postgres (ou spatialite à sqlite).



qgis semble capable de lire du geoparquet via gdal : [lien](https://gis.stackexchange.com/questions/430973/importing-geoparquet-file-in-qgis)

gdal est capable de lire geoparquet à partir de gdal 3.6.2 : [lien](https://gdal.org/drivers/vector/parquet.html), ce que j'ai pu vérifier avec une POC.

## Tools

Beaucoup de tools sont listés sur [la page d'accueil du projet](https://geoparquet.org/).


[gpq](https://github.com/planetlabs/gpq) semble être un utilitaire pour bosser avec geoparquet.

On dirait qu'on peut utiliser gdal via fiona : [lien](https://github.com/Toblerity/Fiona).

J'ai fait une POC de conversion de geoparquet en geojson avec ogr2ogr (mais il faut une version de gdal plus récente que celle de mon poste → j'ai utilisé une image docker d'un gdal récent).

Un petit sample de geoparquet est [ici](https://github.com/opengeospatial/geoparquet/tree/e52ace139a34778331b94251606070ed5259375f/examples), il contient un détourage de l'Amérique du Nord.

D'autres samples semblent être proposés sur [la page d'accueil du projet](https://geoparquet.org/).
