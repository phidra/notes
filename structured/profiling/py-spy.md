* [py-spy c'est quoi](#py-spy-cest-quoi)
* [Lancer un script en enregistrant son profil py-spy](#lancer-un-script-en-enregistrant-son-profil-py-spy)
* [Enregistrer le profil d'un process déjà en train de tourner](#enregistrer-le-profil-dun-process-déjà-en-train-de-tourner)
* [Profiler une extension C/C++](#profiler-une-extension-cc)

# py-spy c'est quoi

https://github.com/benfred/py-spy

Un profiler de code python, qui peut également profiler les extensions C/C++, et qui sait générer sa sortie au format [speedscope](./speedscope.md).

Installation simple :

```
pip install py-spy
```

(py-spy fonctionne même sur des pythons aussi anciens que python3.5)

# Lancer un script en enregistrant son profil py-spy

Le principe de base, c'est de lancer son script avec py-spy :

```
py-spy -- python myscript.py --option=value arg1 arg2
```

Détail 1 = il y a diverses options, la plus importantes est `--format speedscope`, ainsi que `--native` si on veut profiler les extensions C/C++ :

```
py-spy --native --format speedscope -o /tmp/profile.json -- python myscript.py --option=value arg1 arg2
```

Détail 2 = si on utilise pyenv, il faut trouver l'emplacement de l'interpréteur python :

```sh
pyenv which python
# /home/myself/.pyenv/versions/mysupervenv/bin/python

py-spy -- /home/myself/.pyenv/versions/mysupervenv/bin/python myscript.py --option=value arg1 arg2
```

Détail 3 = si on utilise un entry-point, il faut trouver l'emplacement du script exécuté :

```sh
which myentrypoint

# ou si on utilise pyenv :
pyenv which myentrypoint

# /home/myself/.pyenv/versions/mysupervenv/bin/myentrypoint
```

Au global, la commande complète peut ressembler à ça :

```sh
py-spy record --native --idle --threads -o /tmp/profile.speedscope --format speedscope -- /home/myself/.pyenv/versions/mysupervenv/bin/python /home/myself/.pyenv/versions/mysupervenv/bin/myentrypoint --option=value arg1 arg2
```

# Enregistrer le profil d'un process déjà en train de tourner

Même principe, avec la subtilité qu'il faut être **root** pour s'attacher au process :

```sh
ps aux|grep mysuperprocess
sudo env PATH=$PATH py-spy record --native --idle --threads -o /tmp/profile.speedscope --format speedscope --pid 42419
```

Attention aux droits du fichier de profil (et malheureusement, ça n'est qu'à la fin du record que ça échouera s'il y a un souci là-dessus). Le mieux étant soit que le fichier de destination n'existe pas, soit qu'il ait des droits ouverts à tout le monde.

# Profiler une extension C/C++

Ne pas oublier l'option `--native` (mais voir aussi les autres options de `py-spy help record`).

Compiler en `RelWithDebInfo` pour avoir les infos de debug (numéros de lignes, notamment), sinon on n'aura que les noms des symboles sans savoir où on passe le temps.
