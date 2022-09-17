* [perf = c'est quoi](#perf--cest-quoi)
* [installation en deux étapes](#installation-en-deux-étapes)
* [enregistrer un profil](#enregistrer-un-profil)
* [visualiser le profil avec FlameGraph](#visualiser-le-profil-avec-flamegraph)
* [visualiser le profil avec hotspot](#visualiser-le-profil-avec-hotspot)

# perf = c'est quoi

Un autre outil de profiling, qui n'est utilisable que sur linux, mais extrêmement puissant ; [leur wiki](https://perf.wiki.kernel.org/index.php/Main_Page)

J'ai surtout utilisé `perf record`, mais il y a d'autres outils :

- `perf script` ?
- `perf annotate` ?
- `perf report` ? (ça a l'air d'être une interface ncurses pour visualiser les perfs, comme FlameGraph)
- `perf top` ? (pour visualiser ce qui prend de la perf selon une interface similaire à top)

# installation en deux étapes

```sh
sudo apt install linux-tools-common

# pour connaître les versions à installer pour matcher à mon noyau :
perf --help

# e.g. :
sudo apt install linux-tools-4.15.0-62-generic linux-cloud-tools-4.15.0-62-generic
```
# enregistrer un profil

Sauf précaution particulière, il faut sudo.

```sh
sudo perf record -F 99 -ag mysuperbinary

# -F 99 = on prend 100 mesures par seconde
# -a = on mesure sur tous les CPUs
# -g = on enregistre le callgraph
```

Les "précautions particulières" pour se passer de sudo :


```sh
sudo sysctl -w kernel.perf_event_p aranoid=1

# vérification :
cat /proc/sys/kernel/perf_event_paranoid
# 1  (avant, j'avais p.ex. 3)
```

Derrière, on peut lancer perf à condition de supprimer l'option `-a` :

```sh
perf record -F 99 -g mysuperbinary
```

Le résultat est sauvegardé dans `perf.data`, qui est **ÉCRASÉ** à chaque lancement (seul le dernier est sauvegardé dans `perf.data.old`).

# visualiser le profil avec FlameGraph

https://github.com/brendangregg/FlameGraph

NOTE : a priori, hotspot est mieux (EDIT juin 2022 : even better : speedscope).

```sh
git clone https://github.com/brendangregg/FlameGraph /tmp/FlameGraph
perf record -F 99 -ag mysuperbinary
perf script | /tmp/FlameGraph/stackcollapse-perf.pl | /tmp/FlameGraph/flamegraph.pl > /tmp/perfs.svg
firefox /tmp/perfs.svg
```

Il vaut mieux ouvrir le résultat SVG avec firefox :

- on a alors plein d'infos lorsqu'on passe la souris sur une barre
- en cliquant sur une barre, on peut zoomer dessus

À POC-er = on peut filtrer la sortie de stackcollapse pour ne garder (ou exclure) que ce qui nous intéresse :

```sh
perf script | ./stackcollapse-perf.pl > out.perf-folded
grep -v cpu_idle out.perf-folded | ./flamegraph.pl > nonidle.svg
grep ext4 out.perf-folded | ./flamegraph.pl > ext4internals.svg
egrep 'system_call.*sys_(read|write)' out.perf-folded | ./flamegraph.pl > rw.svg
```

# visualiser le profil avec hotspot

https://github.com/KDAB/hotspot/

NOTE : a priori, mieux que FlameGraph de brendangregg (EDIT juin 2022 : even better : speedscope).

- disposer de diverses visualiations des records, y compris un flamegraph
- installation et lancement simple en téléchargeant l'AppImage
- client lourd, mais semble être une IHM intuitive pour comprendre les records perfs
