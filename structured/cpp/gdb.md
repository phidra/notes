* [Généralités](#généralités)
* [Configuration de gdb](#configuration-de-gdb)
   * [Historique des commandes](#historique-des-commandes)
   * [UX = meilleure utilisabilité](#ux--meilleure-utilisabilité)
      * [TUI](#tui)
      * [gdbinit](#gdbinit)
      * [gdb-dashboard](#gdb-dashboard)
* [Breakpoints](#breakpoints)
   * [Types de breakpoings](#types-de-breakpoings)
   * [Savoir où poser un breakpoint](#savoir-où-poser-un-breakpoint)
   * [Poser un breakpoint](#poser-un-breakpoint)
   * [Consulter et supprimer un breakpoint](#consulter-et-supprimer-un-breakpoint)
   * [Breakpoints sur une fonction overloadée](#breakpoints-sur-une-fonction-overloadée)
      * [Position du problème](#position-du-problème)
      * [Ce qui n'est pas une solution au problème](#ce-qui-nest-pas-une-solution-au-problème)
      * [Solution 1 = lever l'ambiguïté en précisant le type des paramètres de la fonction overloadée](#solution-1--lever-lambiguïté-en-précisant-le-type-des-paramètres-de-la-fonction-overloadée)
      * [Solution 2 = lever l'ambiguïté en choisissant le breakpoint à poser dans un menu](#solution-2--lever-lambiguïté-en-choisissant-le-breakpoint-à-poser-dans-un-menu)
      * [Solution 3 = conserver l'ambiguïté lorsqu'on pose le breakpoint, mais disabler ceux qui ne nous intéressent pas](#solution-3--conserver-lambiguïté-lorsquon-pose-le-breakpoint-mais-disabler-ceux-qui-ne-nous-intéressent-pas)
* [Utiliser gdb pour investiguer une situation précise (backtrace ou breakpoint)](#utiliser-gdb-pour-investiguer-une-situation-précise-backtrace-ou-breakpoint)
   * [Naviguer dans la stack-trace](#naviguer-dans-la-stack-trace)
   * [Naviguer dans le programme](#naviguer-dans-le-programme)
   * [Threads](#threads)
   * [Inspecter l'état du programme](#inspecter-létat-du-programme)
   * [Modifier l'état du programme](#modifier-létat-du-programme)
   * [charger les symboles de debug d'une lib](#charger-les-symboles-de-debug-dune-lib)
   * [Divers](#divers)

# Généralités

Trois types d'utilisation de gdb :

- démarrer l'exécution d'un binaire sous gdb
- analyser un core avec gdb (il faut disposer du binaire qui a généré le core)
- analyser un process en cours d'exécution avec gdb

Lancement :

```sh
gdb /path/to/mybinary  # puis : run arg1 arg2
gdb /path/to/mybinary -ex "run arg1 arg2"
```

Flags de compilation pour utiliser gdb :

- pour que gdb accède aux infos (nom des fonctions, code-source), il faut avoir compilé le binaire avec le flag `-g`
- notamment, si gdb affiche ce qui suit quand je `list`, c'est que j'ai oublié le `-g` :
    ```
    <built-in> No such file or directory
    ```
- par ailleurs, attention aux flags de compilation concernant l'optimisation, si j'ai le choix, `-O0` restera proche du code source
- si je n'ai pas le choix (e.g. le programme est trop lent à exécuter, ou bien le bug ne se produit qu'avec `-O2` ou `-O3`), bien penser à tout de même activer `-g` !
- attention, si une variable est marquée `<optimized out>` par gdb, c'est à cause d'un flag `-O2` ou `-O3`
- pour rappel, j'avais fait une POC sur l'impact de `CMAKE_BUILD_TYPE` sur les flags de compilation utilisés :
    ```
    CMAKE_BUILD_TYPE=Debug :
        -g
    CMAKE_BUILD_TYPE=RelWithDebInfo :
        -O2 -g -NDEBUG
    CMAKE_BUILD_TYPE=Release :
        -O3 -NDEBUG
    ```

Pour arrêter l'exécution en cours et redémarrer le programme de nouveau, il suffit de refaire `run`.

Pour quitter, `Ctrl+C` ne quitte pas mais arrête simplement la commande en cours → pour quitter c'est `quit`, `exit`, ou Ctrl+D ([doc](https://sourceware.org/gdb/current/onlinedocs/gdb/Quitting-GDB.html#Quitting-GDB))

Au sujet de la doc : **attention**, certains liens google renvoient vers une ancienne doc de gdb ([BAD = lien deprecated](https://ftp.gnu.org/old-gnu/Manuals/gdb/html_mono/gdb.html)), il faut plutôt utiliser la doc à jour : [GOOD = lien à jour](https://sourceware.org/gdb/current/onlinedocs/gdb/).

Autocomplétion : gdb autocomplète les commandes et leurs arguments :-)

Commentaires : on peut commneter des lignes avec `#` (pratique pour les intégrer à l'historique sans les exécuter), mais attention : on ne peut commenter qu'une ligne complète (pas de commentaire en fin de ligne comme en python).

Aide intégrée :

```
apropos WORD
help COMMAND
```

# Configuration de gdb

gdb a plein d'options de configuration : [doc1](https://sourceware.org/gdb/current/onlinedocs/gdb/Controlling-GDB.html#Controlling-GDB), [doc2](https://sourceware.org/gdb/current/onlinedocs/gdb/Print-Settings.html#Print-Settings)

Pour configurer gdb interactivement (en one-shot : les configs reprennent leur valeur par défaut dès la fin de la session gdb) :

```sh
# HELP :
(gdb) help set MYOPTION
(gdb) help set multiple-symbols
    Set how the debugger handles ambiguities in expressions.
    Valid values are `ask`, `all`, `cancel`, and the default is `all`.

# READ :
(gdb) show MYOPTION
(gdb) show multiple-symbols
    How the debugger handles ambiguities in expressions is `ask`.

# WRITE :
(gdb) set MYOPTION MYVALUE
(gdb) set multiple-symbols all
```

Pour configurer gdb de façon persistante, il faut les pérenniser dans un fichier gdbinit ([doc](https://sourceware.org/gdb/onlinedocs/gdb/gdbinit-man.html)). Je mets mes configs dans `~/.gdbinit`, mais il existe aussi un gdbinit systemwide `/etc/gdb/gdbinit`.

## Historique des commandes

TL;DR = pour faire persister l'historique des commandes entre plusieurs session de gdb, il faut ajouter ceci dans `~/.gdbinit` :

```gdb
    set history save on
    set history size 5000
    set history filename ~/.gdb_history
```

Derrière, l'historique est utilisable comme sous bash ou zsh :

- flèches haut et bas pour naviguer dans l'historique
- `Ctrl+R` pour rechercher dans l'historique
- (par contre, à la différence de bash/zsh, `Ctrl+S` ne recherche pas en forward)

## UX = meilleure utilisabilité

gdb est pas très ergonomique, mais il existe plusieurs projets pour améliorer ça

https://stackoverflow.com/questions/209534/how-to-highlight-and-color-gdb-output-during-interactive-debugging

### TUI

Intégré à gdb en baseline ; [doc](https://www.zeuthen.desy.de/dv/documentation/unixguide/infohtml/gdb/TUI.html)

```sh
gdb -tui
# C-x C-a pour activer/désactiver
```

Notes :

- clairement, la visibilité est meilleure puisque je vois le source et ma ligne
- (par contre, je perds l'accès à l'historique avec le flèches, puisqu'elles font maintenant réagir la fenêtre de source-code — EDIT : non, il suffit de mettre le focus ailleurs)
- d'après la doc, on dirait qu'on peut choisir les fenêtres à afficher
   - C-x 2 pour afficher DEUX panels
   - C-x o pour mettre le focus sur une autre fenêtre
- dans une fenêtre active, on peut scroller avec up/down PgUp/PgDown
- le panel du source code indique les breakpoints actuels
- il existe un SingleKey mode permettant de "piloter" l'exécution de gdb depuis une fenêtre de source / assembly : [lien](https://www.zeuthen.desy.de/dv/documentation/unixguide/infohtml/gdb/TUI-Single-Key-Mode.html#TUI-Single-Key-Mode)
- commandes ([doc](https://www.zeuthen.desy.de/dv/documentation/unixguide/infohtml/gdb/TUI-Commands.html#TUI-Commands)) :
    ```sh
    layout src         # pour afficher les sources
    info win           # pour connaître les fenêtres affichées
    winheight src -10  # pour réduire la taille de la fenêtre des sources
    ```

### gdbinit

https://github.com/gdbinit/gdbinit

Config gdb pour une meilleure utilisabilité.


### gdb-dashboard

https://github.com/cyrus-and/gdb-dashboard

A l'air trop chouette, à essayer !

Ici aussi, ça n'est qu'un gdbinit = une config gdb.

# Breakpoints

## Types de breakpoings

Ce qu'en dit [la doc](https://sourceware.org/gdb/onlinedocs/gdb/Breakpoints.html#Breakpoints) :

- **breakpoint** = break quand on atteint une certaine ligne de code ; quand on exécute une fonction en particulier
- **watchpoint** = break quand une data change ("data breakpoint") ; e.g. faire un break quand une variable change de valeur.
- **catchpoint** = break quand un évènement survient ; e.g. quand une exception est lancée (mais aussi syscall, fork, signal, ...)

Les breakpoints peuvent être conditionnels ([doc](https://sourceware.org/gdb/onlinedocs/gdb/Conditions.html#Conditions)) : `break myfunction if i == 4`

On peut configurer des commandes gdb à exécuter automatiquement quand on atteint un breakpoint ([doc](https://sourceware.org/gdb/onlinedocs/gdb/Break-Commands.html#Break-Commands)) : `commands`

## Savoir où poser un breakpoint

```sh
# afficher les fonctions, éventuellement en se limitant à la REGEX :
# (utile pour savoir sur quelle fonction exactement poser un breakpoint )
info functions [REGEX]

# connaître le code-source de la frame courante :
info source
#    Current source file is /home/myself/myproject/src/./interface_rpc/common/path_worker.h
#    [...]

# lister les fichiers sources dont gdb a connaissance :
info sources [REGEX]
```

## Poser un breakpoint

```sh
# breakpoint sur une fonction en particlier :
break my_namespace::my_super_function

# breakpoint sur une fonction en particlier, dans tel fichier source :
break my_source.cpp::my_super_function

# breakpoint placé à la ligne 43 du fichier main.cpp :
break main.cpp:43

# breakpoint sur le fait de lever une exception :
catch throw

# breakpoint sur le fait d'attraper une exception :
catch catch
```

## Consulter et supprimer un breakpoint

```sh
# supprimer TOUS les breakpoints :
delete breakpoints

# supprimer le 3ième breakpoint :
delete 3

# voir les breakpoints (montre aussi les catchpoints) :
info breakpoints
```

## Breakpoints sur une fonction overloadée

### Position du problème

Si une fonction `run` est overloadée, poser UN SEUL breakpoint sur `run` en posera PLUSIEURS (plus précisément, il posera un breakpoint avec plusieurs locations), et je me retrouve avec des breakpoints parasites.

Illustration :

```
(gdb) info functions run
    All functions matching regular expression `run`:
    File /media/DATA/git_projects/myproject/src/process_main.cpp:
        24:     int run(int, char const**);
    File /usr/include/boost/program_options/detail/parsers.hpp:
        90:     boost::program_options::basic_parsed_options<char> boost::program_options::basic_command_line_parser<char>::run();
(gdb) b process_main.cpp:run
    Breakpoint 1 at 0x137f0: process_main.cpp:run. (2 locations)
(gdb) info breakpoints
    Num     Type           Disp Enb Address            What
    1       breakpoint     keep y   <MULTIPLE>
    1.1                         y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
    1.2                         y   0x000000000001a9a0 in boost::program_options::basic_command_line_parser<char>::run() at /usr/include/boost/program_options/detail/parsers.hpp:90
```


Ici, même si `process_main.cpp` ne définit QU'UNE SEULE fonction `run`, probablement qu'après le preprocesor (et l'include de boost), le source a en fait DEUX fonctions run overloadées...

Ce qu'en dit la doc :

- https://sourceware.org/gdb/current/onlinedocs/gdb/Set-Breaks.html#Set-Breaks
    > When using source languages that permit overloading of symbols, such as C++, a function name may refer to more than one possible place to break. [...] \
    > It is possible that a breakpoint corresponds to several locations in your program. [...] \
    > In all those cases, GDB will insert a breakpoint at all the relevant locations. [...] \
    > A breakpoint with multiple locations is displayed in the breakpoint table using several rows—one header row, followed by one row for each breakpoint location. [...]
- https://sourceware.org/gdb/current/onlinedocs/gdb/Ambiguous-Expressions.html#Ambiguous-Expressions
    - Donne les solutions 1 et 2 ci-dessous (préciser le breakpoint ou utiliser un menu)
- https://sourceware.org/gdb/current/onlinedocs/gdb/Set-Breaks.html#Set-Breaks
    - Donne la solution 3 ci-dessous (disable une location en particluier)
    > You cannot delete the individual locations from a breakpoint. \
    > However, each location can be individually enabled or disabled by passing breakpoint-number.location-number as argument to the enable and disable commands.

### Ce qui n'est pas une solution au problème

Préciser le fichier source n'est pas une solution ultime, car si la fonction `run` est overloadée au sein d'un même fichier source, poser le breakpoint sur `process_main.cpp:run` ne sera pas suffisant pour désambiguer.

De même, essayer de `delete` l'un des sous-breakpoints avec `delete 7.2` ne marche pas car gdb ne permet pas de `delete` une location de breakpoint, on ne peut que `delete` le breakpoint avec _toutes_ ses locations.

### Solution 1 = lever l'ambiguïté en précisant le type des paramètres de la fonction overloadée

```
(gdb) b process_main.cpp:run(int, char const**)
    Breakpoint 3 at 0x137f0: file /media/DATA/git_projects/myproject/src/process_main.cpp, line 24.
(gdb) info b
    Num     Type           Disp Enb Address            What
    3       breakpoint     keep y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
```

### Solution 2 = lever l'ambiguïté en choisissant le breakpoint à poser dans un menu

```
(gdb) set multiple-symbols ask
    NdM : on vient de configurer gdb pour systématiquement nous demander explicitement de lever l'ambiguïté sur un breakpoint overloadé.
(gdb) b process_main.cpp:run
    [0] cancel
    [1] all
    [2] /media/DATA/git_projects/myproject/src/process_main.cpp:run(int, char const**)
    [3] /usr/include/boost/program_options/detail/parsers.hpp:boost::program_options::basic_command_line_parser<char>::run()
    > 2
    > Breakpoint 6 at 0x137f0: file /media/DATA/git_projects/myproject/src/process_main.cpp, line 24.
(gdb) info b
    Num     Type           Disp Enb Address            What
    6       breakpoint     keep y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
```

### Solution 3 = conserver l'ambiguïté lorsqu'on pose le breakpoint, mais disabler ceux qui ne nous intéressent pas

```
(gdb) b process_main.cpp:run
    Breakpoint 7 at 0x137f0: process_main.cpp:run. (2 locations)
(gdb) info br
    Num     Type           Disp Enb Address            What
    7       breakpoint     keep y   <MULTIPLE>
    7.1                         y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
    7.2                         y   0x000000000001a9a0 in boost::program_options::basic_command_line_parser<char>::run() at /usr/include/boost/program_options/detail/parsers.hpp:90
(gdb) disable 7.2
(gdb) info br
    Num     Type           Disp Enb Address            What
    7       breakpoint     keep y   <MULTIPLE>
    7.1                         y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
    7.2                         n   0x000000000001a9a0 in boost::program_options::basic_command_line_parser<char>::run() at /usr/include/boost/program_options/detail/parsers.hpp:90
(gdb) enable 7.2
(gdb) info br
    Num     Type           Disp Enb Address            What
    7       breakpoint     keep y   <MULTIPLE>
    7.1                         y   0x00000000000137f0 in run(int, char const**) at /media/DATA/git_projects/myproject/src/process_main.cpp:24
    7.2                         y   0x000000000001a9a0 in boost::program_options::basic_command_line_parser<char>::run() at /usr/include/boost/program_options/detail/parsers.hpp:90
```

NdM : on voit que le `keep` du breakpoint 7.2 est à `n` (pour NO) et que si besoin, on peut le réactiver avec `enable`

# Utiliser gdb pour investiguer une situation précise (backtrace ou breakpoint)

Si besoin, ce mini-tuto est pas mal : [doc](https://sourceware.org/gdb/current/onlinedocs/gdb/Sample-Session.html#Sample-Session)

## Naviguer dans la stack-trace

```gdb
# afficher la stack-trace :
backtrace (bt)

# remonter d'une frame (i.e. aller dans la frame de la fonction appellante) :
up

# redescendre d'une frame (i.e. aller dans la frame de la fonction appelée) :
down

# afficher le code source de l'emplacement actuel :
list (l)
```

## Naviguer dans le programme

```gdb
# exécuter la prochaine ligne DE LA FONCTION COURANTE (cette commande n'entre donc pas dans les sous-fonctions appelées) :
next (n)

# exécuter la prochaine ligne TOUT COURT (cette commande entre donc dans les sous-fonctions appelées) :
step (s)

# idem, mais pour les N prochaines lignes :
next N
step N

# reprendre l'exécution jusqu'au prochain breakpoint :
continue (c)

# terminer l'exécution de la fonction courante, puis break :
finish
```

## Threads

```gdb
# lister les threads :
info threads

# jumper au thread 3 :
thread 3
```

## Inspecter l'état du programme

cf. https://sourceware.org/gdb/onlinedocs/gdb/Symbols.html

```gdb
# afficher le contenu de VAR (e.g. un paramètre d'appel de fonction, ou une variable de la frame courante) :
print (p) VAR

# donner le type (simplifié) de VAR :
whatis VAR

# donner le type (complet) de VAR :
ptype VAR
```

## Modifier l'état du programme

gdb peut modifier des variables dans le code. La modification est "durable", i.e. elle persiste dans la suite de l'exécution du code, tout se passe comme si c'est le code lui-même (et non gdb) qui avait fait la modification.

Pour faire ça, on "exécute" du code depuis gdb, avec l'une des 3 commandes suivantes, qui semblent à peu près équivalentes :

```gdb
# de façon contre-intuitive, print ÉVALUE l'expression !
# du coup, l'instruction suivante modifie l'état du programme :
print EXPRESSION

# call est similaire à print (évalue expr), mais sans afficher void en retour :
call EXPRESSION

# set variable semble encore une alternative :
set variable EXPRESSION
```

Ce qu'en dit la doc :

- https://sourceware.org/gdb/onlinedocs/gdb/Calling.html
    > print expr \
    > Evaluate the expression expr and display the resulting value. The expression may include calls to functions in the program being debugged.
    >
    > call expr \
    > Evaluate the expression expr without displaying void returned values.
- https://sourceware.org/gdb/onlinedocs/gdb/Assignment.html#index-set-variable
    > set is really the same as print except that the expression’s value is not printed and is not put in the value history
- http://www.jaist.ac.jp/iscenter-new/mpc/altix/altixdata/opt/intel/idb/10.0.026/doc/idb_manual/common/idb_the_assign_and_the_set_variable_commands.htm
    > The only difference between the set variable and the print commands is printing the value - the set variable does not print anything.

J'ai fait deux pocs sur la modification de variables via gdb :

- [POC1](https://github.com/phidra/pocs/tree/c7a082ce09d789eaea4bb31e5697f652091cca7d/cpp/TOOL_gdb/01_modify_int) : pour modifier des variables simples (e.g. `int`), on peut directement écrire à l'adresse mémoire
- [POC2](https://github.com/phidra/pocs/tree/c7a082ce09d789eaea4bb31e5697f652091cca7d/cpp/TOOL_gdb/02_modify_string) pour modidifer des variables plus complexes (e.g. `std::string`), on peut aussi appeler des fonctions : `my_variable.assign("my_value")`

## charger les symboles de debug d'une lib


(note : pas nécessaire si la lib a été compilé avec les symboles de debug)

- https://visualgdb.com/gdbreference/commands/sharedlibrary
- https://stackoverflow.com/questions/33886913/make-gdb-load-a-shared-library-from-a-specific-path
    > If you don't want to do that, set solib-search-path and set sysroot are your friends.
- https://sourceware.org/gdb/onlinedocs/gdb/Files.html

## Divers

convenience variables = on peut définir des variables propre à gdb (indépendante du code) pour se simplifier le débugging : [doc](https://sourceware.org/gdb/onlinedocs/gdb/Convenience-Vars.html)

