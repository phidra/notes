# My Python testing style guide

- **url** = https://blog.thea.codes/my-python-testing-style-guide/
- **type** = post
- **auteur** = [Stargirl Flowers](https://thea.codes/) = une dev python impliquée dans la communauté ; [son github](https://github.com/theacodes)
- **date de publication** = 2017-07-15
- **source** = [son blog](https://blog.thea.codes/)
- **tags** = language>python ; topic>testing ; topic>best-practices ; level>intermediate


**TL;DR** : un article un peu ancien mais intéressant où l'auteure donne des bonnes pratiques sur le testing en python.

* [My Python testing style guide](#my-python-testing-style-guide)
   * [Test names, functions, and classes](#test-names-functions-and-classes)
   * [Assert results and outcome, not the steps needed to get there](#assert-results-and-outcome-not-the-steps-needed-to-get-there)
   * [Use real objects for collaborators whenever possible](#use-real-objects-for-collaborators-whenever-possible)
   * [A mock must always have a spec](#a-mock-must-always-have-a-spec)
   * [Consider using a stub or fake](#consider-using-a-stub-or-fake)
   * [Consider using a spy](#consider-using-a-spy)
   * [Don't give mock/stubs/fakes special names](#dont-give-mockstubsfakes-special-names)
   * [Use factory helpers to create complex collaborators](#use-factory-helpers-to-create-complex-collaborators)
   * [Use fixtures sparingly](#use-fixtures-sparingly)

Les dénominations qu'elle utilise :

- **target** = ce que d'autres articles appellent _System Under Test_ a.k.a _SUT_ vs. **collaborator** = un objet utilisé par le SUT
- **unit-tests** = tout ce qui a un comportement homogène et qu'on peut tester sans interaction avec le monde extérieur (catégorie dans lequel elle inclut ce qui est habituellement appelé **integration tests**) vs. **system tests** = tests qui interagissent avec le monde extérieur

NdM : je remets le titre de chaque chapitre, mais tous ne nécessitent pas le même niveau d'annotations.

## Test names, functions, and classes

RAS

## Assert results and outcome, not the steps needed to get there

Tout est dans le titre, l'idée est que si on teste l'implémentation (e.g. avec l'approche mockist), les tests sont friables.

## Use real objects for collaborators whenever possible

L'objectif est 1. d'attraper des bugs et surtout 2. de se rendre compte que le collaborateur (NdM : supposément écrit par nous-même, et non pas une third-party library) est difficile à utiliser sans side-effect, et donc qu'il faut le refactorer.

## A mock must always have a spec

Si possible, il faut utiliser `mock.create_autospec()` ou `mock.patch(autospec=True)` car sinon, `Mock` accepte sans broncher toutes les utilisations de fonctions/membres imprévus, ce qui empêche de détecter que le mock n'est plus à jour.

C'est un conseil assez répandu, e.g. je suis également tombé dessus récemment [dans cet article](https://hynek.me/articles/what-to-mock-in-5-mins/).

## Consider using a stub or fake

Ses définitions pour ces 3 notions qui visent toutes à remplacer un vrai collaborator, en proposant une interface similaire :

- **mock** = objet sur lequel on peut vérifier les appels effectués
- **stub** = objet "statique" : l'interface renvoie toujours la même donnée, figée
- **fake** = objet dont l'implémentation est "réelle" mais simplifiée par rapport au vrai collaborator (e.g. une _in-memory database_)

Son avis = si on galère à utiliser un mock, ne pas hésiter à utiliser un stub ou un fake.

## Consider using a spy

NdM : du coup c'est pas clair à ses yeux ce qui différencie un mock d'un spy... Possiblement, le caractère "spy" indique simplement qu'on peut enregistrer les appels effectués : un mock est un spy sur un objet vide, mais on peut créer un spy sur un objet réel (et `mock.wraps` permet de faire ça facilement).

## Don't give mock/stubs/fakes special names

Son objectif par ce conseil est que dans le test, on ne se repose pas trop sur le fait que le collaborator n'est pas le vrai collaborator.

## Use factory helpers to create complex collaborators

RAS : tout est dans le titre.

## Use fixtures sparingly

Ce conseil est intéressant : elle suggère de préférer les helpers et autres factories plutôt que d'utiliser les fixtures, car le fonctionnement des fixtures est contre-intuitif. Je n'avais jamais vu les choses sous cet angle, mais à la réflexion, je suis plutôt d'accord avec elle.

Elle suggère de réserver l'usage des fixtures à ce qui nécessite une étape de _setup_ et (surtout) une étape de _teardown_, puisqu'on peut faire ça avec `pytest.fixture` mais qu'on ne peut pas le faire avec un helper :

```python
@pytest.fixture()
def server():
    server = WSGIServer(application=TEST_APP)
    server.start()
    yield server
    server.stop()  # <-- le teardown qui justifie l'usage d'une fixture
```
