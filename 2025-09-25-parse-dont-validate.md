# Parse, don’t validate

- **url** = https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
- **type** = blogpost
- **auteur** = [Alexis KING](https://lexi-lambda.github.io/about.html), dev avec un focus functional programming
- **date de publication** = 2019-11-05
- **source** = [son blog](https://lexi-lambda.github.io/)
- **tags** = language>agnostic ; topic>good-practice ; level>beginner

**TL;DR** = un post centré autour d'un message que je juge très très important = construire des types qui ne permettent que de représenter un état valide (c'est pas tout à fait ce sur quoi il centre le post, mais je reformule avecc mes mots à moi... d'autant que ça va 100% avec [make illegal states unrepresentable](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/))

**EDIT** : [cet article](https://refactoringenglish.com/blog/software-essays-that-shaped-me/#-by-alexis-king-2019) formule le message de l'article original un peu mieux que moi :

> whenever you validate any data, you should convert it to a new type

Dit autrement, lorsqu'on doit valider un input, la bonne pratique est de travailler avec DEUX types différents :

- celui qui encode la donnée brute, non-validée, reçue en entrée (e.g. une `string`)
- celui qui encode la donnée validée (e.g. un type custom `Username` qui ne peut contenir QUE des données valides, p.ex. QUE des username de moins de 20 caractère)

Il faut donc créer un type représentant la donnée valide, et ne véhiculer **QUE** celui-ci une fois la donnée validée.

---

Les exemples sont en Haskell, ça nuit un chouïa à la lisibilité des exemples, mais ça reste du Haskell assez simple et facile à lire.

La situation est celle où on reçoit une donnée en input, qui peut éventuellement être invalide, on peut :

1. stocker l'input dans un type, et permettre de le valider
    - la donnée stockée dans le type peut ou non être valide
2. vérifier la validité de la donnée, et brancher :
    - si la donnée est invalide, ne pas construire d'objet pour la stocker (et à la place, gérer l'erreur)
    - si la donnée est valide, construire un objet pour la stocker
    - le point crucial, c'est que **le type de l'objet ne peut contenir QUE des données valides** : si on a pu construire l'objet, c'est qu'il est valide et utilisable
    - du coup, en baladant ce type "forcément valide" partout ailleurs dans la suite du programme, on n'a plus besoin de vérifier si oui ou non on a une donnée valide : si on reçoit un objet de ce type, par construction, il est valide

Il avance le point que c'est uniquement aux frontières du programme qu'il faut parser : si on parse dans la couche la plus externe du programme, on est tranquille partout ailleurs.

À l'inverse, si on attend d'être dans les couches internes pour parser (et découvrir que la donnée est invalide), ça peut nous conduire à avoir DÉJÀ fait une partie du travail, réaliser trop tard qu'il faut abort, et devoir rollbacker (à supposer même que ça soit faisable) ; cf. **shotgun parsing**.

De mon côté, je ne partage pas ce point de vue à 100% : certes c'est mieux de gérer la donnée invalide le plus à l'extérieur _si c'est possible_. Mais le point qui apporte le plus de valeur à mes yeux est d'avoir un type qui stocke une donnée **obligatoirement valide** ; si jamais on n'a pas pu la construire plus tôt que "dans une couche interne", tant pis.

