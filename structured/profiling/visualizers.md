Une fois qu'on a enregistré les données de profiling (e.g. créées par `py-spy` ou par `perf`), on a besoin d'outils pour les visualiser et les éplucher.

* [SpeedScope](#speedscope)
   * [Visualiser un profil](#visualiser-un-profil)
   * [Notes vrac](#notes-vrac)
* [FlameGraph](#flamegraph)
* [hotspot](#hotspot)

# SpeedScope

https://www.speedscope.app/

https://github.com/jlfwong/speedscope

Tool similaire à FlameGraph, mais en mieux car interactif.

## Visualiser un profil

**Note** : le résultat de `perf script -i perf.data > /tmp/profile.txt` est chargeable directement sur speedscope, [lien](https://github.com/jlfwong/speedscope/wiki/Importing-from-perf-(linux)).

À partir d'un profil speedscope, on peut le visualiser par 2 moyens :

- en le chargeant le profil sur https://www.speedscope.app/ (d'après [la doc](https://github.com/jlfwong/speedscope#usage), le profil n'est pas uploadé, tout est in-browser)
- en chargeant le profil sur un speedscope local : `speedscope myprofile.json` (installé avec `npm install -g speedscope`)

Avec un moyen bonus : une fois qu'on a chargé un profil dans un speedscope local, deux fichier (html+js) sont créés, ils contiennent la visualisation du profil, qui peut alors être consulté juste en ouvrant le html dans le browser :

```sh
/tmp/speedscope-1655208088279-67869.html
/tmp/speedscope-1655208088279-67869.js
```

Note : ces fichiers html+js sont créés dans `os.tmpdir()`, qui renvoit `/tmp` chez moi (et c'est problématique si chromium n'y a pas accès). Pour utiliser un autre répertoire temporaire :

```sh
TMPDIR=/path/to/another/folder speedscope profile.speedscope
```

Par ailleurs, on ne peut pas facilement déplacer ces fichiers, car le chemin vers le js est hardcodé dans le html.

## Notes vrac

- Couplé à [py-spy](./py-spy.md), ça permet de profiler des scripts python, y compris leurs extensions C/C++.
- Les différentes vues sont toutes très utiles :
    - `Time Order` présente les mesures en respectant l'ordre dans lequel elles ont été exécutées
    - `Left Heavy` change cet ordre pour regrouper les mesures entre elles (y compris si elles ne sont pas consécutives), et classer les plus lourdes en premier
    - `Sandwich` liste les symboles (avec un tableau triable par colonne, et greppable avec `Ctrl+F`), puis affiche la fonction choisie en indiquant notamment ses différents appelants, et leur répartition relative.
    - sandwich (très utile pour voir qui appelle une fonction)
- On peut visualiser les différents threads de façon indépendante.
- On peut zoomer sur une fonction (en cliquant / double-cliquant dessus).
- Pour avoir les infos détaillées de la callstack C/C++, il faut avoir compilé avec les infos de debug (e.g. `RelWithDebInfo`)
- J'ai observé parfois une imprécision sur l'attribution d'une mesure à la ligne de code (elle était attribuée à la ligne suivante)

# FlameGraph

https://github.com/brendangregg/FlameGraph

EDIT juin 2022 : speedscope est plus simple à utiliser et plus puissant.

```sh
git clone https://github.com/brendangregg/FlameGraph /tmp/FlameGraph
perf record -F 99 -g  --call-graph=dwarf mysuperbinary
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

# hotspot

https://github.com/KDAB/hotspot/

NOTE : a priori, mieux que FlameGraph de brendangregg (EDIT juin 2022 : even better : speedscope).

- disposer de diverses visualiations des records, y compris un flamegraph
- installation et lancement simple en téléchargeant l'AppImage
- client lourd, mais semble être une IHM intuitive pour comprendre les records perfs
