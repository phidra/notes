Préambule : dans ces notes, je ne m'intéresse qu'à la question de comment représenter "un point fixe dans le temps" ( `timepoint`), et pas du tout à la notion de durée (`duration`). De plus, je considère que le temps est homogène et absolu (pas de relativité).

**TL;DR :**

- un datetime (= sextuplet date + time) n'est un timepoint QUE s'il est couplé à une timezone
- il faut donc distinguer deux besoins :
    - "disposer d'une date et d'un time" (i.e. un sextuplet, p.ex. un `std::tm`) est surtout adapté pour qu'un humain puisse lire la date et l'heure
    - "exprimer un point fixe dans le temps" (i.e. un timepoint, p.ex. un `std::time_t`) est adapté le reste du temps
- en C, il existe des structures et fonctions pour manipuler le temps :
    - `std::tm` = un sextuplet (date + time)
    - `std::time_t` = un timepoint exprimé par "tant de secondes depuis tel moment dans le temps"
    - tout plein de fonctions de création/conversion : `localtime`, `gmtime`, `mktime`, etc.
- en C++, la librairie `chrono` permet de manipuler le temps :
    - `chrono::time_point` est un timepoint (une généralisation de `std::time_t`)
    - tout plein de fonctions de création/conversion : `system_clock::now`, `system_clock::from_time_t`, `system_clock::to_time_t`, etc.
- les différentes horloges de `chrono` ont chacune leur usage :
    - `chrono::steady_clock` : à utiliser pour chronométrer des durées
    - `chrono::system_clock` : à utiliser pour connaître l'heure ou la date actuelle

