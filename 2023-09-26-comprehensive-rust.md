# Comprehensive Rust

- **url** = https://google.github.io/comprehensive-rust/index.html
- **type** = [mdBook](https://github.com/rust-lang/mdBook)
- **auteur** = google
- **date de publication** = ????-??-??
- **source** = [github de google](https://github.com/google)
- **tags** = language>rust ; topic>all ; level>beginner

**TL;DR** = une formation sur trois jours de google, destinée à un public de personnes qui sont **déjà** devs ; l'une des meilleures ressources pour apprendre le rust quand on a mon profil (et notamment quand on a fait un peu de C/C++).

Je n'annote que par ci par là ; les parties les plus utiles à mon niveau de rust au moment où j'ai lu le tuto sont l'après-midi du second jour sur la stdlib et le matin du troisième jour sur les traits.

Il y a des speaker-notes = des suggestions de choses à dire à l'oral par le formateur → à lire absolument, il y a plein d'infos utiles.

En plus de la formation de base sur trois jours, il y a des formations complémentaires :

- [android](https://google.github.io/comprehensive-rust/android.html) (sur une demi-journée)
- [bare-metal](https://google.github.io/comprehensive-rust/bare-metal.html) = embedded (sur une journée)
- [concurrency](https://google.github.io/comprehensive-rust/concurrency.html) (sur une journée)

----

* [Comprehensive Rust](#comprehensive-rust)
* [4. Hello World!](#4-hello-world)
* [5. Why rust ?](#5-why-rust-)
   * [5.4 Modern Features](#54-modern-features)
* [6. Basic Syntax](#6-basic-syntax)
   * [6.3 References](#63-references)
   * [6.4 Slices](#64-slices)
   * [6.5. Functions](#65-functions)
      * [6.5.1. Rustdoc](#651-rustdoc)
      * [6.5.2. Methods](#652-methods)
      * [6.5.3. Overloading](#653-overloading)
* [8. Control Flow](#8-control-flow)
   * [8.1. Blocks](#81-blocks)
   * [8.3. for expressions](#83-for-expressions)
* [9. Variables](#9-variables)
   * [9.2. static &amp; const](#92-static--const)
* [10. Enums](#10-enums)
   * [10.1. Variant Payloads](#101-variant-payloads)
   * [11.1. if let expressions](#111-if-let-expressions)
* [13. Exercises](#13-exercises)
* [15. Memory Management](#15-memory-management)
   * [15.2. Stack Memory](#152-stack-memory)
* [16. Ownership](#16-ownership)
   * [16.1. Move Semantics](#161-move-semantics)
   * [16.2. Moved Strings in Rust](#162-moved-strings-in-rust)
      * [16.2.1. Double Frees in Modern C++](#1621-double-frees-in-modern-c)
   * [16.3. Moves in Function Calls](#163-moves-in-function-calls)
   * [16.4. Copying and Cloning](#164-copying-and-cloning)
   * [16.5. Borrowing](#165-borrowing)
   * [16.6. Lifetimes](#166-lifetimes)
   * [16.7. Lifetimes in Function Calls](#167-lifetimes-in-function-calls)
   * [16.8. Lifetimes in Data Structures](#168-lifetimes-in-data-structures)
* [17. Structs](#17-structs)
   * [17.1. Tuple Structs](#171-tuple-structs)
* [18. Methods](#18-methods)
   * [18.1. Method Receiver](#181-method-receiver)
* [19. Exercises](#19-exercises)
   * [20. Standard Library](#20-standard-library)
   * [20.1. Option and Result](#201-option-and-result)
   * [20.2. String](#202-string)
   * [20.3. Vec](#203-vec)
   * [20.4. HashMap](#204-hashmap)
   * [20.5. Box](#205-box)
      * [20.5.1. Recursive Data Types](#2051-recursive-data-types)
   * [20.6. Rc](#206-rc)
   * [20.7. Cell/RefCell](#207-cellrefcell)
* [21. Modules](#21-modules)
   * [21.1. Visibility](#211-visibility)
   * [21.3. Filesystem Hierarchy](#213-filesystem-hierarchy)
* [22. Exercises](#22-exercises)
   * [22.1. Iterators and Ownership](#221-iterators-and-ownership)
* [25. Traits](#25-traits)
   * [25.1. Trait Objects](#251-trait-objects)
   * [25.2. Deriving Traits](#252-deriving-traits)
   * [25.4. Trait Bounds](#254-trait-bounds)
   * [25.5. impl Trait](#255-impl-trait)
* [26. Important Traits](#26-important-traits)
   * [26.1. Iterator](#261-iterator)
   * [26.2. FromIterator](#262-fromiterator)
   * [26.3. From and Into](#263-from-and-into)
   * [26.4. Read and Write](#264-read-and-write)
   * [26.5. Drop](#265-drop)
   * [26.6. Default](#266-default)
   * [26.7. Operators: Add, Mul, ...](#267-operators-add-mul-)
   * [26.8. Closures: Fn, FnMut, FnOnce](#268-closures-fn-fnmut-fnonce)
* [27. Exercises](#27-exercises)
   * [27.2. Points and Polygons](#272-points-and-polygons)
* [28. Error Handling](#28-error-handling)
   * [28.1. Panics](#281-panics)
   * [28.2. Structured Error Handling](#282-structured-error-handling)
   * [28.3. Propagating Errors with ?](#283-propagating-errors-with-)
      * [28.3.1. Converting Error Types](#2831-converting-error-types)
      * [28.3.2. Deriving Error Enums](#2832-deriving-error-enums)
      * [28.3.3. Dynamic Error Types](#2833-dynamic-error-types)
      * [28.3.4. Adding Context to Errors](#2834-adding-context-to-errors)
* [29. Testing](#29-testing)
   * [29.3. Documentation Tests](#293-documentation-tests)
   * [29.5. Useful crates](#295-useful-crates)
* [30. Unsafe Rust](#30-unsafe-rust)
   * [30.1. Dereferencing Raw Pointers](#301-dereferencing-raw-pointers)
   * [30.2. Mutable Static Variables](#302-mutable-static-variables)
   * [30.3. Unions](#303-unions)
   * [30.4. Calling Unsafe Functions](#304-calling-unsafe-functions)
   * [30.5. Implementing Unsafe Traits](#305-implementing-unsafe-traits)
* [31. Exercises](#31-exercises)
   * [31.1. Safe FFI Wrapper](#311-safe-ffi-wrapper)


# 4. Hello World!

https://google.github.io/comprehensive-rust/hello-world.html


> Rust uses macros for situations where you want to have a variable number of arguments (no function overloading).

^ si on veut un nombre variable d'arguments (comme pour `pritnln!`), les macros sont obligatoires.

> Macros being ‘hygienic’ means they don’t accidentally capture identifiers from the scope they are used in. Rust macros are actually only partially hygienic.

^ meilleures que les macros C.

# 5. Why rust ?

https://google.github.io/comprehensive-rust/why-rust.html

## 5.4 Modern Features

https://google.github.io/comprehensive-rust/why-rust/modern.html

^ plein d'arguments sur "rust est un langage moderne"

# 6. Basic Syntax

## 6.3 References

https://google.github.io/comprehensive-rust/basic-syntax/references.html

> We must dereference ref_x when assigning to it, similar to C and C++ pointers.
>
> Rust will auto-dereference in some cases, in particular when invoking methods (try ref_x.count_ones()).

^ ce n'est que pour les appels de méthodes qu'on dispose du sucre syntaxique autorisant à ne pas déréférencer une référence.

Dit autrement, tout se passe comme si le rôle de l'opérateur `->` était joué par l'opérateur `.`.

> Be sure to note the difference between let mut ref_x: &i32 and let ref_x: &mut i32. The first one represents a mutable reference which can be bound to different values, while the second represents a reference to a mutable value.

^ une référence qu'on peut réassigner, mais toujours sur des entiers constats, sera `let mut ref_x: &i32` ; une référene non-réassignable, sur un entier mutable sera `let ref_x: &mut i32`.

## 6.4 Slices

https://google.github.io/comprehensive-rust/basic-syntax/slices.html

> Slices borrow data from the sliced type.
>
> We create a slice by borrowing a and specifying the starting and ending indexes in brackets.
>
> Slices always borrow from another object. In this example, a has to remain ‘alive’ (in scope) for at least as long as our slice.

^ les slices sont des borrows i.e. sont des références ! C'est juste qu'au lieu d'être une référence sur un seul objet, un slice est une réfénce sur une séquence d'objets consécutifs.

## 6.5. Functions

https://google.github.io/comprehensive-rust/basic-syntax/functions.html

### 6.5.1. Rustdoc

https://google.github.io/comprehensive-rust/basic-syntax/rustdoc.html

> It is idiomatic to document all public items in an API using this pattern.

^ les commentaires avec triples slashs `///` sont à placer à l'extérieur de ce qu'ils documentent, et sont à utiliser pour documenter l'API publique.

> Rustdoc comments can contain code snippets that we can run and test using cargo test.

^ on peut intégrer des doctests, ils seront automatiquement lancés avec `cargo test`.

### 6.5.2. Methods

https://google.github.io/comprehensive-rust/basic-syntax/methods.html

> While technically, Rust does not have custom constructors, static methods are commonly used to initialize structs (but don’t have to).

^ Des méthodes statiques jouent le rôle de constructeurs.

### 6.5.3. Overloading

https://google.github.io/comprehensive-rust/basic-syntax/functions-interlude.html

> Overloading is not supported:
>
> - Each function has a single implementation:
>     - Always takes a fixed number of parameters.
>     - Always takes a single set of parameter types.
> - Default values are not supported:
>     - All call sites have the same number of arguments.
>     - Macros are sometimes used as an alternative.

^ encore des points qui font que rust est un langage explicite. J'aime beaucoup car ça évite les surprises ou les arrachages de cheveux pour trouver d'où sort tel comportement.

# 8. Control Flow

https://google.github.io/comprehensive-rust/control-flow.html

## 8.1. Blocks

https://google.github.io/comprehensive-rust/control-flow/blocks.html

> A block in Rust contains a sequence of expressions. Each block has a value and a type, which are those of the last expression of the block: (...) If the last expression ends with ;, then the resulting value and type is ().

^ de façon un peu contre-intuitive vis-à-vis d'autres langages (mais très très logique quand on s'y est fait), les blocs sont typés et ont une valeur = celle de leur dernière expression !

## 8.3. for expressions

https://google.github.io/comprehensive-rust/control-flow/for-expressions.html

> The for loop (...) will automatically call into_iter() on the expression and then iterate over it:

^ pour utiliser une for-loop, c'est `into_iter()` dont on a besoin, et qui est appelé under-the-hood.

# 9. Variables

https://google.github.io/comprehensive-rust/basic-syntax/variables.html

## 9.2. static & const

https://google.github.io/comprehensive-rust/basic-syntax/static-and-const.html

`const` = la variable est inlinée par le compilo (elle n'a ps d'adresse mémoire propre, vu qu'elle n'existe pas pendant l'exécution du programme).

`static` = la variable est stockée dans une adresse mémoire, qui reste constante pendant toute la durée de vie du programme.

> When a globally-scoped value does not have a reason to need object identity, const is generally preferred.

^ guideline pour choisir entre `const` et `static`.

> Because static variables are accessible from any thread, they must be Sync. Interior mutability is possible through a Mutex, atomic or similar.

^ contraintes sur une variable statique.

> It is also possible to have mutable statics, but they require manual synchronisation so any access to them requires unsafe code

^ à moins de faire de l'unsafe rust et de gérer sa synchro soi même, les variables statiques sont immutables.

> Mention that const behaves semantically similar to C++’s constexpr.  static, on the other hand, is much more similar to a const or mutable global variable in C++.

^ parallèles avec la terminologie c++

# 10. Enums

https://google.github.io/comprehensive-rust/enums.html

## 10.1. Variant Payloads

https://google.github.io/comprehensive-rust/enums/variant-payloads.html

Annotation `#[rustfmt::skip]` pour ne pas formater une fonction (et garder un formatage manuel custom).

> match inspects a hidden discriminant field in the enum. (...) It is possible to retrieve the discriminant by calling std::mem::discriminant()

^ les enums ont un champ caché identifiant le variant ! Voir aussi [ceci](https://doc.rust-lang.org/std/mem/fn.discriminant.html) et [ceci](https://doc.rust-lang.org/reference/items/enumerations.html#custom-discriminant-values-for-fieldless-enumerations)

## 11.1. if let expressions

https://google.github.io/comprehensive-rust/control-flow/if-let-expressions.html

> Since 1.65, a similar let-else construct allows to do a destructuring assignment, or if it fails, execute a block which is required to abort normal control flow (with panic/return/break/continue):

^ pas sûr que ce soit très utile, mais au moins maintenant je connais la syntaxe si je tombe dessus : il existe un `else` au `if-let`.

# 13. Exercises

https://google.github.io/comprehensive-rust/exercises/day-1/afternoon.html

J'ai fait les deux exercices, bon ratio investissement/ROI.

# 15. Memory Management

https://google.github.io/comprehensive-rust/memory-management.html

## 15.2. Stack Memory

https://google.github.io/comprehensive-rust/memory-management/stack.html

Le buffer de la string est sur le heap, mais ses métadonnées (pointeur vers le buffer, length, capacité) sont sur la stack.

# 16. Ownership

https://google.github.io/comprehensive-rust/ownership.html

> At the end of the scope, the variable is dropped and the data is freed. (...) We say that the variable owns the value.

^ conceptuellement, il y a une différence entre la data (= la value) et la variable bindée à cette value.

## 16.1. Move Semantics

https://google.github.io/comprehensive-rust/ownership/move-semantics.html

> An assignment will transfer ownership between variables
>
> (...) There is always exactly one variable binding which owns a value.
>
> (...) this is the opposite of the defaults in C++, which copies by value unless you use std::move
>
> It is only the ownership that moves. Whether any machine code is generated to manipulate the data itself is a matter of optimization, and such copies are aggressively optimized away.

^ beaucoup de choses intéressantes ici :

- il n'y a jamais qu'un unique **binding** qui owne une value.
- ce n'est pas parce qu'on assigne `s1` à `s2` qu'il y a forcément des copies
- en effet, c'est l'ownership qui est transféré, et c'est un concept du compilateur plutôt que du code machine !

## 16.2. Moved Strings in Rust

https://google.github.io/comprehensive-rust/ownership/moved-strings-rust.html

### 16.2.1. Double Frees in Modern C++

https://google.github.io/comprehensive-rust/ownership/double-free-modern-cpp.html

En C++, par défaut, l'assignation copie (plutôt que de move, i.e. de transférer l'ownership) ; en rust, c'est le contraire.

> Unlike Rust, = in C++ can run arbitrary code as determined by the type which is being copied or moved.

^ apparemment, en rust `=` n'exécute jamais de code custom (pas de constructeur de copie ou de surcharge de l' `operator=`), ici aussi à la différence du C++.

## 16.3. Moves in Function Calls

https://google.github.io/comprehensive-rust/ownership/moves-function-calls.html

> When you pass a value to a function, the value is assigned to the function parameter. This transfers ownership

^ à moins qu'une fonction accepte explicitement une référence, le fait de passer une valeur à une fonction transfère l'ownership : par défaut, la fonction **consomme** l'argument.

> Rust makes it harder than C++ to inadvertently create copies by making move semantics the default, and by forcing programmers to make clones explicit.

^ conséquence = comme tout est move par défaut, les copies sont explicites, on ne risque pas de les rater.

https://google.github.io/comprehensive-rust/ownership/copy-clone.html

## 16.4. Copying and Cloning

> Copying and cloning are not the same thing
>
> - Copying refers to bitwise copies of memory regions and does not work on arbitrary objects.
> - Copying does not allow for custom logic (unlike copy constructors in C++).
> - Cloning is a more general operation and also allows for custom behavior by implementing the Clone trait.
> - Copying does not work on types that implement the Drop trait.

^ conceptuellement, il y a une différence entre l'action de copier et l'action de cloner :

- copier est une opération sur la mémoire, pas sur les objets (i.e. au mieux, on copie le layout mémoire d'un objet)
- cloner consiste à fabriqer un nouvel objet calqué sur le premier, ce qui autorise des actions particulières (e.g. enregistrer le nouveal objet dans un registry)

## 16.5. Borrowing

https://google.github.io/comprehensive-rust/ownership/borrowing.html

> Instead of transferring ownership when calling a function, you can let a function borrow the value

^ une fonction peut se contenter de borrow une value lorsqu'elle la prend par référence ; le point important, c'est que c'est explicite car pas le défaut.

> (...) The Rust compiler can do return value optimization (RVO).
>
> In C++, copy elision has to be defined in the language specification because constructors can have side effects. In Rust, this is not an issue at all.

^ tout comme en C++, il y a de la copy elision, mais celle-ci est rendue plus simple par le fait que les constructeurs sont gérés par le compilo, et n'exécutent donc pas de code custom.

## 16.6. Lifetimes

https://google.github.io/comprehensive-rust/ownership/lifetimes.html

> A borrowed value has a lifetime (...) Read &'a Point as “a borrowed Point which is valid for at least the lifetime a”.

^ **TOUTES** les références ont des lifetimes ; mais la plupart du temps, on n'a pas besoin de l'expliciter.

> Lifetimes are always inferred by the compiler: you cannot assign a lifetime yourself.
>
> Lifetime annotations create constraints; the compiler verifies that there is a valid solution.

^ on ne peut pas assigner soi-même une lifetime : elle est gérée par le compilo. Du coup, les lifetime annotations ne servent PAS à assigner une lifetime (mais à exprimer des contraintes entre des lifetimes existantes.

> Lifetimes for function arguments and return values must be fully specified, but Rust allows lifetimes to be elided in most cases with a few simple rules.

^ les lifetimes apparaissant dans la signature d'une fonction (arguments ou valeur de retour) sont obligatoires, mais élisées la plupart du temps.

## 16.7. Lifetimes in Function Calls

https://google.github.io/comprehensive-rust/ownership/lifetimes-function-calls.html

Un intéressant exemple ici :

```rs
#[derive(Debug)]
struct Point(i32, i32);

fn left_most<'a>(p1: &'a Point, p2: &'a Point) -> &'a Point {
    if p1.0 < p2.0 { p1 } else { p2 }
}

fn main() {
    let p1: Point = Point(10, 10);
    let p2: Point = Point(20, 20);
    let p3: &Point = left_most(&p1, &p2);
    println!("p3: {p3:?}");
}
```

^ c'est le cas de base, il compile.

Je reproduis l'un de ses dérivés, qui ne compilera pas (car `p3` vit plus longtemps que `p2`, or une référence ne peut pas vivre plus longtemps que la valeur référencée) :

```rs
#[derive(Debug)]
struct Point(i32, i32);

fn left_most<'a>(p1: &'a Point, p2: &'a Point) -> &'a Point {
    if p1.0 < p2.0 { p1 } else { p2 }
}

fn main() {
    let p1: Point = Point(10, 10);
    let p3: &Point;
    {
        let p2: Point = Point(20, 20);
        p3 = left_most(&p1, &p2);
    }
    println!("p3: {p3:?}");
}
```

Par ailleurs, un autre dérivé qui ne compile pas est intéressant :

```rs
fn left_most<'a, 'b>(p1: &'a Point, p2: &'a Point) -> &'b Point
```

> Another way to explain it:
>
> Two references to two values are borrowed by a function and the function returns another reference.
>
> It must have come from one of those two inputs (or from a global variable).
>
> Which one is it? The compiler needs to know, so at the call site the returned reference is not used for longer than a variable from where the reference came from.

^ ça dit explicitement que la lifetime d'une référence retournée par une fonction est OBLIGATOIREMENT liée à la lifetime de l'un des arguments de la fonction, ou bien d'une variable globale. C'est assez logique : si la fonction renvoie une référence, c'est soit qu'elle pointe sur un objet passé en argument, soit qu'elle pointe sur un objet global : si elle pointait sur un objet du corps de la fonction, alors celui-ci cesserait d'exister lorsque la fonction retourne, et la référence serait dangling.

NDM : je pense quand même qu'il y a des cas plus bâtards (ou en tout cas plus difficile à exprimer avec les lifetime annotations de rust, que je trouve un peu simplistes), comme par exemple le fait de retourner une référence sur un sous-objet d'un argument de la fonction, dans le cas où le sous-objet n'a pas la même durée de vie que l'argument. Mais je suppose que c'est un peu tiré par les cheveux (et qu'on s'approche du domaine des self-referential data structures, mal gérées par rust).


## 16.8. Lifetimes in Data Structures

https://google.github.io/comprehensive-rust/ownership/lifetimes-data-structures.html

Un intéressant exemple illustratif ici aussi :

```rs
#[derive(Debug)]
struct Highlight<'doc>(&'doc str);

fn erase(text: String) {
    println!("Bye {text}!");
}

fn main() {
    let text = String::from("The quick brown fox jumps over the lazy dog.");
    let fox = Highlight(&text[4..19]);
    let dog = Highlight(&text[35..43]);
    // erase(text);
    println!("{fox:?}");
    println!("{dog:?}");
}
```

La structure `Highlight` dépend de la lifetime de la référence qu'on lui a passée.

> In the above example, the annotation on Highlight enforces that the data underlying the contained &str lives at least as long as any instance of Highlight that uses that data.

^ le borrow checker va vérifier que si une `Highlight` existe, alors la string sur laquelle on a construit cette `Highlight` existe aussi (ce qui explique que le code ne compilera pas si on decommente `erase`).

> Types with borrowed data force users to hold on to the original data. This can be useful for creating lightweight views, but it generally makes them somewhat harder to use.

^ **clé de compréhension** = les types dont l'un des membres est une référence servent à créer des _views_ sur d'autres structures. Ça peut être utile, mais ça rend le type plus compliqué à utiliser (NDM : car on se tape tout un tas de lifetime annotations à propager).

> When possible, make data structures own their data directly.

^ recommandation = essayer d'éviter les références dans les data structures, et de préférer les structures qui ownent leurs membres (NDM : dommage, selon moi... On a parfois besoin de modéliser son problème par des références)

# 17. Structs

https://google.github.io/comprehensive-rust/structs.html

## 17.1. Tuple Structs

https://google.github.io/comprehensive-rust/structs/tuple-structs.html

> This is often used for single-field wrappers (called newtypes)

^ pattern **newtype**, très fréquent en rust, et très pratique, notamment pour faire du strong-typing.

À signaler aussi = un pattern pratique pour profiter des fields par défaut :

```rs
let sam = Person {
    name: "Sam".to_string(),
    ..Person::default()
};
```

# 18. Methods

https://google.github.io/comprehensive-rust/methods.html

## 18.1. Method Receiver

https://google.github.io/comprehensive-rust/methods/receiver.html

> The &self above indicates that the method borrows the object immutably.

^ une méthode peut recevoir son objet selon les 4 possibilités offertes par les deux axes suivants :

- mutable vs immutable
- borrowed vs owned.

> self: takes ownership of the object and moves it away from the caller. The method becomes the owner of the object. The object will be dropped (deallocated) when the method returns, unless its ownership is explicitly transmitted.

^ une bonne façon de voir les choses pour une fonction/méthode qui owne un de ses arguments : l'objet sera droppé à la fin de la méthode/fonction... à moins que son ownership n'ait été à quelqu'un d'autre dans le corps de la fonction, ou au plus tard à la valeur de retour de la fonction.

> Complete ownership does not automatically mean mutability.

^ un point intéressant et un peu contre-intuitif : ownership et mutabilité sont des concepts orthogonaux : même si une méthode reçoit l'ownership d'un de ses arguments, elle n'est pas pour autant systématiquement autorisée à le muter.

La speaker-note est intéressante aussi :

> Consider emphasizing “shared and immutable” and “unique and mutable”. These constraints always come together in Rust due to borrow checker rules, and self is no exception. It isn’t possible to reference a struct from multiple locations and call a mutating (&mut self) method on it.

^ Le borrow-checker assure qu'on est forcément dans l'un des deu cas suivants :

- shared and immutable
- unique and mutable

# 19. Exercises

https://google.github.io/comprehensive-rust/exercises/day-2/morning.html

Rien de très intéressants dans ces exercices...

## 20. Standard Library

https://google.github.io/comprehensive-rust/std.html

> In fact, Rust contains several layers of the Standard Library: core, alloc and std.
>
> - core includes the most basic types and functions that don’t depend on libc, allocator or even the presence of an operating system.
> - alloc includes types which require a global heap allocator, such as Vec, Box and Arc.

^ la stdlib est découpée en morceaux ! Et une partie de la stdlib ne nécessite pas d'allocator (et ne dépend donc ni de la libc, ni des fonctionnalités offertes par un OS).

NDM : je suppose que cette partie de la stdlib est ainsi utilisable pour implémenter un OS from scratch, du coup.

## 20.1. Option and Result

https://google.github.io/comprehensive-rust/std/option-result.html

Deux petits trucs utiles ici :

> Option<&T> has zero space overhead compared to &T.

^ l'utilisation d'une `Option` est gratuite.

> try_into attempts to convert the vector into a fixed-sized array.

^ on peut convertir un `Vector` (dynamically sized, heap allocated) en un `array` (statically sized, stack allocated)

## 20.2. String

https://google.github.io/comprehensive-rust/std/string.html

> String implements Deref<Target = str>, which means that you can call all str methods on a String.

^ de mémoire, `Deref` permet de manipuler ses smart-pointers comme les objets qu'ils contiennent. Et une `String` est bien un smart-pointer, qui contient et manage un buffer d'octets représentant une chaîne encodée en UTF-8.

> String is implemented as a wrapper around a vector of bytes, many of the operations you see supported on vectors are also supported on String, but with some extra guarantees.

^ conceptuellement, une `String` est un `Vector` de bytes particulier.

## 20.3. Vec

https://google.github.io/comprehensive-rust/std/vec.html

> Vec implements Deref<Target = [T]>, which means that you can call slice methods on a Vec.

^ c'est logique, mais je ne m'étais jamais fait la remarque : grâce à `Deref`, un `Vec` se manipule comme un slice.

> Show iterating over a vector and mutating the value:

```rs
for e in &mut v { *e += 50; }
```

^ comment muter un `Vec`.

## 20.4. HashMap

https://google.github.io/comprehensive-rust/std/hashmap.html

`myhashmap.entry()` permet de récupérer un enum sur l'item pour le gérer de façon monadique (e.g. pour ajouter une valeur si elle n'existe pas).

Pas de macro pour construire directement une hashmap remplie, mais on peut en construire depuis un array de tuple (ou depuis un iterator de tuple).

## 20.5. Box

https://google.github.io/comprehensive-rust/std/box.html

> Box is an owned pointer to data on the heap

^ tout est dans la définition :

- **OWNED** = le cycle de vie de la data pointée est liée au cycle de vie de la `Box`
- **POINTER** = la `Box` est un smart-pointer qui gère une ressource (= la data pointée)
- **ON THE HEAP** = la data est allouée sur le heap.

> Box<T> implements Deref<Target = T>, which means that you can call methods from T directly on a Box<T>.

Tout comme une référence vers un `T` (i.e. un `&T`) se manipule comme un `T` grâce à `Deref`, une `Box<T>` se manipule comme un `T` grâce à `Deref`.

> Box is like std::unique_ptr in C++, except that it’s guaranteed to be not null.

^ une vision utile pour mon profil : un `Box` est l'équivalent de `unique_ptr`.

> A Box can be useful when you:
>
> - have a type whose size that can’t be known at compile time, but the Rust compiler wants to know an exact size.
> - want to transfer ownership of a large amount of data. To avoid copying large amounts of data on the stack, instead store the data on the heap in a Box so only the pointer is moved.

^ les deux cas d'usage d'une `Box` :

- quand on a besoin de connaître au compile-time la taille d'un type dynamiquement sizé
- quand on veut transférer l'ownership de données sans les copier car elles sont trop volumineuses (on transfère alors juste l'ownership des données sur le heap, plutôt que les données).

### 20.5.1. Recursive Data Types

https://google.github.io/comprehensive-rust/std/box-recursive.html

> Recursive data types or data types with dynamic sizes need to use a Box

^ utiliser un `Box` est indispensable pour les data-types récursifs, car sinon on ne connaît pas la taille du type au compile-time.

> If Box was not used and we attempted to embed a List directly into the List, the compiler would not compute a fixed size of the struct in memory (List would be of infinite size).
>
> Box solves this problem as it has the same size as a regular pointer and just points at the next element of the List in the heap.

^ c'était aussi l'exemple du rustbook pour introduire les Box.

## 20.6. Rc

https://google.github.io/comprehensive-rust/std/rc.html

> Rc is a reference-counted shared pointer. Use this when you need to refer to the same data from multiple places

^ on veut utiliser un Rc quand on veut faire référence à la même donnée depuis plusieurs endroits différents, (NDM : dans un contexte **monothread** uniquement).

> You can downgrade a shared pointer into a Weak pointer to create cycles that will get dropped.

^ comme à chaque fois avec les shared-pointers, il y a la question des références cycliques et des weak-pointers.

> Rc in Rust is like std::shared_ptr in C++.

^ Rc est l'équivalent rust des `shared_ptr` du C++

> Rc::clone is cheap: it creates a pointer to the same allocation and increases the reference count. Does not make a deep clone and can generally be ignored when looking for performance issues in code.

^ ajouter une nouvelle référence vers un objet déjà existant est peu coûteux.

> make_mut actually clones the inner value if necessary (“clone-on-write”) and returns a mutable reference.

^ pour récupérer une référence mutable sur un Rc, on appelle `make_mut`, ce qui clonera l'objet sous-jacent s'il y avait déjà des Rc pointant dessus, ou réutilisera l'objet sinon : **clone on write**. Attention que du coup, de façon un peu contre-intuitive, on ne garantit pas de récupérer une référence mutable sur l'objet de la Rc sur laquelle on a appelé `make_mut`.

> Rc::downgrade gives you a weakly reference-counted object to create cycles that will be dropped properly (likely in combination with RefCell, on the next slide).

^ on peut créer une nouvelle référence sur le même objet, mais weak. NDM : derrière, il faut appeler [upgrade](https://doc.rust-lang.org/std/rc/struct.Weak.html#method.upgrade) sur le Weak pointer pour pouvoir l'utiliser, et cette opération peut échouer (si l'objet pointé a été détruit entretemps), cf. [la doc](https://doc.rust-lang.org/std/rc/struct.Weak.html).

## 20.7. Cell/RefCell

https://google.github.io/comprehensive-rust/std/cell.html

> Cell and RefCell implement what Rust calls interior mutability: mutation of values in an immutable context.

^ **interior mutability** = capacité à muter un bout d'un objet constant (la notion est à rapprocher du keyword `mutable` utilisé sur un attribut d'un objet `const` en C++).

> Cell is typically used for simple types, as it requires copying or moving values. More complex interior types typically use RefCell, which tracks shared and exclusive references at runtime and panics if they are misused.

^ `Cell` copie/move les valeurs ; `RefCell` ne fait les checks qu'au runtime (on a donc un risque de panic).

> To do anything with a Node, you must call a RefCell method, usually borrow or borrow_mut

^ ma compréhension, c'est que quand on voit `RefCell<Node>`, ça veut dire "un Node qu'on peut éventuellement muter même s'il est immutable, à condition d'appeler les fonctions spéciales `borrow` et `borrow_mut`, et de prendre le risque de voir celles-ci panic au runtime".

NDM : le pattern `Rc<RefCell<T>>` est fréquent, car les Rc ne sont pas mutables, cf. [leur doc](https://doc.rust-lang.org/std/rc/index.html) :

> Shared references in Rust disallow mutation by default, and Rc is no exception: you cannot generally obtain a mutable reference to something inside an Rc. If you need mutability, put a Cell or RefCell inside the Rc

Du coup, `Rc<RefCell<T>>` fait bien ce qu'on veut si on veut "un Rc mutable" = un Rc, qui, comme tous les Rc, est autour de quelque chose d'immutable... mais qu'on peut muter quand même car ce quelque chose est un RefCell (qui autorise à muter des trucs immutables).


# 21. Modules

## 21.1. Visibility

https://google.github.io/comprehensive-rust/modules/visibility.html

> Modules are a privacy boundary:
>
> - Module items are private by default (hides implementation details).
> - Parent and sibling items are always visible.
> - In other words, if an item is visible in module foo, it’s visible in all the descendants of foo.

^ Le sujet le plus important du point de vue "architecture" : les modules permettent de définir explicitement ce qui est public ou privé.

## 21.3. Filesystem Hierarchy

https://google.github.io/comprehensive-rust/modules/filesystem.html

> Omitting the module content will tell Rust to look for it in another file:
>
> ```rs
> mod garden;
> ```
> This tells rust that the garden module content is found at src/garden.rs.

^ ici, c'est le fait de ne pas préciser de contenu au module qui indique au compilo d'aller rechercher le module dans un fichier externe.

# 22. Exercises

https://google.github.io/comprehensive-rust/exercises/day-2/afternoon.html

## 22.1. Iterators and Ownership

https://google.github.io/comprehensive-rust/exercises/day-2/iterators-and-ownership.html

> The Iterator trait tells you how to iterate once you have created an iterator. The related trait IntoIterator tells you how to create the iterator

^ différence entre le trait `Iterator` (qui permet d'itérer, i.e. "de récupérer le suivant" d'un objet, en proposant la fonction `next()`) et le trait `IntoIterator` (qui permet de récupérer un itérator, en proposant la fonction `into_iter()`).

La fin de l'exercice est intéressante :

```rs
let v: Vec<String> = ...
for word in &v { ... }
for word in v { ... }
```

La question est de savoir quel est le type de `word` dans les deux cas. L'exercice encourage à aller regarder les docs :

- [impl IntoIterator for &Vec<T>](https://doc.rust-lang.org/std/vec/struct.Vec.html#impl-IntoIterator-for-%26'a+Vec%3CT,+A%3E) : quand on appelle `into_iter()` sur un `Vec<T>`, l'itérateur créé renvoie des `&T` ; il ne consomme donc pas le vector qui continuera d'être utilisable.
- [impl IntoIterator for Vec<T>](https://doc.rust-lang.org/std/vec/struct.Vec.html#impl-IntoIterator-for-Vec%3CT,+A%3E) : quand on appele `into_iter()` sur un `Vec<T>`, l'itérateur créé renvoie des `T`, ce qui consommera le vector : au fur et à mesure qu'on itère, on move les `T` en dehors du vector (_The vector cannot be used after calling this._)

# 25. Traits

## 25.1. Trait Objects

https://google.github.io/comprehensive-rust/traits/trait-objects.html

> Trait objects allow for values of different types, for instance in a collection:

```rs
struct Dog { name: String, age: i8 }
struct Cat { lives: i8 }

// ...je skippe les implementations des traits

fn main() {
    let pets: Vec<Box<dyn Pet>> = vec![
        Box::new(Cat { lives: 9 }),
        Box::new(Dog { name: String::from("Fido"), age: 5 }),
    ];
    for pet in pets {
        println!("Hello, who are you? {}", pet.talk());
    }
}
```

> Types that implement a given trait may be of different sizes. This makes it impossible to have things like Vec<dyn Pet> in the example above.

^ on pourrait croire que le Vector contient des items de tailles différentes, mais comme il contient des Box, tout le monde est de la même taille.

La page contient un schéma intéressant, avec le layout mémoire.

> In the example, pets is allocated on the stack and the vector data is on the heap. The two vector elements are fat pointers:
>
> A fat pointer is a double-width pointer. It has two components: a pointer to the actual object and a pointer to the virtual method table (vtable) for the Pet implementation of that particular object.
>
> The data for the Dog named Fido is the name and age fields. The Cat has a lives field.

^ chaque item du Vector est à la fois un pointeur classique (vers l'objet Dog ou Cat) car le Vector contient des Box, et un pointeur vers la vtable de l'implémentation du trait pour cet objet ; l'ensemble des deux forme un **fat-pointer**.

## 25.2. Deriving Traits

https://google.github.io/comprehensive-rust/traits/deriving-traits.html

> Rust derive macros work by automatically generating code that implements the specified traits for a data structure.

Les macros `derive` génèrent du code qui implémentent automatiquement un trait pour un type donné.

## 25.4. Trait Bounds

https://google.github.io/comprehensive-rust/traits/trait-bounds.html

Un **trait bound**, c'est la restriction d'un type générique à un trait donné :

```rs
fn duplicate<T>(a: T) -> (T, T)
where  T: Clone,
```

^ ici, le trait bound est :  `T: Clone`

Littéralement (dans mon franglais) un "binding sur un trait".

## 25.5. impl Trait

https://google.github.io/comprehensive-rust/traits/impl-trait.html

> For a return type, it means that the return type is some concrete type that implements the trait, without naming the type. This can be useful when you don’t want to expose the concrete type in a public API.

^ `impl Trait` comme return type sert à renvoyer un type implémentant `Trait`, peu importe ce qu'il est concrètement ; c'est une forme d'interface (une interface au compile-time uniquement : le dynamic dispatch = le polymorphisme au runtime est plutôt obtenu avec `dyn Trait`)

# 26. Important Traits

https://google.github.io/comprehensive-rust/traits/important-traits.html

## 26.1. Iterator

https://google.github.io/comprehensive-rust/traits/iterator.html

Implémenter le trait `Iterator` pour un de ses propres types revient à lui ajouter une fonction `next()`. Ça revient donc à considérer qu'on peut "récupérer le suivant" de son type.

Une fois que mon type implémente `Iterator`, il gagne tout un tas de fonctions pratiques (e.g. `enumerate`).

> This is why you can iterate over a vector with for i in some_vec { .. } but some_vec.next() doesn’t exist.

^ `IntoIterator` permet d'itérer avec une boucle for.

## 26.2. FromIterator

https://google.github.io/comprehensive-rust/traits/from-iterator.html

Sur un type qui implémente `FromIterator`, on peut appeler `collect()` pour reconstruire la collection à partir de l'itérateur .

## 26.3. From and Into

https://google.github.io/comprehensive-rust/traits/from-into.html

Implémenter `Into` n'est pas nécessaire, ça vient tout seul si on implémente `From`.

> When declaring a function argument input type like “anything that can be converted into a String”, the rule is opposite, you should use Into.

^ comment déclarer une fonction qui accepte "tout ce qui peut être converti en String".

## 26.4. Read and Write

https://google.github.io/comprehensive-rust/traits/read-write.html

`Read` nécessite d'implémenter une méthode `read` qui remplit un buffer de bytes. Le trait est donc là pour représenter un reader, p.ex. un objet qui lit le contenu d'un fichier.

`Write` est le contraire = représente un objet capable d'écrire des bytes.

## 26.5. Drop

https://google.github.io/comprehensive-rust/traits/drop.html

> Values which implement Drop can specify code to run when they go out of scope

Permet de bénéficier du RAII

> All its fields will then be dropped too, whether or not it implements Drop.

Implémenter `Drop` n'override pas le comportement habituel = dropper aussi les membres du type.

## 26.6. Default

https://google.github.io/comprehensive-rust/traits/default.html

> Default trait produces a default value for a type

Une unique méthode `default` qui renvoie une instance de `Self`

> It can be implemented directly or it can be derived via #[derive(Default)].

## 26.7. Operators: Add, Mul, ...

https://google.github.io/comprehensive-rust/traits/operators.html

> Operator overloading is implemented via traits in std::ops

RAS ici, c'est intuitif.

## 26.8. Closures: Fn, FnMut, FnOnce

https://google.github.io/comprehensive-rust/traits/closures.html

> Closures or lambda expressions have types which cannot be named. However, they implement special Fn, FnMut, and FnOnce traits

^ les traits `Fn` , `FnOnce` et `FnMut` servent à décrire des lambdas.

Les speaker notes sont une excellente synthèse de comment ça fonctionne, donc je les reproduis presqu'entièrement :

> An Fn (e.g. add_3) neither consumes nor mutates captured values, or perhaps captures nothing at all. It can be called multiple times concurrently.
>
> An FnMut (e.g. accumulate) might mutate captured values. You can call it multiple times, but not concurrently.
>
> If you have an FnOnce (e.g. multiply_sum), you may only call it once. It might consume captured values.
>
> FnMut is a subtype of FnOnce. Fn is a subtype of FnMut and FnOnce. I.e. you can use an FnMut wherever an FnOnce is called for, and you can use an Fn wherever an FnMut or FnOnce is called for.
>
> The compiler also infers Copy (e.g. for add_3) and Clone (e.g. multiply_sum), depending on what the closure captures.
>
> By default, closures will capture by reference if they can. The move keyword makes them capture by value.


# 27. Exercises

https://google.github.io/comprehensive-rust/exercises/day-3/morning.html

## 27.2. Points and Polygons

https://google.github.io/comprehensive-rust/exercises/day-3/points-polygons.html

Exercice très formateur sur un paquet de trucs.

Notamment, à cause d'une erreur de ma part, j'ai dû réimplémenter moi-même l'itérateur sur les points du `Polygon`, ce qui était un bon exercice (mais je n'avais pas besoin de m'embêter : j'aurais pu renvoyer l'itérateur sur le Vector... quand j'avais essayé, je n'avais pas réussi, j'avais dû faire une erreur ; du coup j'ai créé un objet `PolygonIte` pour représenter l'itérateur).

Note : c'est `PolygonIte` et non `Polygon` qui implémente le trait `Iterator` (sinon, un `Polygon` devrait garder l'état de son itération = savoir où il en est...)


Du coup, je me suis un peu intéressé aux différences entre le trait `Iterator` et `IntoIterator` :

- `Iterator` est à implémenter par l'objet représentant l'itérateur = `PolygonIte`
- `IntoIterator` serait à implémenter par l'objet sur lequel on veut itérer = `Polygon`, si on voulait l'utiliser dans une boucle for

Mais en pratique, tant que notre objet (`Polygon`) a une méthode (`iter`) pour renvoyer un truc implémentant `Iterator` (`PolygonIte`), il n'est pas indispensable de dériver `IntoIterator` ; c'est le cas ici.

Ce qui est un peu moins clair, c'es la différence entre `iter()` et `into_iter()`.

[Cette réponse stackoverflow](https://stackoverflow.com/questions/34733811/what-is-the-difference-between-iter-and-into-iter) est bien détaillée : on dirait que les deux fonctions ont vocation à renvoyer un `Iterator` (donc un `PolygonIte`), mais dans des circonstances un peu différentes.

# 28. Error Handling

https://google.github.io/comprehensive-rust/error-handling.html

## 28.1. Panics

https://google.github.io/comprehensive-rust/error-handling/panics.html

> Panics are for unrecoverable and unexpected errors.
> Panics are symptoms of bugs in the program.

^ en cas de bug dans le programme, i.e. d'erreurs de programmation (par opposition aux erreurs "normales" comme un disque dur plein), il n'y a rien de mieux à faire que de panic...

> Use non-panicking APIs (such as Vec::get) if crashing is not acceptable.

^ si on veut absolument éviter le crash (NDM : et on peut avoir de bonnes raisons de le faire ! Par exemple si le service est très très long à redémarrer, ou si on veut un fort taux de disponibilité), il faut utiliser des fonctions qui ne panic pas.

## 28.2. Structured Error Handling

https://google.github.io/comprehensive-rust/error-handling/result.html

> In the case where an error should never happen, unwrap() or expect() can be called, and this is a signal of the developer intent too.

^ utiliser `unwrap()` communique une intention = l'appel a beau renvoyer un `Result`, il n'est pas censé échouer dans le fonctionnement normal du programme (ou formulé alternativement, s'il échoue, on choisit délibérément de ne pas traiter l'erreur et de crasher).

## 28.3. Propagating Errors with ?

### 28.3.1. Converting Error Types

https://google.github.io/comprehensive-rust/error-handling/converting-error-types.html

> It is good practice for all error types that don’t need to be no_std to implement std::error::Error, which requires Debug and Display.

^ bonne pratique = implémenter le trait `Error` sur ses erreurs.

### 28.3.2. Deriving Error Enums

https://google.github.io/comprehensive-rust/error-handling/deriving-error-enums.html

> The thiserror crate is a popular way to create an error enum like we did on the previous page:

^ c'est donc à ça que sert [thiserror](https://docs.rs/thiserror/latest/thiserror/) = à faire l'équivalent de ceci :

```rs
#[derive(Debug)]
enum ReadUsernameError {
    IoError(io::Error),
    EmptyUsername(String),
}

impl Error for ReadUsernameError {}

impl Display for ReadUsernameError {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        match self {
            Self::IoError(e) => write!(f, "IO error: {e}"),
            Self::EmptyUsername(filename) => write!(f, "Found no username in {filename}"),
        }
    }
}

impl From<io::Error> for ReadUsernameError {
    fn from(err: io::Error) -> ReadUsernameError {
        ReadUsernameError::IoError(err)
    }
}
```

... mais en beaucoup plus concis :

```rs
use thiserror::Error;

#[derive(Debug, Error)]
enum ReadUsernameError {
    #[error("Could not read: {0}")]
    IoError(#[from] io::Error),
    #[error("Found no username in {0}")]
    EmptyUsername(String),
}
```

Last but not least :

> It doesn’t affect your public API, which makes it good for libraries.

### 28.3.3. Dynamic Error Types

https://google.github.io/comprehensive-rust/error-handling/dynamic-errors.html

On utilise `Box<dyn Error>` pour représenter une erreur générique = un truc implémentant le trait `Error`, mais non connu au compile-time.

> This saves on code, but gives up the ability to cleanly handle different error cases differently in the program. As such it’s generally not a good idea to use Box<dyn Error> in the public API of a library, but it can be a good option in a program where you just want to display the error message somewhere.

^ cette astuce peut être nécessaire, mais elle empêche de gérer proprement les erreurs, donc à éviter dans l'API publique de sa lib.

### 28.3.4. Adding Context to Errors

https://google.github.io/comprehensive-rust/error-handling/error-contexts.html

La crate [anyhow](https://docs.rs/anyhow/latest/anyhow/) propose une façon idiomatique de gérer les erreurs, en ajoutant du contexte à toute exception implémentant le trait `Error`.

Du coup, on a un peu moins besoin de créer des types d'erreur custom.

> anyhow::Result<V> is a type alias for Result<V, anyhow::Error>.

^ au lieu de renvoyer des types d'erreur custom, on renvoie `anyhow::Error`.

> anyhow::Error is essentially a wrapper around Box<dyn Error>. As such it’s again generally not a good choice for the public API of a library, but is widely used in applications.

^ l'inconvénient (NDM : quand même important à mes yeux...) est qu'on n'utilise plus un type exact pour une erreur, mais un type général.

> Actual error type inside of it can be extracted for examination if necessary

^ cependant, on ne perd pas le type d'erreur reçue, on peut toujours y accéder.

# 29. Testing

https://google.github.io/comprehensive-rust/testing.html

> Unit tests are supported throughout your code.
>
> Integration tests are supported via the tests/ directory.

^ les **tests unitaires** vont avec le code. Les **tests d'intégration** vont dans un répertoire externe à l'app/lib, ils n'ont accès qu'à l'API publique de la librairie.

> Unit tests are often put in a nested module (...) This lets you unit test private helpers.

^ en plaçant des tests U dans un sous module du code, ça donne accès au code privé si besoin.

## 29.3. Documentation Tests

https://google.github.io/comprehensive-rust/testing/doc-tests.html

> Code blocks in /// comments are automatically seen as Rust code. The code will be compiled and executed as part of cargo test.

^ les doctests sont supportés out-of-the-box via du code dans un comment `///`

>  Adding # in the code will hide it from the docs, but will still compile/run it.

^ tiens, je ne savais pas, mais on a un deuxième niveau de commentaires dans les commentaires.

## 29.5. Useful crates

https://google.github.io/comprehensive-rust/testing/useful-crates.html

- googletest: Comprehensive test assertion library in the tradition of GoogleTest for C++.
- proptest: Property-based testing for Rust.
- rstest: Support for fixtures and parameterised tests.

# 30. Unsafe Rust

https://google.github.io/comprehensive-rust/unsafe.html


> The Rust language has two parts:
>
> - Safe Rust: memory safe, no undefined behavior possible.
> - Unsafe Rust: can trigger undefined behavior if preconditions are violated.

^ ici aussi on retrouve le fait qu'il y a deux langages en rust.

Attention, unsafe rust ne signifie pas dire buggé ! Ça dit simplement : "c'est à toi de vérifier les pré-conditions" (ce qui ouvre la porte aux erreurs) alors que d'habitude, le compilo le vérifie automatiquement (ce qui garantit l'absence d'erreurs).

> Unsafe code is usually small and isolated, and its correctness should be carefully documented. It is usually wrapped in a safe abstraction layer.

^ bonnes pratiques d'unsafe rust :

- petits bouts de code isolés
- on documente les pré-conditions nécessaires
- on essaye de wrapper le code unsafe dans une abstraction safe, i.e. dans un bout de code 1. qui vérifie les pré-conditions (plutôt que de laisser le client le faire) et 2. qui n'est pas estampillé `unsafe`

> Unsafe Rust gives you access to five new capabilities:
>
> - Dereference raw pointers.
> - Access or modify mutable static variables.
> - Access union fields.
> - Call unsafe functions, including extern functions.
> - Implement unsafe traits.

^ les 5 superpouvoirs d'unsafe rust.

> Unsafe Rust does not mean the code is incorrect. It means that developers have turned off the compiler safety features and have to write correct code by themselves. It means the compiler no longer enforces Rust’s memory-safety rules.

^ un rappel qu'unsafe rust veut juste dire "c'est au client (et non plus au compilo) de vérifier la correctness du code".

## 30.1. Dereferencing Raw Pointers

https://google.github.io/comprehensive-rust/unsafe/raw-pointers.html

> It is good practice (and required by the Android Rust style guide) to write a comment for each unsafe block explaining how the code inside it satisfies the safety requirements of the unsafe operations it is doing.

^ bonne pratique = commenter les blocs unsafe.

> In the case of pointer dereferences, this means that the pointers must be valid, i.e.:
>
> The pointer must be non-null.
> The pointer must be dereferenceable (within the bounds of a single allocated object).
> The object must not have been deallocated.
> There must not be concurrent accesses to the same location.
> If the pointer was obtained by casting a reference, the underlying object must be live and no reference may be used to access the memory.
> In most cases the pointer must also be properly aligned.

^ les (assez nombreuses, mais logiques) règles pour déréférencer un raw pointer tout en évitant l'UB.

## 30.2. Mutable Static Variables

https://google.github.io/comprehensive-rust/unsafe/mutable-static-variables.html

> since data races can occur, it is unsafe to read and write mutable static variables

^ de base, c'est unsafe de muter une variable globale, vu que ça peut conduire à des data races. Pour le faire, il faut un bloc `unsafe`

> Using a mutable static is generally a bad idea, but there are some cases where it might make sense in low-level no_std code, such as implementing a heap allocator or working with some C APIs.

^ mais ceci dit, les cas où on doit muter une variable globale et où ça n'est pas un code-smell sont rares...

## 30.3. Unions

https://google.github.io/comprehensive-rust/unsafe/unions.html

> Unions are like enums, but you need to track the active field yourself (...) Unions are very rarely needed in Rust as you can usually use an enum. They are occasionally needed for interacting with C library APIs.

^ les unions sont rares en rust, elles sont surtout utilisées pour s'interfacer avec du C.

## 30.4. Calling Unsafe Functions

https://google.github.io/comprehensive-rust/unsafe/calling-unsafe-functions.html

Faire appel à une fonction unsafe nécessite d'être dans un bloc unsafe.

> Functions from other languages might violate the guarantees of Rust

^ appeler une fonction externe aussi, car on ne sait pas si la fonction externe offre les mêmes garanties que ce que garantit le compilo rust.

## 30.5. Implementing Unsafe Traits

https://google.github.io/comprehensive-rust/unsafe/unsafe-traits.html

> Like with functions, you can mark a trait as unsafe if the implementation must guarantee particular conditions to avoid undefined behaviour.

^ marquer un trait `unsafe` peut être utile si on doit vérifier des pré-conditions pour l'utiliser de façon safe.

Implémenter un trait marqué `unsafe` nécessite d'être unsafe.

> There should be a # Safety section on the Rustdoc for the trait explaining the requirements for the trait to be safely implemented.

^ on retrouve la bonne pratique = documenter les conditions d'utilisation du trait à vérifier pour rester safe.

# 31. Exercises

https://google.github.io/comprehensive-rust/exercises/day-3/afternoon.html

Mon premier vrai contact avec unsafe rust et l'utilisation de ffi.

J'ai galéré (sans doute plus que ce qui était nécessaire) avec tout un tas de conversions...

## 31.1. Safe FFI Wrapper

https://google.github.io/comprehensive-rust/exercises/day-3/safe-ffi-wrapper.html
