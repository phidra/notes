# Is Inheritance That Evil?

- **url** = https://thevaluable.dev/guide-inheritance-oop/
- **type** = post
- **auteur** = Matthieu CNEUDE, dev français assez actif [site perso](https://matthieucneude.com/), [twitter](https://twitter.com/Cneude_Matthieu), [github](https://github.com/Phantas0s), [son blog](https://thevaluable.dev/page/about/)
- **date de publication** = 2021-03-27
- **source** = https://thevaluable.dev/

**Tl;DR** :
- Avis critique de l'héritage et du lieu commun "il ne faut jamais utiliser l'héritage, il vaut mieux utiliser la composition".
- Intéressante remise en contexte historique.
- Montre notamment quelques défauts de la composition, c'est d'ailleurs l'un des messages de l'article : _la composition n'est pas une silver bullet_.
- Selon lui, c'est parce qu'on a oublié que "hériter c'est coupler" qu'on utilise mal l'héritage. Mais si on l'utilise à bon escient (par exemple là où le couplage est pertinent), alors l'héritage est tout à fait acceptable.
- Propose des conseils pour utiliser l'héritage correctement, par exemple à utiliser pour les classes "mécaniques" (qui varient peu), ou suggère de considérer une hiérarchie d'héritage comme un tout cohérent, et c'est cet ensemble cohérent qui est encapsulé.
- Deux décisions importantes : 1. cohesion : qu'est-ce qui devrait être ensemble et qu'est-ce qui ne devrait pas ? 2. Quand est-il judicieux d'utiliser ce qu'il appelle _interface constructs_ (héritage d'interface) ?


> At that point, both Davina and Dave agree to speak mostly about single inheritance.

----

> For now, we only talked about inheritance as a way to inherit the implementation of superclasses in subclasses. That’s not all: many programming languages allow you to substitute a superclass by one of its subclass thanks to polymorphism. The subclasses can be seen as specialization of their superclasses.

Cette utilisation de l'héritage est la plus importante à mes yeux.

> OOP was never meant to represent objects or living creatures surrounding our daily life, like a dog, a car, or a coffee machine.

----

> What problem Dan Ingalls tried to solve with inheritance? Code reuse.

Hum, c'est justement une mauvaise utilisation de l'héritage à mes yeux...

> Similar, but not identical: only the interface was inherited, not the implementation.

----

> When you read in a random tutorial that a class Dog and a class Platypus inherit from a class Animal, do they speak about code reuse?” [...] Not at all. You speak about specialization: a Cat is a specialized form of Animal.

----

> inheritance is a concept which can bring many powerful features,

----

> In languages with inheritance, a data abstraction implementation (i.e., a class) has two kinds of users. There are the “outsiders” who simply use the objects by calling the operations. But in addition there are the “insiders.” These are the subclasses, which are typically permitted to violate encapsulation

Intéressante distinction entre **outsiders** qui voient la classe comme une boîte noire, et **insiders**, qui ont accès aux rouages de celle-ci, et qui sont donc pas concernés par l'encapsulation.

> If you modify the base class Parser, six other classes will be affected!

Ça, c'est dans le cas où on utilise l'héritage pour réutiliser du code.

> Don’t get me wrong: it can be beneficial if you want that each change of a superclass affects every subclass at every level below [...] But you need to be sure that your classes are very cohesive, that is, the change of any superclass needs to affect every subclass.

Cohésion = les classes ont les mêmes raisons de changer -> quand on hérite d'une classe, il faut avoir les mêmes raisons de changer qu'elle.

> Objects of subtypes should behave like those of supertypes if used via supertype methods. (Barbara Liskov)

----

> Said differently: don’t override anything.

NdM : est-ce que ça ne contredit pas l'intérêt même du polymorphisme ? Pas nécessairement (cf. le pattern template-method, où on n'override rien, on se contente d'implémenter des méthodes virtuelles pures).

> From “the behavior should stay the same”, we know think that “the behavior shouldn’t break the application”

----

> From the example above, the class JSONParser implements the same interface as Parser, but one method count count the number of lines and the other count the number of JSON objects. Interface substitution is not what Barbara Liskov was speaking about, and it won’t save your codebase if you try to mix specialization and overriding.

C'est important : overrider la méthode `count` (pour lui faire compter des objets json plutôt que des lignes) **a violé le LSP** tel que défini par Barbara Liskov !

> inheritance is only interesting for Liskov in the context of subtyping; she doesn’t see any value to use it for inheriting implementation.
> Why? Because inheritance is not the only solution for code reuse. Many prefer using composition.

----

> It’s one of the reason why Mathematical notation was invented at the first place: to avoid the ambiguity of natural language.

`is` vs `has` vs `use`

> It doesn’t matter if it’s called aggregation, composition, or delegation. At the end, it boils down to the same simple mechanism: injecting an object into another one.
> [...]
> One thing is certain: this book gave to beginners the perfect pretext to show how smart they are by instantly changing a healthy codebase into a legacy mess full of Singleton and Abstract Factories.
> [...]
> Inheritance and object composition thus work together.
> [...]
> According to this book, we should favor composition not because inheritance is evil, but because nobody uses it correctly. Well, hopefully we understand it better now.

Pas clair...

> As we saw with inheritance, the problem of tight coupling (or the benefit of cohesion) will affect every layer down the inheritance tree.

Important : **l'héritage, c'est du couplage !**

(et il est (un peu) caché : si A utilise B, et que B hérite de C, alors A est couplé à C de façon pas forcément très apparente dans le code)

> Let’s say that you want to use 10 methods from 3 different objects and you want to add some implementation on top: you’ll need to inject your 3 objects, create 10 methods in your new class wrapping the 10 methods of the objects injected, and add more methods to take care of your new functionality.

Intéressant : si on veut **exposer** des méthodes de sous objets (par opposition à **utiliser**), la composition peut être lourde à implémenter, car elle nécessite de wrapper ce qu'on veut exposer...

> That’s true, answers Davina. But it would break the encapsulation of our objects Shipment in that case.

...et la "solution" consistant à exposer l'objet membre lui même plutôt que de le wrapper n'en est pas une, car elle viole l'encapsulation !

> Composition doesn’t bring you the benefits of subtyping either. When you inject an object to a class, you’ll need to inject a precise object if your language has some sort of type checking, and nothing else. On that regards, it constrains you (which can be a good thing!). If your language doesn’t have any type checking and you can just give any object to the constructor of your class, it doesn’t mean that it will work as intended. The problem stays the same.

Je suis en désaccord sur le fait que c'est problématique : par exemple, c'est justement ce qui permet de mocker (à mes yeux l'un des intérêts principaux de l'injection de dépendance).

> At least, an inheritance hierarchy can indicate what object you can use instead of another and, if it follows a strict form of LSP, nothing should break.

Mouais... Mais à quel prix ? Au prix de la lourdeur et de la rigidité d'une hiérarchie d'héritage.

> This power is its biggest problem: many developers, not knowing all the implications we saw above, have a tendency to misuse inheritance, tightly coupling everything in huge inheritance hierarchies, which led to the Mantra of Composition.

Intéressant : selon lui, c'est parce qu'on a oublié que "hériter c'est coupler" qu'on utilise mal l'héritage. Mais si on l'utilise à bon escient (par exemple là où le couplage est pertinent), alors l'héritage est "acceptable".

> But multiple inheritance is even more flawed: the possibility for a subclass to have more than one superclass is making everything very ambiguous. As an example, you can look at the diamond problem. Additionally, multiple inheritance is very complex to implement in a programming language.

----

> but doesn’t guarantee that the substitution will work!

Ce point est important : le LSP n'est pas automatiquement respecté !

> Never use inheritance for classes which are related to the business domain.
> Sometimes use inheritance for reusing implementation of classes bringing some mechanics.
> If you use subtypes, always try to respect the original strictness of LSP as much as possible.
> If you use the interface constructs for subtyping, keep in mind that the implementation of the interface can still break everything.

Ces rules of thumb sont intéressantes.

> You ask Davina: “what do you mean by mechanics”?
> “These classes are everything which are not domain objects. For example, our classes to parse files represent some mechanisms: they don’t represent anything from our business, they’re just general constructs to parse some files.
> [...]
> For example, it’s not very likely that the world will come up tomorrow with a different definition of stacks. That’s why the object Stack won’t need many changes overtime.

Selon lui, l'héritage est acceptable pour les classes "mécaniques", car elles ne vont pas beaucoup changer.

> the hierarchy itself is encapsulated from its outside.

Intéressant point de vue : il faut considérer une hiérarchie d'héritage comme un tout cohérent. C'est cet ensemble cohérent qui est encapsulé

> Your system can become hard to maintain if you mix two of the Three Power Gems of Inheritance in the same soup:
> Inheritance of implementation.
> Substituting superclasses with their subclasses (subtyping if it follows some rules).
> Multilevel inheritance.
> [...]
> Most of the time, composition seems to be the best alternative to inheritance of implementation.
> Everything is tightly coupled in an inheritance hierarchy, but this blurb of classes is still encapsulated from the outside.
> [...]
> Inheritance can be useful for the part of your system which won’t change too much (mechanical part).

----

> If you need to retain one thing from all of that: don’t use DRY, or inheritance, or composition before you understand clearly what’s the problem you’re trying to solve and its context. These concepts should be used when you refactor you code; consider the first writing as a messy draft and, in that spirit, defer all the important decisions making your design hard to change as much as you can.

Je me retrouve parfaitement dans ce conseil : j'ai du mal à prévoir une hiérarchie définie à l'avance sans bien connaître mon problème, du coup ça m'est souvent plus facile de faire un premier draft non-pérenne pour bien maîtriser le sujet avant de choisir la bonne archi à utiliser.

> What should be together and what should not (cohesion) is one of these important decision. What should be under a layer of indirection using interface constructs is another.

Deux points également très importants !


