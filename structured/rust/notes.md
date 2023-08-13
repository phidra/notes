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
   * [fields d'une struct publics vs utiliser new](#fields-dune-struct-publics-vs-utiliser-new)

# Ressources

[Rust by Example](https://doc.rust-lang.org/rust-by-example/)
[Rust By Practice](https://practice.rs/why-exercise.html)
[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
[Effective Rust](https://www.lurklurk.org/effective-rust/)
[Rust Language Cheat Sheet](https://cheats.rs/)
[Easy Rust](https://dhghomon.github.io/easy_rust/Chapter_1.html)
[A Gentle Introduction to Rust](https://stevedonovan.github.io/rust-gentle-intro/readme.html#a-gentle-introduction-to-rust)


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
