# rsync

**C'est quoi ?** Tool pour transférer des données de façon efficace = en ne transférant que le diff nécessaire (du coup, on peut interrompre+reprendre des transferts sans tout refaire de zéro).

* [rsync](#rsync)
   * [Commandes utiles](#commandes-utiles)
   * [Slash final dans le répertoire source](#slash-final-dans-le-répertoire-source)
   * [Pré-existence du répertoire de destination](#pré-existence-du-répertoire-de-destination)
   * [Options qui peuvent être utiles](#options-qui-peuvent-être-utiles)

Références :

- man rsync
- http://doc.ubuntu-fr.org/rsync

Précisions :

- les fichiers cachés sont copiés aussi
- rsync ne copie que dans un sens (i.e. ça ne met à jour que la destination, la source est considérée comme en lecture seule)
- en cas de copie vers une machine différente, rsync doit obligatoirement être installé sur la machine de destination.
- ATTENTION : l'option `-a` préserve les groupes et les permissions, mais ce sont les _numéros_ de groupes qui sont préservés : si un groupe n'existe pas sur la destination, c'est un *autre* groupe qui aura les mêmes droits !
- sauf à utiliser l'option `-t` (éventuellement via l'option `-a`), les 3 time-fields (ATIME/MTIME/CTIME) sont mis à jour à la date de la copie
- l'option `--delete` supprime dans la destination les fichiers ayant disparu de la source
- si un fichier de la source va écraser un fichier de la destination (car il y existait déjà), l'options `--backup` permet de backuper ce dernier en le suffixant

Plus généralement, il y a beaucoup beaucoup d'options et de comportements à tuner, ne pas hésiter à digger la doc pour des besoins évolués.

## Commandes utiles

```sh
# Faire un miroir EXACT d'un répertoire local sur une machine distante :
rsync   -avz --delete  --progress  path/to/my/local/directory/   USER@IP:path/to/destination
# ATTENTION : ça supprimera sur la destination les fichiers qui ont été supprimés de la source !

# Idem, mais en backupant les fichiers existants dans le répertoire "mybackup" :
rsync   -avz --delete --progress  -b --backup-dir="mybackup"  path/to/my/local/directory/   USER@IP:path/to/destination

# Idem, mais en excluant le répertoire caché ".git", les fichiers ".pyc", et les fichiers ".swp" :
rsync   -avz --progress  --exclude=".git" --exclude="*.pyc" --exclude="*.swp" path/to/my/local/directory/   USER@IP:path/to/destination

# Lister un répertoire distant (il suffit de ne pas préciser la destination) :
rsync   USER@IP:directory/to/list

# Voir le statut du démon rsync :
service rsync status
```

## Slash final dans le répertoire source

**ATTENTION** : le fait que le répertoire source soit passé avec ou sans `/` final a un impact très important :

```sh
rsync -r source/ dest   # AVEC le slash, on copie plusieurs objets : tous ceux contenus par le répertoire source
rsync -r source  dest   # SANS le slash, on ne copie qu'un seul objet : le répertoire source
```

En revanche, ça ne joue pas pour la destination. Ainsi, les deux commandes suivantes sont équivalentes :

```sh
rsync -r source dest/
rsync -r source dest
```

Exemple :

```
arborescence de src :
	src
	├── file1
	├── file2
	└── file3

arborescence de dst avec la commande   "rsync -avz src/ dst"   :
	dst
	├── file1
	├── file2
	└── file3

arborescence de dst avec la commande   "rsync -avz src dst"    :
	dst
	└── src
	    ├── file1
	    ├── file2
	    └── file3
```

## Pré-existence du répertoire de destination

Pour rsync, le fait que le répertoire de destination existe déjà ou non ne change strictement rien au résultat.

En fait, on peut considèrer que le répertoire de destination existe déjà (il sera créé au tout début de synchro si ça n'est pas le cas)


## Options qui peuvent être utiles

(Notes vrac directement issues de mes notes OTL)

```
-v  = augmente la verbosité du transfert
-z  = compression des fichiers pendant le transfert
-a  = mode "archive" (équivalent de -rlptgoD, sans -H, -A, -X), correspond plus ou moins à la reproduction à l'identique de la source.
----------------------------------------
--progress = montrer une barre de progression pendant le transfert.
----------------------------------------
-r  = recursive   → traite un répertoire et tous ses sous-répertoires/sous-fichiers
-l  = symlinks    → copie les liens symboliques comme des liens symboliques
-p  = permissions → les permissions de la destination prendront comme valeurs celles de la source
-t  = modif-time  → la date de modification de la destination prendra comme valeur celle de la source
-g  = groups      → les groupes de la destination prendront comme valeurs celles de la source
-o  = owner       → préserve le propriétaire du fichier (attention à avoir les droits si le propriétaire n'est pas le login de connexion)
-D  = devices     → présever les fichiers spéciaux (genre ceux représentant un périphérique)
----------------------------------------
-A  = ACLs        → préserve les ACLs (Access Control List, permettent le contrôle fin des permissions)
-H  = hard-links  → préserve les liens physiques
-X  = xattrs      → préserve les attributs étendus
----------------------------------------
--delete          → supprime dans l'arborescence de destination les fichiers qui n'existent pas dans la source.
--delete-after    → idem, mais l'opération de suppression a lieu en dernier (alors que par défaut, rsync commence par la suppression)
----------------------------------------
-b  = --backup    → si un fichier source va écraser un fichier de la destination, on backupe ce dernier en le suffixant (par défaut par "~")
--suffix=ARG      → ARG est utilisé comme suffixe de backup à la place de "~"
--backup-dir=ARG  → tous les fichiers de backup sont placés dans un sous-répertoire ARG du répertoire de destination
----------------------------------------
--exclude=PATTERN → ne transfère pas les fichiers et répertoires dont le nom matche le pattern (on répéte l'option autant de fois que nécessaire)
--exclude-from=FILE → chaque ligne du fichier représente un pattern, dont on ne transfèrera pas les fichiers/répertoires le matchant.
```
