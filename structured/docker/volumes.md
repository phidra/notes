# Volumes docker

* [Volumes docker](#volumes-docker)
   * [bind mounts vs. volumes vs. tmpfs](#bind-mounts-vs-volumes-vs-tmpfs)
      * [Quand utiliser quoi ?](#quand-utiliser-quoi-)
   * [suppression des données](#suppression-des-données)
   * [--mount vs. -v](#--mount-vs--v)
   * [RUN --mount = mount à l'étape de build](#run---mount--mount-à-létape-de-build)

https://docs.docker.com/storage/

> By default all files created inside a container are stored on a writable container layer.

Quand on `docker run` un container, même sans aucun volume, on peut écrire des trucs sur le filesystem du container ; par contre ces trucs seront définitivement perdus une fois le container arrêté : si on veut **persister** des données, il faut monter un volume, par exemple :

```
docker run --rm -it --mount type=bind,source="$(pwd)"/mysubdir,target=/app mycontainer:latest
```

> No matter which type of mount you choose to use, the data looks the same from within the container. It is exposed as either a directory or an individual file in the container’s filesystem.

^ du point de vue du container, le type de `--mount` ne change rien.

## bind mounts vs. volumes vs. tmpfs

Les différences entre les 3 types de `--mount` :

- **bind-mount** = on rend visible depuis le container un répertoire/fichier du filesystem HOST ; du coup le container PARTAGE les données avec l'host :
    > Bind mounts may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.
- **volume** = on laisse docker gérer l'espace de stockage, celui-ci est inaccessible à quelqu'un d'autre que docker (en revanche, plusieurs containers différents peuvent monter le même volume)
    > Volumes are stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/ on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.
- **tmpsfs** = l'espace de stockage est en RAM, rien n'est persisté sur disque-dur
    > tmpfs mounts are stored in the host system’s memory only, and are never written to the host system’s filesystem.

Tout ceci est résumé par ce schéma, qui indique où vivent les données :

![types of mount](./types-of-mounts.png)

Quelques notes complémentaires :

- un `volume` est "juste" un répertoire particulier sous `/var/lib/docker/volumes/`, géré par docker
- les `bind-mounts` donnent accès au filesystem du host, alors que les `volume` sont isolés (même si techniquement, les `volume` vivent sur le filesystem du host aussi)
- on peut gérer les `volume` indépendamment des containers (API ou CLI, p.ex. scripter leur création) et plusieurs containers peuvent se partager un même volume
- peu importe le type (`volume` ou `bind-mount`) le stockage monté pourra être créé à la volée au moment de la création du container : inutile qu'il existe en avance de phase
- un `bind-mount` doit être référencé par son chemin absolu sur la machine host
- l'un des inconvénients d'utiliser un `bind-mount` est qu'on a besoin d'une structure pré-existante du filesystem host pour que ça fonctionne :
    > Bind mounts are very performant, but they rely on the host machine’s filesystem having a specific directory structure available.
    >
    > When the Docker host is not guaranteed to have a given directory or file structure, volumes help you decouple the configuration of the Docker host from the container runtime.
- un `dangling volume` est un volume qui n'est actuellement référencé par aucun container
- j'ai pu vérifier expérimentalement que le `type=` par défaut d'un `--mount` est `volume` ; les deux commandes suivantes sont équivalentes :
    ```
    docker run --rm -it --mount type=volume,source=myvolume,target=/hosttmp mycontainer
    docker run --rm -it --mount             source=myvolume,target=/hosttmp mycontainer
    ```

### Quand utiliser quoi ?

La doc recommande de préférer les `volumes` et de n'utiliser les `bind-mounts` que dans certains cas :

> In general, you should use volumes where possible. Bind mounts are appropriate for the following types of use case:
>
> - Sharing configuration files from the host machine to containers. This is how Docker provides DNS resolution to containers by default, by mounting /etc/resolv.conf from the host machine into each container.
> - Sharing source code or build artifacts between a development environment on the Docker host and a container. (...)
> - When the file or directory structure of the Docker host is guaranteed to be consistent with the bind mounts the containers require.

Et pour `tmpfs` ? On pourrait se dire à tort que comme le writable layer du container sera de toutes façons perdu, on n'aura jamais besoin de ça ? Mais en fait on a les mêmes besoins qu'un `tmpfs` classique (hors docker) = ça va plus vite d'écrire en RAM + si on ne veut pas persister de secrets sur HDD.

## suppression des données

La durée de vie d'un `bind-mount` n'est par définition pas liée à celle du container (vu que c'est en fait un répertoire du filesystem host).

De façon peut-être plus contre-intuitive, la durée de vie d'un `volume` n'est pas non-plus liée à celle d'un container, même si celui-ci l'a créé ; **le volume a son existence propre, indépendante des containers** ; il ne sera jamais supprimé sauf demande explicite de l'utilisateur (`docker volume rm/prune`) :

> If you don’t explicitly create it, a volume is created the first time it is mounted into a container. When that container stops or is removed, the volume still exists. (...) Volumes are only removed when you explicitly remove them.

(ça permet p.ex. de partager des données entre plusieurs exécutions successives d'un container ; par contre ça rajoute une charge de cleaning à faire pour penser à détruire les données)

## --mount vs. -v

Utiliser la syntaxe `--mount` est recommandé :

> Bind mounts and volumes can both be mounted into containers using the -v or --volume flag, but the syntax for each is slightly different.

Exemple :

```
docker run --rm -it --mount type=bind,source="$(pwd)"/mysubdir,target=/app mycontainer:latest
```

Même si une syntaxe alternative (`docker run -v  aka  docker run --volume`) existe.

## RUN --mount = mount à l'étape de build

Les notes suivantes concernent une étape différente : on ne parle plus du stockage au moment de **l'exécution** du container, mais au moment du **build d'une image**.

https://docs.docker.com/engine/reference/builder/#run---mount

Il y a alors d'autres types de mounts (notamment `secret` et `ssh`), et celui qui m'intéresse le plus est `cache` ([lien](https://docs.docker.com/engine/reference/builder/#run---mounttypecache)) :

> This mount type allows the build container to cache directories for compilers and package managers. (...) Contents of the cache directories persists between builder invocations without invalidating the instruction cache.

^ ce type de mount persiste entre les builds → un build n°2 peut bénéficier d'un cache qui aurait été peuplé par un build n°1.

> Cache mounts should only be used for better performance. Your build should work with any contents of the cache directory as another build may overwrite the files or GC may clean it if more storage space is needed.

^ Attention que le build n°2 ne doit pas _nécessiter_ un cache peuplé pour fonctionner.
