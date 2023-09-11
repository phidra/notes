**C'est quoi ?** Un client graphique pour travailler avec des databases. À la différence de pgadmin4, il sait gérer autre chose que postgresql ; notamment sqlite, et [plein plein d'autres](https://dbeaver.io/about/).

* [Installation](#installation)
* [Notes vrac](#notes-vrac)
* [Bosser avec des données géographiques](#bosser-avec-des-données-géographiques)
   * [Avec une base postGIS](#avec-une-base-postgis)
   * [Avec une base spatialite](#avec-une-base-spatialite)

# Installation

Un `.deb` est téléchargeable sur [la page officielle](https://dbeaver.io/download/), puis :

```
sudo dpkg -i dbeaver-ce_23.2.0_amd64.deb
```


# Notes vrac

Globalement, le tool est plus ergonomique que pgadmin4.

- c'est un client lourd
- au démarrage, il propose de se connecter à une database sample (sqlite) permettant de jouer avec les features
    - on peut la recréer à tout instant via un bouton dans le menu `Aide`
- lorsqu'on créée une connexion vers une nouvelle database, après confirmation, il se débrouille tout seul pour installer le pilote nécessaire
- dans le panel des database, on peut filtrer les tables avec une chaîne
- en double-cliquant sur une table, on accède à des outils utiles :
    - Propriétés = la liste de ses colonnes
    - une vue sur les données
    - un diagramme Entity Relashionship Diagram !
- c'est général : on peut consulter les Propriétés de chaque objet : database, schéma, table, colonne...
    - et il y a un fil d'Ariane pour naviguer facilement

# Vue des données

La vue des données a plein de features UX pratiques :

- on peut filtrer facilement en entrant le contenu d'une clause `WHERE`, ça évite d'avoir à taper toute la requête :
    ```
    my_field = 'this_value'
    ```
- par défaut, n'affiche que les 200 premières lignes, mais en cliquant droit sur les colonnes, on peut télécharger les pages suivantes voire toute la table (attention à la RAM dans ce cas)
- pour faciliter leur copier/coller, on peut afficher les résultats sous une forme textuelle (et dans ce cas, la sélection est par colonne = équivalent de Ctrl+V sous vim)
- on peut masquer des colonnes
- on peut afficher en grille ou en textuel
- il y a un viewer de géométrie basé sur leaflet
- il y a un fil d'Ariane sur ce qu'on visualise
- il y a de la pagination par défaut, et des tools pour facilement récupérer le reste / la page suivante
- on peut facilement exporter les données visualisées
- on peut passer en mode "record" pour visualiser une lignes en changeant de disposition (ses champs sur deux colonnes key + value)
- on peut détacher la fenêtre de visualisation des données, si ça aide

Bémol 1 = je m'y perds un peu sur tous mes onglets (mais on peut "fermer les autres onglets" facilement)

Bémol 2 = pour passer une requête sur la table dont on visualise actuellement les données, je ne trouve pas de shortcut rapide (même si l'autocomplétion fait le taf).


# Bosser avec des données géographiques

https://dbeaver.com/docs/dbeaver/Working-with-Spatial-GIS-data/

Lorsqu'un table contient des géométries, en plus des onglets verticaux `Grille` et `Texte` à gauche de la visualisation principale, on gagne un onglet `Spatial` qui est un viewer leaflet affichant les géométries sur un fond de carte OSM.

Les géométries s'affichent en bleu, on peut cliquer dessus pour voir les propriétés de la ligne correspondante.

L'affichage des tuiles de fond de plan semble parfois déconner.

## Avec une base postGIS

Tout semble fonctionner out-of-the-box.

## Avec une base spatialite

Déjà, il faut charger l'extension spatialite sur la connexion sqlite (à ne faire qu'une fois) :

```sql
SELECT load_extension('mod_spatialite')
```

Si on oublie de charger le module, on aura une erreur du genre :

> SQL Error [1]: [SQLITE_ERROR] SQL error or missing database (no such function: ST_Transform)

Derrière, le format des géométries spatialite n'a pas l'air d'être pris en compte, mais on peut contourner en générant une colonne [au format WKT](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (que dBeaver comprend nativement) avec `ST_AsText` :

- exécuter un script SQL pour construire les données dérivées :
   ```sql
   -- en conservant le SRID initial des données :
   SELECT ST_AsText(geom) AS the_geometry, * FROM my_table  -- conserver le SRID des données

   -- en convertissant pour travailler directement en 4326 = WGS84 = GPS :
   SELECT ST_AsText(ST_Transform(geom, 4326)) AS the_geometry, * FROM my_table
   ```
- à ce stade, la colonne n'est curieusement pas encore vue comme géométrique par dbeaver, il faut setter manuellement le type de colonne :
    - dans la vue des données, clic-droit sur la colonne `the_geometry`
    - **View/Format > Set "the_geometry" transform > Geometry**  (sans quoi la colonne est vue comme textuelle)
    - choisir le SRID des géométries
    - l'onglet vertical `Spatial` apparaît, sous `Grille` et `Texte`
    - si besoin, dans le viewer leaflet, choisir le SRID d'affichage des géométries (l'IHM lagge un peu)

C'est un peu moins pratique que postGIS, car il faut travailler sur une vue dérivée de la table (construite à partir de `ST_AsText`), mais ça fait le taf.
