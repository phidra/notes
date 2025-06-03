**TL;DR** = une image docker windows n'est pas pullable depuis Linux.

Explication : une image docker n'est pas un unique fichier, c'est l'agrégat de :

- un manifest qui décrit les layers
- les différents fichiers qui contiennent les layers

Du coup, quand on `docker pull` une image docker, on ne télécharge pas un (unique) fichier représentant l'image !

Sous le capot, on télécharge d'abord le manifest, puis chacun des layers, puis docker reconstruit l'image à partir des layers (+ éventuellement de config additionnelle dans le manifest).

(l'un des intérêts de faire ça, c'est qu'on peut mutualiser les layers intermédiaires entre plusieurs images les utilisant)

Or, un filesystem linux ne pourra pas reconstruire un filesystem windows !

[source](https://forums.docker.com/t/docker-daemon-on-ubuntu-pull-windows-containers-or-create-my-own/28823/8) :

> Unfortunately, a Linux system won’t be able to handle Windows image nor vice-versa.
>
> The reason is that when you docker pull an image, the image layers are unpacked into the graph-driver on the system so that the image is ready to run at a moments notice. Docker running on Linux will not be able to unpack Windows filesystem layers because the Windows and Linux filesystems are not compatible.

Et effectivement, manuellement, depuis mon host linux, j'arrive à télécharger des layers d'une image windows, mais pas à reconstruire l'image totale.
