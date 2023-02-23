# Visualising Test Terminology

- **url** = http://www.natpryce.com/articles/000772.html
- **type** = blogpost
- **auteur** = [Nat PRYCE](http://www.natpryce.com/bio.html), auteur d'un livre sur le testing + utilisateur d'XP/TDD
- **date de publication** = 2009-11-18
- **source** = [son blog](http://www.natpryce.com/)
- **tags** = language>agnostic ; topic>testing ; level>beginner

Ses définition des tests (qui datent de 2009) :

**Unit tests** :

> exercise individual objects or value types or small clusters of objects within in a single process.

NDM : j'ajoute un point qui apparaît sur son schéma (mais pas dans cette description) : les unit tests ne passent pas par le monde extérieur.


**Integration Tests** :

> The term "integration test" can apply to many different kinds of test. In the book, we use the term specifically to mean the test of an abstraction that we own but have implemented with some third-party package. We want to test that our code implementing the abstraction integrates with that third-party package correctly.

^ en gros, c'est la définition moderne des tests d'intégration = on teste uniquement le code de communication avec le monde extérieur.

**Acceptance Tests** :

> Acceptance tests are customer-facing tests that capture the domain logic the system must perform and demonstrates that it performs them.

^ si je comprends bien, on teste l'ensemble de l'appli, mais avec des dépendances mockées permettant que ça aille vite et que ça soit répétable. A priori, ces tests-là, nous les appelons plutôt "tests unitaires" également (sans discriminer avec ce que lui appelle "tests unitaires")

**System Tests** :

> System tests exercise the entire system end-to-end, driving the system through its published remote interfaces and user interface. They also exercise the packaging, deployment and startup of the system. 

^ on utilise la même dénomination, et avec au moins l'un des objectifs communs = vérifier le packaging + déploiement + l "vraie" interface d'utilisation en conditions réelles.
