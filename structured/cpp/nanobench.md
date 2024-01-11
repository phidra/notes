**CONTEXTE** = janvier 2024, on cherche un outil de benchmarking de fonctions.

**TL;DR** : [nanobench](https://github.com/andreas-abel/nanoBench) a l'air mieux à tout point de vue que [google benchmark](https://github.com/google/benchmark).

Cf. ma POC sur le sujet.

Parmi les features chouettes de nanobench :

- simple à installer = header-only
- simple à utiliser = pas de macro, un objet `Bench` qu'on peut optionnellement configurer, puis qui lance une lambda.
- divers outputs possibles : console, CSV, HTML, json
- intégration avec pyperf possible pour des analyses plus poussées
- out-of-the-box, on peut comparer deux benchs successifs entre eux
- [la doc](https://nanobench.ankerl.com/) est bien (et mieux que google benchmark)
- (annoncé comme beaucoup plus rapide que google benchmark, et avec un overhead plus léger)
- si `perf_event_paranoid` l'autorise, l'outil rapporte les branch mispredictions et le nombre d'instructions nécessaires à faire tourner la lambda

Note :

- dans l'output :
    - `op/s` = combien de fois on peut lancer la fonction par seconde
    - `ns/op` = combien de temps prend la fonction pour s'exécuter
- on peut setup des choses et les passer à la lambda
- la lib permet une phase de warmup qui ne compte pas dans la mesure finale
- la lib lance plusieurs fois la lambda, on dirait qu'on n'a pas directement dans les résultats le nombre de lancements, mais on peut le déduire du temps total et de la moyenne d'un lancement
- Le même header sert à la fois d'API publique, et d'implémentation si le define `ANKERL_NANOBENCH_IMPLEMENT` est présent. Ainsi, la lib suggère de définir un fichier cpp pour ne la compiler qu'une fois, p.ex. dans un fichier `nanobench.cpp` :
    ```cpp
    #define ANKERL_NANOBENCH_IMPLEMENT
    #include <nanobench.h>
    ```
- la lib a une doc où elle se compare avec d'autres tools similaires : [lien](https://github.com/martinus/nanobench/blob/master/src/docs/comparison.rst)
- [ce post reddit](https://www.reddit.com/r/cpp/comments/nwjmct/suggestions_for_c_microbenchmarking_libraries/) suggère de préférer nanobench à google benchmark notamment pour la visu
- on dirait qu'il existe DEUX projets github : [projet1](https://github.com/martinus/nanobench), [projet2](https://github.com/andreas-abel/nanoBench) ; le lien entre les deux (et entre leurs auteurs) n'est pas clair...
    - Le premier est la lib que j'ai étudiée, la doc est servie par https://ankerl.com/ qui est le site de Martin ANKERL = Martinus
    - Le second est basé sur [ce papier](https://arxiv.org/abs/1911.03282) = _nanoBench: A Low-Overhead Tool for Running Microbenchmarks on x86 Systems_

