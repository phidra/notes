* [Speedscope c'est quoi](#speedscope-cest-quoi)
* [Visualiser un profil](#visualiser-un-profil)
* [Notes vrac](#notes-vrac)

# Speedscope c'est quoi

https://www.speedscope.app/

https://github.com/jlfwong/speedscope

Tool similaire à FlameGraph, mais en mieux car interactif.

# Visualiser un profil

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

# Notes vrac

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
