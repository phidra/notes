# Really Simple CQRS

- **url** = https://kalele.io/really-simple-cqrs/
- **type** = blogpost
- **auteur** = [Vaughn VERNON](https://vaughnvernon.com/), tech consultant avec 30 ans d'expérience
- **date de publication** = ???
- **source** = [le blog de KALELE](https://kalele.io/), une boîte de consulting qui fait aussi de la domotique
- **tags** = language>agnostic ; topic>architecture ; level>intermediate

**TL;DR** = description succinte du pattern CQRS = Command-Query Responsability Segregation, qui consiste à utiliser deux interfaces séparées pour interagir avec un objet métier : une pour la lecture (Query), l'autre pour l'écriture (Command).

> users tend to **view** data in quantities, shapes, and sizes that is different from the way they **modify** data.

^ C'est la situation que vise à adresser CQRS

> the strengths of CQRS are in enabling developers to design for state mutation separately from state queries. The state mutation operations are optimized for creates and updates. The operations and data used for querying are optimized for how the user tends view the system state. Both of these sets of operations and states can be designed and scaled separately

CQS par Bertrand Meyer (Eiffel) est le précurseur de CQRS :

> CQS, or Command-Query Separation states that a software interface abstraction is designed with two types of methods. One method type is known as a Command method, and the other method type is known as a Query method. A Command method modifies or mutates the state underneath the interface, but does not answer any portion of that state. A Query method answers the current state beneath the interface, but must not modify that state before answering it.

CQRS = Command-Query Responsibility Segregation va plus loin : ségrégation= on n'a plus une seule interface à deux méthodes, on a DEUX interfaces séparées (l'une pour Command, l'autre pour Query)
