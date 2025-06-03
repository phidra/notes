**C'est quoi** : _systemd-resolved is a systemd service that provides network name resolution to local applications_ ([source](https://wiki.archlinux.org/title/Systemd-resolved))

* [Savoir si systemd-resolved est utilisé](#savoir-si-systemd-resolved-est-utilisé)
* [Paramétrage](#paramétrage)
* [Connaître sa config DNS](#connaître-sa-config-dns)
* [Vider le cache DNS](#vider-le-cache-dns)
* [Passer des queries DNS](#passer-des-queries-dns)
* [Mode stub](#mode-stub)

(NDM : il faut sans doute entendre `RESOLVE-Daemon` avec le `D` final à prononcer à part, plutôt que prononcer en un seul mot comme le participe-passé du verbe _to resolve_)

systemd n'est pas qu'un gestionnaire de services : il propose lui-même des services, et notamment ([source](https://www.linuxtricks.fr/wiki/systemd-la-resolution-de-nom-avec-systemd-resolved)) :

- `systemd-networkd.service` : le service réseau
- `systemd-resolved.service` : le service de résolution de nom (qui peut être utilisé même si on n'utilise pas `systemd-networkd`)

# Savoir si systemd-resolved est utilisé

Savoir si le service est up and running :

```sh
systemctl status systemd-resolved
```

Par ailleurs, si `resolvectl status` renvoie quelque chose, c'est qu'on utilise systemd-resolved (car resolvectl est la CLI permettant d'interagir avec ce service).

On peut aussi vérifier qu'une query est résolue avec resolvectl (même principe : si resolvectl fonctionne, c'est qu'on utilise systemd-resolved) :


```sh
resolvectl query google.com
```

Enfin, si le fichier `/etc/resolv.conf` est en fait un lien vers un fichier `systemd-resolved`, c'est qu'on utilise le service :

```sh
ls -l /etc/resolv.conf
# /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
```

# Paramétrage

```sh
sudo vim /etc/systemd/resolved.conf
# ...

sudo systemctl restart systemd-resolved
# à faire après chaque modif
```

# Connaître sa config DNS

```sh
resolvectl status
#   resolv.conf mode: stub
# serveur DNS principal :
#        Current DNS Server: 10.20.30.40
# serveurs DNS de secours :
#        DNS Servers: 10.20.30.40 10.20.30.41
```

# Vider le cache DNS

```sh
sudo resolvectl flush-caches
```

# Passer des queries DNS

Avec `resolvectl query`, ce qui peut être utile pour vérifier si le système fonctionne.


```sh
# DIRECTE :
resolvectl query google.com
# google.com: 2a00:1450:4007:80d::200e           -- link: tun0
#             216.58.214.174                     -- link: tun0
# -- Information acquired via protocol DNS in 9.6ms.
# -- Data is authenticated: no; Data was acquired via local or encrypted transport: no
# -- Data from: network

# REVERSE :
resolvectl query 216.58.214.174
# 216.58.214.174: mad01s26-in-f14.1e100.net      -- link: tun0
#                 mad01s26-in-f174.1e100.net     -- link: tun0
#                 par10s42-in-f14.1e100.net      -- link: tun0
# -- Information acquired via protocol DNS in 10.8ms.
# -- Data is authenticated: no; Data was acquired via local or encrypted transport: no
# -- Data from: network
```

# Mode stub

TL;DR = le mode stub est un mode où `/etc/resolv.conf` utilise systemd-resolved.

Je cite https://wiki.archlinux.org/title/Systemd-resolved :

> To provide domain name resolution for software that reads /etc/resolv.conf directly, such as web browsers, Go and
> GnuPG, systemd-resolved has four different modes for handling the file—stub, static, uplink and foreign. (...) We will focus here only on the recommended mode, i.e. the stub mode which uses /run/systemd/resolve/stub-resolv.conf.
>
> /run/systemd/resolve/stub-resolv.conf contains the local stub `127.0.0.53` as the only DNS server and a list of search
> domains. This is the recommended mode of operation that propagates the systemd-resolved managed configuration to all
> clients. To use it, replace /etc/resolv.conf with a symbolic link to it.


Ma compréhension des choses est que :

- systemd-resolved lance un serveur DNS qui écoute sur `127.0.0.53`
- ce serveur redirige les requêtes qu'il reçoit vers les serveurs DNS paramétrés dans la config systemd-resolved
- ce serveur étant le SEUL listé dans `/etc/resolv.conf`,  les applications qui utilisent directement `/etc/resolv.conf` (plutôt que de passer par la config système) pourront elles aussi utiliser les serveurs configurés de systemd-resolved
- corrolaire = si on utilise systemd-resolved, il ne faut plus éditer manuellement `/etc/resolv.conf` !
