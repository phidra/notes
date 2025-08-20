* [Synthèse](#synthèse)
   * [TL;DR](#tldr)
   * [La plupart des types sont Send et Sync](#la-plupart-des-types-sont-send-et-sync)
   * [Quels sont les types non-Send et non-Sync](#quels-sont-les-types-non-send-et-non-sync)
      * [Types qui ne sont pas Send](#types-qui-ne-sont-pas-send)
      * [Types qui ne sont pas Sync](#types-qui-ne-sont-pas-sync)
   * [Marker traits](#marker-traits)
   * [Utiliser un objet non-Send dans un autre thread](#utiliser-un-objet-non-send-dans-un-autre-thread)
* [Source dans cette réponse StackOverflow](#source-dans-cette-réponse-stackoverflow)
   * [Send](#send)
   * [Sync](#sync)
      * [Rendre Sync un type non-Sync](#rendre-sync-un-type-non-sync)
         * [Arc&lt;Mutex&lt;T&gt;&gt;](#arcmutext)
* [Source dans cette autre réponse](#source-dans-cette-autre-réponse)
   * [Send](#send-1)
   * [Sync](#sync-1)
   * [Et la copie ?](#et-la-copie-)
* [Source dans le rustonomicon](#source-dans-le-rustonomicon)
* [Source dans un blogpost intéressant](#source-dans-un-blogpost-intéressant)
   * [Send](#send-2)
      * [Explication](#explication)
      * [Quels objets ne sont pas Send](#quels-objets-ne-sont-pas-send)
      * [Modèle mental](#modèle-mental)
   * [Sync](#sync-2)
      * [Explication](#explication-1)
      * [Quels objets ne sont pas Sync](#quels-objets-ne-sont-pas-sync)
      * [Modèle mental](#modèle-mental-1)
* [Source dans cette question](#source-dans-cette-question)



# Synthèse

## TL;DR

Pour être autorisé à utiliser un objet dans un autre thread que celui qui l'a créé, son type doit _forcément_ être `Sync` et/ou `Send` :

- **Send** pour être autorisé à passer l'objet lui-même (et non une référence) à un thread, en transférant donc son ownership
- **Sync** pour être autorisé à passer une référence vers l'objet à un thread, son ownership restant dans le thread original

La différence entre `Send` et `Sync` :

- **Send** : un type est marqué `Send` si on peut l'utiliser de façon safe dans un thread, **PUIS (pas en même temps)** transférer son ownership et l'utiliser dans un autre :
    - dans un thread A, on créée l'objet et on l'utilise
    - puis on transfére son ownership vers un thread B, qui peut l'utiliser et le dropper
- **Sync** : un type est marqué `Sync` si c'est safe que plusieurs threads utilisent **en même temps** des références (constantes) vers le même objet

Les noms des traits reflètent ce qu'ils indiquent :

- **Send** = on **envoie** l'objet vers un nouveau thread (qui en prend l'ownership)
- **Sync** = plusieurs threads différents utilisent l'objet en même temps ; il y a donc plusieurs accès concurrents, l'objet doit être **synchronisé**

## La plupart des types sont Send et Sync

La plupart des types sont `Send` et `Sync`, en effet :

- **Send** : la plupart du temps, rien ne s'oppose à ce qu'on droppe dans un thread B un type qui a été créé dans un thread A
- **Sync** : en rust, si on a plusieurs références vers un objet, elles seront forcément constantes (non-mutables) ; la plupart du temps, on peut donc les utiliser dans plusieurs threads en même temps sans risquer le data-race, vu qu'aucune référence ne permettra de muter l'objet...

## Quels sont les types non-Send et non-Sync

### Types qui ne sont pas Send

Un type non-Send veut dire "l'objet doit être droppé par le thread qui l'a créé".

Ou dit autrement : "l'objet ne doit pas être copié/moved vers un autre thread".

Exemples de raisons :

- l'objet partage un état avec le thread original (c'est le cas de `Rc` : le compteur de référence est utilisé par `Drop`, mais peut aussi être encore utilisé dans le thread original par _autre_ clone du même Rc)
- le `Drop` de l'objet utilise le [thread-local storage](https://fr.wikipedia.org/wiki/Thread_Local_Storage)
- des carabistouilles de l'allocateur, e.g. son allocateur impose de le désallouer dans le même thread que celui dans lequel il a été construit

### Types qui ne sont pas Sync

Un type non-Sync veut dire "seule une référence (constante) de l'objet peut être utilisée à la fois".

Dit autrement : il n'est pas safe d'utiliser de façon concurrente deux référence constantes vers le même objet.

Ça signifie donc obligatoirement qu'une méthode constante mute l'objet = le type a l'interior mutability.

Ici aussi, un exemple est Rc : des méthodes constantes de l'objet vont muter le compteur de référence, d'une façon non-threadsafe.

## Marker traits

Les traits `Send` et `Sync` sont des [marker traits](https://doc.rust-lang.org/std/marker/trait.Copy.html) = il n'y a pas de méthodes associées à ces traits, ils permettent "juste" de changer la sémantique des types.

Dit autrement, ces traits sont "juste" un moyen manuel de faire passer un message au compilateur concernant un type `T` :

- j'ai marqué `T` comme étant `Send`, tu peux donc autoriser le type à être moved dans un autre thread (ce qui est interdit sinon)
- j'ai marqué `T` comme étant `Sync`, tu peux donc autoriser plusieurs références à coexister dans différents threads (ce qui est interdit sinon)

## Utiliser un objet non-Send dans un autre thread

D'après ce que je comprends, en théorie rien n'interdirait d'avoir un objet non-Send et non-Sync utilisé dans un autre thread : l'objet resterait owned par son thread, et l'autre thread utiliserait une unique référence (constante ou mutable, peu importe).

Mais j'ai testé [dans une POC](https://github.com/phidra/pocs/blob/e0805b6b3806e4ab3be624c97e8ebf84d33ad958/rust/LANGUAGE_sendsync_01_pass_a_single_ref_to_a_thread/src/main.rs) : rust est un peu conservateur et interdit même de passer une UNIQUE référence dans un thread si le type n'est pas `Sync`.

C'est cohérent avec la définition donnée par [le rustonomicon](https://doc.rust-lang.org/nomicon/send-and-sync.html) :

> - T is Sync if and only if &T is Send

Pour envoyer une référence à un thread détaché, même pour une unique référence, le type doit être `Sync`.

# Source dans cette réponse StackOverflow

[Cette réponse StackOverflow](https://stackoverflow.com/questions/59428096/understanding-the-send-trait/60109068#60109068) explique l'esprit derrière `Send` et `Sync` :

> - `Sync` allows an object to to be used by two threads A and B at the same time.
> - `Send` allows an object to be used by two threads A and B at different times. Thread A can create and use an object, then send it to thread B, so thread B can use the object while thread A cannot.

## Send

`Send` = l'objet peut être créé/utilisé de façon safe dans un thread A **PUIS, PLUS TARD** dans un thread B.

Dit autrement, l'objet n'est pas marié à un thread en particulier, et peut être utilisé ailleurs, à condition que les deux utilisations ne soient pas faites en même temps.

C'est le cas de la plupart des types.

Un contre-exemple serait un objet "couplé" à un thread en particulier, par exemple :

- il utilise le [thread-local storage](https://fr.wikipedia.org/wiki/Thread_Local_Storage)
- son allocateur impose de le désallouer dans le même thread que celui dans lequel il a été construit
- il reste couplé à un état partagé avec le thread original, et cet état partagé n'est pas safe à muter depuis plusieur threads (c'est le cas de `Rc`)

## Sync

`Sync` = l'objet peut être utilisé de façon safe dans un thread A **ET EN MÊME TEMPS** dans un thread B.

C'est ce qu'on entend habituellement par _threadsafe_.

Sachant que rust interdit de toutes façons de voir coexister une référence mutable (en écriture) avec des références const (en lecture-seule), les deux threads utilisent forcément deux références non-mutables. La question est donc : _"est-il safe que deux threads utilisent en même temps deux références constantes sur un objet ?"_.

Pour la plupart des types, ce sera le cas : tant que personne ne _mute_ une data, on peut avoir autant de threads qu'on veut qui _lisent_ la data en même temps.

Les contre-exemples sont tous dus à _l'interior mutability_, qui permet de muter un objet "en le lisant", i.e. à partir d'une référence constante, par exemple avec un `Rc` :

- l'appel à la méthode `clone` va muter l'objet (pour incrémenter le compteur de référence)
- cette mutation n'est pas threadsafe ! (le compteur de référence n'est pas protégé par un mutex, car `Rc` est censé être utilisé en monothread)
- donc si deux threads différents appellent `clone` en même temps sur deux références vers le _même_ objet de type `Rc`, on a un data-race (et on peut p.ex. n'incrémenter le compteur qu'une seule fois...)

### Rendre Sync un type non-Sync

Une "solution" à ce problème avec `Rc` est de protéger le compteur de références avec un mutex, c'est justement ce que fait `Arc` : deux threads différents peuvent muter (via des références constantes, donc via l'interior mutability) deux références différentes sur un même `Arc` "en même temps", il n'y aura pas de data-race sur le compteur de référence.

Plus généralement, si on protège ce qui peut être muté par l'interior-mutability (avec un `Mutex`, un `RwLock`, ou un atomic), alors le type devient `Sync`.

#### `Arc<Mutex<T>>`

Attention que `Arc<T>` n'est `Sync` **QUE** si son type `T` est lui-même `Sync`... Quand ça n'est pas le cas, il "suffit" de rendre `T` Sync avec un `Mutex`, d'où la prolifération de `Arc<Mutex<T>>`.

# Source dans cette autre réponse

[Cette autre réponse juste après](https://stackoverflow.com/questions/59428096/understanding-the-send-trait/59428741#59428741) formule les choses de façon intéressante + précise ce qui concerne la copie.

## Send

> Non-Send types can only ever be owned by a single thread, since they cannot be moved or copied to other threads.

^ Les types qui ne sont PAS `Send` sont en quelque sorte _couplés à un unique thread_.

NDM : mais même si c'est un peu tarabiscoté, si le type est tout de même `Sync` on peut éventuellement passer des références à d'autres threads, de sorte qu'ils peuvent être _utilisés_ par d'autres threads, même s'ils restent _owned_ par un unique thread.

> Most types that own data will be Send, as there are few cases where data can't be moved from one thread to another (and not be accessed from the original thread afterwards).

^ La plupart du temps, un type sera `Send`, car les cas où on ne peut pas move un objet d'un thread à un autre sont rares.

La réponse donne quelques exceptions :

- Raw pointers are never Send nor Sync.
- Types that share ownership of data without thread synchronization (for instance Rc).
- Types that borrow data that is not Sync.
- Types from external libraries or the operating system that are not thread safe

Enfin, sous la réponse, un commentaire donne un exemple de type `Sync` mais pas `Send` :

> One contrived example of Sync but no Send: (...) - Struct Foo{} uses thread local storage to keep some state, and exposes thread safe get/set methods for that state. Now the &Foo can be passed around, but not Foo itself.

## Sync

> Non-Sync types can only be used by a single thread at any single time, since their references cannot be moved or copied to other threads.

^ Les types qui ne sont PAS `Sync` ne peuvent être utilisés que par _un thread à la fois_.

> They can still be moved between threads if they implement Send.

^ tant qu'il n'y a jamais qu'un seul thread qui utilise l'objet à la fois (y compris via une référence), on est bons ; et notamment, on peut éventuellement _transférer_ l'objet (son ownership) d'un thread à un autre.

NDM : et je suppose que l'absence de `Sync` empêche d'envoyer une référence dans un autre thread, même si en théorie, des références pourraient être utilisées par plusieurs threads, à condition que ce soit obligatoirement l'une après l'autre (pas en même temps).


## Et la copie ?

En rust, lorsqu'on "move" un objet quelque part, et que le type de l'objet implémente `Copy` ([un autre marker trait](https://doc.rust-lang.org/std/marker/trait.Copy.html), d'ailleurs), alors l'objet peut être plutôt copié, i.e. on peut "transférer" l'objet via un équivalent de `memcpy` de ses bits

Ça reste vrai pour les threads : lorsqu'on move un objet qui implémente `Copy` dans un thread, alors il peut être copié dans le thread.

# Source dans le rustonomicon

Ce qu'en dit [le rustonomicon](https://doc.rust-lang.org/nomicon/send-and-sync.html) :


> - A type is Send if it is safe to send it to another thread.
> - A type is Sync if it is safe to share between threads
> - T is Sync if and only if &T is Send

Ma compréhension de cette phrase (un peu obscure, qui ne nous aide pas beaucoup concrètement...), c'est que si on peut move de façon sécure une **référence** dans un thread détaché, alors le type est `Sync` par définition.

C'est cohérent avec les autres explications que j'ai annotées : si on peut move de façon safe une référence sur un objet dans un thread détaché, alors le type est safe à utiliser en même temps depuis plusieurs threads, donc le type est `Sync`.

> Since they're marker traits (they have no associated items like methods), correctly implemented simply means that they have the intrinsic properties an implementor should have. Incorrectly implementing Send or Sync can cause Undefined Behavior.

En gros, ces traits sont "simplement" des façons pour un dev d'indiquer des propriétés d'un type (ici, sur comment ils peuvent être utilisés de façon safe dans un contexte concurrent) sans rien changer à part cette indication.

> Almost all primitives are Send and Sync, and as a consequence pretty much all types you'll ever interact with are Send and Sync.
>
> Major exceptions include:
>
> - raw pointers are neither Send nor Sync (because they have no safety guards).
> - UnsafeCell isn't Sync (and therefore Cell and RefCell aren't).
> - Rc isn't Send or Sync (because the refcount is shared and unsynchronized).

Comme je dis ailleurs, la plupart des types usuels sont `Send` et `Sync`.

`Rc` n'est pas `Send` (selon moi, car le fait de dropper un `Rc` va muter le shared-state = le compteur de référence), ni `Sync` (selon moi, car il est possible de muter le shared-state via l'utilisation d'une méthode de `Rc` = `obj.clone()`)

`UnsafeCell` n'est pas `Sync`, car c'est la primitive de base qui permet l'interior mutability (donc utiliser une méthode de son API, même depuis une référence constante, **peut** muter l'objet). Plus tricky, et à confirmer : `UnsafeCell` est `Send` car le fait de dropper une `UnsafeCell` ne déclenche pas d'opération non-threadsafe.

---

Enfin, [cette autre page](https://doc.rust-lang.org/nomicon/races.html) du rustonomicon confirme que c'est surtout à cause de l'interior mutability qu'on a besoin de `Send` et `Sync` :

> Data races are prevented mostly through Rust's ownership system alone: it's impossible to alias a mutable reference, so it's impossible to perform a data race. Interior mutability makes this more complicated, which is largely why we have the Send and Sync traits

# Source dans un blogpost intéressant

[Ce blogpost](https://blog.cuongle.dev/p/this-sendsync-secret-separates-professional-and-amateur) (que j'ai lu le 15 août 2025) explique très bien les traits `Send` et `Sync`.

## Send

### Explication

Un objet d'un type qui a le trait `Send` peut sans risque être **moved** dans un thread séparé (_envoyé_, d'où le nom du trait).

C'est donc une notion liée à **l'ownership de l'objet** : l'ownership peut être transférée dans le thread séparé.

Ma compréhension des choses, c'est que la plupart des types usuels sont `Send`, car il ne se passe rien de particulier quand on move un objet usuel quelque part.

### Quels objets ne sont pas Send

Les cas qui peuvent ne pas être `Send` sont ceux où le fait de move un objet "laisse une trace" de l'objet à l'endroit d'où on l'a moved from.

L'exemple donné dans l'article et celui de `Rc` : deux objets `Rc` différents peuvent partager un état (le compteur de référence), comme dans l'exemple suivant :


```rs
let obj1 = Rc::new(42);
let obj2 = obj1.clone();
// À ce stade, obj1 et obj2 sont deux objets différents.
// Mais ils partagent un état caché = le compteur de référence.
```

Dans ce cas, est-il safe de transférer l'un des deux objets dans un thread séparé ?

```rs
thread::spawn(move || {
    drop(obj1);
});
drop(rc2);
```

Dans le code ci-dessus, l'un des deux objest (de type `Rc`) est moved dans un thread détaché, et l'autre reste dans le thread original. Est-ce safe ?

La réponse est... ça dépend :

- stricto-sensu, **NON** : si jamais l'objet dont l'ownership est transféré dans le thread n°2 partage un état avec un objet dans le thread n°1, les deux objets peuvent se marcher sur les pieds quand ils muteront l'état partagé : le type n'est pas `Send`
- mais en pratique, la plupart du temps **SI** : la plupart des objets n'ont pas d'état partagé comme le compteur de référence de `Rc`...
- et parfois, même pour des objets qui ont un état partagé, **OUI** : si le type est _spécifiquement conçu_ pour que ce partage d'état soit threadsafe (comme `Arc`, qui protège le compteur de référence avec un mutex), alors on peut sereinement transférer l'ownership de l'un des objets dans un thread séparé, le type est `Send`

### Modèle mental

J'aime beaucoup son "modèle mental" pour savoir si un type est `Send` :

- Deux objets du type peuvent-ils partager un état de façon cachée ?
    - si non, le type est `Send` (on peut transférer son ownership sans risquer de laisser des traces derrière soi)
    - si oui, le type n'est `Send` que si ce partage d'état est conçu pour être threadsafe ; sinon, il n'est pas `Send`

**EDIT** : NDM = la question plus générale est "l'objet est-il couplé au thread dans lequel il était ?" Ça peut inclure autre chose qu'un état partagé, comme l'utlisation du thread-local storage.

## Sync

### Explication

Avec un objet d'un type qui a le trait `Sync`, on peut sans risque partager des références entre plusieurs threads.

(en rust, les multiples références sont forcément en lecture-seule ! On ne peut jamais avoir qu'une unique référence mutable, qui ne peut donc pas être partagée entre plusieurs threads)

C'est donc une notion liée à **l'utilisation de références vers un objet** : plusieurs threads peuvent sans risque utiliser des références constantes vers le même objet.


### Quels objets ne sont pas Sync

Ma compréhension des choses, c'est que la plupart des types usuels sont `Sync`, car comme on a plusieurs références **non-mutables**, on ne peut que les "lire", et la plupart du temps, le simple fait de _lire_ l'état d'un objet ne le mute pas.

Ainsi, plusieurs threads peuvent sans risque lire en même temps l'état d'un objet.

Les cas qui peuvent ne pas être `Sync` sont ceux où le fait de "lire" l'état d'un objet se retrouve en fait à _modifier_ l'objet : c'est **l'interior mutability** où une opération sur une référence constante peut tout de même muter l'objet sous-jacent (e.g. `Cell<T>`).

### Modèle mental

Un objet est-il `Sync` :

- La "lecture" d'un objet (= son utilisation via une référence const) peut-elle muter l'objet ?
    - si non, le type est `Sync` (plusieurs threads peuvent sans risque lire l'objet en même temps)
    - si oui, le type n'est `Sync` que si la mutation de l'objet est conçue pour être threadsafe

# Source dans cette question

Sur le forum de rust-lang.org, il y a [une question](https://users.rust-lang.org/t/sync-but-not-send/21551) _Sync but not Send_ sur le sujet :

> We have several types which are Send but not Sync (...) Reasoning is easy to understand: when the type has interior mutability, we must be sure that we mutate it from one place only, but this place can be everywhere as long as it is singular.

^ Quand le type a de l'interior mutability, il faut faire attention à ne le muter que d'un endroit à la fois.

---

Le thread est intéressant car il donne des exemples de types qui sont `Sync` mais pas `Send` :

> I've heard that there are some ffi types which must be destroyed on the same thread on which they are created, but I can't give a concrete example.

^ Certains types externes à rust doivent être détruits dans le même thread que celui qui les ont créés.

> A contrived example would be a struct which, on creation, puts something to the thread local storage, and accesses that info in Drop.

^ une raison pour qu'un type ne soit pas `Send` (tout en pouvant être `Sync` si besoin).

> Here's a concrete example using the "thread local storage in Drop" use case: OpenSpan<Attached>.The crate tracks the current tracing context in thread local storage and restores the previous on drop, so it can't be moved across threads.

(le lien d'OpenSpan pointe [ici](https://docs.rs/zipkin/0.3.1/zipkin/struct.OpenSpan.html))

^ Cas concret du cas précédent = une crate place quelque chose dans le thread-local storage, et y accède lors de son `Drop`.






