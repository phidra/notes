# Still No Consensus On Testing Private Methods

- **url** = https://jesseduffield.com/Testing-Private-Methods/
- **type** = post
- **auteur** = [Jesse DUFFIELD](https://jesseduffield.com/about/) = dev australien, auteur de lazygit ; on dirait que j'aime bien son approche
- **date de publication** = 2022-03-05
- **source** = [son blog](https://jesseduffield.com/)
- **tags** = language>agnotic ; topic>testing ; level>intermediate


**TL;DR** : Faut-il tester les méthodes privées ? Un article un peu long, mais très bien écrit, avec un point de vue agréablement nuancé qui me parle énormément, et plein de trucs utiles.

Son conseil en takeaway, c'est qu'il faut rester pragmatique :

- essayer de designer ses APIs publiques de façon frugale, en y mettant le strict minimum nécessaire pour effectuer le métier
- si on se retrouve à vouloir tester une méthode privée, c'est peut-être le un avertissement d'un mauvais design : ne faudrait-il pas sortir la méthode privée dans une classe à part ?
- si la réponse est oui _indépendamment de notre problématique de testing_, on corrige le design, ce qui règle en passant notre problème de testing
- si la réponse est non, alors on s'autorise à tester la méthode privée

----

* [Still No Consensus On Testing Private Methods](#still-no-consensus-on-testing-private-methods)
   * [Viewpoint 1: Don’t Use Private Methods In The First Place](#viewpoint-1-dont-use-private-methods-in-the-first-place)
   * [Viewpoint 2: Always Test Private Methods](#viewpoint-2-always-test-private-methods)
   * [Viewpoint 3: Never Test Private Methods](#viewpoint-3-never-test-private-methods)
   * [Viewpoint 4: Test Private Methods Sometimes](#viewpoint-4-test-private-methods-sometimes)
   * [Viewpoint 5: Extract Private Methods Into A Separate Class](#viewpoint-5-extract-private-methods-into-a-separate-class)
   * [Sa proposition :](#sa-proposition-)


## Viewpoint 1: Don’t Use Private Methods In The First Place

- argument = difficile de prévoir comment les clients vont utiliser le code, donc mieux vaut tout mettre public par défaut
- c'est un point de vue partagé plutôt par les devs de librairie (pour qui une modification de la lib est compliquée puisqu'il faut sortir une nouvelle version, alors qu'un dev d'une app pourra plus facilement refactorer son propre code)
- ça a beaucoup d'inconvénient :
    - rendre public quelque chose qui était privé est facile ; l'inverse est très difficile (breaking change)
    - l'API public a valeur d'explications sur comment utiliser quelque chose, si on la pollue avec plein de trucs, elle devient difficile à lire
    - supprimer toute encapsulation complique la vie de l'utilisateur de la classe

## Viewpoint 2: Always Test Private Methods

- avantage = TDD friendly
- avantage = un test clarifie le rôle de chaque méthode, y compris celles qui sont privées
- inconvénient = tester certaines méthodes privées via l'API publique peut être très compliqué

## Viewpoint 3: Never Test Private Methods

- argument philosophique = comme les clients d'une classe n'accèdent qu'à l'API publique, on ne devrait tester que l'API publique (et si la méthode privée n'est pas accessible depuis l'API publique, c'est du code mort)
- argument point de vue pratico-pratique = si le test ne dépend que de l'API publique, alors on est libre de refactorer comme on l'entend l'implémentation sans que le test soit impacté
- avantage = avec ce viewpoint, quand un test échoue, c'est que le comportement PUBLIC a régressé (alors que dans d'autres viewpoints, un test peut échouer à cause d'une modification sur un détail d'implémentation PRIVÉ)
- à l'inverse, si on testait des trucs privés, alors une modification d'un truc privé nécessitera de mettre à jour les tests, ce qui sera 1. coûteux, et 2. empêche de se reposer sur les tests avant/après pour confirmer l'absence de régression, vu que ceux-ci sont modifiés entretemps

## Viewpoint 4: Test Private Methods Sometimes

NDM : mon point de vue personnel est proche de celui présenté ici.

- une point de vue intéressant : pour les application finales (i.e. pas les librairies), la seule vraie interface publique est l'utilisation par le client final
- dans le cas d'une app graphique, la seule vraie API publique à tester, c'est les clics de souris et les inputs utilisateurs -> théoriquement, la vraie bonne façon de tester ça est via un test end-to-end qui simule ces actions utilisateurs
- à quelques rares exceptions près (e.g. quand tu veux faire passer exactement les mêmes tests sur deux apps implémentées différemment), ce n'est pas une bonne idée
- en effet, les tests e2e ont beaucoup d'inconvénients : très longs à exécuter, très longs à écrire, l'intention du test n'apparaît pas clairement (car obfusquée par la complexité des actions utilisateurs à décrire), interdépendance des tests des features entre eux
- phrase très intéressante = It’s for these very reasons that unit tests exist in the first place.  <-- les tests unitaires SERVENT JUSTEMENT à corriger ces défauts
- son message est qu'il existe différent degrés dans le caractère "public" : un spectre continu allant du plus public (l'interface réellement publique) au plus privé (une méthode privée d'une sous-classe profondément enfouie dans la hiérarchie de l'app)
- comme on ne teste jamais la vraie interface publique de notre application (mais plutôt une fonction haut dans la hiérarchie publique), on fait un choix un peu arbitraire par rapport à où on se place sur ce spectre continue
- et du coup, la réponse à "que faut-il tester", i.e. "où doit-on se placer sur ce spectre continue" n'est pas absolue, c'est un trade-off entre se placer proche du bout public (résultat = refactoring factilité, mais testing de trucs bas-niveau compliqué) ou plus proche du bout privé (résutlat = le contraire)
- le conseil de son paragraphe est de rester pragmatique : si une fonction privée est suffisamment auto-porteuse + difficile à tester via iune interface publique, alors ne pas hésiter à la tester directement

## Viewpoint 5: Extract Private Methods Into A Separate Class

- si on veut tester un truc privé d'une classe, c'est peut-être que la classe a trop de responsabilités et viole le SRP
- l'auteur ne partage pas vraiment ce point de vue (i.e. le fait de vouloir tester une méthode n'est pas une raison suffisante à ses yeux pour la sortir dans une clsse publique), mais le point de vue est partagé par de grands noms
- d'une certaine façon, ce viewpoint s'oppose au précédent ; qui disait que le caractère public/privé était un spectre continu, et que l'endroit où on s'y plaçait pour choisir quoi tester était arbitraire
- ici, l'idée est de dire qu'il y a une règle absolue = si on veut tester une méthode privée, c'est qu'elle est en fait censée être public dans une granularité plus fine
- c'est secondaire, mais on gagne en bonus la possibilité d'injecter un mock de la nouvelle classe dans l'ancienne en inversion de dépendance si besoin

## Sa proposition :

- s'intéresser avant tout à l'API publique de ses classes : essayer de la réduire au strict minimum (tout privé par défaut, sauf ce qu'il est indispensable de passer public)
- si on veut tester une méthode privée, c'est peut-être le signe d'un mauvais design : regarder si ça a du sens EN SOI (i.e. indépendamment de notre besoin de testing) de la sortir dans une classe ou fonction indépendante
- si oui, le faire (et le problème de testing est résolu : on peut tester l'interface publique de cette nouvelle classe) ; si non, convertir la méthode en une fonction pure (qui ne référence pas l'instance), et tester celle-ci
- ce dernier point permet de garder une certaine liberté sur la façon dont ce qu'on teste évolue : comme la fonction pure est indépendante, on peut la déplacer librement, avec un impact de mise à jour du test correspondant assez petit
