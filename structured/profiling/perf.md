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
* [script tout-en-un](#script-tout-en-un)


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
sudo perf record -F 99 -g --call-graph=dwarf -o NOGIT_perf.data mysuperbinary
```

- `-F` est la fréquence de sampling (ici, on prend 99 samples par seconde)
- `-g` enregistre la call-stack
- `--call-graph=dwarf` : si je laisse la valeur par défaut `fp`, j'ai des callstacks très imprécises. Peut-être lié à ce point de `man perf-record` :
    > In some systems, where binaries are build with gcc --fomit-frame-pointer, using the "fp" method will produce bogus call graphs, using "dwarf", if available (perf tools linked to the libunwind or libdw library) should be used instead.
- `-o` renseigne le fichier de sortie (par défaut : `perf.data`)
- (si besoin, `-a` permet d'enregistrer ce qui se passe sur tous les CPUs plutôt que juste les events d'une cible particulière, PID ou commande)

Le résultat est sauvegardé dans `NOGIT_perf.data`, qui est **ÉCRASÉ** à chaque lancement (seul le dernier est sauvegardé dans `NOGIT_perf.data.old`).

Derrière, il faut postprocesser `NOGIT_perf.data` pour en faire quelque chose, e.g. avec `perf script` :

```sh
perf script -i NOGIT_perf.data > /tmp/profile.txt
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

(on peut également lancer `perf record` sans `sleep`, et l'arrêter manuellement avec Ctrl+C quand on a récolté suffisamment de données)

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
perf script -i NOGIT_perf.data -F+srcline > /tmp/profile.txt
```

Mais non seulement c'est démesurément lent (même pour quelques centaines de sample, ça prend de longues minutes), mais surtout, ça ne fait pas ce que je veux : ça ajoute des events artificiels avec les source lines.

# lenteur de perf script

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

# script tout-en-un

Le script suivant wrappe :

- le lancement d'un service en tâche de fond
- l'exécution de perf
- le passage de requêtes


<details>
  <summary>Click to expand!</summary>

```sh
#!/bin/bash


PERF_PROFILE="${1:-/tmp/profile.txt}"
PERF_DATA="$(mktemp)" || exit 1


function run_service() {
    /path/to/my/service --port 7777 &
    server_pid=$!

    function kill_server() {
        kill -SIGINT $server_pid
        wait $server_pid
    }
    trap 'kill_server' EXIT

    # on boucle jusqu'à ce que le serveur soit démarré :
    max_attempts=30
    attempt=1
    success=false
    while [[ $attempt -le $max_attempts ]]; do
        set +o errexit
        response=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:7777/is-server-started")
        set -o errexit
        if [[ $response -eq 200 ]]; then
            success=true
            break
        fi
        echo "On attend que le serveur démarre et réponde sur l'endpoint de version..."

        sleep 0.5
        ((attempt++))
    done

    if [[ $success == false ]]; then
        echo "ERROR = le serveur n'a pas démarré dans le temps imparti"
        exit 1
    fi
}



function send_requests() {
    REQUESTS="""toto,tata,titi
hoho,haha,hihi
yoyo,yaya,yiyi"""

    failure_count=0
    while IFS= read -r line; do
      arg1="$(echo "$line" | cut -d ';' -f 1)"
      arg2="$(echo "$line" | cut -d ';' -f 2)"
      arg3="$(echo "$line" | cut -d ';' -f 3)"
      set +o errexit
      curl "http://localhost:7777/endpoint?arg1=${arg1}&arg2=${arg2}&arg3=${arg3}" &> /dev/null
      if [[ $? -ne 0 ]]; then
        ((failure_count++))
      fi
      set -o errexit
    done <<< "$REQUESTS"

    if [[ $failure_count -ne 0 ]]; then
        echo "ERROR : $failure_count échecs de curl"
        exit 1
    fi
}


# STEP 1 = lancer le service :
run_service

# STEP 2 = requêter, sandwiché entre démarrage et arrêt du profiling perf :
perf record -F 99 -g --call-graph=dwarf -p "$server_pid" -o "$PERF_DATA" &
perf_pid=$!

start=$(date +%s%3N)
send_requests
end=$(date +%s%3N)
requesting_time_ms=$((end - start))
echo "REQUESTING TIME = $requesting_time_ms ms"

kill -SIGINT $perf_pid
set +o errexit  # perf_pid n'existe peut-être plus au moment où on appelle wait
wait $perf_pid
set -o errexit

# STEP 3 = profil utilisable par https://www.speedscope.app/
perf script -i "$PERF_DATA" > "$PERF_PROFILE"
```

</details>
