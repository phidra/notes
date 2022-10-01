* [Installation](#installation)
* [Configuration](#configuration)
* [Cluster](#cluster)
   * [Retour d'expérience de migration de postgresql 12 à 14](#retour-dexpérience-de-migration-de-postgresql-12-à-14)
* [Administration vrac](#administration-vrac)
   * [version](#version)
   * [port d'écoute](#port-découte)
   * [base de maintenance](#base-de-maintenance)
   * [client psql](#client-psql)
   * [client pgadmin4](#client-pgadmin4)
   * [(re)démarrer postgresql + savoir si postgresql est actif](#redémarrer-postgresql--savoir-si-postgresql-est-actif)
   * [consulter les logs](#consulter-les-logs)
   * [où sont stockées les données ?](#où-sont-stockées-les-données-)
   * [taille sur disque d'une database :](#taille-sur-disque-dune-database-)
   * [lister les sessions](#lister-les-sessions)
   * [extensions](#extensions)
   * [stats et sessions en cours](#stats-et-sessions-en-cours)
   * [durcissement de conf pour usage local](#durcissement-de-conf-pour-usage-local)

D'une façon générale, la doc est bien faite : https://docs.postgresql.fr/14/

# Installation

```sh
apt install postgresql postgresql-client postgresql-doc postgresql-doc-9.4
```

Pour utilisation sous python via psycopg2 :

```sh
apt install libpq-dev
pip install psycopg2
```

# Configuration

Fichiers de configuration :

- `/etc/postgresql/12/main/postgresql.conf` = fichier de configuration principal
- `/etc/postgresql/12/main/pg_hba.conf`     = fichier de configuration de l'HBA (Host Based Authentication)
- `/etc/postgresql/12/main/pg_ident.conf`   = fichier de configuration pour la correspondance d'utilisateurs
- `/etc/postgresql/postgresql-common`       = fichier de configuration commun à tous les clusters

Commandes SQL de configuration :
- `[PSQL]> ALTER SYSTEM`   = permet de configurer le système comme le fait postgresql.conf
- `[PSQL]> ALTER DATABASE` = permet de surcharger le paramétrage global pour une base donnée
- `[PSQL]> ALTER ROLE`     = permet de surcharger le paramétrage global pour une base donnée et un utilisateur donné

Dans les deux derniers cas, la surcharge a lieu au démarrage d'une session (et certains paramètres ne peuvent pas être surchargés)

Consulter / modifier un paramètre de configuration :

```
Consulter :
[PSQL]> SHOW max_locks_per_transaction;
[PSQL]> SHOW ALL;

Modifier :
[PSQL]> SET param TO value;
[PSQL]> SET param TO DEFAULT;
```

# Cluster

**TL;DR** :

- un cluster est _a collection of databases served by a postgres instance_ (un cluster écoute sur un port donné)
- on les administre avec divers binaires comme `pg_createcluster`, fournis par le paquet `postgresql-common`
- un cluster est identifié par le couple {version + nom), e.g. {`14` + `main`}
- à l'installation de postgresql sur un poste, un cluster "main" est automatiquement créé pour la version de postgresql installée :
    - p.ex. sur Ubuntu 22.04, le cluster installé par le paquet postgresql vit donc sous `/etc/postgresql/14/main/`
- pour lister les clusters, et savoir s'ils écoutent (et si oui, sur quel port) :
    ```sh
    pg_lsclusters
    # Ver Cluster Port Status Owner    Data directory              Log file
    # 14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

    ```


**man pg_createcluster**

> pg_createcluster creates a new PostgreSQL server cluster (i. e. a collection of databases served by a postgres(1) instance) and integrates it into the multi-version/multi-cluster architecture of the postgresql-common package.
>
> [...] The default cluster that is created on installation of a server package is main.
>
> [...] Given a major PostgreSQL version (like "8.2" or "8.3") and a cluster name, it creates the necessary configuration files in /etc/postgresql/version/name/;
>
> [...] postgresql.conf is automatically adapted to use the next available port, i.  e. the first port (starting from 5432) which is not yet used by an already existing cluster.


Le paquet `postgresql-common` fournit des tools pour gérer ses clusters postgresql :

- https://packages.debian.org/fr/sid/postgresql-common :
    > Le paquet postgresql-common fournit une structure qui permet d'installer simultanément des versions différentes de PostgreSQL et de maintenir plusieurs grappes PostgreSQL.
    ```sh
    dpkg-query -L postgresql-common | grep '^/usr/bin/pg_'
    # /usr/bin/pg_backupcluster
    # /usr/bin/pg_buildext
    # /usr/bin/pg_config
    # /usr/bin/pg_conftool
    # /usr/bin/pg_createcluster
    # /usr/bin/pg_ctlcluster
    # /usr/bin/pg_dropcluster
    # /usr/bin/pg_lsclusters
    # /usr/bin/pg_renamecluster
    # /usr/bin/pg_restorecluster
    # /usr/bin/pg_upgradecluster
    # /usr/bin/pg_virtualenv
    # /usr/bin/pg_archivecleanup
    ```

## Retour d'expérience de migration de postgresql 12 à 14

- quand j'ai migré de Ubuntu 20.04 à 22.04 (donc de la version 12 à 14 de postgresql), je me suis retrouvé avec deux clusters, l'un en 12 (mon ancien cluster) et l'autre en 14 (le cluster par défaut créé par postgresql14)
- le cluster actif (`14/main`) répondait sur le port `5433`, comme l'indique `man pg_lsclusters` :
    > postgresql.conf is automatically adapted to use the next available port, i.  e. the first port (starting from 5432) which is not yet used by an already existing cluster.
- mon ancien cluster (`12/main`) qui utilisait le port `5432` ne répond plus, il est arrêté. Je ne peux pas le redémarrer (et pour cause : je n'ai plus postgresql12 d'installé !) :
    ```sh
    pg_ctlcluster 12 main start
    # Warning: the cluster will not be running as a systemd service. Consider using systemctl:
    #   sudo systemctl start postgresql@12-main
    # Error: Could not create /var/run/postgresql/12-main.pg_stat_tmp: Permission denied

    sudo systemctl start postgresql@12-main
    # [sudo] Mot de passe de myself :
    # Job for postgresql@12-main.service failed because the service did not take the steps required by its unit configuration.
    # See "systemctl status postgresql@12-main.service" and "journalctl -xeu postgresql@12-main.service" for details.

    sudo systemctl status postgresql@12-main.service
    # [sudo] Mot de passe de myself :
    # × postgresql@12-main.service - PostgreSQL Cluster 12-main
    # 	 Loaded: loaded (/lib/systemd/system/postgresql@.service; disabled; vendor preset: enabled)
    # 	 Active: failed (Result: protocol) since Fri 2022-09-30 09:40:53 CEST; 38min ago
    # 	Process: 14708 ExecStart=/usr/bin/pg_ctlcluster --skip-systemctl-redirect 12-main start (code=exited, status=1/FAILURE)
    # 		CPU: 33ms
    #
    # sept. 30 09:40:53 pcfixe systemd[1]: Starting PostgreSQL Cluster 12-main...
    # sept. 30 09:40:53 pcfixe postgresql@12-main[14708]: Error: Could not find pg_ctl executable for version 12
    # sept. 30 09:40:53 pcfixe systemd[1]: postgresql@12-main.service: Can't open PID file /run/postgresql/12-main.pid (yet?) after start: Operation not permitted
    # sept. 30 09:40:53 pcfixe systemd[1]: postgresql@12-main.service: Failed with result 'protocol'.
    # sept. 30 09:40:53 pcfixe systemd[1]: Failed to start PostgreSQL Cluster 12-main.
    ```
- en théorie, il est possible de migrer un cluster :
    ```sh
    sudo pg_upgradecluster -v 14 12 main
    ```
- mais en pratique, il faut les deux versions de postgresql, et comme je n'ai pas de données pertinentes sur mon ancien cluster, et que je n'ai pas envie de voir coexister deux versions concurrentes, je finis par tout dropper et recréer un seul et unique cluster 14 :
    ```sh
    sudo pg_dropcluster --stop 14 main
    sudo pg_dropcluster 12 main
    sudo pg_createcluster 14 main
    sudo systemctl start postgresql
    pg_lsclusters
    # Ver Cluster Port Status Owner    Data directory              Log file
    # 14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
    ```
- notamment, je vérifie bien que le nouveau cluster 14 a été créé sur le port `5432`

# Administration vrac

## version

```sql
SELECT version();
```

## port d'écoute

Savoir sur quel port écoute postgres :

```sh
grep port /etc/postgresql/9.5/main/postgresql.conf
# port = 5432

# EDIT : encore mieux, car liste plusieurs ports d'écoute :
pg_lsclusters

# port d'écoute + fichier de config utilisé :
SELECT name, setting, sourcefile FROM pg_settings WHERE name='port';
```

## base de maintenance

Notion de **base de maintenance** = la base à la quelle se connecte psql si on ne lui a pas précisé de base ([doc](https://www.postgresql.org/docs/current/manage-ag-templatedbs.html)) :

> The postgres database is also created when a database cluster is initialized. \
> This database is meant as a default database for users and applications to connect to. \
> It is simply a copy of template1 and can be dropped and recreated if necessary.

## client psql

Utilisation du client **psql** et des utilitaires qui vont avec (il y a beaucoup de commandes et de détails dans [ma cheatsheet dédiée](./ADMINISTRATION_psql_clients.md)) :

```sh
sudo -u postgres psql
```

## client pgadmin4

pgadmin4 = tool graphique, très pratique, pour visualiser les données (notamment, on peut facilement peek une DB, en regardant les 100 premières lignes)

(pgadmin3 est trop vieux pour fonctionner avec postgresql12)

```sh
# install :
pipx install pgadmin4
pipx install pgadmin4-desktop-mode

# lancer un serveur pgadmin4 :
pgadmin4_desktop_mode

# utiliser le lien indiqué pour s'y connecter
```

master password : à la première utilisation, je créée un master password (par contre, on dirait qu'il me le demande à chaque lancement). Il est possible de le désactiver (ce qui revient à stocker en clair les mots de passe, I guess) : [lien](https://www.pgadmin.org/faq/#10)

NOTE : pour rafraîchir une DB, il ne suffit pas de faire clic-droit > rafraîchir, le mieux est de double-cliquer dessus.

## (re)démarrer postgresql + savoir si postgresql est actif

Démarrer / redémarrer :

```sh
sudo systemctl restart postgresql
sudo systemctl start postgresql
```

Savoir si postgresql est actif :

```sh
service postgresql status
systemctl status postgresql
pg_lsclusters
```

## consulter les logs

```sh
cat /var/log/postgresql/postgresql-9.4-main.log
```

## où sont stockées les données ?

Par défaut, où est le répertoire des données à l'installation :

```
/var/lib/postgresql/9.4/main
```

## taille sur disque d'une database :

```sql
-- connaître la taille sur disque de la base "mysuperdb"
SELECT pg_size_pretty(pg_database_size('mysuperdb'));
```

## lister les sessions

```sh
SELECT pid, usename, datname, application_name, TO_CHAR(backend_start, 'YYYY-MM-DD HH24:MI') FROM pg_stat_activity;

#   pid  | usename  | datname  | application_name |     to_char
# -------+----------+----------+------------------+------------------
#  10680 | postgres | postgres | psql             | 2020-08-08 11:19
#  10953 | postgres | postgres | psql             | 2020-08-08 11:22
# (2 rows)
```

## extensions


```sql
-- lister les extensions
SELECT * FROM pg_extension;

-- installer l'extension postgis pour la base courante :
CREATE EXTENSION postgis;
```

## stats et sessions en cours

```sql
-- tuer toutes les connexions à une DB donnée :
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid <> pg_backend_pid() AND datname = 'dbname';

-- accéder aux stats / administration de postgres
-- cf. https://www.postgresql.org/docs/9.5/monitoring-stats.html
SELECT * FROM pg_stat_activity;
```


## durcissement de conf pour usage local

HOWTO — interdire les accès à PostgreSQL autres que locaux :

- (note : j'ai pu vérifier que c'était le cas avec la configuration par défaut à l'installation)
- Déjà, m'assurer qu'iptables bloque bien le port de PostgreSQL (5432 par défaut)
- Puis, vérifier que `listen_addresses` est vide (pour n'accepter que les unix domain sockets) ou ne contient que `localhost` (ce qui semble le cas par défaut) :
    ```
    postgres=# SHOW listen_addresses;

     listen_addresses
    ------------------
     localhost
    ```
- Le cas échéant, modifier la valeur (on ne peut pas le faire dynamiquement, il faut redémarrer le serveur) :
    ```sh
    vim /etc/postgresql/9.4/main/postgresql.conf
    # listen_addresses = 'localhost'
    ```

HOWTO — vérifier que les passwords sont chiffrés :

- Vérifier la valeur du paramètre de configuration :
    ```
    postgres=# SHOW password_encryption;

    password_encryption
    ---------------------
    md5
    ```
- On peut également vérifier que le nom d'utilisateur n'est pas stocké en clair :
    ```
    postgres=# SELECT rolname, rolpassword FROM pg_authID WHERE rolname='myusername';

       rolname    |             rolpassword
    --------------+-------------------------------------
    myusername    | md54c2383f5c88e9110642953b5dd7c88a1
    ```
