# super, Python's most misunderstood feature.

- **url** = https://www.youtube.com/watch?v=X1PQ7zzltz4
- **type** = vidéo
- **auteur** = [James MURPHY](https://mcoding.io/about-james-murphy) = mathématicien et codeur C++/python avec un background dans la finance
- **date de publication** = 2022-03-15
- **source** = [sa chaîne youtube](https://www.youtube.com/c/mCodingWithJamesMurphy/about)

- **tags** = language>python ; topic>super ; level>intermediate

**TL;DR** : quelques misconceptions et explications sur `super` (qui ne renvoie **pas** la classe parente)

* [super, Python's most misunderstood feature.](#super-pythons-most-misunderstood-feature)
   * [Ce que renvoie super](#ce-que-renvoie-super)
   * [Bonnes pratiques quand on design des hiérarchies de classes](#bonnes-pratiques-quand-on-design-des-hiérarchies-de-classes)
   * [Comment fonctionne super](#comment-fonctionne-super)

## Ce que renvoie super

NdM = en préambule, si on utilise `super`, c'est qu'on overloade une méthode (ou un attribut) dans une hiérarchie de classe.

En deux mots, **super** ne renvoie pas la classe parent, mais la classe _next-in-line_ dans le [MRO](https://docs.python.org/3/glossary.html#term-method-resolution-order) de l'objet courant.

Au passage, le `MRO = Method Resolution Order` est mal nommé, car il permet d'accéder non seulement aux méthodes, mais plus généralement à tout attribut d'un objet.

99% du temps, ça correspond bien à la classe parente, car on est dans une situation dans ce genre-là :

```python
class Parent:
    def hello(self, name: str):
        print(f"Hello from Parent, {name}")

class Child(Parent):
    def hello(self, name: str):
        super().hello(name)  # <-- appellera Parent.hello(name)
        print(f"And also hello from Child, {name}")
```

Mais comme `super` suit aveuglément le MRO, ça peut être le sibling ! Par exemple :

```python
class Root:
    def f(self):
        print("from Root")

class A(Root):
    def f(self):
        super().f()
        print("from A")

class B(Root):
    def f(self):
        super().f()
        print("from B")

class C(A, B):
    def f(self):
        super().f()
        print("from C")

c = C()
c.f()
```

Ici, on a l'héritage en diamant suivant :

```
   Root
   /  \
  A    B
   \  /
    C
```

Le MRO d'une instance de la classe `C` est donc `C A B Root object`. Du coup, quand `A.f` est appelée (par l'appel à `super().f()` de `C.f`), et qu'elle appelle à son tour `super().f()`, ça n'est **PAS** `Root.f()`, i.e. la méthode de la classe PARENTE de `A` qui est appelée, mais c'est plutôt `B.f()`, méthode de la classe SOEUR de `A` !

Un autre cas où `super` n'appelle pas la classe parente (mais plutôt la classe grand-parente) :

```python
class GrandParent:
    def f(self):
        print("from GrandParent")

class Parent(GrandParent):
    pass  # pas de méthode f !

class Child(Parent):
    def f(self):
        super().f()
        print("from Child")

child = Child()
child.f()
```

Ici, le MRO d'une instance de `Child` est `Child Parent GrandParent object`. Quand `child.f()` appelle `super().f()`, comme `Parent.f` est en fait `GrandParent.f`, c'est bien `GrandParent.f` qui est appelée !

## Bonnes pratiques quand on design des hiérarchies de classes

Il explique quelques règles de fonctionnement du MRO :
- commence par la classe et finit par `object`
- un enfant apparaît avant ses parents + le MRO respectera l'ordre des parents tel que déclaré dans la classe : `class C(A, B)` → dans le MRO, on aura obligatoirement `C < A < B`
- le MRO d'un enfant est une extension du MRO de chacun de ses parents (i.e. si on prend le MRO d'un enfant, il suffit de lui retirer des items pour pouvoir retrouver le MRO de chacun de ses parents). Cette proprité garantit que tout parent d'un parent d'un parent (etc.) apparaît forcément dans le MRO d'un fils, même lointain
- si les règles précédentes ne peuvent pas êtrerespectées, la définition de classe échouera, p.ex. cette définition échouera :
    ```python
    class A:
        pass

    class B:
        pass

    class C(A, B):  # dans le MRO, C < A < B
        pass

    class D(A, C):  # dans le MRO, D < A < C
        pass

    # on a la contradiction C < A et A < C, ce qui provoque l'exception :
     # TypeError: Cannot create a consistent method resolution
     # order (MRO) for bases A, C
    ```

Puis, il donne des bonnes pratiques pour designer une hiérarchie de classes. Problèmes à résoudre :

- comment être sûr que la méthode parente est appelée ?
- comment être sûr que toutes les méthodes overloadées, appelées par `super` ont la même interface en terme d'arguments attendus ?

Bonnes pratiques, qu'il appelle _cooperative inheritance_ (sans lesquelles `super` n'est pas garanti de marcher avec une hiérarchie de classes arbitraire) :

 1. Avoir une classe racine de laquelle tout le monde hérite.
 2. si on utilise `super` dans UNE des méthodes overloadées des classes de la hiérarchie, il faut l'utiliser PARTOUT, de sorte que la méthode de la classe racine finisse par être appelée
 3. toutes les différentes méthodes overloadées doivent respecter la même signature (ou faire de _l'argument peeling_ = utiliser `*args` et `**kwargs` et chaque méthode overloadée prélève les arguments qui l'intéressent, mais je ne suis pas fan de cette "bonne pratique"...)

## Comment fonctionne super

- `super` est une classe, `super()` est une instanciation
- la classe "`super`" est un proxy qui wrappe un objet, et qui forwarde les _attribute-lookups_ à cet objet
- conceptuellement, elle inspecte la stackframe de l'appelant pour récupérer la variable `V` sur laquelle on a appelé `f`
- python implémente un comportement particulier pour accéder à des attributs spéciaux dès qu'on utilise `super` dans une méthode, c'est notamment ce qui permet à `super` de connaître la classe de `V`
- la "vraie" forme de `super` est décrite à 16:40 = c'est un wrapper, qui connait la classe et le self de l'objet courant (depuis lequel on a appelé `super`), et qui utilise donc son MRO pour trouver le "next in line"

