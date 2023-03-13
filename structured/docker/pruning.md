# Pruning docker

https://docs.docker.com/config/pruning/

Ces notes concernent le ménage qu'on peut faire dans l'espace utilisé par docker ; et par défaut, docker va en bouffer, de la place :

> Docker takes a conservative approach to cleaning up unused objects (...), such as images, containers, volumes, and networks: these objects are generally not removed unless you explicitly ask Docker to do so. This can cause Docker to use extra disk space.


* [Pruning docker](#pruning-docker)
   * [Images](#images)
   * [Containers](#containers)
   * [Volumes](#volumes)
   * [Networks](#networks)

> For each type of object, Docker provides a prune command. In addition, you can use docker system prune to clean up multiple types of objects at once.

^ `docker system prune` n'est qu'un sucre syntaxique sur un job que plusieurs commande `prune` individuelles peuvent faire.

> The docker system prune command is a shortcut that prunes images, containers, and networks. Volumes are not pruned by default, and you must specify the --volumes flag for docker system prune to prune volumes.

^ les volumes ne sont pas prunés par défaut (même avec `-a`).

## Images

> The docker image prune command allows you to clean up unused images. By default, docker image prune only cleans up
> dangling images. A dangling image is one that is not tagged and is not referenced by any container. (...) To remove
> all images which are not used by existing containers, use the `-a` flag. You can limit which images are pruned using filtering expressions with the `--filter` flag. For example, to only consider images created more than 24 hours ago:

```
docker image prune -a --filter "until=24h"
```

^ Suppression des images dangling (sans `-a`), ou des images dangling ET des images non-associées à un container (avec `-a`) ; possibilité d'être moins aggressif en filtrant sur la date de création.

## Containers

> When you stop a container, it is not automatically removed unless you started it with the `--rm` flag. To see all
> containers on the Docker host, including stopped containers, use docker `ps -a`

^ par défaut, lister les containers ne montre que les containers en cours d'exécution, il faut ajouter `-a` pour lister également les containers arrêtés.

Pour supprimer les containers arrêtés, on peut utiliser cette commande, et ici aussi on peut filtrer sur la date de création :

```
docker container prune --filter "until=24h"
```

Note : certes ça fait du ménage et c'est bon à prendre, mais les containers ne prennent pas forcément beaucoup de place, vu que c'est surtout leur writable layer qui prend de la place, donc si le container n'a rien écrit, le supprimer ne libèrera pas de place.

## Volumes

> Volumes are never removed automatically, because to do so could destroy data.

^ cf. mes notes sur le sujet ; ici, on parle des `--mount type=volume` (et non pas des `bind-mounts`, qui sont en fait des répertoires du filesystem host, et que docker n'a donc pas vocation à cleaner).

On peut filtrer, mais uniquement à partir du `label` du volume (label = une metadata associée au volume lors de sa création).

Pour faire des suppressions plus évoluées, il faut utiliser un filtre plus complexe avec `docker volume ls --filter`, puis supprimer le volume manuellement.

## Networks

> Docker networks don’t take up much disk space, but they do create iptables rules, bridge network devices, and routing table entries. To clean these things up, you can use docker network prune to clean up networks which aren’t used by any containers.
