Notes sur les différents **storages** (notamment le storage `static` et la façon d'initialiser les variables statiques) et les différents **linkages**.

Une variable peut exister dans trois types de storages (mais je n'annote que `static`) :

- `stack`
- `heap`
- `static`

À noter que les deux sujets sont ORTHOGONAUX (le linkage n'a RIEN À VOIR avec le storage) mais ils sont influencés en partie par les mêmes keywords : déclarer une variable `static` peut changer son storage et son linkage :

- déclarer `static` une variable globale dans un cpp va modifier son linkage (qui va passer de `external` à `internal`) sans modifer son storage qui sera `static` dans tous les cas
- déclarer `static` une variable locale d'une fonction va modifier son storage (qui va passer de `stack` à `static`) sans modifier son linkage, qui sera `no linkage` dans tous les cas

* [Storage](#storage)
   * [Constant initialization](#constant-initialization)
   * [Dynamic initialization](#dynamic-initialization)
   * [Que préférer entre constant et dynamic initialization des variables statiques](#que-préférer-entre-constant-et-dynamic-initialization-des-variables-statiques)
   * [Cas particulier des variables statiques dans des méthodes ou fonctions](#cas-particulier-des-variables-statiques-dans-des-méthodes-ou-fonctions)
   * [Static Initialization Order Fiasco](#static-initialization-order-fiasco)
* [Linkage](#linkage)
   * [Les trois différents linkage](#les-trois-différents-linkage)
   * [Impact de static sur une variable globale](#impact-de-static-sur-une-variable-globale)
   * [Impact de extern sur une variable globale](#impact-de-extern-sur-une-variable-globale)
   * [Bonnes pratiques autour de internal linkage](#bonnes-pratiques-autour-de-internal-linkage)
      * [Pourquoi préférer internal linkage ?](#pourquoi-préférer-internal-linkage-)
      * [Namespace anonyme](#namespace-anonyme)
      * [Testing des variables et fonctions avec linkage internal](#testing-des-variables-et-fonctions-avec-linkage-internal)



# Storage static

[Ce post](https://pabloariasal.github.io/2020/01/02/static-variable-initialization/) est une très bonne synthèse du sujet, à la fois claire et concise.

En résumé, les variables avec le storage `static` sont stockées dans une zone mémoire du process qui existe avant même le début du programme ; **MAIS** cette zone mémoire n'est pas nécessairement correctement initialisée dès le chargement du programme : elle peut être remplie de zéros, auquel cas il y aura une phase d'initialisation dynamique au runtime.

Dit autrement : même s'il n'y a aucune allocation nécessaire au runtime pour les variables statiques (vu que leur zone mémoire est DÉJÀ créée lors du mapping du process en mémoire), la variable n'est pas pour autant utilisable directement, car il peut y avoir un besoin d'initialization de cette zone mémoire, qui aura lieu juste avant le `main`.

Il existe deux façons d'initialiser une variable avec un storage `static` :

- au compile-time (const)
- au runtime (dynamic)

## Constant initialization

S'il le peut, le compilo attribuera directement leur valeur "définitive" (hors mutation ultérieure) à une variable statique. C'est la **const-initialization** :

```cpp
struct MyStruct
{
    static int pouet;
};
int MyStruct::pouet = 67;
```

^ ici, l'entier `pouet` a sa valeur définitive `67` dès la compilation du programme (il est stocké dans la zone `initialized data segment` = `data segment` de la mémoire du process, qui est lue directement depuis le fichier binaire).

## Dynamic initialization

Si la _constant initialization_ n'est pas possible, alors le compiler initialize la mémoire statique avec des **ZÉROS** ; et une phase d'initialization dynamique a lieu au runtime, pour l'initializer avec sa valeur définitive.


```cpp
static std::string pouet{"coucou"};
```

^ ici, la zone mémoire de la string `pouet` (attention, on parle du pointeur vers le buffer de `char`, ainsi que la taille et la capacité : on ne parle PAS du buffer lui-même), est initialisée avec des zéros.

Ça n'est qu'au runtime, juste avant l'exécution du main, que le constructeur de string est appelé dynamiquement pour initialiser la zone mémoire.

Ma compréhension des choses, c'est qu'on peut exécuter du code arbitraire avant même que la fonction `main` ne soit appelée, via les constructeurs des objets statiques... Ici par exemple, le constructeur de `std::string` a fait une allocation de son buffer. De façon contre-intuitive, au moment où le `main` est exécuté, le heap n'est pas vide !

## Que préférer entre constant et dynamic initialization des variables statiques

D'une façon générale, il faut préférer la **constant initialization** des variables statiques si on le peut :

- gain de temps : au lieu de passer par deux étapes (zero-initialization + construction de l'objet statique), la mémoire est simplement `memcpy` depuis le binaire du process vers le static-storage
- pas d'exécution de code arbitraire au runtime
- pas de initialization order fiasco possible (cf. plus bas)

Si on utilise `constexpr` (ou `constinit` pour initialiser des variables globales qui doivent être non-const), alors on est certain d'utiliser la const-initialization.

Mais ça n'est pas toujours possible ; par exemple si une variable statique globale doit posséder une valeur random, on va vouloir que cette valeur change à chaque exécution, et celle-ci doit donc être initialisée au lancement du programme, et non à la compilation.

## Cas particulier des variables statiques dans des méthodes ou fonctions

Les variables statiques qui sont locales à une fonction sont des variables statiques comme les autres (leur storage est bien `static` : elles ne sont ni sur le heap ni sur la stack).

Mais elles sont initialisées d'une façon particulières = au premier appel de la fonction.

(et elles sont réutilisables d'un appel à un autre, mais c'est pas le sujet du jour)

Ainsi, avec le code suivant :

```cpp
struct Pouet {
    Pouet() { std::cout << "CONSTRUCTION" << std::endl; }
    int val = 42;
};

void do_something() {
    static Pouet pouet;
    //           ^ c'est de cette variable statique dont on parle ici
    std::cout << "val = " << pouet.val++ << std::endl;
}

int main void() {
    do_something();
    do_something();
    do_something();
    return 0;
}
```

Alors la variable `pouet` sera bien utilisable d'un appel à l'autre (l'exécution du programme écrit `42... 43... 44`), mais la construction de la variable (et l'affichage de `CONSTRUCTION` sur cout) n'est faite QU'UNE SEULE FOIS, même si elle est dans le corps d'une fonction appelée trois fois !

## Static Initialization Order Fiasco

**Au sein d'un même fichier cpp**, les variables statiques qui n'ont pas été const-initializées seront initialisées dans l'ordre de leur déclaration, ça ça marche bien :+1:

**MAIS** l'ordre d'initialization respectif des variables statiques de DEUX fichiers cpp est aléatoire ! :-1:

Du coup, si une variable statique dépend d'une autre variable statique dans un autre cpp, c'est cuit (pire : parfois, le problème sera invisible pendant longtemps avant de planter). Exemple :

```cpp
// dans le fichier a.cpp
sdt::string pouet_a = "Voici une super variable statique";

// dans le fichier b.cpp :
std::string pouet_b = pouet_a;  // pouet_b dépend de pouet_a
```

À noter : ceci ne concerne QUE les variables dynamic-initialized (vu que les variables const-initialized sont **déjà** initialisées au moment où le programme démarre), raison supplémentaire de préférer la const-initialization à la dynamic-initialization pour les variables statiques.

La solution à préférer est de refactorer son programme pour 1. ne pas utiliser de variables globales ! ou sinon 2. ne pas faire dépendre les variables statiques les unes des autres. Si ça n'est pas possible, on peut contourner le problème en s'appuyant sur le fait que les variables statiques dans des fonctions sont initialisées de façon déterministe = au premier appel de la fonction :


```cpp
// dans le fichier a.cpp
std::string& get_pouet_a()
{
    // pouet_a ne sera initialisée que la première fois que `get_pouet_a` sera appelée :
    static std::string pouet_a = a_complicated_initialization_possibly_using_random_value();
    return pouet_a;
}

// dans le fichier b.cpp :
std::string pouet_b = get_pouet_a();
```

^ ceci marche, même si les variables statiques de `b.cpp` sont initialisées en premier :

- on essaye de construire `pouet_b`, ce qui appelle `get_pouet_a()`
- on passe par le corps de `get_pouet_a` pour la première fois, ce qui initialise `pouet_a`
- `get_pouet_a` renvoie une référence sur `pouet_a`, qui sert à initialiser `pouet_b`

# Linkage

Dans les grandes lignes, le linkage indique si une variable définie dans une translation-unit est visible ou non depuis l'extérieur de cette translation-unit.

## Les trois différents linkage

On a trois niveaux de visibilité :

- `no linkage` : la variable n'est visible que dans son scope :
    - exemple : une variable locale d'une fonction
- `internal linkage` : la variable n'est visible que dans la translation-unit dans laquelle elle est définie :
    - exemple : une variable globale marquée `static` en haut d'un fichier cpp
    - comme les variables ne sont pas visibles par les autres translation-units, elles ont le droit de s'appeler pareil :
    ```cpp
    // dans fichier1.cpp :
    static int pouet;

    // dans fichier2.cpp :
    static int pouet;
    ```
    - ^ ceci compile et linke sans souci (vu que les variables sont "privées" au cpp qui les définit)
- `external linkage` : la variable est visible par d'autres translation-unit :
    - dit autrement, _la variable est visible par TOUT le programme_
    - exemple : une variable globale qui n'est pas `static` en haut d'un fichier cpp
    - comme les variables sont visibles par les autres translation-units, elles doivent s'appeler différemment :
    ```cpp
    // dans fichier1.cpp :
    int pouet;

    // dans fichier2.cpp :
    int pouet;
    ```
    - ^ ceci compile (car individuellement, il n'y a pas de souci avec chaque translation-unit)...
    - ...mais ne linke pas : la One Definition Rule est violée !

## Impact de static sur une variable globale

À noter que de façon contre-intuitive, pour une variable globale dans un fichier cpp (i.e. définie en dehors d'une fonction ou d'une classe, mais éventuellemnt dans un namespace) est statique de toutes façons : le mot-clé `static` ne change que son linkage :

```cpp
static int ma_variable_globale = 42;  // INTERNAL linkage + STATIC storage
int ma_variable_globale = 42;         // EXTERNAL linkage + STATIC storage aussi !
```

## Impact de extern sur une variable globale

Et si on veut utiliser dans un fichier cpp une variable globale d'un AUTRE fichier cpp (qui doit donc avoir le linkage external), on utilise `extern` :

```cpp
// dans fichier1.cpp :
int pouet = 42;  // EXTERNAL linkage + STATIC storage

// dans fichier2.cpp :
extern int pouet;  // simple déclaration, on utilise en réalité la variable du fichier1

void ma_fonction() {
    std::cout << "La valeur de pouet est : " << pouet << std::endl;
}
```

Dit autrement, les deux instructions suivantes sont très différentes :

- `int pouet;` :
    - non-seulement ça déclare l'identifier `pouet` (i.e. la suite du programme sait que `pouet` est un entier, et peut l'utiliser comme tel)...
    - ... mais également, ça alloue de la mémoire (`stack` ou `static`, selon que la déclaration `int pouet;` soit faite dans le corps d'une fonction ou en global au fichier cpp)
- `extern int pouet;` :
    - ici aussi, ça déclare l'identifier `pouet`
    - MAIS ça n'alloue aucune mémoire : l'identifier `pouet` référence un entier qui existe dans une autre translation-unit
    - (et le linker échouera si jamais il n'existe pas d'autre translation-unit qui définit un entier appelé `pouet`, avec un external linkage)


Dit autrement, avec `extern`, on se contente de **déclarer** une variable plutôt que de la définir. C'est un peu plus visible avec une variable plus complexe qu'un int, telle qu'une struct :

```cpp
// dans mylib.h :
struct Pouet {
  int val;
  Pouet(int val_) : val{val_} {}
  // note qu'il n'existe pas de constructeur sans argument !
};


// dans mylib.cpp :
#include "f.h"
Pouet pouet{42}; // variable globale, qui fait appel au constructeur


// dans main.cpp :
#include <iostream>

#include "f.h"

// Pouet pouet;  // ne compile pas (car pas de constructeur adéquat)
extern Pouet pouet;  // compile (car c'est une simple déclaration, pas une définition)

int main() {
    std::cout << "La valeur de pouet est : " << pouet.val << std::endl;
    return 0;
}
```


## Bonnes pratiques autour de internal linkage

### Pourquoi préférer internal linkage ?

1. pour éviter que quelqu'un d'externe utilise par erreur la variable
2. pour éviter les conflits de noms accidentels
3. pour véhiculer de façon explicite l'intention du programmeur :
    > _cette fonction n'est utile que LOCALEMENT, dans CE cpp : c'est un détail d'implémentation privé_
4. d'une façon générale, c'est une bonne pratique de ne rendre public que ce qui doit l'être

### Namespace anonyme

Pour donner un internal linkage à quelque chose, la pratique moderne est plutôt de préférer les namespaces anonymes au mot-clé `static` :

```cpp

// old :
static int myvar = 42;

// modern :
namespace {
    int myvar = 42;
}
```

En effet, ça joue le même rôle (car le namespace anonyme n'est accessible que depuis le cpp dans lequel il est défini), mais ça permet d'appliquer l'internal linkage à plus de choses, p.ex. les `using XXX = YYY;`;

### Testing des variables et fonctions avec linkage internal

Il y a conflit entre deux potentiels opposés :

- avoir un linkage `internal` pour une fonction/variable, car c'est un détail d'implémentation privé au cpp
- avoir un linkage `external` pour une fonction/variable, car on veut la tester en y accédant depuis un test unitaire (défini dans un autre cpp)

On peut jongler pour contourner ce conflig, j'en ai parlé un peu [dans ces notes](./testing_private_things.md).

(à noter que rust a une approche différente, où la bonne pratique est de définir le test dans le module)
