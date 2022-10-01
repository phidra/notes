NOTE : cette page regroupe des notes sur les clients qui accompagnent postgresql : `psql`, `createdb`, etc.

* [utilisation de psql et des utilitaires qui accompagent psql](#utilisation-de-psql-et-des-utilitaires-qui-accompagent-psql)
   * [Utilisateur postgres](#utilisateur-postgres)
   * [Connexion 'local' vs. 'host'](#connexion-local-vs-host)
   * [psql 101](#psql-101)
* [Utilisation de psqsl](#utilisation-de-psqsl)
* [CRUD de bases et utilisateurs](#crud-de-bases-et-utilisateurs)

# utilisation de psql et des utilitaires qui accompagent psql

NOTE : c'est le paquet `postgresql-client` qui fournit psql, ainsi que d'autres utilitaires CLI (e.g. `createuser`, `createdb`, etc.) :

```sh
dpkg-query -L postgresql-client-common
# [...]
# /usr/bin/createdb
# /usr/bin/createuser
# /usr/bin/dropdb
# /usr/bin/dropuser
# /usr/bin/pg_dump
# /usr/bin/pg_restore
# /usr/bin/psql
# [...]
```

## Utilisateur postgres

Préalable à toute commande :

```sh
sudo su - postgres
sudo -u postgres COMMAND  # alternative
```

(on utilise l'user UNIX postgres, créé par l'installation de postgresql, à ne pas confondre avec le RÔLE postgres, cf. mes notes sur l'authentication)

## Connexion 'local' vs. 'host'

- si on ne précise pas d'host, les clients psql essayeront de se connecter avec une socket unix ('local')
- pour forcer une connexion "host" plutôt qu'une connexion locale, il suffit de passer l'host (à 'localhost') :
    ```sh
    sudo -u anakin psql -l -h localhost  # <---- fonctionne, EN DEMANDANT le mot de passe du rôle anakin
    ```
- pour "annuler" le host (i.e. forcer le fait de passer par une connexion locale), on peut passer un host vide :
    ```sh
    sudo -u anakin psql -l -h ""         # <---- fonctionne, SANS DEMANDER le mot de passe du rôle anakin
    ```

## psql 101

- connecter psql à la base CUSTOM :
    ```sh
    sudo su - anakin
    psql deathstar
    ```
- lister les bases de données (c'est un bon moyen de tester qu'on peut se connecter, sans nécessiter une base en particulier) :
    ```sh
    psql -l
    ```
- charger un script sql :
    ```sh
    psql -f MYFILE.SQL -U MYUSERNAME -h localhost -d MYDATABASE
    ```

NOTE : par défaut (sans option `-h`), psql se connecte en 'local' avec un rôle postgres de même nom que l'utilisateur linux qui a lancé psql.

# Utilisation de psql

- quitter le client :
    ```
    \q
    CTRL+d
    ```
- aide des commandes propres à psql :
    ```
    \?
    ```
- aide SQL :
    ```
    \help
    \h
    ```
- lister les bases existantes :
    ```
    \list
    ```
- me connecter à une base
    ```
    \connect MYBASE
    ```
- à quelle database suis-je connecté :
    ```
    \conninfo
    You are connected to database "MYBASE" as user "myself" via socket in "/var/run/postgresql" at port "5432".
    ```
- lister les utilisateurs (=rôles) existants :
    ```
    \du
    \dg
    ```
- lister les schémas
    ```
    \dn
    ```
- montrer le schéma par défaut actuel :
    ```sql
    SHOW search_path;
    ```
- changer le schéma par défaut actuel :
    ```sql
    SET search_path TO monsuperschema;
    ```
- lister les tables du schéma courant
    ```
    \dt
    \dt *.    ->   de tous les schémas (marche même pour les tables qui ne sont pas dans le search_path)
    \dt pouet.   ->   du schéma 'pouet'
    ```
- lister les colonnes de `myschema.mytable`
    ```
    \d+ myschema.mytable
    ```

# CRUD de bases et utilisateurs

- créer un utilisateur CUSTOM avec son passowrd :
    ```sh
    createuser anakin
    psql
    # postgres=# \password anakin  # Entrer le mot de passe
    ```
- créer un utilisateur CUSTOM avec son passowrd (ALTERNATIVE) :
    ```sh
    createuser -P luke  # un prompt demandera le password
    ```
- créer une base CUSTOM, et l'attribuer à l'utilisateur CUSTOM:
    ```sh
    createdb --owner=anakin deathstar
    ```
- supprimer une base CUSTOM :
    ```sh
    dropdb deathstar
    ```
- supprimer un utilisateur LUKE :
    ```sh
    dropuser anakin
    ```
- cloner une database 
    ```sh
    createdb -O owner -T src_db dst_db
    ```
    - cf. https://stackoverflow.com/questions/876522/creating-a-copy-of-a-database-in-postgresql
    - attention : la DB doit être idle (personne ne doit y être connecté), sinon on aura l'erreur suivante :
        ```
        createdb: error: database creation failed: ERROR:  source database "src_db" is being accessed by other users
        DETAIL:  There are 3 other sessions using the database.
        ```
    - et dans ce cas, pour forcer brutalement la déconnexion des clients :
        ```sql
        SELECT pg_terminate_backend(pg_stat_activity.pid) FROM pg_stat_activity
        WHERE pg_stat_activity.datname = 'originaldb' AND pid <> pg_backend_pid();
        ```
- le même genre de CRUD, directement en SQL :
    ```sql
    CREATE USER myself WITH PASSWORD 'myself';  -- création de l'utilisateur 'myself'
    ALTER USER myself CREATEDB;  -- autoriser myself à créer des DB
    ALTER USER myself SUPERUSER;  -- tout autoriser (nécessaire pour installer postgis)
    GRANT ALL PRIVILEGES ON DATABASE mysuperdb TO myself;  -- tout autoriser à myself, pour une database donnée uniquement
    DROP USER myusername;  -- supprimer l'utilisateur
    DROP DATABASE mydatatable;  -- supprimer la base
    ```

ATTENTION il y a deux utilisateurs à ne pas confondre :
    - `--owner` = -O = utilisateur à qui appartiendra la database créée
    - `--username` = -U = login qu'on utilise pour la connexion au serveur
