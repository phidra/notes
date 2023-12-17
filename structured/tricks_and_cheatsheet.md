* [imagemagick pour calculer la différence entre deux images](#imagemagick-pour-calculer-la-différence-entre-deux-images)
* [plotter avec sqliteviz des valeurs loggées avec timestamp](#plotter-avec-sqliteviz-des-valeurs-loggées-avec-timestamp)
* [grepper un process](#grepper-un-process)
* [retrouver un process à partir de son pid](#retrouver-un-process-à-partir-de-son-pid)
* [récupérer l'id de commit de master sur un repo distant](#récupérer-lid-de-commit-de-master-sur-un-repo-distant)
* [mesurer la RAM consommée par un process](#mesurer-la-ram-consommée-par-un-process)
* [envoyer une touche littérale dans le terminal](#envoyer-une-touche-littérale-dans-le-terminal)
* [mélanger les lignes d'un fichier](#mélanger-les-lignes-dun-fichier)
* [sauter à la ligne dans une commande zsh multiligne de l'historique](#sauter-à-la-ligne-dans-une-commande-zsh-multiligne-de-lhistorique)
* [créer ou extraire une archive .tar.7z de tout un répertoire](#créer-ou-extraire-une-archive-tar7z-de-tout-un-répertoire)
* [chiffrer un fichier](#chiffrer-un-fichier)
* [éditer un fichier dans un container docker qui n'a pas d'éditeur](#éditer-un-fichier-dans-un-container-docker-qui-na-pas-déditeur)
* [matcher toute l'arborescence ou juste les fichiers avec zsh](#matcher-toute-larborescence-ou-juste-les-fichiers-avec-zsh)
* [vider le cache bash](#vider-le-cache-bash)


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

# retrouver un process à partir de son pid

**tags** : sysadmin

```sh
# version longue :
ps -o command= -p PID
# /full/path/to/myprocess with args --and --options

# version courte :
ps -o comm
# myprocess
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

# mélanger les lignes d'un fichier

**tags** : terminal, unix-tool

`shuf` pour mélanger les lignes d'un fichier

Utile pour tirer au hasard des lignes de logs :

```sh
# 5 logs au hasard :
shuf logs.txt | head --lines=5
```

# sauter à la ligne dans une commande zsh multiligne de l'historique

**tags** : terminal, zsh

Quand on rappelle une commande de l'historique zsh et qu'on veut l'éditer, si cette commande est multiligne, `Entrée` l'exécute au lieu d'insérer un saut de ligne.

Solution = `Escape` puis `Entrée`.

# créer ou extraire une archive .tar.7z de tout un répertoire

**tags** : tarball, 7z, compression

Créer une archive de tout un répertoire :

```sh
tar -cvf - monrepertoire | 7z a -si /tmp/monarchive.tar.7z
```

Extraire l'archive :

```sh
7z x -so /tmp/monarchive.tar.7z | tar -xvf -
```

# chiffrer un fichier

**tags** : chiffrement, openssl

Chiffrer un fichier (l'exemple est donné avec une archive 7z, car ça fait un bon combo)

```sh
openssl enc -e -aes-256-cbc -salt -in /tmp/monarchive.tar.7z -out /tmp/monarchive.tar.7z.xxx
# le tool demande le mot de passe au prompt
```

Déchiffrer :

```sh
openssl enc -d -aes-256-cbc -salt -in /tmp/monarchive.tar.7z.xxx -out /tmp/monarchive.tar.7z
# le tool demande le mot de passe au prompt
```


# éditer un fichier dans un container docker qui n'a pas d'éditeur

**tags** : docker

Même si l'image docker ne dispose pas de container, on peut éditer un fichier de façon externe, et le copier dans le container via `docker cp` :

```sh
docker cp <container>:/path/to/myfile /tmp
vim /tmp/myfile
docker cp /tmp/myfile <container>:/path/to/myfile
```

# matcher toute l'arborescence ou juste les fichiers avec zsh

**tags** : zsh

```sh

# éditer toute l'arborescence actuelle, fichiers ET répertoires :
nvim **/*

# éditer tous les FICHIERS de l'arborescence actuelle :
nvim **/*(.)
```

# vider le cache bash

**tags** : bash, cache

Pour éviter d'avoir à parcourir tous les chemins du `PATH` à chaque commande, bash maintient un cache ; bash indique que le fichier est `hashed`.

Lorsqu'on désinstalle un programme, on peut vouloir faire oublier une entrée au cache :

```sh
# oublier l'entrée meson
hash -d meson

# tout rehasher :
hash -r

# si besoin :
man bash
```
