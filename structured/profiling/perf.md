**C'est quoi ?** Un outil de profiling, qui n'est utilisable que sur linux, mais extrêmement puissant.

La [page de Brendan Gregg](https://www.brendangregg.com/perf.html) donne des one-liners utiles, et sinon il y a [le wiki](https://perf.wiki.kernel.org/index.php/Main_Page) (qui a notamment un [tutorial](https://perf.wiki.kernel.org/index.php/Tutorial)) ou `man perf-record`.

* [installation](#installation)
* [enregistrer un profil](#enregistrer-un-profil)
   * [avec sudo](#avec-sudo)
   * [enregistrer un profil partiel](#enregistrer-un-profil-partiel)
   * [se passer de sudo](#se-passer-de-sudo)
   * [numéros de lignes](#numéros-de-lignes)
   * [lenteur de perf script](#lenteur-de-perf-script)
* [autres usages](#autres-usages)

# installation

Ça se fait en deux étapes

```sh
# STEP 1 = connaître les paquets à installer selon mon noyau :
sudo apt install linux-tools-common
perf --help

# STEP 2 = installer ce qui a été suggéré, p.ex. :
sudo apt install linux-tools-4.15.0-62-generic linux-cloud-tools-4.15.0-62-generic
```
# enregistrer un profil

## avec sudo

Sauf précaution particulière, il faut utiliser sudo :

```sh
sudo perf record -F 99 -g --call-graph=dwarf mysuperbinary
```

- `-F` est la fréquence de sampling (ici, on prend 99 samples par seconde)
- `-g` enregistre la call-stack
- `--call-graph=dwarf` : si je laisse la valeur par défaut `fp`, j'ai des callstacks très imprécises. Peut-être lié à ce point de `man perf-record` :
    > In some systems, where binaries are build with gcc --fomit-frame-pointer, using the "fp" method will produce bogus call graphs, using "dwarf", if available (perf tools linked to the libunwind or libdw library) should be used instead.
- (si besoin, `-a` permet d'enregistrer ce qui se passe sur tous les CPUs plutôt que juste les events d'une cible particulière, PID ou commande)

Le résultat est sauvegardé dans `perf.data`, qui est **ÉCRASÉ** à chaque lancement (seul le dernier est sauvegardé dans `perf.data.old`).

Derrière, il faut postprocesser `perf.data` pour en faire quelque chose, e.g. avec `perf script` :

```sh
perf script -i perf.data > /tmp/profile.txt
# visualiser profile.txt, e.g. avec speedscope
```

## enregistrer un profil partiel

Profil partiel = on ne record qu'une partie de l'exécution d'une commande.

Cas d'usage = si la commande complète mets trop de temps à s'exécuter, elle génère une quantité de samples trop grande pour `perf script` ou pour `speedscope` → en ne recordant qu'une partie de son exécution, on réduit la quantité de samples.

```sh
# on exécute la commande :
./mysupercommand arg1 arg2

# on récupère son PID :
pgrep mysupercommand

# quand on le souhaite, on enregistre la prochaine minute :
perf record -F 99 -g --call-graph=dwarf -p <PID> sleep 60
```

## se passer de sudo

À refaire à chaque boot :

```sh
# si besoin, mémoriser la valeur avant sa modification :
cat /proc/sys/kernel/perf_event_paranoid
# e.g. 3

# réduire la parano :
sudo sysctl -w kernel.perf_event_paranoid=1

# si besoin, vérifier :
cat /proc/sys/kernel/perf_event_paranoid
# 1
```

## numéros de lignes

TL;DR = inutile d'avoir les numéros de ligne car on a déjà les fully-qualified noms des fonctions.

En pratique, j'ai essayé de suivre [ce conseil](https://stackoverflow.com/questions/44865551/source-line-numbers-in-perf-call-graph) de modifier l'appel à `perf script` :

```sh
perf script -i perf.data -F+srcline > /tmp/profile.txt
```

Mais non seulement c'est démesurément lent (même pour quelques centaines de sample, ça prend de longues minutes), mais surtout, ça ne fait pas ce que je veux : ça ajoute des events artificiels avec les source lines.

## lenteur de perf script

TL;DR = `perf script` peut être très lent, c'est un problème connu, il existe des contournements que je n'ai pas encore testés.

Pour convertir une adresse binaire en le nom de la fonction correspondante, perf fait un appel à `addr2line` pour chaque adresse, ce qui est très lent.

Les contournements consistent à utiliser une version patchée de perf :

- https://michcioperz.com/post/slow-perf-script/
- https://eighty-twenty.org/2021/09/09/perf-addr2line-speed-improvement
- https://github.com/flamegraph-rs/flamegraph/issues/74

# autres usages

J'ai surtout utilisé `perf record`, mais il y a d'autres outils :

- `perf script` = convertir les records en des stacks exploitables par les visualiseurs (tels que FlameGraph ou speedscope)
- `perf annotate` = afficher le code-source annoté par le nombre d'events par ligne
- `perf report` = ça a l'air d'être une interface ncurses pour visualiser les perfs, comme FlameGraph
- `perf top` = visualiser à quoi sont occupés les CPU actuellements, selon une interface similaire à top

Par ailleurs, il y a surtout beaucoup d'autres events à enregistrer que "simplement" le temps CPU, par exemple les cache hit ou miss L2 (mais j'ai pas creusé).
