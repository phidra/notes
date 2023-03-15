# Images docker

**Contexte** : à la base, je m'intéressais à la différence entre _dangling image_ et _unreferenced image_ dans le contexte du pruning (les premières sont supprimées par `docker system prune`, les secondes par `docker system prune -a`), et j'ai tiré le fil vers des trucs un peu plus généraux.

Les notes sont assez brutes pour ne pas perdre trop de temps à mettre en forme (ce que je compte faire de toutes façons dans un blogpost).

Résumé :

- il y a deux builders docker pour builder une image à partir d'un dockerfile : le builder legacy, et le builder moderne a.k.a [BuildKit](https://docs.docker.com/build/buildkit/)
- avec le builder legacy :
    - chaque commande dans le dockerfile conduit à une image intermédiaire
    - l'image finale buildée est celle qui est associée au tag choisi par l'option `-t` ; elle pointe sur son image intermédiaire parente, elle-même pointant sur son propre parent, etc.
    - une image **dangling** est une image non-taggée, qui n'est pas parente (éloignée) d'une image taggée
    - une image non-dangling peut devenir dangling lorsqu'on rebuilde l'image : ça déplace le tag, et l'image précédente devient dangling
- avec BuildKit :
    - les étapes intermédiaires ne créent plus des images, mais des objets de cache
    - la seule image crée l'est donc sur l'état final du Dockerfile


## vrac source 1 = image taggée vs. image dangling

https://vsupalov.com/docker-image-layers/

> TL;DR
>
> - Each layer is an image itself, just one without a human-assigned tag. They have auto-generated IDs though.
> - Each layer stores the changes compared to the image it’s based on.
> - An image can consist of a single layer (that’s often the case when the squash command was used).
> - Each instruction in a Dockerfile results in a layer. (Except for multi-stage builds, where usually only the layers in the final image are pushed, or when an image is squashed to a single layer).
> - Layers are used to avoid transferring redundant information and skip build steps which have not changed (according to the Docker cache).



En gros, une image, c'est un layer, qui contient un diff par rapport à son image parente.

Toutes les images ont un id (sha256), mais il existe également une autre façon de désigner les images = leur nom + tag, e.g. `myimage:latest`.

Une image **taggée**, c'est une image qui a un nom+tag pointant vers elle ; toutes les images ne sont pas taggées (et plusieurs tags peuvent pointer vers la même image).

Les images intermédiaires sont des vraies images : quand on liste les images avec l'option `-a`, on les voit apparaître aussi.

Toute ligne indépendante dans le `Dockerfile` créée une image intermédiaire ; `docker build -t mytag` se contente d'associer le tag `mytag` à la dernière image du dockerfile.

(il est possible de squasher les images intermédiaires, ce qui peut être utile pour masquer des secrets, en les ajoutant à une ligne 1, les utilisant dans une ligne 2, puis les supprimant à une ligne 3 : le squashing fera disparaître le secret)

Une image **dangling** est une image non-taggée **ET** qui n'est pas un parent d'une image taggée. L'idée est que les images dangling ne sont jamais utiles, p.ex. c'étaient peut-être des images parentes d'une image taggée, mais le tag a été depuis déplacé vers une image plus récente (ce qui a rendu obsolète l'ancienne image taggée et ses parentes). Plus rarement, elles peuvent être issues d'un build sans tag (mais si on veut utiliser l'image fraîchement buildée, on veut toujours tagger l'image).

Du coup, les images intermédiaires ne seront pas considérées comme **dangling**, du moins pas tant que l'image taggée finale n'aura pas été supprimée/aura perdu son tag.

## vrac source 2 = image taggée vs image dangling

https://stackoverflow.com/questions/45142528/what-is-a-dangling-image-and-what-is-an-unused-image/45143234#45143234

> An unused image means that it has not been assigned or used in a container. For example, when running docker ps -a - it will list all of your exited and currently running containers. Any images shown being used inside any of containers are a "used image".
>
> On the other hand, a dangling image just means that you've created the new build of the image, but it wasn't given a new name. So the old images you have becomes the "dangling image". Those old image are the ones that are untagged and displays "<none>" on its name when you run docker images.
>
> When running docker system prune -a, it will remove both unused and dangling images. Therefore any images being used in a container, whether they have been exited or currently running, will NOT be affected.

```
docker system prune --all --filter "until=24h"
```

---


> Images in docker are referenced by a sha256 digest, often referred to as the image id. That digest is all you need for the image to exist on the docker host. Typically, you will have tags that point to these digests, e.g. the tag 'busybox:latest' currently points to image id c30178c523... on my system.
>
> Multiple tags can point to the same image, and any tag can be changed to point to a different id, e.g. when you pull a new copy of busybox:latest or build a new version of your application image.
>
> Dangling images are images which do not have a tag, and do not have a child image (e.g. an old image that used a different version of FROM busybox:latest), pointing to them. They may have had a tag pointing to them before and that tag later changed. Or they may have never had a tag (e.g. the output of a docker build without including the tag option).

