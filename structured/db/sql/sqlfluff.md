**C'est quoi** = un linter/formatter SQL, qui sait parler plusieurs dialectes.

# Configuration

Fichier `.sqlfluff` accessible depuis le `pwd` ; exemple pour faire sauter la limite de taille :

```ini
[sqlfluff]
large_file_skip_byte_limit = 0
```

Fichier `.sqlfluffignore` pour ignorer des fichiers/répertoires.

# Utilisation

```sh
# lister les dialectes :
sqlfluff dialects

# juste vérifier que le fichier est parsable :
sqlfluff parse --dialect t-sql /tmp/code.sql

# formatter inline le fichier :
sqlfluff fix --dialect tsql /tmp/code.sql > /tmp/fixed.sql\n
```
