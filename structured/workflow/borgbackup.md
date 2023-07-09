
* [C'est quoi ça sert à quoi](#cest-quoi-ça-sert-à-quoi)
* [Concepts clé](#concepts-clé)
   * [repo vs. archive](#repo-vs-archive)
   * [chiffrement et compression](#chiffrement-et-compression)
   * [remote repository](#remote-repository)
   * [interrompre et reprendre un backup](#interrompre-et-reprendre-un-backup)
* [Commandes utiles](#commandes-utiles)
   * [Passphrase](#passphrase)
   * [Créer le repo](#créer-le-repo)
   * [Backuper des données](#backuper-des-données)
   * [Accéder aux données backupées](#accéder-aux-données-backupées)
      * [Lister le contenu](#lister-le-contenu)
      * [Extraire le contenu dans un répertoire](#extraire-le-contenu-dans-un-répertoire)
      * [Monter une archive](#monter-une-archive)
      * [Downloader une tarball tgz de l'archive](#downloader-une-tarball-tgz-de-larchive)
   * [Faire du ménage](#faire-du-ménage)
   * [Changer la passphrase](#changer-la-passphrase)

# C'est quoi ça sert à quoi

https://www.borgbackup.org/

Un outil pour backuper ses données sur un repo local ou remote, avec plein de features utiles :

- **déduplication** = si j'ai des photos en plusieurs exemplaires, elles ne sont backupées qu'une fois
- **mutualisation du stockage entre plusieurs archives** = en quelque sorte, la déduplication est effective aussi entre plusieurs archives
- **chiffrement** = les archives peuvent être chiffrées côté client pour que l'hébergeur du repo n'ait pas connaissance du contenu
- **compression** = les archives peuvent être compressées
- **pruning** = très configurable, on peut choisir p.ex. de ne conserver qu'une archive par mois

NOTE : cf. `K2_JOURNAL_ADMINISTRATIF.otl` pour les détails sur comment j'utilise borg pour backuper mes journaux.

# Concepts clé

## repo vs. archive

Selon la terminologie borg, à chaque fois que je veux créer une copie de backup de mes données, je crée une nouvelle **archive**. Le **repository** est le conteneur des différentes archives.

Par exemple, on crée un repository une seule et unique fois avec `borg init`, mais on y place des archives autant de fois qu'on veut avec `borg create`.

Certaines commandes borg utilisent le repo uniquement, d'autre le repo + une archive en particulier, et les deux sont agrégés sous la forme `REPO::ARCHIVE` :

```
borg init REPO
borg list REPO
borg create REPO::ARCHIVE
borg mount REPO::ARCHIVE
```

## chiffrement et compression

C'est le repo complet qui est chiffré → le choix de chiffrer les données (et la définition de la clé/passphrase) se fait à l'appel de `borg init` :

```sh
borg init -e repokey-blake2 REPO
# il faut entrer la passphrase du repo nouvellement créé
```

À l'inverse, la compression est choisie au moment de la création d'une archive à l'appel de `borg create` :

```sh
borg create --compression lz4 REPO::ARCHIVE folder_to_backup
```

La déduplication a lieu AVANT la compression → même si deux archives utilisent deux algos de compression différents, la déduplication fonctionnera quand même (on peut donc mixer plusieurs compressions dans le même repo si besoin).

La compression par défaut est la plus rapide/moins efficace = `lz4` ; pour connaître les compressions disponibles :

```sh
borg help compression
```

## remote repository

Le repo borg peut-être un répertoire local ou un remote repository accessible par SSH :

```sh
borg init /media/DATA/myborgrepo
borg init ssh://user@host:port/path/to/repo
borg init ssh://user@host:port/./repo
```

Dans les présentes notes, `REPO` désigne un repository, peu importe qu'il soit local ou remote via ssh.

## interrompre et reprendre un backup

Si un backup met longtemps à se faire, une sauvegarde intermédiaire "checkpoint" a lieu régulièrement (par défaut, toutes les 30 minutes).

Ce checkpoint est une archive intègre (sauf qu'elle ne contient pas toutes les données), elle peut donc servir à la déduplication.

Du coup, après un backup interrompu (e.g. avec Ctrl+C), je n'ai qu'à lancer une nouvelle sauvegarde différente, le checkpoint sera utilisé pour ne pas avoir à ré-uploader toutes les données depuis zéro.

Cf. [la FAQ](https://borgbackup.readthedocs.io/en/stable/faq.html#if-a-backup-stops-mid-way-does-the-already-backed-up-data-stay-there)

# Commandes utiles

## Passphrase

Toutes les commandes borg nécessitent d'entrer la passphrase pour accéder au repo.

On peut utiliser l'envvar `BORG_PASSPHRASE` pour ne pas le faire à chaque fois :

```sh
# soit en exportant :
export BORG_PASSPHRASE=DoNotRepeat
borg list REPO::ARCHIVE

# soit en préfixant :
BORG_PASSPHRASE=DoNotRepeat borg list REPO::ARCHIVE
```

Il y a une autre envvar pour ne pas préciser le repo, et même beaucoup d'autres pour plein de trucs : https://borgbackup.readthedocs.io/en/stable/usage/general.html#environment-variables

## Créer le repo

```sh
borg init -e repokey-blake2 REPO
# c'est à la création du repo qu'un prompt nous demande de choisir une passphrase
```


## Backuper des données

Pour créer une archive :

```sh
# commande de base :
borg create REPO::ARCHIVE  FOLDER_TO_BACKUP1  FOLDER_TO_BACKUP2

# commande complète :
borg create -v --stats --progress --compression lz4 REPO::ARCHIVE  FOLDER_TO_BACKUP

# on peut utiliser la date dans le nom de l'archive :
borg create -v --stats --progress --compression lz4 REPO::ARCHIVE-{now:%Y-%m-%dT%H:%M:%S}  FOLDER_TO_BACKUP
```

Les archives étant dédupliquées, elles seront très légères si les données backupées ont peu évolué → ne pas hésiter à en créer plein.


## Accéder aux données backupées

### Lister le contenu

```sh
# lister le contenu du repo = voir la liste des archives :
borg list REPO

# lister le contenu d'une archive = voir les fichiers qu'elle contient :
borg list REPO::ARCHIVE
```

NOTE : lister les archives du repo semble un préalable indispensable, afin d'avoir l'intitulé exact de l'archive à extraire.

### Extraire le contenu dans un répertoire


```sh
borg extract REPO::ARCHIVE

borg help extract
```

**Attention** :

- le backup est TOUJOURS extrait DANS LE REPERTOIRE COURANT !
- le chemin COMPLET est extrait (si le `FOLDER_TO_BACKUP` passé à `borg create` valait `/path/to/this/folder`, et que je suis actuellement dans le répertoire `/tmp/EXTRACT`, alors j'obtiens l'arborescence `/tmp/EXTRACT/path/to/this/folder`)

### Monter une archive

```sh
mkdir /tmp/my-mounted-archive
borg mount REPO::ARCHIVE /tmp/my-mounted-archive

# later...
borg umount /tmp/my-mounted-archive
```

À vérifier, mais je suppose que seuls les données consultées sont réellement downloadées depuis le repo.

### Downloader une tarball tgz de l'archive

```sh
borg export-tar REPO::ARCHIVE /tmp/my-backuped-data.tar.gz
```

## Faire du ménage

À creuser ; la commande de base est `borg prune`, elle a plein de règles possibles pour avoir du pruning intelligent, du genre "ne garder qu'une sauvegarde par mois", cf. [la doc](https://borgbackup.readthedocs.io/en/stable/usage/prune.html)

## Changer la passphrase

```sh
BORG_NEW_PASSPHRASE="ThisIsANewPassphrase" borg key change-passphrase REPO
```