^ s'il n'y a ni tag, ni autre image l'utilisant, alors l'image est dangling.

---

> Dangling images are layers that have no relationship to any tagged images. They no longer serve a purpose and consume disk space.
>
> An unused image is an image that has not been assigned or used in a container.

---

> - Dangling images are untagged images.
>     - docker image prune deletes all dangling images.
> - Unused images are images that have tags but currently not being used as a container. You may or may not need it in future.
>     - docker image prune -a delete all dangling as well as unused images.
> - You generally don't want to remove all unused images until some time. Hence it is better to remove with a filter.
>     - docker image prune -a -f --filter "until=6h"

---

https://www.reddit.com/r/docker/comments/idzlr1/comment/g2cn7qt/?utm_source=share&utm_medium=web2x&context=3

> Unused = images that aren't tied to any container, either in use or stopped. \
> Danglings = unnamed image, when you only see the id, or <none> instead of the tag-name.

^ ce que j'en comprends :

- une image peut devenir dangling lorsqu'on l'avait précédemment taggée, puis qu'on a taggé une image plus récente
- on ne veut pas garder les images dangling : personne ne les utilise
- on peut vouloir garder les images unused : ce n'est pas parce qu'aucun container ne les utilise actuellement


## vrac source 3 = les layers intermédiaires sont des images

cf. ma POC pour montrer que les layers intermédiaires sont bien des images, et qu'elles ne sont pas dangling (même si non-taggées)

Connaissance acquise =:

- les layers intermédiaires sont listés comme des images (mais ne sont visibles qu'avec `-a`)
- malgré leur absence de tags, ils ne sont pas considérés comme dangling, car leur image fille pointe sur eux

EDIT : ceci est valable avec le builder legacy : avec [BuildKit](https://docs.docker.com/build/buildkit/), c'est différent.

## vrac source 4 = builder une image sans lui attribuer de tag

Dockerfile de test :

```
FROM busybox:latest

RUN echo "test1" > /tmp/contenu1
RUN echo "test2" > /tmp/contenu2
RUN echo "test3" > /tmp/contenu3
RUN echo "test4" > /tmp/contenu4
RUN echo "test5" > /tmp/contenu5

CMD ["cat", "/tmp/test1", "/tmp/test2", "/tmp/test3", "/tmp/test4", "/tmp/test5"]
```

Essai :

```sh
docker build .
docker image list
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# <none>       <none>    eb32f77bff31   1 second ago   4.86MB
# busybox      latest    bab98d58e29e   6 days ago     4.86MB
```

L'image buildée n'a pas de tag.

À noter que si on fait ça, l'image est aussitôt considérée comme dangling (et sera supprimée aussitôt par un `docker system prune`).

## vrac source 5 = inspection d'une image

cf. ma POC sur l'inspection d'une image

Connaissance acquise :

- une image est un diff par rapport à son image parente
- du coup, une image pointe vers sont parent, et contient la commande qui a généré le diff

## vrac source 6 = layers

Et en regardant rapidement les layers :

```
/var/lib/docker/overlay2
```

Après un `docker system prune -a`, le répertoire est vide.

Après un build, j'ai 6 layers :

- un correspondant à busybox
- 5 correspondant à mes 5 commandes `RUN`

On dirait que chaque layer contient les fichiers ajoutés (et supprimés, je suppose) par le layer. Exemple avec mon troisième layer :

```
880d04b76920eef1834bc81506c3241c887fa4b22be128c80eead258fde5fb0d
├── committed
├── diff
│   └── tmp
│       └── contenu3
├── link
├── lower
└── work
```


Et effectivement, dans l'image adéquate, je retrouve ce layer `880d0` dans le `docker inspect` de cette image intermédiaire, sous `MergedDir`, `UpperDir` et `WorkDir` (par ailleurs, les layers mentionnés dans `LowerDir` sont ceux des images parentes) :

```
"Cmd": [
	"/bin/sh",
	"-c",
	"echo \"test3\" > /tmp/contenu3"
],
[...]
"GraphDriver": {
	"Data": {
		"LowerDir": "/var/lib/docker/overlay2/d457c2a8fecfcdf1f46a486b4cf8605604051f9ff1fa3c060a008fce5cafb23c/diff:/var/lib/docker/overlay2/37e7c03d897051a7f867fa105ab65e5faed4b7cc19553508eff042cbeb30080b/diff:/var/lib/docker/overlay2/fe7b9dcf33fcb9ae0357426b73bdb731201e9b528b827cbef7d2d059e5bc684b/diff",
		"MergedDir": "/var/lib/docker/overlay2/880d04b76920eef1834bc81506c3241c887fa4b22be128c80eead258fde5fb0d/merged",
		"UpperDir": "/var/lib/docker/overlay2/880d04b76920eef1834bc81506c3241c887fa4b22be128c80eead258fde5fb0d/diff",
		"WorkDir": "/var/lib/docker/overlay2/880d04b76920eef1834bc81506c3241c887fa4b22be128c80eead258fde5fb0d/work"
	},
	"Name": "overlay2"
},
```
