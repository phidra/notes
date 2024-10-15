# The Ultimate Guide to Error Handling in Python

- **url** = https://blog.miguelgrinberg.com/post/the-ultimate-guide-to-error-handling-in-python
- **type** = blogpost
- **auteur** = [Miguel GRINBERG](https://blog.miguelgrinberg.com/post/about-me) = software engineer
- **date de publication** = 2024-10-07
- **source** = [son blog](https://blog.miguelgrinberg.com/index)
- **tags** = language>pythonn ; topic>error-handling ; level>intermediate


L'article est un peu verbeux pour pas grand chose, mais je l'annote car :

1. il résonne avec mes questionnements actuels sur la gestion des erreurs, c'est l'occasion de résumer un brin
2. il apporte un concept intéressant = les erreurs propres au code (`new`) vs. les erreurs d'un code appelé (`bubbled-up`)


En gros, plusieurs éléments permettent de savoir quoi faire vis-à-vis d'une erreur

**Error `new` vs `bubbled-up` :**

- `new` = l'erreur a été générée par mon propre code
- `bubbled-up` = l'erreur a été générée non pas par mon code, mais par une fonction que mon code a appelé

**Error recoverable vs. non-recoverable :**

- `recoverable` = mon code SAIT comment corriger/contourner l'erreur
- `non-recoverable` = mon code ne sait pas quoi faire de cette erreur → il propage plus haut en espérant qu'un appelant saura le faire

NDM : j'ajoute un troisième cas très important = mon code SAIT que l'erreur est une erreur logique qui ne devrait pas arriver (bug) ; conséquence = il SAIT que l'appelant ne saura pas gérer cette erreur :

- en dev, je PANIC en exitant tout de suite (inutile de lever une exception)
- en prod, je peux lever une exception non-traitable avec dans l'idée qu'un appelant pourra sauver les meubles (e.g. sur un serveur web, il fera en sorte que la requête répond en `HTTP 500` sans faire planter tout le serveur)

**Prod vs. dev :**

- dev = planter salement le plus tôt possible, avec une stacktrace pour investiguer (objectif = détecter les soucis et les corriger)
- prod = lever une exception pour 1. logger proprement le souci 2. permettre à un appelant de haut-niveau d'essayer de contourner le souci, mais continuer à traiter autant que possible pour éviter les crashs/downtime

