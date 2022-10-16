* [imagemagick pour calculer la différence entre deux images](#imagemagick-pour-calculer-la-différence-entre-deux-images)
* [plotter avec sqliteviz des valeurs loggées avec timestamp](#plotter-avec-sqliteviz-des-valeurs-loggées-avec-timestamp)
* [grepper un process](#grepper-un-process)
* [récupérer l'id de commit de master sur un repo distant](#récupérer-lid-de-commit-de-master-sur-un-repo-distant)
* [mesurer la RAM consommée par un process](#mesurer-la-ram-consommée-par-un-process)
* [envoyer une touche littérale dans le terminal](#envoyer-une-touche-littérale-dans-le-terminal)

# imagemagick pour calculer la différence entre deux images

**tags** : image-processing

```sh
compare modified.png reference.png -compose src /tmp/diff.png
```

# plotter avec sqliteviz des valeurs loggées avec timestamp

**tags** : dataviz + sqlite

Par exemple, si je greppe des logs dans ce genre :

```
2022-06-13 00:00:31,870	INFO	[pouet] 12345 this is my log, that took 138 ms
```

Je peux transformer une liste de tels logs en CSV (à deux colonnes : datetime + mesure) avec un coup de vim :

```
2022-06-13 00:00:31,138
```

Avec [sqliteviz](https://github.com/lana-k/sqliteviz), je peux importer mon CSV, stocker le datetime dans une colonne, et la valeur dans l'autre :

```sql
SELECT datetime(col1) AS dt, col2 FROM "times"
```

Derrière, il faut cliquer sur "play" pour exécuter la requête et charger les données, et je peux alors tracer des histogrammes ou des courbes.

Bonus = si je veux plotter des _différences de timestamp_ entre deux lignes de log :
- je fais un coup de vim pour avoir un CSV où chaque ligne à 3 colonnes :
    - datetime avant (sans les millisecondes, inutilisable par la fonction `datetime` de sqlite)
    - datetime après
    - (la mesure)
- exemple de ligne :
    ```
    2022-06-13 00:43:18,2022-06-13 00:44:07,2945
    ```
- derrière, pour plotter sous sqlite le diff de deux colonnes datetime :
    ```sql
    SELECT datetime(col1) AS premier, datetime(col2) AS deuxieme, ROUND((JULIANDAY(col2) - JULIANDAY(col1)) * 86400) AS diff FROM "times"
    ```

# grepper un process

**tags** : sysadmin

```sh
pgrep -f myprocessname -a
```

# récupérer l'id de commit de master sur un repo distant

**tags** : sysadmin + git

```sh
git ls-remote https://github.com/lviennot/hub-labeling/ | grep refs/heads/master | head --lines=1 | cut -f 1
# a3fe302014273377ad30779fd13f7bffb8d01afd
```

# mesurer la RAM consommée par un process

**tags** : sysadmin

```sh
# nécessite d'être ROOT
pmap -x <pid>
```

La deuxième colonne indique en kio le _Resident Set Size_ = RSS (= l'ensemble de la RAM physique actuellement utilisée par le process, including la RAM qui est partagée avec d'autres process comme les libs partagées).

En alternative, `htop` indique aussi tout ça (mais avec une précision un peu moindre).

# envoyer une touche littérale dans le terminal

**tags** : terminal

`Ctrl+V` suivi de la touche à envoyer.

E.g. sous vim, je veux une tabulation, mais l'éditeur est configuré pour les convertir en espaces.