* [TimePoint et DateTime](#timepoint-et-datetime)
   * [Représentation d'un timepoint](#représentation-dun-timepoint)
   * [Dépendance du datetime à la timezone](#dépendance-du-datetime-à-la-timezone)
   * [Conséquence sur le nommage des variables](#conséquence-sur-le-nommage-des-variables)
* [Les types C de ctime](#les-types-c-de-ctime)
* [Les types C++ de std::chrono](#les-types-c-de-stdchrono)
* [Création et conversion des structures](#création-et-conversion-des-structures)
   * [Création d'un std::tm](#création-dun-stdtm)
   * [Création d'un std::time_t](#création-dun-stdtime_t)
   * [Création d'un chrono::time_point](#création-dun-chronotime_point)
   * [Formatter un datetime](#formatter-un-datetime)


# TimePoint et DateTime

## Représentation d'un timepoint

Un "point fixe dans le temps" = un `timepoint` peut-être représenté classiquement de deux façons :

- un nombre de secondes écoulées depuis un timepoint précis connu
- un datetime = un sextuplet (année+mois+jour+heure+minute+seconde), à condition de **FAIRE ATTENTION** !

Attention : pour qu'un datetime représente un timepoint de façon non-ambigüe, il doit **OBLIGATOIREMENT** être couplé à une timezone dans laquelle l'interprèter :

- le datetime _"le 1er janvier 2024 à 22h18"_ correspondra à des timepoints différents selon que c'est un Français ou un Canadien qui parle.
- dans l'autre sens, le timepoint _"2 heures après l'explosion du Krakatoa"_ est non-ambigü, mais sera représenté par des sextuplets différents en France ou au Canada.

Le monde Python a des dénominations que j'aime bien ([la doc datetime](https://docs.python.org/3/library/datetime.html#aware-and-naive-objects)):

- **timezone AWARE datetime** = un sextuplet couplé à une timezone :
    > An aware object represents a specific moment in time that is not open to interpretation.
- **timezone NAIVE datetime** = un sextuplet sans timezone :
    > A naive object does not contain enough information to unambiguously locate itself relative to other date/time objects.

## Dépendance du datetime à la timezone

Stricto-sensu, un datetime est l'association d'une date et d'un time ; en pratique, un objet de type datetime peut ou non comporter une timezone.

Un datetime ne représente un point fixe dans le temps **QUE** s'il est associé à une timezone ! Du coup, _un datetime n'est pas forcément un timepoint_ !

Plus généralement, la notion de date ou d'heure est très liée à la planète Terre :

- la notion de date est liée à la rotation autour du soleil
- la notion d'heure à liée la rotation de la Terre sur elle-même

À l'inverse, la notion de timepoint = "point fixe dans le temps" existe même hors de la planète Terre.

**EDIT = ATTENTION** : selon la définition qu'on donne au mot _timezone_, même un datetime couplé à une timezone peut être ambigü. Par exemple, les deux exemples suivants couplent un datetime à une "timezone", mais l'un est ambigü :

- :warning: = _"le 1er janvier 2024 à 22h18 heure de Paris"_ : la "timezone" est `heure de Paris` mais ceci est **PARFOIS AMBIGÜ**
- :white_check_mark: = _"le 1er janvier 2024 à 22h18 UTC+01:00"_ : la "timezone" est `UTC+01:00` et n'est **JAMAIS AMBIGÜ**

La première forme est ambigüe, car lors du passage à l'heure d'hiver, on recule d'une heure (à 3h du matin, il est 2h du matin). Par conséquent, on passe _DEUX FOIS_ par l'heure "2h17 du matin heure de Paris".

À l'inverse, la deuxième forme décrit une heure précise par rapport à UTC (qui est toujours bien définie), typiquement, le premier 2h17 sera `2h17 UTC+01:00` et le second sera `2h17 UTC+02:00`.

## Conséquence sur le nommage des variables

Appeler `datetime` une variable qui représente un timepoint est possiblement imprécis ("le datetime a-t-il une timezone ?") voire trompeur (j'ai appelé "datetime" quelque chose qui est un nombre de secondes depuis epoch)

À partir de maintenant, je vais peut-être plutôt nommer ce type de variable `timepoint`, et réserver le nom "datetime" à un sextuplet (année+mois+jour+heure+minute+seconde). Les datetimes auront donc surtout du sens pour exprimer un timepoint sous une forme interprétable par un humain.

En résumé, le nommage de mes variables dépendra de la sémantique que j'associe à la variable :

- sémantique de "point fixe dans le temps" = mieux vaut nommer la variable `timepoint`
- sémantique de "une date et heure" (e.g. parce que je veux l'afficher dans une IHM) = mieux vaut nommer la variable `datetime`

^ Ce dernier distinguo est important : vouloir "une date et heure" n'est important que si un humain doit interagir avec la structure (e.g. affichage dans une IHM) : si je veux identifier un timepoint pour un ordinateur, un datetime n'est qu'un moyen parmi d'autres de le faire, et il n'est pas super robuste vu qu'il y a des grosses erreurs à faire avec les timezone...

# Les types C de ctime

La lib standard C dispose de structures pour manipuler le temps, parmi lesquelles les deux représentations "classiques" que j'ai décrites :

- `std::time_t` ([doc](https://en.cppreference.com/w/cpp/chrono/c/time_t)) identifie de façon non-ambigüe un timepoint, typiquement en indiquant le nombre de secondes écoulées depuis le 1er janvier 1970 à minuit UTC
- `std::tm` ([doc](https://en.cppreference.com/w/cpp/chrono/c/tm)) est un datetime (= un sextuplet) **SANS LA TIMEZONE**, elle n'identifie donc PAS un timepoint de façon non-ambigüe ! Il faut associer cette structure à une timezone pour l'interpréter.
    - notamment, le `std::tm` en convertissant un `std::time_t` via `std::gmtime` ([doc](https://en.cppreference.com/w/cpp/chrono/c/gmtime)) est exprimé à la timezone UTC
    - certaines implémentations (non-standard C, mais standard POSIX) étendent le type avec deux champs `tm_gmtoff` et `tm_zone` pour qu'il contiennent également l'information de timezone, le rendant capable d'exprimer un timepoint

# Les types C++ de std::chrono

- `chrono::time_point` ([doc](https://en.cppreference.com/w/cpp/chrono/time_point)) = représente un timepoint non-ambigü
    - même si un `chrono::time_point` identifie un timepoint, il est toujours associé à une clock : c'est le nombre de ticks écoulés depuis l'epoch de la clock
    - c'est donc une généralisation de `std::time_t`

Les clocks sont des "machins qui tickent à intervalle régulier" (la date à laquelle une clock a commencé a ticker est son **epoch**). Il existe plusieurs clocks standard, adaptées à différents usages :

- `chrono::steady_clock` ([doc](https://en.cppreference.com/w/cpp/chrono/steady_clock)) : à utiliser pour chronométrer des durées i.e. c'est pas tant la valeur stockée dans la clock qui nous intéresse ici, c'est plutôt le diff entre deux mesures.
- `chrono::system_clock` ([doc](https://en.cppreference.com/w/cpp/chrono/system_clock)) : à utiliser pour connaître l'heure ou la date actuelle ; elle représente l'horloge de l'ordinateur.
    - à partir de C++20, son epoch est défini :
    > system_clock measures Unix Time (i.e., time since 00:00:00 Coordinated Universal Time (UTC), Thursday, 1 January 1970, not counting leap seconds).
    - NDM = je suppose que ça permet de transférer des `chrono::time_point` d'une machine à l'autre, car ils peuvent alors être interprétés de la même façon ?
    - à l'inverse de `chrono::steady_clock`, elle n'est **PAS ADAPTÉE** pour chronométrer des durées (i.e. faire des diffs) car elle peut être modifiée par un utilisateur externe à tout moment → la différence entre deux mesures de temps consécutives peut être négative (i.e. le temps peut rebrousser chemin avec cette clock, p.ex. si un utilisateur externe l'a remise à l'heure entre nos deux mesures)
    - c'est la seule clock qui peut convertir ses `chrono::time_point` en `std::time_t`
- `chrono::high_resolution_clock` ([doc](https://en.cppreference.com/w/cpp/chrono/high_resolution_clock)) : en théorie, l'horloge ayant le plus petit tick possible sur la machine ; en pratique, cette définition tend à vouloir l'utiliser pour chronométrer des durées, alors que cette horloge peut (ou pas, selon les implémentations) être un alias vers `chrono::system_clock`, qui peut remonter le temps...

# Création et conversion des structures

## Création d'un std::tm

- C'est une structure C → on peut remplir manuellement les champs de la structure à "ce qui nous intéresse"
- `std::localtime` ([doc](https://en.cppreference.com/w/cpp/chrono/c/localtime)) = transformer un `std::time_t` en un équivalent `std::tm` **EXPRIMÉ DANS LA TIMEZONE LOCALE**
    - i.e. le `std::tm` obtenu n'est plus un timepoint : il faut savoir qu'il est exprimé dans la timezone locale
- `std::gmtime` ([doc](https://en.cppreference.com/w/cpp/chrono/c/gmtime)) = transformer un timepoint de type `std::time_t` en un datetime équivalent `std::tm` **EXPRIMÉ DANS LA TIMEZONE UTC**
    - c'est l'équivalent de `std::localtime`, mais pour une timezone différente
    - ici aussi, `std::tm` obtenu n'est plus un timepoint : il faut savoir qu'il est exprimé dans la timezone UTC)
- `std::get_time` ([doc](https://en.cppreference.com/w/cpp/io/manip/get_time)) = parser une string du genre `2024-04-12T18:21:36` en un `std::tm` la représentant

On dirait qu'on ne peut pas récupérer `now`, ou bien convertir un `chrono::time_point` directement en `std::tm` : il faut d'abord passer par un `std::time_t` puis le convertir en `std::tm`.

## Création d'un std::time_t

- `std::time` ([doc](https://en.cppreference.com/w/cpp/chrono/c/time)) = renvoie le timepoint actuel dans un `std::time_t`
- `chrono::system_clock::to_time_t` ([doc](https://en.cppreference.com/w/cpp/chrono/system_clock/to_time_t)) = convertit un `chrono::time_point` d'une `system_clock` vers un `std::time_t`
- `std::mktime` ([doc](https://en.cppreference.com/w/c/chrono/mktime)) = convertit un `std::tm` supposé être exprimé dans la timezone LOCALE en `std::time_t` (c'est l'opération inverse de `std::localtime`)

L'opération équivalente à `std::mktime` avec UTC (i.e. convertir un `std::tm` en timepoint absolu, en supposant que le `std::tm` est exprimé dans UTC) n'a pas l'air possible de façon standard, mais il semble exister une fonction de la lib C GNU dédiée à ça = `timegm` ([page de man](https://man7.org/linux/man-pages/man3/timegm.3.html)) :

- elle est non-standard, c'est spécifique à GNU (donc il faut tester, peut-être que ça ne marche pas sur LLVM ?)
- le man semble indiquer que la fonction est MT-safe, i.e. qu'on peut l'utiliser dans un contexte multithreadé

## Création d'un chrono::time_point

- Les clocks steady et system disposent toutes les deux d'une fonction `now` ([doc steady](https://en.cppreference.com/w/cpp/chrono/steady_clock/now), [doc system](https://en.cppreference.com/w/cpp/chrono/system_clock/now)) pour créer un `chrono::time_point` représentant le timepoint actuel.
- `std::chrono::system_clock::from_time_t` ([doc](https://en.cppreference.com/w/cpp/chrono/system_clock/from_time_t)) permet de convertir un `std::time_t` en `std::time_point`.

On dirait qu'on ne peut pas créer un `chrono::time_point` à partir d'une string du type `2024-04-12T18:21:36` : il faut passer par `std::get_time`, puis `mktime` et `from_time_t`.

## Formatter un datetime

On s'intéresse donc ici au formatting d'un `std::tm` (i.e. un sextuplet = année+mois+jour+heure+minute+seconde) :

`std::put_time` ([doc](https://en.cppreference.com/w/cpp/io/manip/put_time)) / `std::strftime` ([doc](https://en.cppreference.com/w/cpp/chrono/c/strftime)) = ce sont deux APIs différentes pour faire la même chose = printer un `std::tm` selon un formattage user-defined
`std::asctime` ([doc](https://en.cppreference.com/w/cpp/chrono/c/asctime)) = printer un `std::tm`, mais sans laisser le choix à l'utilisateur du formattage : il est fixé (en "ascii")

Note importante : formatter un datetime de type `std::tm` est important, car si on utilise un tel datetime pour stocker un timepoint, c'est donc qu'on veut qu'un humain interagisse avec (sinon, on utiliserait un `std::time_t`) !

Conséquence = il n'y a pas de moyen direct de formatter un `std::time_t`.

Et le formatting d'un `chrono::time_point` a été introduit dans C++20 avec `std::format` (grepper `chrono` dans [la doc de std::format](https://en.cppreference.com/w/cpp/utility/format/format)).
