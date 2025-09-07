**nmap** permet de scanner des ports d'un ou plusieurs hosts.

Utile pour p.ex. vérifier pourquoi je n'arrive pas à échanger des messages sur une socket UDP.

* [Vérifier si un port TCP ou UDP est ouvert](#vérifier-si-un-port-tcp-ou-udp-est-ouvert)
* [Scanner tous les ports TCP](#scanner-tous-les-ports-tcp)
* [Scanner tous les ports UDP :](#scanner-tous-les-ports-udp-)
* [Différents états des ports](#différents-états-des-ports)


# Vérifier si un port TCP ou UDP est ouvert

```sh
# ========= TCP :
nmap -Pn  -p 80 172.18.148.20

# PORT    STATE  SERVICE
# 80/tcp  open   http


# ========= UDP :
nmap -Pn -sU -p 50001 172.18.148.5

# PORT       STATE          SERVICE
# 50001/udp  open|filtered  unknown
```

NOTE : l'option `-Pn` dit juste à nmap de skipper le fait de vérifier si l'host est up.

NOTE : pour certaines commandes, il faut être root.

# Scanner tous les ports TCP

```sh
nmap -sS -Pn -p- 172.18.149.5

# (...)
# Not shown: 65516 closed ports
# PORT      STATE SERVICE
# 22/tcp    open  ssh
# 53/tcp    open  domain
# 80/tcp    open  http
# 111/tcp   open  rpcbind
# 443/tcp   open  https
# 1534/tcp  open  micromuse-lm
# 1883/tcp  open  mqtt
# 3000/tcp  open  ppp
# 4369/tcp  open  epmd
# 5672/tcp  open  amqp
# 8400/tcp  open  cvd
# 9000/tcp  open  cslistener
# 15672/tcp open  unknown
# 15674/tcp open  unknown
# (...)
```

**ATTENTION** : il y a une différence entre :

- `nmap -sS -Pn -p- 172.18.149.5` = scanne TOUS les ports de 0 à 65535
- `nmap -sS -Pn     172.18.149.5` = ne scanne apparemment qu'un subset de 1000 ports les plus communs ?

> By default, Nmap scans the most common 1,000 ports for each protocol.

(source = `LANG=en man nmap`)

# Scanner tous les ports UDP :

```sh
nmap -sU -Pn -p- 172.18.149.5
```

Ici aussi, il faut passer explicitement les ports sous peine d'être limités aux 1000 plus communs.

# Différents états des ports

Le man nmap (`LANG=en man nmap`) détaille les 6 états dans lesquels on peut trouver les ports :

```
open
closed
filtered
unfiltered
open|filtered
closed|filtered
```

