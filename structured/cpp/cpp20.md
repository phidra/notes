**Contexte** : septembre 2023, on doit choisir le standard+compilo pour un nouveau projet. Le choix se limite à C++17 ou C++20 (avant c'est trop ancien, après c'est trop récent) ; je m'intéresse donc à ce qu'apporte C++20.

* [Mon avis en quelques mots](#mon-avis-en-quelques-mots)
* [ce qu'apporte C++20](#ce-quapporte-c20)
   * [Explications vrac sur quelques features mentionnées](#explications-vrac-sur-quelques-features-mentionnées)
      * [constant expression](#constant-expression)
      * [CTAD Class Template Argument Deduction](#ctad-class-template-argument-deduction)
      * [span](#span)
      * [feature test macros](#feature-test-macros)
      * [three-way comparison](#three-way-comparison)
      * [designated initializers](#designated-initializers)
      * [features liées aux range-based for-loops](#features-liées-aux-range-based-for-loops)
      * [likely / unlikely](#likely--unlikely)
   * [Nouvelles features](#nouvelles-features)
      * [Liste officielle](#liste-officielle)
      * [Core language features with examples](#core-language-features-with-examples)
      * [Features of C++20](#features-of-c20)
      * [C++20/17/14/11](#c20171411)
* [support de C++20 par les compilateurs et outils](#support-de-c20-par-les-compilateurs-et-outils)
      * [Comment vérifier les outils dispos](#comment-vérifier-les-outils-dispos)
   * [les compilateurs disponibles par défaut sur divers OS](#les-compilateurs-disponibles-par-défaut-sur-divers-os)
   * [support par les compilateurs](#support-par-les-compilateurs)
      * [clang++](#clang)
      * [g++](#g)
      * [commun](#commun)
   * [features non-supportées](#features-non-supportées)
   * [support par les outils](#support-par-les-outils)

# Mon avis en quelques mots

De base, grosse incitation à utiliser C++20 pour rester à jour : il n'est pas exclu de rester en C++17, mais il faut alors de solides arguments en face de cette décision.

- globalement, code plus expressif, moins buggé, plus efficace
- quelques ajouts majeurs qui apportent beaucoup : concepts, coroutines (j'exclus les modules), span, (format ? likely/unlikely ?)
- mais surtout beaucoup beaucoup BEAUCOUP de choses moins majeures, mais très utiles : elles finissent par s'additionner pour faire un langage beaucoup plus agréable à utiliser. Liste non-exhaustive :
    - ranged based for loops
    - lambdas
    - structured bindings
    - source_location
    - constant expressions
    - CTAD
    - floating points atomics
    - osyncstream
    - starts_with / contains / midpoint
    - make_shared

# ce qu'apporte C++20

## Explications vrac sur quelques features mentionnées

### constant expression

https://en.cppreference.com/w/cpp/language/constant_expression
> Defines an expression that can be evaluated at compile time.
>
> Such expressions can be used as non-type template arguments, array sizes, and in other contexts that require constant expressions

### CTAD Class Template Argument Deduction

https://en.cppreference.com/w/cpp/language/class_template_argument_deduction

- avant C++17, l'argument deduction ne marchait que pour les fonctions, pas pour les classes
- à partir de C++17, l'argument deduction marche aussi pour les classes

### span

https://en.cppreference.com/w/cpp/container/span

> The class template span describes an object that can refer to a contiguous sequence of objects with the first element of the sequence at position zero. A span can either have a static extent, in which case the number of elements in the sequence is known at compile-time and encoded in the type, or a dynamic extent.
>
> If a span has dynamic extent, a typical implementation holds two members: a pointer to T and a size. A span with static extent may have only one member: a pointer to T.

https://github.com/AnthonyCalandra/modern-cpp-features#stdspan

> - A span is a view (i.e. non-owning) of a container providing bounds-checked access to a contiguous group of elements.
> - Since views do not own their elements they are cheap to construct and copy -- a simplified way to think about views is they are holding references to their data.
> - As opposed to maintaining a pointer/iterator and length field, a span wraps both of those up in a single object.

### feature test macros

https://en.cppreference.com/w/cpp/feature_test

- macros pour vérifier le support de telle feature par le compilateur
- intérêt mineur : permet d'adapter le code à ce que permet le compilateur

### three-way comparison

https://en.cppreference.com/w/cpp/language/operator_comparison#Three-way_comparison

- opérateur `<=>` pour faire plusieurs comparaison d'un coup
- intérêt : code moins verbeux et plus expressif, nice to have

### designated initializers

https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers

- initialisation des structs en précisant leur champs
- intérêt : plus expressif, réduit les risques de bug, serait un plus

### features liées aux range-based for-loops

https://en.cppreference.com/w/cpp/language/range-for

- Init-statements and initializers in range-for
- intérêt = nice to have : permet une initialisation optionnelle des range-loops.

### likely / unlikely

https://en.cppreference.com/w/cpp/language/attributes/likely

> New attributes: `[[no_unique_address]]`, `[[likely]]`, `[[unlikely]]`

Intérêt = nice to have : permet sans doute de meilleures optimisations.

## Nouvelles features

### Liste officielle

https://en.cppreference.com/w/cpp/20

(elle indique aussi le support par les compilateurs, cf. plus bas)

### Core language features with examples

https://oleksandrkvl.github.io/2021/04/02/cpp-20-overview.html

Une très bonne ressource : c'est une liste de toutes les nouvelles features du C++20 assorties d'exemples et d'explications. Attention, la liste se limite aux features du language, et non de la stdlib (alors qu'il y a aussi plein de trucs intéressants dans la stdlib).

Quelques notes en vrac :

- Concepts
- Modules
- Coroutines
- Three-way comparison
- Lambda expressions
    - explicit lambda-capture this
    - template parameter for generic lambdas ([]<typename T>(std::vector<T> vector){ })
    - Lambdas in unevaluated contexts
    - Default constructible and assignable stateless lambdas
    - Pack expansion in lambda init-capture
    - (et plus globalement, plein de petits trucs qui font que c'est plus simple de bosser avec les lambdas)
- Constant expressions
    - (...)
    - consteval pour les fonctions compile-time only
    - (et plus globalement plein de petits trucs qui font que c'est plus simple de bosser avec constexpr)
- Aggregates
    - (...)
    - Class template argument deduction for aggregates
    - Parenthesized initialization of aggregates  (This allows seamless usage of factory functions like std::make_unique<>()/emplace() with aggregates.)
- Non-type template parameters
- Structured bindings
    - (globalement quelques petits trucs qui font que c'est plus simple de bosser avec les structured bindings)
- Range-based for loop
    - (globalement plein de petits trucs qui font que c'est plus simple de bosser avec les ranged-based for loops)
- Attributes
    - quelques trucs utiles pour les attributs
    - notamment, `likely` et `unlikely` auront probablement un effet sur les perfs
- Character encoding (`char8_t` pour représenter un literal UTF-8)
- Sugar
    - plein plein de petits trucs souhaitables ici
    - e.g. typename un peu plus optionnel, designated initializers, conversions plus safe, etc.
- ...plein d'autres petits trucs...

### Features of C++20

https://www.geeksforgeeks.org/features-of-c-20/

Dans la même veine que la ressource précédente (mais en moins bonne qualité) ; intéressante tout de même car moins exhaustive donc plus synthétique.

- concepts
- threeway operator (more expressiveness in how to define our relations, allows us to write less code to define them, and avoids some performance pitfalls of manually implementing some comparison operators in terms of others)
- map contains (plus expressif)
- range-based for loop with initializations (  `for (std::vector v{ 1, 2, 3 }; auto& e : v)` )
- modules (a priori inutilisables pour le moment sauf sous windows)
- Calendar and time zone library (The chrono library of C++ 11/14 was extended with a calendar and time-zone feature. The calendar consists of types, which represent a year, a month, a day of a weekday, or n-th weekday of a month.)
- std::string functions  (`start_with` / `ends_with`)
- Array bounded/unbounded (ça a l'air d'être des fonctions d'introspection permettant de savoir si un type array est bounded ou non)
- std::to_array (conversion en array)
- Likely and unlikely attributes (likely/unlikely to have its body executed. Both attributes allow giving the Optimizer a hint)

### C++20/17/14/11

https://github.com/AnthonyCalandra/modern-cpp-features#c20171411

Même principe ici aussi, moins d'exhaustivité ; mais contenu volumineux malgré tout, car couvre tous les standards depuis C++11.

Le point intéressant = en plus des features du langage, on s'intéresse aussi aux features de la stdlib, dont certaines sont très utiles :

- quelques concepts utiles dans la stdlib
- synchronized buffered outputstream (i.e. no interleaving of output)
- spans
- string : starts_with / ends_with
- bit_cast
- .contains()
- std::midpoint()

# support de C++20 par les compilateurs et outils

### Comment vérifier les outils dispos

Un moyen simple de checker ce qui est dispo sous un OS est d'utiliser docker :

```sh
docker run --rm -it ubuntu:20.04

# dans le container :
apt update && apt upgrade
apt install build-essential clang
g++ --version
clang++ --version
```

## les compilateurs disponibles par défaut sur divers OS

Contexte = je m'intéresse surtout à ce qui est dispo sur ubuntu 20.04 et 22.04.

```
ubuntu:20.04 :
    g++     = 9.4.0
    clang++ = 10.0.0

ubuntu:22.04 :
    g++     = 11.4.0
    clang++ = 14.0.0

ubuntu:23.04 :
    g++     = 12.3.0
    clang++ = 15.0.7

debian:bookworm :
    g++     = 12.2.0
    clang++ = 14.0.6

debian:bullseye :
    g++     = 10.2.1
    clang++ = 11.0.1-2

alpine:3.18 :
    g++     = 12.2.1
    clang++ = 16.0.6
```

## support par les compilateurs

Préambule : comme le support de C++20 est réparti feature par feature, il est important de s'être d'abord familiarisé avec les features avant d'analyser le support.

TL;DR : les features C++20 non-supportées ont l'air assez mineures : on peut considérer que g++11 / clang14 supportent C++20 (le support a l'air un peu meilleur par g++11). Les deux features notables non-supportées sont les modules et le header `<format>`.

Note C++17 : aussi bien en 20.04 qu'en 22.04, le C++ 17 est pleinement supporté [par g++](https://gcc.gnu.org/projects/cxx-status.html#cxx17) et [par clang++](https://clang.llvm.org/cxx_status.html#cxx17).

### clang++

TL;DR = clang 14 est loin de supporter C++20 entièrement, mais c'est sans doute largement utilisable en pratique (et clang 10 ne supporte pas C++20).

Le support de C++20 par clang++ est [détaillé ici](https://clang.llvm.org/cxx_status.html#cxx20) :

- à la louche, la moitié des features de C++20 ne sont pas supportées par clang 10
- plus embêtant, une proportion non-négligeable des features de C++20 ne sont pas supportées en clang 14...
- (bon, en pratique, les features non-supportées n'ont pas l'air très embêtantes)
- pour avoir un support statistiquement correct (et encore, à l'exception des modules), il faut un clang 16 voire clang 17

### g++

Le support de C++20 par g++ est [détaillé ici](https://gcc.gnu.org/projects/cxx-status.html#cxx20) :

- la plupart des features sont accessibles dès la version 10, mais il y a un petit nombre accessibles uniquement en 11

### commun

La [page officielle du C++20](https://en.cppreference.com/w/cpp/20) indique également le support par les compilateurs :

- côté g++ 11, tout a l'air supporté pour le core-language, et la stdlib est très majoritairement supportée (exception notable = text formatting)
- côté clang 14, le core-language est majoritairement supporté (les coroutines ne sont indiquées en partial que pour windows : elles sont pleinement supportées sous linux), mais la stdlib est moins bien supportée que g++

CONCLUSION : g++11 ne supporte pas C++20 entièrement, mais c'est sans doute largement utilisable en pratique.

## features non-supportées

Aucun compilo sous linux ne semble supporter les modules pour le moment.

Aucun compilo de ubuntu:22.04 (g++11 / clang++14) ne supporte le header `<format>`, mais à la différence des modules, des compilos un poil plus récents le supportent ; de plus il reste toujours la possibilité d'utiliser la lib `{fmt}`.

Le reste du support de C++20 semble ok.

## support par les outils

**cmake**

- https://cmake.org/cmake/help/latest/prop_tgt/CXX_STANDARD.html
- le support de C++20 a été ajouté avec cmake 3.12 (et C++23 en cmake 3.20)
- la version de cmake sur ubuntu:20.04 est déjà la 3.16, et en ubuntu:22.04, on a la 3.22
- cmake ne posera donc pas souci pour l'adoption du C++20 (ni en 20.04 ni en 22.04)

**clang-format**

- en 20.04, clang-format est en version 10 (cohérent avec le compilo)
- en 22.04, clang-format est en version 14 (cohérent avec le compilo) :
- (mais si besoin, on peut apt-installer des versions de clang-format plus récentes que la version du compilo)

**clang-tidy**

- ubuntu 20.04, clang-tidy est en version 10 (cohérent avec le compilo)
- ubuntu 22.04, clang-tidy est en version 14 (cohérent avec le compilo)


**cppcheck**

- ubuntu 20.04 = Cppcheck 1.90
- ubuntu 22.04 = Cppcheck 2.7

La [page des releases](https://github.com/danmar/cppcheck/releases) de cppcheck indique le support :

> Cppcheck-2.5
> - checked that all features in c++11, c++14, c++17 are supported
> - c++20 support is improved but not complete yet

Pas d'indication dans une release plus récente que C++20 est pleinement supporté... Mais `cppcheck --help` indique que C++20 est le standard utilisé par défaut (pour les deux versions), donc ça devrait être ok.
