# Really Simple CQRS

- **url** = https://kalele.io/really-simple-cqrs/
- **type** = post
- **auteur** = [Vaughn VERNON](https://vaughnvernon.com/), auteur, formateur
- **date de publication** = 2022-10-22
- **source** = [le blog de Kalele](https://kalele.io/blog/), boîte de consulting/formation
- **tags** = language>agnostic ; topic>CQRS ; topic>CQS ; level>beginner

TL;DR = CQRS va plus loin que le CQS principle : non seulement il faut avoir des méthodes différentes pour consulter d'une part et modifier d'autre part l'état de l'application, mais il faut carrément que pour le client externe, les deux utilisations soient faites selon des interfaces différentes.

> users tend to **view** data in quantities, shapes, and sizes that is different from the way they **modify** data.
>
> the strengths of CQRS are in enabling developers to design for state mutation separately from state queries. The state mutation operations are optimized for creates and updates. The operations and data used for querying are optimized for how the user tends view the system state. Both of these sets of operations and states can be designed and scaled separately

CQS par Bertrand MEYER (Eiffel) est le précurseur de CQRS :

> CQS, or Command-Query Separation states that a software interface abstraction is designed with two types of methods. One method type is known as a Command method, and the other method type is known as a Query method. A Command method modifies or mutates the state underneath the interface, but does not answer any portion of that state. A Query method answers the current state beneath the interface, but must not modify that state before answering it.

CQRS, or Command-Query Responsibility Segregation

Ségrégation= on n'a plus une seule interface qui propose deux méthodes différentes (comme dans CQS), on a DEUX interfaces séparées :

- une pour C = les Commands = ce qui modifie l'état de l'application, qui a des side-effects et renvoie void.
- une pour Q = les Queries = ce qui interroge l'état de l'application, qui n'a pas de side-effects et qui renvoie une valeur.
