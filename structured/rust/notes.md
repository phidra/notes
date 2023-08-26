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
