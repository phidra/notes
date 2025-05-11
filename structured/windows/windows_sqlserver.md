**Contexte** = début 2025, je dois travailler avec SqlServer.

# Lancer SqlServer via docker

```
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=toto999+" -p 1433:1433  mcr.microsoft.com/mssql/server:2019-latest
```

(il est casse-bonbons sur la force du mot de passe)

À noter que le port par défaut est `1433`.

# sqlcmd

Ça semble être la CLI utilisable sous linux pour aller passer des commandes au server SqlServer.

## tester le setup

Pour tester la disponibilité du tool + la connexion, le plus simple est de requêter la version du serveur :

```sh
/opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P "toto999+" -C -Q 'SELECT @@VERSION;'
```

(dans ce qui suit, je me contente de `sqlcmd` sans le préfixe)


## utiliser un port différent

ÉVIDEMMENT, la syntaxe n'est pas habituelle, et utilise une virgule `IP,PORT` plutôt que `IP:PORT`... grmbl :

```sh
sqlcmd -S localhost,1435 -U SA -P "toto999+" -C -Q "SELECT @@VERSION;"
```

(pour tester, binder le port docker 1433 sur le port de l'host 1444)
