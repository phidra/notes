* [Première connexion au serveur postgresql après installation](#première-connexion-au-serveur-postgresql-après-installation)
* [Fonctionnement de l'authentication postgresql](#fonctionnement-de-lauthentication-postgresql)
   * [user linux/system et user postgresql](#user-linuxsystem-et-user-postgresql)
   * [unix domain socket et TCP](#unix-domain-socket-et-tcp)
   * ['local' et 'host'](#local-et-host)
   * [fichier pg_hba.conf](#fichier-pg_hbaconf)
   * [différents types d'authentication](#différents-types-dauthentication)
* [Mes tests pour comprendre le fonctionnement de l'authentication postgresql](#mes-tests-pour-comprendre-le-fonctionnement-de-lauthentication-postgresql)

# Première connexion au serveur postgresql après installation

**TL;DR** = après l'installation, le seul moyen de se connecter au serveur postgresql est :

```sh
sudo -u postgres psql -l
```

Côté linux :

- à l'installation, l'utilisateur système `postgres` est créé
- il ne possède pas de mot de passe (et ça doit rester comme ça : l`utilisateur `postgres' est locké) :
    ```sh
    grep postgres /etc/shadow
    postgres:*:.....
    ```
- on ne peut donc s'y connecter QUE via `root` (qui shunte le mécanisme de vérification des mots de passe) :
    ```
    ROOT> su - postgres   # OK
    USER> su - postgres   # KO : il demande un mot de passe impossible à fournir, car n'existant pas
    ```

Côté postgresql :

- à l'installation, le seul rôle postgresql créé est `postgres`, il est superuser
- il n'a pas de mode passe non-plus :
    ```sql
    SELECT rolname, rolpassword FROM pg_authID WHERE rolname='postgres';
    --  rolname  | rolpassword
    -- ----------+-------------
    --  postgres |
    ```

Références sur le sujet du mot de passe de l'utilisateur système `postgres` :

- https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install/325596#325596
- https://superuser.com/questions/623881/what-means-and-at-second-field-of-etc-shadow/623882#623882


# Fonctionnement de l'authentication postgresql

## user linux/system et user postgresql

Le terme "utilisateur" est ambigü, il faut distinguer :

- l'utilisateur système (créé par `adduser`) qui existe indépendamment du serveur postgresql
- l'utilisateur postgreql (aussi appelé "rôle", dénomination que je vais garder pour limiter les confusions), qui n'a pas d'existence en dehors du serveur postgresql

L'installation de postgresql créé à la fois un utilisateur système `postgres` et le rôle postgresql correspondant `postgres`.

## unix domain socket et TCP

Il existe deux types de sockets (et même plus) :

- unix domain socket = permettant à deux process SUR LA MÊME MACHINE de communiquer
- INET socket        = permettant à deux process SUR LE MÊME RÉSEAU de communiquer

## 'local' et 'host'

- deux types de connexions à postgres sont possibles = avec socket unix ou avec socket TCP
- `local` = par socket unix :
    - comme ça utilise une unix domain socket, le client qui essaye de se connecter doit être sur la même machine que le serveur postgresql
    - ça concerne donc soit les clients fournis avec postgresql (comme psql ou createdb), soit les clients qui utilisent la libpq (je suppose que psycopg2 en fait partie)
- `host` = par socket INET :
    - rien de particulier : on utilise une socket (donc un host et un port) pour se connecter au serveur
    - attention : même si le client est sur la même machine que le serveur postgresql, il peut choisir d'utiliser une socket TCP (via l'interface loopback) plutôt qu'une socket unix
- ATTENTION : du coup, le terme "connexion locale" est ambigü :
    - habituellement, ça désigne une connexion TCP sur la même machine (i.e. l'host de la socket est localhost ou 127.0.0.1)
    - dans un contexte postgresql, ça désigne surtout une connexion à partir d'une socket UNIX plutôt qu'une socket TCP
- dans `pg_hba.conf`, 'host' (ainsi que toutes les autres méthodes d'authentification en fait) passent par une socket INET, la seule exception concerne les lignes qui commencent par 'local'
    - https://docs.postgresql.fr/12/auth-pg-hba-conf.html

## fichier pg_hba.conf

- le fichier qui configure les authentifications acceptées :
    ```
    /etc/postgresql/12/main/pg_hba.conf
    ```
- chaque ligne définit la façon d'authentifier un utilisateur :
- Le premier paramètre est le type de connexion concernée 'local' vs. 'host' :
    ```
    local    DATABASE  USER  METHOD  [OPTIONS]
    host     DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
    ```
- Exemples :
    ```
    # Autoriser l'accès "peer" pour tous les utilisateurs locaux :
    local      all       all   peer

    # Autoriser la connexion par mot de passe pour tous les utilisateurs sur localhost
    host       all       all   127.0.0.1/32   md5
    ```
- Le dernier paramètre est la façon d'authentifier l'utilisateur :
    - `trust`     autorise toutes les connexions (peut rester intéressant pour les "unix socket" = les connexions 'local', qui correspondent à une connexion depuis la machine locale, p.ex. par `createdb`)
    - `peer`      récupère le nom d'utilisateur identifié par le système d'exploitation du client et vérifie que cela correspond au rôle postgresql demandé (peer ne peut être utilisée que pour les connexions locales)
    - `md5`       connexion par login/mot de passe, chiffré par md5.
    - `ident`     semble être un équivalent de 'peer' pour les connexions TCP (si on l'utilise à tort pour une connexion locale, postgresql lui substitue automatiquement 'peer')

## différents types d'authentication

https://docs.postgresql.fr/12/auth-methods.html
- TL;DR : connexionx locales = trust ou peer / connexions distantes = password (chiffré : md5)
- PostgreSQL fournit différentes méthodes pour l'authentification des utilisateurs :
    > Authentification Trust, qui fait confiance aux utilisateurs à l'identification connue. \
    > Authentification Trust, qui fait confiance aux utilisateurs à l'identification connue. \
    > Authentification Password, qui réclame un mot de passe aux utilisateurs. \
    > [...] \
    > Authentification Ident, qui se base sur le service « Identification Protocol » (RFC 1413) de la machine cliente. (Pour des connexions locales par socket Unix, ceci est traité comme une authentification peer.) \
    > Authentification Peer, qui se base sur les capacités du système d'exploitation pour identifier le processus à l'autre bout d'une connexion locale. Ceci n'est pas supporté pour les connexions distantes.
- L'authentification peer est recommandable généralement pour les connexions locales, même si l'authentification trust pourrait être suffisante dans certains contextes.
- L'authentification password est le choix le plus simple pour les connexions distantes.
- Toutes les autres options nécessitent une forme d'infrastructure de sécurité externe (habituellement un serveur d'authentification ou une autorité de certificat pour créer des certificats SSL) ou sont spécifiques à la plateforme.

QUESTION : quelle différence entre ident et peer ?

- j'ai l'impression que ident est la version "locale" de peer...
- et on dirait surtout que je n'ai PAS à utiliser ident (car si je me connecte à distance, je veux utiliser username+password)
- https://serverfault.com/questions/515277/difference-of-postgresqls-trust-and-ident

https://docs.postgresql.fr/12/auth-peer.html

- La méthode d'authentification peer utilise les services du système d'exploitation afin d'obtenir le nom de l'opérateur ayant lancé la commande client de connexion et l'utilise (après une éventuelle mise en correspondance) comme nom d'utilisateur de la base de données.
- Cette méthode n'est supportée que pour les connexions locales.

https://docs.postgresql.fr/12/auth-ident.html

- La méthode d'authentification ident fonctionne en obtenant le nom de l'opérateur du système depuis le serveur ident distant et en l'appliquant comme nom de l'utilisateur de la base de données (et après une éventuelle mise en correspondance).
- Cette méthode n'est supportée que pour les connexions TCP/IP.
- Lorsqu'ident est spécifié pour une connexion locale (c'est-à-dire non TCP/IP), l'authentification peer (voir Section 20.9) lui est automatiquement substituée.

https://docs.postgresql.fr/12/auth-password.html documente les différentes authentications utilisant un password, concerne plusieurs valeurs :
- `md5`           = ce que postgresql utilise par défaut (= mot de passe chiffré par md5)
- `password`      = mot de passe en clair, BAD !
- `scram-sha-256` = mot de passe chiffré fortement (C'est actuellement la méthode interne la plus sécurisée, mais elle n'est pas supportée par les anciennes bibliothèques clients.)

On dirait qu'identd est un serveur permettant de récupérer l'utilisateur d'une connexion TCP.

- http://manpages.ubuntu.com/manpages/bionic/man8/identd.8.html
- https://en.wikipedia.org/wiki/Ident_protocol
- NdM = a priori, il faut un serveur identd distant... De plus, pour toute connexion distante, il faut sans doute plutôt une authent par password que par ident...


# Mes tests pour comprendre le fonctionnement de l'authentication postgresql

**Note** : je ne convertis pas ces notes OTL en markdown (le jeu n'en vaut pas la chandelle).

```otl
pourquoi tester via docker = je peux facilement tester la configuration par défaut sur une machine vierge, et modifier la configuration de postgresql sans dégouliner sur le reste de mon système
setup un docker pour faire mes tests :
	commandes docker pour faire mes tests :
		docker run --rm -it --name mycontainer debian:latest /bin/bash
			je démarre un container debian 10 from scratch pour partir d'un système vierge
			possibilité 1 = avec --rm = on peut la mettre et COMMIT l'image pour la réutiliser
			possibilité 2 = sans --rm = on peut ne PAS mettre l'option --rm, pour pouvoir arrêter le container sans le supprimer, et le réutiliser pour d'autres tests
		docker commit mycontainer testpgsql:ready_to_test
			quand je suis satisfait de l'état de mon container pour faire mes tests, je l'enregistre dans une iamge
		docker run --rm -it testpgsql:ready_to_test
			je peux alors instancier cette image custom dans un container autant que je veux
		NOTE : je n'arrive pas à enregistrer une image docker avec un postgresql qui tourne...
			du coup, je me contente d'une image avec toute la config système prête, et tant pis si je dois 'service postgresql start' à chaque instanciation dans un container
	installation et démarrage de postgres :
		apt update
		apt install postgresql
		apt install sudo  # plus pratique pour lancer des commandes en tant que 'postgres'
		apt install vim   # pour éditer 'pg_hba.conf'
		pg_ctlcluster 11 main start
		service postgresql status
			11/main (port 5432): online
		NOTE : on dirait que si on redémarre l'image, postgres est automatiquement arrêté, et on doit le redémarrer :
			service postgresql start
	créer un user LINUX et un rôle POSTGRES de même nom (qui est superuser)
		adduser anakin
			utiliser 'anakinlinux' comme mot de passe
			NOTE1 : of course, l'utilisation d'anakin pour se connecter à postgresql n'est pas possible tant qu'on n'a pas créé le rôle postgres 'anakin' en face :
				sudo -u anakin psql -l
				psql: FATAL:  role "anakin" does not exist
			NOTE2 : comme sur le container debian, on est loggé en tant que root, on peut utiliser "sudo -u anakin" sans nécessiter le mot de passe d'anakin.
		sudo -u postgres createuser --pwprompt --superuser anakin
			utiliser 'anakinpg' comme mot de passe
			NOTE : alternative pour définir le mot de passe du rôle anakin :
				sudo -u postgres psql -c "ALTER USER anakin WITH PASSWORD 'anakinpg'"
			ATTENTION : initialement, j'avais incorrectement utilisé --password au lieu de --pwprompt !
		derrière, je vérifie que je peux utiliser le rôle 'anakin' :
			sudo -u anakin psql -l
				'local' = OK sans entrer de mot de passe : j'essaye de me connecter en local avec un utilisateur existant (peer)
			sudo -u anakin psql -l -h localhost
				'host'  = OK en entrant le mot de passe : j'essaye de me connecter en host avec un utilisateur existant (md5)
à partir du setup docker, tester l'authentication postgres :
	NOTE : mon test de connexion consiste à lancer "psql -l" (qui liste les databases), mais c'est le même principe avec 'createdb'.
	TEST 0 = analyse de la configuration par défaut :
		cat /etc/postgresql/11/main/pg_hba.conf
			# TYPE  DATABASE        USER            ADDRESS                 METHOD
			local   all             postgres                                peer
			local   all             all                                     peer
			host    all             all             127.0.0.1/32            md5
			host    all             all             ::1/128                 md5
		Mon interprétation de la conf par défaut :
			local + peer
				sur la machine sur laquelle tourne le serveur, tout utilisateur linux qui a son rôle dans postgresql peut se connecter sans mot de passe
				ça inclut le rôle postgres, qui est l'administrateur par défaut du serveur postgresql → l'utilisateur LINUX postgres peut toujours se connecter
			host + md5
				pour les connexions TCP qui sont faites DEPUIS LA MÊME MACHINE, les connexions sont autorisées, mais nécessitent login + mot de passe
				pour les connexions TCP qui sont faites depuis une machine remote, les connexions sont interdites
		Que permet la configuration par défaut ?
			avec la conf par défaut, je peux me connecter en 'local' avec postgres, sans mot de passe :
				sudo -u postgres psql -h "" -l         # OK
			avec la conf par défaut, je peux me connecter en 'local' avec anakin, sans mot de passe :
				sudo -u postgres psql -h "" -l         # OK
			avec la conf par défaut, je peux me connecter en 'host' avec anakin, avec mot de passe :
				sudo -u anakin psql -h localhost -l    # OK
			NOTE : avec la conf par défaut, je ne peux PAS me connecter en 'host' avec postgres, vu que ce rôle n'a PAS de mot de passe...
				sudo -u postgres psql -h localhost -l  # KO !!!
				Password for user postgres:            # je suis bien embêté, je n'ai pas de password à entrer...
	TEST 1 = il est possible de se connecter sur un rôle postgresql dont l'utilisateur LINUX n'existe pas :
		TL;DR : se connecter par login/mdp avec un rôle postgres sans utilisateur linux en face est tout à fait possible ; obviously, la connexion locale "peer" est impossible.
		----------------------------------------
		PRÉPARATION = création d'un rôle postgresql (sans aucun utilisateur linux en face) :
			sudo -u postgres createuser --superuser inexisting_in_system
			sudo -u postgres psql -c "ALTER USER inexisting_in_system WITH PASSWORD 'pouet'"
		sudo -u postgres psql -U inexisting_in_system -l
			psql: FATAL:  Peer authentication failed for user "inexisting_in_system"
		sudo -u postgres psql -U inexisting_in_system -h localhost -l
			OK
	TEST 2 = désactiver les connexions 'local' :
		TL;DR : si on désactive les connexions 'local', on ne peut plus se connecter qu'en 'host' (avec mot de passe, donc, vu le reste de la config par défaut de pg_hba.conf)
		----------------------------------------
		PRÉPARATION = modifier la config par défaut pour désactiver les connexions 'local' :
			vim pg_hba.conf
			commenter les deux lignes "local"
			service postgresql restart
		sudo -u postgres psql -l
			psql: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "postgres", SSL off
		sudo -u anakin psql -l
			psql: FATAL:  no pg_hba.conf entry for host "[local]", user "anakin", database "postgres", SSL off
		sudo -u anakin psql -l -h "localhost"
			OK
		(impossible d'essayer en 'host' avec postgres, vu que le rôle 'postgres' n'a pas de password)
	TEST 3 = désactiver connexions host :
		TL;DR : si on désactive les connexions 'host', on ne peut plus se connecter qu'en 'local'
		----------------------------------------
		PRÉPARATION = modifier la config par défaut pour désactiver les connexions 'local' :
			vim pg_hba.conf
			commenter les deux lignes "host"
			service postgresql restart
		sudo -u postgres psql -l
			OK
		sudo -u anakin psql -l
			OK
		sudo -u anakin psql -l -h localhost
			psql: FATAL:  no pg_hba.conf entry for host "127.0.0.1", user "anakin", database "postgres", SSL on
			FATAL:  no pg_hba.conf entry for host "127.0.0.1", user "anakin", database "postgres", SSL off
		sudo -u postgres psql -l -h localhost
			psql: FATAL:  no pg_hba.conf entry for host "127.0.0.1", user "postgres", database "postgres", SSL on
			FATAL:  no pg_hba.conf entry for host "127.0.0.1", user "postgres", database "postgres", SSL off
	TEST 4 = avec libpq, on peut passer le password en envar (TODO = retrouver le lien vers la doc dans la conf slack)
		TL;DR : avec l'envvar PGPASSWORD, je peux faire sauter le prompt qui me demande le password.
		----------------------------------------
		PRÉPARATION = se connecter avec anakin (pour éviter de réfléchir à si sudo propage les envvar ou non...)
			su - anakin
		psql -h localhost -l
			OK, mais me demande le password dans un prompt.
		PGPASSWORD=anakinpg psql -h localhost -l
			OK, sans me demander le password :-)
```
