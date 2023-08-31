* [Ressources](#ressources)
* [Conversions](#conversions)
* [Tricks](#tricks)
   * [cargo clean](#cargo-clean)
   * [faire en sorte que le compio affiche le type exact d'une expression](#faire-en-sorte-que-le-compio-affiche-le-type-exact-dune-expression)
   * [n'itérer que sur un variant d'enum](#nitérer-que-sur-un-variant-denum)
   * [laisser rust inférer le type avec collect()](#laisser-rust-inférer-le-type-avec-collect)
   * [tester si un enum est un variant particulier](#tester-si-un-enum-est-un-variant-particulier)
   * [dire à clippy d'ignorer un warning](#dire-à-clippy-dignorer-un-warning)
   * [dire au compilo d'ignorer un morceau de dead-code](#dire-au-compilo-dignorer-un-morceau-de-dead-code)
   * [indiquer explicitement que le corps d'une fonction n'est pas implémenté](#indiquer-explicitement-que-le-corps-dune-fonction-nest-pas-implémenté)
   * [utiliser la macro debug pour formater un affichage](#utiliser-la-macro-debug-pour-formater-un-affichage)
   * [définir un field d'une struct avec le même nom qu'un keyword](#définir-un-field-dune-struct-avec-le-même-nom-quun-keyword)
* [Notes diverses](#notes-diverses)
   * [orphan rule](#orphan-rule)
   * [ownership](#ownership)
   * [ownership le retour](#ownership-le-retour)
   * [fields d'une struct publics vs utiliser new](#fields-dune-struct-publics-vs-utiliser-new)
   * [mutabilité et move des références](#mutabilité-et-move-des-références)
* [Rust for C++ Programmers](#rust-for-c-programmers)
   * [Unique pointers](#unique-pointers)
   * [Borrowed pointers](#borrowed-pointers)
   * [Reference counted and raw pointers](#reference-counted-and-raw-pointers)
   * [Data types](#data-types)
   * [Destructuring](#destructuring)
   * [Destructuring pt2 - match and borrowing](#destructuring-pt2---match-and-borrowing)
   * [Arrays and Vectors](#arrays-and-vectors)
   * [Graphs and arena allocation](#graphs-and-arena-allocation)
   * [Closures and first-class functions](#closures-and-first-class-functions)

# Ressources

- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Rust By Practice](https://practice.rs/why-exercise.html)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Effective Rust](https://www.lurklurk.org/effective-rust/)
- [Rust Language Cheat Sheet](https://cheats.rs/)
- [Easy Rust](https://dhghomon.github.io/easy_rust/Chapter_1.html)
- [A Gentle Introduction to Rust](https://stevedonovan.github.io/rust-gentle-intro/readme.html#a-gentle-introduction-to-rust)
- beaucoup, beaucoup de liens qui ont l'air intéressants dans [la cheatsheet rust](https://cheats.rs/#links-services)


# Conversions

- `Option<T>` en `Option<&T>`
    ```rs
    my_option.as_ref()
    ```
- slice en vector, et le contraire :
    ```rs
    // slice en vector
    let my_vec = my_slice.to_vec();

    // vector en slice :
    let my_slice = &my_vec[..];
    ```
- pour convertir un iterator de références en iterator de values, on utilise soit `cloned()` pour cloner les values, soit `into_iter()` pour mover les values :
    - [doc d'into_iter()](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter-1) :
        > Creates a consuming iterator, that is, one that moves each value out of the vector (from start to end). The vector cannot be used after calling this.
    - sans `cloned()` ni `into_inter()` = on récupère un itérable de RÉFÉRENCES :
        ```rs
        let my_vec: Vec<String> = Vec::new();
        let _my_ite = my_vec.iter().collect::<Vec<_>>();
        // ^ _my_ite est de type Vec<&String> (des RÉFÉRENCES), et my_vec est toujours utilisable (il n'a pas été moved)
        ```
    - avec `cloned()`, on récupère un itérable de VALEURS clonées :
        ```rs
        let my_vec: Vec<String> = Vec::new();
        let _my_ite = my_vec.iter().cloned().collect::<Vec<_>>();
        // ^ _my_ite est de type Vec<String> (des VALEURS), et my_vec est toujours utilisable (il n'a pas été moved)
        ```
    - avec `into_iter()`, on récupère un itérable de VALEURS movées :
        ```rs
        let my_vec: Vec<String> = Vec::new();
        let _my_ite = my_vec.into_iter().collect::<Vec<_>>();
        // ^ _my_ite est de type Vec<String> (des VALEURS), et my_vec n'est plus utilisable (il a été moved)
        ```
- pour convertir une option en erreur (pour utiliser l'operator `?` dans une fonction qui doit retourner une erreur), on peut utiliser [`ok_or`](https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or)

# Tricks

## cargo clean

Si `cargo check --all` échoue alors que `cargo build` réussit (et que le build est correct), cleaner le cache peut résoudre le souci :

```
cargo clean
```

## faire en sorte que le compio affiche le type exact d'une expression

```rs
let () = expr;
```

## n'itérer que sur un variant d'enum

Quand j'ai un itérable d'enums, pour n'itérer que sur un variant en particulier, je peux utiliser `filter_map()` (qui combine le filtrage et le mapping permettant de downcaster : [doc](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map)).

## laisser rust inférer le type avec collect()

Dans les situations où il n'y a pas d'ambiguïtés, on peut laisser rust faire l'inférence de type avec `<_>` :

```rs
let res = myhashmap.iter().collect::<Vec<_>>();
let res = myhashmap.iter().collect::<HashMap<_, _>>();  // <--- on peut même l'utiliser deux fois dans la même expression
```

## tester si un enum est un variant particulier

La macro `matches!` testee si un item est un variant d'un enum :

```rs
let only_reds = reds_and_blue.iter().filter(|item| matches!(item, Color::Red));
```

## dire à clippy d'ignorer un warning

Pour silencer clippy, utiliser `#[allow]` (e.g. ici, faire ce `let` inutile est en fait utile, puisqu'il permet de documenter le contenu de la variable)

```rs
let between_zero_and_360 = (between_minus360_and_360 + 360) % 360;
#[allow(clippy::let_and_return)]
between_zero_and_360
```

## dire au compilo d'ignorer un morceau de dead-code

```rs
#[allow(dead_code)]
fn f() {}
```

## indiquer explicitement que le corps d'une fonction n'est pas implémenté

```rs
fn not_yet_implemented() {
    unimplemented!("FIXME : fill this body");
}
```

La macro `unimplemented!` panic avec un message précisant que ce n'est pas implémenté : https://doc.rust-lang.org/std/macro.unimplemented.html

## utiliser la macro debug pour formater un affichage

On peut utiliser `{:?}` :

```rs
println!("My obj is {:?}", obj);
```

## définir un field d'une struct avec le même nom qu'un keyword

C'est à éviter (mais ça peut servir), on le préfixe par `r#` :

```rs
pub struct MyStruct {
    regular_field: i32,
    r#type: String,
    other_regular_field: i32,
}
```

# Notes diverses

## orphan rule

**orphan rule** = on ne peut pas arbitrairement implémenter n'importe quel trait pour n'importe quelle struct : on ne peut implémenter un trait `T` pour une struct `S` que si :

- on fait cette implémentation dans le même module que T
- on fait cette implémentation dans le même module que S

(mais si besoin, on peut contourner cette limitation, en implémentant le trait `T` pour un `NewType(S)`)

## ownership

Une façon de comprendre l'ownership : dire qu'une variable V prends l'ownership d'une autre variable W (i.e. W est moved dans V) revient à dire :

> "c'est quand V arrivera en fin de scope que la valeur à laquelle V était bindée sera détruite, alors qu'avant le move, c'était plutôt quand W arrivait en fin de scope que la valeur était détruite"

## ownership le retour

> Ownership = Instantiating a type and binding it to a variable name creates a memory resource that the Rust compiler will validate through its whole lifetime. The bound variable is called the resource's owner.

^ beaucoup de choses dans ce résumé [issu de tourofrust.com](https://tourofrust.com/43_en.html), notamment :

- la variable n'est pas la valeur
- la valeur est une ressource = la zone mémoire
- la variable **possède** la valeur

> When an owner is passed as an argument to a function, ownership is moved to the function parameter. (...) During a move the stack memory of the owners value is copied to the function call's parameter stack memory.

^ du coup grâce au résumé précédent, [cette autre citation de tourofrust.com](https://tourofrust.com/46_en.html) est claire : la ressource (= la zone mémoire) est initialement possédée par la variable externe. Lorsqu'on la passe à une fonction, la ressource est transmise à la variable de la stackframe de la fonction, qui est le nouvel owner de la ressource.



> References allow us borrow access to a resource with the & operator. References are also dropped like other resources.

^ enfin, ce [troisième extrait de tourofrust.com](https://tourofrust.com/48_en.html) complète la boucle : les références se contentent d'emprunter la ressource (= la zone mémoire) mais la rendent quand elles arrivent en fin de vie, d'où leur nom : borrow.

## fields d'une struct publics vs utiliser new

Question : comment doit-on permettre la construction d'une struct :

- en gardant ses fields privés, et en fournissant une fonction `new()` ?
- en passant ses fields publics ?

Réponse : c'est une question de design, qui dépend de la structure et de son utilisation :

- si la structure n'est rien d'autre qu'un agrégat de données sans invariant, on peut passer tous les fields publics, ajouter une fonction `new()` n'apporte rien
- mais si la structure doit maintenir un invariant (e.g. un entier qui doit être toujours supérieur à 10, qui serait un `NewType(u32)`), alors il ne faut pas donner accès au fields, et forcer l'appelant à passer par `new()`, qui construit ou vérifie l'invariant

Un autre exemple, une structure qui représenterait un groupe d'amis et leur âge moyen :

```rs
struct FriendsGroup {
    members: Vec<Person>,
    mean_age: u32,
}
```

Alors les fields `members` et `mean_age` sont liés l'un à l'autre : lorsque `members` change, il faut mettre à jour l'âge moyen → les fields ne doivent donc pas être publics, mais settés par des accesseurs.

**CONCLUSION** = il faut voir la question "field publics" vs. "new + fields privés" comme une question d'encapsulation : a-t-on besoin d'accesseurs ? Le constructeur `new()` est un accesseur comme un autre pour "setter" les attributs.

## mutabilité et move des références

[Cet article](https://dev.to/wrongbyte/rust-references-5ehc) m'a aidé à comprendre des trucs sur l'impossibilité de _move-out of a reference_ ou sur le fait d'avoir des variables mutables sur les références immutables;

> Due to its immutability, we are not allowed to either change what's inside of it or move what's inside of it. In other words, we can't "move out".

^ à partir d'une référence immutable, on n'a pas le droit 1. de muter la valeur pointée (ça c'est intuitif), mais également 2. de move-out la valeur pointée. Si on le fait, on a cette erreur :

```
cannot move out of `*my_vec_ref` which is behind a shared reference
move occurs because `*my_vec_ref` has type `Vec<i32>`, which does not implement the `Copy` trait
```

^ du coup, l'erreur ci-dessus est à reconnaître comme "j'essaye de move une valeur depuis un borrow".

> Additionally, in Rust the . operator automatically dereferences reference types,

^ en rust, de façon un peu contre-intuitive à mes yeux, les références s'utilisent comme des valeurs : on peut utiliser la syntaxe `x.member` de la même façon, que `x` peu importe que `x` soit une référence ou une valeur.

Et du coup, l'erreur `cannot move out` ci-dessus surviendra également lorsque j'essaye de move-out un membre d'une struct, si on manipule la struct avec un immutable borrow :

```rs
struct MyStruct {
    str_vec: Vec<i32>
}

fn main() {
    let strct = MyStruct { str_vec: vec![1, 2, 3]};
    let strct_ref = &strct;
    let my_vec = strct_ref.str_vec; // error!
}
```

Et pour les références mutables ?

> Similarly to shared references, Rust also does not allow us to move out of mutable references. In fact, moving out from any kind of reference - mutable or not - is not allowed.

^ avec une référence mutable, on peut muter la valeur pointée par la référence... mais on ne peut pas non plus move-out la valeur référencée !

**Conclusion à retenir** : on ne peut move que des valeurs, et pas des références ! Les références ne permettent que de lire les valeurs référencées, ou de les muter si on a une mutable référence.

> What is the difference between p: &T, mut p: &T, p: &mut T and mut p: &mut T?

^ en résumé, si la référence est mutable (`p: &mut T` et `mut p: &mut T`), on peut muter la valeur référencée.

Et si la variable qui stocke la référence est elle-même mutable (`mut p: &T` et `mut p: &mut T`), ça permet de réassigner la variable sur une autre référence :

```rs
let vec_number = vec![1, 2, 3];
let another_vec = vec![2, 3, 4];
let mut ref_vec = &vec_number;
ref_vec = &another_vec;
```

# Rust for C++ Programmers

Notes éparses issues de [Rust For Systems Programmers](https://github.com/nrc/r4cppp/tree/master) = un mini-book expliquant les concepts de rust en s'adressant à un public de devs C++/

## Unique pointers

https://github.com/nrc/r4cppp/blob/master/unique.md

> As shown above, owning pointers must be dereferenced to use their values. However, method calls automatically dereference, so there is no need for a -> operator or to use * for method calls.

^ ça clarifie une question que je me posais : c'est uniquement pour les appels de méthodes qu'on omet `*` : dans les autres cas, il faut bien déréférencer les références avec `*` !

Au passage, ça donne une situation que je ne suis pas sûr d'apprécier, mais rigolote = on peut utiliser les méthodes des pointeurs de pointeurs (ou de références de références) de multiples fois, comme s'il n'y avait pas d'indirection :

```rs
fn bar(x: Box<Foo>, y: Box<Box<Box<Box<Foo>>>>) {
    x.foo();
    y.foo();
}
```

----

> Calling Box::new() with an existing value does not take a reference to that value, it copies that value.

```rs
fn foo() {
    let x = 3;
    let mut y = Box::new(x);
    *y = 45;
    println!("x is still {}", x);
}
```

^ dans le cas général (i.e. les types qui ne sont pas `Copy`), `Box::new(something)` alloue une zone mémoire sur le heap capable de stocker `something`, puis y move `something`, et pointe sur cette zone. Dans le cas particulier des types `Copy`, `Box::new` INSTANCIE une nouvelle valeur sur le heap, vers laquelle il pointe.

## Borrowed pointers

https://github.com/nrc/r4cppp/blob/master/borrowed.md

> Note that the reference may be mutable (or not) independently of the mutableness of the variable holding the reference. This is similar to C++ where pointers can be const (or not) independently of the data they point to.

^ J'ai déjà vu ce point dans des notes récentes (un peu plus haut dans ce fichier) : la mutabilité de la référence permet de muter la valeur, la mutabilité de la variable permet de réassigner la référence.

> If a mutable value is borrowed, it becomes immutable for the duration of the borrow. Once the borrowed pointer goes out of scope, the value can be mutated again (...) The same thing happens if we take a mutable reference to a value - the value still cannot be modified.

^ référence mutable sur valeur immutable et vice-versa


> pointer types will automatically be converted to a reference:

```rs
fn foo(x: &i32) { ... }

fn bar(x: i32, y: Box<i32>) {
    foo(&x);
    // foo(x);   // Error - expected &i32, found i32
    foo(y);      // Ok
    foo(&*y);    // Also ok, and more explicit, but not good style
}
```

^ techniquement, si on dispose d'un `Box<i32>` et qu'on veut passer une référence vers l'inner i32 à une fonction qui attend un `&i32`, il faudrait 1. déréférencer le Box avec `*`, puis 2. prendre la référence vers l'inner-value avec `&`, soit `&*my_box`. En pratique, les pointer-types (dont Box fait partie, je suppose qu'il s'agit de ce que le rustbook appelait smart pointers, i.e. ce qui implémente `Deref`) peuvent être passés là où on attend une référence.

> As we mentioned above, all mutable variables are unique. So if you have a mutable value, you know it is not going to change unless you change it. Furthermore, you can change it freely since you know that no one else is relying on it not changing.

^ résumé de pourquoi les règles enforcées par le borrow-checker apportent de la sécurité

## Reference counted and raw pointers

https://github.com/nrc/r4cppp/blob/master/rc-raw.md

> To pass a ref-counted pointer you need to use the clone method. This kinda sucks, and hopefully we'll fix that, but that is not for sure (sadly). You can take a (borrowed) reference to the pointed at value, so hopefully you don't need to clone too often.

^ pour passer des `Rc` à droite à gauche, il faut les cloner. Mais on n'est pas absolument obligés de passer des `Rc` : on peut aussi passer directement un borrow sur la valeur pointée par le `Rc`. La syntaxe n'est pas très jolie though : `baz(&*x);` on déréférence le Rc pour accéder à son inner value, puis on prend une référence vers cette inner value. Et comme tout borrow, un mutable borrow interdira d'utiliser les autres Rc.

> if you want a mutable, ref counted object you need a Cell or RefCell wrapped in an Rc. As a first approximation, you probably want Cell for primitive data (NDM : objects Copy) and RefCell for objects with move semantics

^ résumé de `Rc<RefCell>` et `Rc<Cell>`.

Les raw pointers sont créés avec la même syntaxe que les références (`&`) mais en précisant explicitement que c'est un raw pointer qu'on veut récupérer :

```rs
let mut x = 5;
let x_p: *mut i32 = &mut x;
```

Les raw pointers peuvent être nuls (ce sont les seuls pointeurs de rust à le pouvoir)

Les raw pointers ne sont pas déréférencés automatiquement comme le sont les références, ce qui force à :

```rs
(*x).foo();
```

## Data types

https://github.com/nrc/r4cppp/blob/master/data-types.md

> We don't mark fields in structs or enums as mutable, their mutability is inherited. This means that a field in a struct object is mutable or immutable depending on whether the object itself is mutable or immutable.

^ la mutability est héritée : la mutability d'un field est la même que celle de sa struct....

> Inherited mutability in Rust stops at references. (...) If you want a reference field to be mutable, you have to use &mut on the field type:

^ ... mais apparemment pas si on manipule la struct par une référence ?! (Ou plutôt si le champ lui même est une référence, d'après les exemples ?)

> In Rust we have the Cell and RefCell structs. (...) Whilst that is useful, it means you need to be aware that when you see an immutable object in Rust, it is possible that some parts may actually be mutable.

^ une bonne remarque à garder en tête : quand on dispose d'un objet immutable, il se peut qu'il ne soit pas immutable à 100% s'il contient des Cell/RefCell.

## Destructuring

https://github.com/nrc/r4cppp/blob/master/destructuring.md

> When destructuring structs, the fields don't need to be in order and you can use .. to elide the remaining fields

^ quand on déstructure une struct, on peut ignorer ses fields (y compris tous ses fields) avec `..`

> To create a reference to something in a pattern, you use the ref keyword. For example,

```rs
fn foo(b: Big) {
    let Big { field3: ref x, ref field6, ..} = b;
    println!("pulled out {} and {}", *x, *field6);
}
```

> Here, x and field6 both have type &int and are references to the fields in b.

^ le fait de déstructurer une struct va move les champs de la struct déstructurée ; pour l'éviter (i.e. pour prendre un borrow sur un champ), on utilise cette syntaxe avec `ref field`.

## Destructuring pt2 - match and borrowing

https://github.com/nrc/r4cppp/blob/master/destructuring-2.md

Je n'annote pas, mais ce chapitre traite du cas assez complexe où on déstructure une référence vers un enum

## Arrays and Vectors

https://github.com/nrc/r4cppp/blob/master/arrays.md

> Just like other data structures in Rust, arrays are immutable by default and mutability is inherited.

^ la mutabilité est héritée dans les data structures.

> The size is known dynamically (as opposed to statically in the case of fixed length arrays), and we say that slice types are dynamically sized types

^ les slices sont des DST

> Since a slice is just a sequence of values, the size cannot be stored as part of the slice. Instead it is stored as part of the pointer (remember that slices must always exist as pointer types). A pointer to a slice (like all pointers to DSTs) is a fat pointer - it is two words wide, rather than one, and contains the pointer to the data plus a payload. In the case of slices, the payload is the length of the slice

^ la length des slices est stockée dans le pointeur, qui est donc un fat pointeur.

> A slice can be thought of as a (borrowed) view of an array

^ résumé conceptuel des slices.

## Graphs and arena allocation

https://github.com/nrc/r4cppp/blob/master/graphs/README.md

^ l'article qui m'a fait lire la série. Il décrit une façon d'implémenter un graphe, ce qui n'est pas si évident que ça en rust, à cause du borrow-checker et de l'ownership.

## Closures and first-class functions

https://github.com/nrc/r4cppp/blob/master/closures.md

> A FnMut represents an object which can be called and can be mutated during that call. This doesn't apply to normal functions, but for closures it means the closure can mutate its environment.

^ `FnMut` veut dire qu'on a une closure qui peut muter son environnement.

> You can use methods in the same way as functions - take pointers to them store them in variables, etc

^ on peut passer des méthodes dans des higher order functions comme on le ferait avec des fonctions classiques ou des closures.

C'est un sujet avancé, mais à noter que les fonctions qu'on donne à manger à une higher order function peuvent être générique sur les lifetimes, avec une syntaxe spéciale `for<'a>` :

```rs
fn foo<'b, 'c, F>(x: &'b Bar, y: &'c Bar, f: F) -> (&'b Baz, &'c Baz)
    where F: for<'a> Fn(&'a Bar) -> &'a Baz
{
    (f(x), f(y))
}
```

> A function type which is generic in this way is called a higher-ranked type

Le paragraphe **Closure flavours** synthétise très bien beaucoup de choses intéressantes sur les closures, je le reproduis quasiment à l'identique :

A closure has two forms of input:

- the arguments which are passed to it explicitly and
- the variables it captures from its environment.

(...)

**For the arguments**, you can declare types instead of letting Rust infer them. You can also declare a return type. Rather than writing `|x| { ... }` you can write `|x: i32| -> String { ... }`.

Whether an argument is owned or borrowed is determined by the types (either declared or inferred).

**For the captured variables**, the type is mostly known from the environment, but Rust does a little extra magic. Should a variable be captured by reference or value?

Rust infers this from the body of the closure. If possible, Rust captures by reference. E.g.,

```rs
fn foo(x: Bar) {
    let f = || { ... x ... };
}
```

- All being well, in the body of `f`, `x` has the type `&Bar` with a lifetime bounded by the scope of `foo`.
- However, if x is mutated, then Rust will infer that the capture is by mutable reference, i.e., `x` has type `&mut Bar`
- If `x` is moved in `f` (e.g., is stored into a variable or field with value type), then Rust infers that the variable must be captured by value, i.e., it has the type `Bar`.

This can be overridden by the programmer (sometimes necessary if the closure will be stored in a field or returned from a function). By using the `move` keyword in front of a closure. Then, all of the captured variables are captured by value. E.g., in `let f = move || { ... x ... };`, `x` would always have type `Bar`.
