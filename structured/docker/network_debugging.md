# Debugging du réseau docker

([source](https://bobcares.com/blog/docker-image-dns-lookup-error/))

Lorsqu'un container n'arrive pas à utiliser le réseau, le débugger directement n'est pas toujours pratique ; p.ex. une image debian n'aura pas l'utilitaire `ping`... et ne pourra pas l'installer vu qu'elle n'a pas accès au réseau !

Contournement : on peut débugger avec l'image `busybox`.

Hors de tout container, on récupère déjà une IP de test :

```sh
nslookup google.com  # p.ex. 142.250.74.238
```

Puis, on peut débugger l'accès des containers au réseau avec `busybox` :

```sh
# vérifier que le container a accès à l'IP de test :
docker run --rm busybox ping -c 1 142.250.74.238

# si oui, c'est peut être un problème DNS :
docker run --rm busybox nslookup google.com

# sortie possible en cas d'échec :
# ;; connection timed out; no servers could be reached

# dans ce cas, on peut essayer avec un autre serveur DNS :
docker run --rm --dns=8.8.8.8 busybox nslookup google.com

# sortie possible en cas de succès :
# Server:		8.8.8.8
# Address:	8.8.8.8:53
#
# Non-authoritative answer:
# Name:	google.com
# Address: 142.250.74.238
#
# Non-authoritative answer:
# Name:	google.com
# Address: 2a00:1450:4007:80d::200e
```

Si le problème du container est un souci DNS, aller jeter un oeil au contenu du fichier `/etc/docker/daemon.json`.

Une fois corrigé : `sudo service docker restart`
