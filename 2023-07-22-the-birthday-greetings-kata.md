# The birthday greetings kata

- **url** = http://matteo.vaccari.name/blog/archives/154
- **type** = post
- **auteur** = [Matteo VACCARI](http://matteo.vaccari.name/), _I am an Extreme Programmer. I help individuals, teams and organization become more effective. I work as a developer for Thoughtworks Italia._
- **date de publication** = 2017-??-??
- **source** = [son blog](http://matteo.vaccari.name/blog/)
- **tags** = language>agnostic ; topic>architecture ; topic>architecture-hexagonale ; level>intermediate

TL;DR = le post est une description d'un kata, mais il est surtout intéressant car il fournit de très bonnes descriptions des soucis de la layered architecture et de comment on y répond avec l'architecture hexagonale.

----

> Note that the collaborators of the birthdayService objects are injected in it. Ideally domain code should never use the new operator. The new operator is called from outside the domain code, to set up an aggregate of objects that collaborate together

^ le domain n'instancie pas ses collaborateurs, ils lui sont injectés.

> The goal of this exercise is to come up with a solution that is
>
> - Testable; we should be able to test the internal application logic with no need to ever send a real email.
> - Flexible: we anticipate that the data source in the future could change from a flat file to a relational database, or perhaps a web service. We also anticipate that the email service could soon be replaced with a service that sends greetings through Facebook or some other social network.
> - Well-designed: separate clearly the business logic from the infrastructure.

^ les atouts de l'archi hexagonale.

> A test is not a unit test if:
>
> -It talks to a database
> -It communicates across the network
> -It touches the file system
> -You have to do things to your environment to run it (eg, change config files, comment line)
>
> Tests that do this are integration tests.
> Integration tests have their place; but they should be clearly marked as such, so that we can execute them separately.

^ les tests qui interagissent avec le monde extérieur ne sont pas des unit-tests mais des tests d'intégration.

> The reason we draw this sharp distinction is that unit tests should be
>
> - Very fast; we expect to run thousands of tests per second.
> - Reliable; we don’t want to see tests failing because of random technical problems in external systems.
>
> One way to make code more testable is to use Dependency Injection. This means that an object should never instantiate its collaborator by calling the new operator. It should be passed its collaborators instead. When we work this way we separate classes in two kinds.
>
> - Application logic classes have their collaborators passed into them in the constructor.
> - Configuration classes build a network of objects, setting up their collaborators.
>
> Application logic classes contain a bunch of logic, but no calls to the new operator. Configuration classes contain a bunch of calls to the new operator, but no application logic.

^ un point très intéressant, avec deux types de classes : celles qui font le boulot (et qui se voient injecter leurs collaborateurs), et celles qui préparent le terrain pour les premières (notamment en instanciant les collaborateurs, et en les injectant).

> The traditional way to structure an application in layers is :
>
> ```
> +--------------+
> | presentation |
> |--------------|
> |    domain    |
> |--------------|
> | persistence  |
> +--------------+
> ```
>
> The meaning of a layer diagram is that
>
> - a layer is a set of classes;
> - a class cannot reference classes in layers above; a class can only reference other classes in the same layer, or in the lower layers.
>
> In other words, it should be possible to compile a layer without access to the source code of the layers above it, as long as we have the source code of the layers below.

^ explications sur la layered architecture. Ce n'est pas la première fois que je vois une présentation de l'archi hexagonale comme une évolution de la layered architecture.

> The traditional three-layers architecture has many drawbacks.
>
> - It assumes that an application communicates with only two external systems, the user (through the user interface), and the database. Real applications often have more external systems to deal with than that (...)
> - It links domain code to the persistence layer in a way that makes external APIs pollute domain logic. References to JDBC, SQL or object-relational mapping frameworks APIs creep into the domain logic.
> - It makes it difficult to test domaain logic without invoving the database; that is, it makes it difficult to write unit tests for the domain logic, which is where unit tests should be more useful.

^ les soucis de la layered-architecture, qui sont corrigés en transitionnant vers l'archi hexaghonale. NDM : le point le plus important IMO est le fait que la domain-logic connaisse la database.

> The hexagonal architecture avoids these problems by treating all external systems as equally external. (...) Every external systems is hidden behind a facade that:
>
> - Provides a simplified view of the external system, with only the operations that we need to do with it.
> - Is expressed in terms of the domain model.
>
> The domain model does not depend on any other layer; all other layers depend on the domain model.
>
> ```
> +-----+-------------+----------+
> | gui | file system | database |
> |-----+-------------+----------+
> |          domain              |
> |------------------------------+
> ```
>
> How can we make the domain independent, for instance, of the database? We should define a repository interface that returns domain objects. The interface is defined in the domain layer, and is implemented in the database layer.

^ très bonnes explications sur l'archi hexagonale.
