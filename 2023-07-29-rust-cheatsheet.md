# Rust Language Cheat Sheet

- **url** = https://cheats.rs/
- **type** = cheatsheet
- **auteur** = [Ralf BIEDERT](https://github.com/ralfbiedert), principal engineer qui fait de l'eye-tracking
- **date de publication** = N/A (le contenu est mis à jour régulièrement)
- **source** = N/A (la page est autoporteuse)
- **tags** = language>rust ; topic>language ; level>beginner

TL;DR = une cheatsheet très (trop ?) complète sur rust, très bonne ressource pour un débutant éclairé tel que moi, pour orienter mes recherches.

* [Rust Language Cheat Sheet](#rust-language-cheat-sheet)
   * [Structures et syntaxes de base](#structures-et-syntaxes-de-base)
   * [Macros](#macros)
   * [Pattern-matching et destructuring](#pattern-matching-et-destructuring)
   * [Trait bound](#trait-bound)
   * [Memory &amp; Lifetimes](#memory--lifetimes)
   * [Memory Layout](#memory-layout)
   * [Standard Library](#standard-library)
      * [Thread Safety](#thread-safety)
      * [Iterators](#iterators)
      * [String conversion](#string-conversion)
   * [Tooling](#tooling)
   * [Working with Types](#working-with-types)
   * [Coding Guides](#coding-guides)
      * [Async-Await 101](#async-await-101)
   * [Unsafe, Unsound, Undefined](#unsafe-unsound-undefined)
   * [Links &amp; Services](#links--services)

## Structures et syntaxes de base

Différence :

- `S(x)` = instanciation d'une tuple-struct à un unique élément
- `S{x}` = instanciation d'une struct classique qui a un field `x`, en lui passant le contenu de la variable locale `x`

Les ranges avec `a..b`

> References & Pointers = Granting access to un-owned memory.

Une référence est un type, un borrow est une variable de type "référence".

Pour utiliser une référence, il faut la déréferencer.

> Trait = common behavior types can adhere to.

```rs
impl S {}
// ^ Implementation of functionality for a type S, e.g., methods.

impl T for S {}
// ^ Implement trait T for type S; specifies how exactly S acts like T.

fn f() {}
// ^ Definition of a function;  or associated function if inside impl.
// fonction dans un bloc impl = associated function (un peu l'équivalent des méthodes statiques)

fn f(&self) {}
// ^ Define a method
```

Quand on utilise le pattern newtype, on gagne un constructeur automatique depuis le type wrappé vers le newtype :

```rs
struct S(T);
```

rust a une structure de contrôle plus rare, qui peut parfois être plus expressive :

```rs
loop {}
```

## Macros

> Macros & Attributes = Code generation constructs expanded before the actual compilation happens.

^ comme en C(++) , les macros sont une forme de preprocessor = elles _génèrent_ du code.

> #[attr] Outer attribute, annotating the following item.
>
> #![attr] Inner attribute, annotating the upper, surrounding item.

^ `Outer` car l'attribut est placé à l'extérieur de l'item annoté.

## Pattern-matching et destructuring


```rs
let S(x) = get();     // Notably, let also destructures
let S { x } = s;      // Only x will be bound to value s.x.
let (_, b, _) = abc;  // Only b will be bound to value abc.1.
let (a, ..) = abc;
```

^ Quelques exemples de destructuring, c'est très puissant et expressif !

```rs
if let Some(x) = get() {}  // Branch if pattern can be assigned (e.g., enum variant)

fn f(S { x }: S)           //Function parameters also work like let, here x bound to s.x of f(s).

E::A => {}         // Match enum variant A, c. pattern matching.
E::B ( .. ) => {}  // Match enum tuple variant B, ignoring any index.
E::C { .. } => {}  // Match enum struct variant C, ignoring any field.
```

^ quelques exemples de pattern-matching utiles.

```rs
0 | 1 => {}        // Pattern alternatives, or-patterns
Some(A | B) => {}  // Same, can also match alternatives deeply nested.
```

^ or-pattern pour matcher plusieurs trucs.

```rs
// Pathological or-pattern, leading | ignored, is just x | x, therefore x.
|x| x => {}
```

^ attention, ça ressemble à une closure, mais ce n'en est pas une !

## Trait bound

```rs
// Trait bound, limits allowed T, guarantees T has R; R must be trait :
S<T> where T: R
```

^ un **trait bound** est l'expression d'une contrainte sur un type générique.

## Memory & Lifetimes

https://cheats.rs/#memory-lifetimes

Très bon chapitre : les schémas aident à se faire rapidement un modèle mental de ce qui se passe → ne pas hésiter à revenir les consulter.

```rs
let t = S(1);
// Reserves memory location with name t of type S and the value S(1) stored inside.
// If declared with let that location lives on stack
```

^ une variable `let` est toujours sur la stack.

> Note the linguistic ambiguity,2 in the term variable, it can mean the:
>
> - name of the location in the source file ("rename that variable"),
> - location in a compiled app, 0x7 ("tell me the address of that variable"),
> - value contained within, S(1) ("increment that variable").

^ une "variable" peut représenter plusieurs concepts :

- une zone mémoire
- la valeur qu'elle contient
- ce par quoi on manipule les deux précédents concepts dans le code source

```rs
let a = t;
// This will move value within t to location of a, or copy it, if S is Copy
```

^ les schémas montrent que `let a = t;` est un move, ce qui transfère de contenu de la mémoire d'un emplacement à un autre (toujours sur la stack, a priori), sauf si le type est `Copy`.


> `let c: S = M::new();`
>
> The type of a variable serves multiple important purposes, it:
>
> - dictates how the underlying bits are to be interpreted,
> - allows only well-defined operations on these bits
> - prevents random other values or bits from being written to that location

^ la façon dont on interprète des bits en mémoire est liée au type de la variable.


```rs
fn f(x: S) { … }

let a = S(1); // <- We are here
f(a);
```

> When a function is called, memory for parameters (and return values) are reserved on stack.
>
> Here before f is invoked value in a is moved to 'agreed upon' location on stack, and during f works like 'local variable' x.

^ ça confirme ce que je dis plus haut = à moins d'utiliser une référence, les arguments d'une fonction sont moved dans la callstack de la fonction, d'un emplacement de la stack à un autre emplacement de la stack.

```rs
let a = S(1);
let r: &S = &a;
```

> A reference type such as &S or &mut S can hold the location of some s.
>
> Here type &S, bound as name r, holds location of variable a (0x3), that must be type S, obtained via &a.

^ les schémas aident à se faire un modèle mental : les références sont des pointeurs vers d'autres variables.

```rs
let mut a = S(1);
let r = &mut a;
let d = r.clone();  // Valid to clone (or copy) from r-target.
*r = S(2);          // Valid to set new S value to r-target.
```

> References can read from (&S) and also write to (&mut S) location they point to. \
> The dereference `*r` means to neither use the location of or value within r, but the location r points to. \
> In example above, clone d is created from `*r`, and S(2) written to `*r`. \
> Method Clone::clone(&T) expects a reference itself, which is why we can use r, not `*r`.

^ si la variable qui la contient est mutable, on peut réassigner une référence (à la différence du C++). Par ailleurs, en clonant une référence (appel explicite de `.clone()`, c'est bien la valeur référencée qu'on clone.

```rs
let mut a = …;
let r = &mut a;
let d = *r;       // Invalid to move out value, `a` would be empty.
*r = M::new();    // invalid to store non S value, doesn't make sense.
```

> While bindings guarantee to always hold valid data, references guarantee to always point to valid data. \
> Esp. &mut T must provide same guarantees as variables, and some more as they can't dissolve the target: \
> They do not allow writing invalid data. \
> They do not allow moving out data (would leave target empty w/o owner knowing).

^ les références sont safe vis-à-vis de la valeur qu'elles pointent, notamment on ne peut pas move ce qui est pointé par la référence (sans quoi l'owner se ferait "piquer" sa valeur), d'où le nom "borrow" = les références sont non-owning  (cf. aussi mes notes rust où j'ai annoté un article lié à ce sujet).


> Assume you got a r: &'c S from somewhere it means:
>
> - r holds an address of some S,
> - any address r points to must and will exist for at least 'c,
> - the variable r itself cannot live longer than 'c.

^ résumé de la signification de la **lifetime** d'une référence.

> Once the address of a variable is taken via &b or &mut b the variable is marked as borrowed.
> While borrowed, the content of the address cannot be modified anymore via original binding b.
> Once address taken via &b or &mut b stops being used (in terms of LOC) original binding b works again.

^ fonctionnement du **borrowing**

## Memory Layout

https://cheats.rs/#memory-layout

Représentation en mémoire des différents types.

Les `char` stockent grosso-modo un point de code unicode sur 4 bytes (donc ils sont **différents** des char en C/C++ qui stockent un byte)

Les `str` ne sont **PAS** des tableaux de char, mais des tableaux de bytes, stockant l'encodage UTF-8 d'une chaîne unicode (= l'encodage UTF-8 d'une liste de code-points).

> Many reference and pointer types can carry an extra field, pointer metadata. It can be the element- or byte-length of the target, or a pointer to a vtable. Pointers with meta are called fat, otherwise thin.

^ certains pointeurs peuvent stocker des métadonnées.

## Standard Library

https://cheats.rs/#standard-library

### Thread Safety

https://cheats.rs/#thread-safety

> Assume you hold some variables in Thread 1, and want to either move them to Thread 2, or pass their references to Thread 3. Whether this is allowed is governed by Send and Sync respectively:

^ `Send` définit si on peut move une valeur d'un thread à un autre ; `Sync` définit si on peut passer une référence vers une valeur d'un thread à un autre.

### Iterators

https://cheats.rs/#iterators

La cheatsheet indique quand utiliser un style fonctionnel pour les iterators, et quand faire de l'impératif :

> - Imperative, useful when side effects, interdependencies, or need to break flow early.
> - Functional, often much cleaner when only results of interest.

Donc en gros, si seul le résultat d'un traitement (simple) appliqué à tout l'iterable nous intéresse, le style fonctionnel est bien adapté ; sinon, ne pas hésiter à utiliser un style impératif.

> Assume you have a collection c of type C you want to use:
>
> `c.into_iter()` — Turns collection c into an Iterator i and consumes c. Standard way to get iterator.
> `c.iter()` — Courtesy method some collections provide, returns borrowing Iterator, doesn't consume c.
> `c.iter_mut()` — Same, but mutably borrowing Iterator that allow collection to be changed.

^ la méthode standard pour itérer sur une collection est `into_iter` et non `iter` !

Par ailleurs, avec `into_iter`, on "consomme" la collection, ce qui veut dire (je suppose) que son contenu est moved vers la suite du pipeline.

**EDIT** : c'est un peu plus nuancé : à la base, quand on parle de "consommer", on parle de consommer l'itérateur = ça veut dire qu'on modifie l'état interne de l'itérateur pour le faire avancer, cf. [le rustbook](https://doc.rust-lang.org/book/ch13-02-iterators.html) :

> Note that we needed to make v1_iter mutable: calling the next method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. In other words, this code consumes, or uses up, the iterator. Each call to next eats up an item from the iterator. (...)

Mais en pratique, lorsqu'on utilise `into_iter`, la collection est **moved** dans l'itérateur, qui retourne donc des owned-values :

> Also note that the values we get from the calls to next are immutable references to the values in the vector. The iter method produces an iterator over immutable references. If we want to create an iterator that takes ownership of v1 and returns owned values, we can call into_iter instead of iter. Similarly, if we want to iterate over mutable references, we can call iter_mut instead of iter.

Le `into` du nom `into_iter` est donc à interpréter comme "move cette collection vers l'itérateur".

```rs
for x in c {}
// ^ Syntactic sugar, calls c.into_iter() and loops i until None.
```

^ fonctionnement interne des boucles for = into_iter

La cheatsheet explique comment itérer sur ses propres structs : `IntoIter` et `IntoIterator` (plus d'autres)

### String conversion

https://cheats.rs/#string-conversions

Il y a tout un (long) paragraphe sur les conversions de strings.

## Tooling

https://cheats.rs/#tooling

Il y a un paragraphe qui décrit le layout d'un projet (je n'annote pas car j'en connais déjà l'essentiel)

Clippy et fmt sont des composants optionnels de rustup.

Il y a un résumé sur comment cross-compiler.

## Working with Types

https://cheats.rs/#working-with-types

Notamment une grosse synthèse sur les conversions de type.

## Coding Guides

https://cheats.rs/#coding-guides

Le paragraphe "Idiomatic Rust" présente [un tableau](https://cheats.rs/#idiomatic-rust) **très très** intéressant, plutôt que de le copier-coller inégralement, je me contente de lister ses titres, mais ne pas hésiter à retourner le consulter :

- Think in Expressions
- Think in Iterators
- Handle Absence with ?
- Use Strong Types
- Illegal State: Impossible
- Provide Builders
- Don't Panic
- Generics in Moderation
- Split Implementations
- Unsafe
- Implement Traits
- Tooling
- Documentation

### Async-Await 101

https://cheats.rs/#async-await-101

L'asynchrone fonctionne de façon similaire aux autres langages, avec des nommages un brin différent :

> Anything declared async always returns an `impl Future<Output=_>`. \
> (...) \
> `let sm = f();` \
> Calling `f()` that is async will not execute `f`, but produce state machine `sm`.

On intègre l'exécution d'une fonction asynchrone dans un code synchrone avec `runtime.block_on(sm);` où `runtime` provient d'une crate externe, telle que [tokio](https://tokio.rs/), un peu un équivalent d'asyncio.

La syntaxe pour qu'une fonction asynchrone await une autre fonction asynchrone est un peu inhabituelle, postfixée : `sm.await`

## Unsafe, Unsound, Undefined

https://cheats.rs/#unsafe-unsound-undefined

Attention que du code **safe** en rust peut quand même faire des trucs très dangereux (comme effacer une database) : en rust, la signification de safe est **pas d'UB** (et c'est déjà pas mal).

Le code unsafe a des permissions spéciales, et le dev s'engage à rempir lui même les préconditions qui feront que le code unsafe ne produira pas d'UB.

**Unsound code** = code "safe" mais qui peut produire de l'UB, p.ex. wrapper dans une fonction safe un appel à `mem::transmute` ([un équivalent de reinterpret_cast](https://doc.rust-lang.org/std/mem/fn.transmute.html)) sans faire les vérifications des préconditions nécessaires pour éviter que cette utilisation de `mem::transmute` conduise à un UB.

## Links & Services

https://cheats.rs/#links-services

Il y a une liste longue comme le bras de ressources pour apprendre différents aspects de rust !

Plein plein de trucs ont l'air intéressants, ne pas hésiter à y revenir !
