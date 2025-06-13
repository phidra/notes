**Contexte** = début 2025, je dois travailler avec SqlServer.

* [Lancer SqlServer via docker](#lancer-sqlserver-via-docker)
* [sqlcmd](#sqlcmd)
   * [installation](#installation)
   * [tester le setup](#tester-le-setup)
   * [passer des requêtes](#passer-des-requêtes)
   * [utiliser un port différent](#utiliser-un-port-différent)
   * [utiliser un résultat dans un script](#utiliser-un-résultat-dans-un-script)
   * [boucler sur les tables](#boucler-sur-les-tables)
* [Transact-SQL aka t-sql](#transact-sql-aka-t-sql)
   * [Procédures stockées](#procédures-stockées)



# Lancer SqlServer via docker

```
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=toto999+" -p 1433:1433  mcr.microsoft.com/mssql/server:2019-latest
```

(il est casse-bonbons sur la force du mot de passe)

À noter que le port par défaut est `1433`.

# sqlcmd

Ça semble être le client approprié côté linux pour requêter une base SqlServer.



## installation

Je suis [la doc de microsoft](https://learn.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools?view=sql-server-ver16&tabs=ubuntu-install#install-tools-on-linux), mais en gros, après avoir installé le repo APT de microsoft, il faut :

```sh
sudo apt install mssql-tools18 unixodbc-dev
```

## tester le setup

Pour tester la disponibilité du tool + la connexion, le plus simple est de requêter la version du serveur :

```sh
/opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P "toto999+" -C -Q 'SELECT @@VERSION;'
```

(dans ce qui suit, je me contente de `sqlcmd` sans le préfixe)

## passer des requêtes

En passant la query directement :

```sh
sqlcmd -S localhost -U SA -P "toto999+" -C -Q "SELECT @@VERSION;"
```

En pipant :

```sh
echo 'SELECT @@VERSION;' | sqlcmd -S localhost -U SA -P "toto999+" -C
```

En exécutant les queries d'un fichier :

```sh
sqlcmd -S localhost -U SA -P "toto999+" -C -i /tmp/queries.sql
```

En lançant un prompt interactif :

```sh
sqlcmd -S localhost -U SA -P "toto999+" -C

# EXIT = quitter
# GO   = exécuter la ou les commandes (car non, ha ha, c'est du windows, donc bien sûr c'est pas intuitif...)
```

## utiliser un port différent

ÉVIDEMMENT, la syntaxe n'est pas habituelle, et utilise une virgule `IP,PORT` plutôt que `IP:PORT`... grmbl :

```sh
sqlcmd -S localhost,1435 -U SA -P "toto999+" -C -Q "SELECT @@VERSION;"
```

(pour tester, binder le port docker 1433 sur le port de l'host 1444)

## utiliser un résultat dans un script

```sh
sqlcmd -S localhost  -U SA -P "toto999+" -C -h-1 -W -Q "SET NOCOUNT ON SELECT name FROM sys.databases;"

# -h-1           = enlever le header
# SET NOCOUNT ON = ne pas afficher le nombre de lignes
# -W             = ne pas afficher d'espaces en fin de ligne
```

## boucler sur les tables

Plus une astuce à pérenniser qu'un point spécial sur sqlcmd, mais voici un exemple qui boucle sur toutes les bases pour compter les procédures stockées :

```sh
sqlcmd -S localhost -U SA -P "toto999+" -C -h-1 -W -Q "SET NOCOUNT ON SELECT name FROM sys.databases;" | while read db_name
do
echo "$(sqlcmd -S localhost -d ${db_name} -U SA -P "toto999+" -C -h-1 -W -Q "SET NOCOUNT ON SELECT COUNT(*) FROM sys.procedures;"),${db_name}"
done | sort -r -n > /tmp/count_procedures.csv
```

(ça permet d'éviter d'avoir à créer une table temporaire, ou de faire des queries complexes ; au prix de N requêtes au lieu d'une)

# Transact-SQL aka t-sql

C'est le dialecte SQL de SqlServer.

Quelques notes en vrac :

```sql
-- connaître la version du serveur :
SELECT @@VERSION;

-- connaître la base actuelle :
SELECT DB_NAME();

-- lister les bases :
SELECT name FROM sys.databases;

-- compter les procédures :
SELECT COUNT(*) FROM sys.procedures;
```

On peut construire des requêtes dynamiques (i.e. on construit une string contenant la requête, puis on l'exécute) :

```sql
DECLARE @szOS VARCHAR(50);
SELECT @szOS = host_platform FROM sys.dm_os_host_info;
IF (@szOS LIKE '%Windows%')
BEGIN
     DECLARE @sql VARCHAR(100);
     SET @sql = 'GRANT ADMINISTER BULK OPERATIONS TO ThisSuperRole;';
     EXEC sp_executesql @sql;
END
GO
```

^ ci-dessus, on utilise une requête dynamique qui ne sera exécutée **que** sous Windows ; on pourrait penser qu'il suffit de mettre la requête dans le bloc `IF`, mais ça ne marche pas, car même le simple parsing de la requête échoue sous Linux :

> Keyword or statement option 'ADMINISTER BULK OPERATIONS' is not supported on the 'Linux' platform.

## Procédures stockées

```sql
EXEC mon_schema.ma_procedure @mon_param=219
-- à noter que la PS renvoie des valeurs : ce que je récupère en l'exécutant est un équivalent d'un SELECT
```

