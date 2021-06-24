# Les bases de la data science avec Python

- **url** = https://www.youtube.com/watch?v=4bccU9eMrkw
- **type** = vidéo
- **auteur** = [Docstring](https://www.docstring.fr/), site de formations python.
- **date de publication** = 2021-06-21
- **source** = [la chaîne youtube de Docstring](https://www.youtube.com/channel/UCo2zkK2d_frGSXctIkftALw)
- **tags** = language>python ; topic>data-science ; tool ; level>beginner

**TL;DR** : vidéo présentant les bases (vraiment très basiques) de l'analyse de données avec pandas. Rien d'introuvable avec un grepping de quelques minutes dans la doc, mais montre en live l'utilisation de pandas (dans un jupyter notebook). 

# Notes vrac

- Commande prefixee de ! sous jupyterlab pour lancer une commande shell ? (Ex: `!{sys.executable} -m pip install pandas`)
- [Mockaroo](https://www.mockaroo.com/) = site pour générer des données random
- [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/) = IHM web pour Jupyter notebook : `pip install jupyterlab`
    * `Shift+entrée` pour exécuter la cellule (`[*]`=l'exécution est en cours / `[7]`=l'exécution terminée, c'est la 7ième instruction exécutée par le kernel)
    * Cellule markdown vs cellule avec du code
    * Relancer cellule vs relancer kernel
- Index d'un dataframe pandas
- Dataframe (tableau complet) vs series (colonne) dans pandas
- Les filtres pandas ont l'air chouette
- Paramètre `inplace` pour muter la dataframe, plutôt que renvoyer une copie.
- Commencer par supprimer les colonnes inutiles au test est souvent une bonne idée
- Détecter et gérer les valeurs manquantes (enlever, ou setter une valeur, fixe ou calculée à partir des voisines)
- Ajouter des colonnes (éventuellement, dont la valeur est le résultat d'un calcul utilisant d'autres colonnes)
- quelques fonctions chouettes : `describe` / `value_counts` / `groupby`
- Pandas est packagé de base avec matplotlib
