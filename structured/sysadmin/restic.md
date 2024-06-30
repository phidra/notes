(contexte juin 2024 = transfert à l'identique de mes vieilles notes sur restic, sans les mettre à jour)

* [Installation](#installation)
* [Notes vrac](#notes-vrac)
* [Commandes :](#commandes-)
   * [Backup](#backup)
   * [Lister les backups](#lister-les-backups)
   * [Lister les fichiers d'un backup](#lister-les-fichiers-dun-backup)
   * [Full restore](#full-restore)
   * [Partial restore](#partial-restore)
   * [Compare](#compare)
   * [Check pour vérifier l'intégrité](#check-pour-vérifier-lintégrité)
   * [Forget + prune](#forget--prune)


# Installation

Télécharger la dernière version sur https://github.com/restic/restic/releases

```sh
bunzip2 restic_0.9.6_linux_386.bz2
chmod +x restic_0.9.6_linux_386
./restic_0.9.6_linux_386 version
```

# Notes vrac

Chaque snapshot a un id, qui permet par exemple de les comparer avec diff.

Pour ne pas avoir à entrer le password à chaque fois, **NE PAS OUBLIER l'ESPACE INITIAL POUR NE PAS SAUVEGARDER LE PASSWORD DANS L'HISTORIQUE BASH/ZSH !** :

```sh
 export RESTIC_PASSWORD=MonPassword
```

# Commandes :

## Backup

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo init
./restic_0.9.6_linux_386 --repo /tmp/resticrepo backup /path/to/workflow
```

## Lister les backups

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo snapshots
```

## Lister les fichiers d'un backup

ATTENTION : sur un snapshot complet, j'ai ~ 500k lignes !

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo ls latest > /tmp/snapshot_content
```

## Full restore

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo restore 3290249 --target /tmp/restored
./restic_0.9.6_linux_386 --repo /tmp/resticrepo restore latest  --target /tmp/restored
```


## Partial restore

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo restore latest  --target /tmp/restored --include /path/to/myspecificfolder
# --> /tmp/restored/path/to/myspecificfolder
```

ATTENTION : l'option `--path` ne permet PAS de restaurer un fichier en particulier (mais plutôt de choisir le snapshot à utiliser)

## Compare

```sh
./restic_0.9.6_linux_386 --repo /tmp/resticrepo diff 8f4a925c c8f9ad12
```

## Check pour vérifier l'intégrité

		./restic_0.9.6_linux_386 --repo /tmp/resticrepo check
		./restic_0.9.6_linux_386 --repo /tmp/resticrepo check --read-data   # plus complet, mais plus long

## Forget + prune

forget + prune = le combo pour faire de la place sur le repo ([doc](https://restic.readthedocs.io/en/v0.9.6/060_forget.html)) :

```sh
sudo /path/to/restic_0.9.6_linux_amd64 --repo /my/restic/repo snapshots
# ID        Time                 Host        Tags        Paths
# ------------------------------------------------------------
# 6b6992a7  2020-04-20 18:14:43  myhost                  /
# [...]
# ------------------------------------------------------------
# 6 snapshots
sudo /path/to/restic_0.9.6_linux_amd64 --repo /my/restic/repo forget 6b6992a7
sudo /path/to/restic_0.9.6_linux_amd64 --repo /my/restic/repo prune
```

ATTENTION : le `prune` est très long.

