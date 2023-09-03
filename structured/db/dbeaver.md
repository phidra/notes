**C'est quoi ?** Un client graphique pour travailler avec des databases. À la différence de pgadmin4, il sait gérer autre chose que postgresql ; notamment sqlite, et [plein plein d'autres](https://dbeaver.io/about/).

# Installation

Un `.deb` est téléchargeable sur [la page officielle](https://dbeaver.io/download/), puis :

```
sudo dpkg -i dbeaver-ce_23.2.0_amd64.deb
```


# Notes vrac

- c'est un client lourd
- au démarrage, il propose de se connecter à une database sample (sqlite) permettant de jouer avec les features
- lorsqu'on créée une connexion vers une nouvelle database, après confirmation, il se débrouille tout seul pour installer le pilote nécessaire
- dans le panel des database, on peut filtrer les tables avec une chaîne
- en double-cliquant sur une table, on accède à des outils utiles :
    - Propriétés = la liste de ses colonnes
    - une vue sur les données
    - un diagramme Entity Relashionship Diagram !
- c'est général : on peut consulter les Propriétés de chaque objet : database, schéma, table, colonne...
    - et il y a un fil d'Ariane pour naviguer facilement
- la vue des données permet de filtrer facilement en entrant le contenu d'une clause `WHERE`, ça évite d'avoir à taper toute la requête :
    ```
    my_field = 'this_value'
    ```
- globalement, le tool a l'air plus ergonomique que pgadmin4
