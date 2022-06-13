* [Conseils généraux](#conseils-généraux)
   * [Release ou debug ?](#release-ou-debug-)
   * [CPU time vs. wall time](#cpu-time-vs-wall-time)
   * [Fréquence de sampling](#fréquence-de-sampling)
   * [cumulative time vs. local time ?](#cumulative-time-vs-local-time-)

# Conseils généraux

## Release ou debug ?

Profiler d'abord en release (avec symboles), et seulement si ça ne marche pas, en debug (en effet, c'est le mode release qui nous intéresse).

## CPU time vs. wall time

ATTENTION : ne pas confondre clock CPU vs. wall clock :

- **CPU clock** = le temps vu par le programme (n'augmente pas si le programme ne progresse pas, p.ex. parce qu'il n'a pas de time slice d'allouée, ou parce qu'il sleep)
- **wall clock** = le temps vu par l'humain qui lance le programme : il avance tout le temps, y compris si le programme n'avance pas.

Par défaut, le profiler ne va sampler que quand la cpu-clock de son programme va avancer, pas quand la wall clock va avancer. C'est pour ça qu'on ne verras pas un "sleep" dans le profiling : du point de vue du programme, le temps n'avance pas.

## Fréquence de sampling

ATTENTION : ne pas mettre une fréquence de sampling trop élevée, car avec une fréquence de sampling trop élevée, le temps "interne" (mesuré par l'outil de perf) est plus petit que le temps "externe" (mesuré par `/usr/bin/time`)

- Problème observé :
    - `/usr/bin/time` montre que l'exécution complète prend ~ 11 secondes (10.84s en userland + 0.04s en kernelland)
    - pourtant, le noeud "root" du flamegraph ne fait que 5.4 secondes en tout
    - → où passe le temps perdu ?!
- hypothèses :
    - H1 : il faut un peu de temps pour charger le programme en RAM, que le dynamic linker travaille -> le programme commence un peu en retard (EDIT : non, c'est mineur : la fonction match_graphs elle-même, mesurée avec std::chrono prend presque 10 secondes)
    - H2 : on ne mesure pas le temps passé à attendre les IO : disque-dur / RAM (EDIT : non, je peux reproduire le problème sur une fonction qui ne fait AUCUN io disque)
- RÉPONSE : j'avais une fréquence de sampling trop élevée de 500 Hz, et ça avait l'effet ci-dessus de "perte de temps". Il m'a suffi de revenir à une fréquence raisonnable de 100 Hz (par défaut) ou 200 Hz pour ne plus avoir cette "perte de temps".

## cumulative time vs. local time ?

Comment interpréter les temps et pourcentages indiqués par gperftools / pprof ?

TL;DR : c'est le cumulative time qui m'intéresse pour savoir où le programme passe son temps.

https://gperftools.github.io/gperftools/cpuprofile.html

Local time : temps passé dans la procédure uniquement (et les fonctions qui y ont été inlinées) ; si je veux diminuer ce temps, il faut que j'optimise le code DE LA FONCTION ELLE-MÊME (et non d'une de ses filles).

Cumulative time = temps passé dans la procédure ET LES FONCTIONS QU'ELLE APPELLE.

Note : les pourcentages sont exprimés en pourcentage du temps TOTAL (du temps du node root).
