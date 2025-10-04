Quelques keypoints sur les lambdas (sources vrac: [lien1](https://stackoverflow.com/questions/5501959/why-does-c11s-lambda-require-mutable-keyword-for-capture-by-value-by-defau), [lien2](https://stackoverflow.com/questions/7627098/what-is-a-lambda-expression-and-when-should-i-use-one ; cf. aussi [ma POC sur les lambdas mutables](https://github.com/phidra/pocs/blob/e14178ea5660520be6707e85414822061ab3d24d/cpp/CATEGORY_language/mutable_lambdas/main.cpp)).

Le plus utile, c'est d'adopter un mindset où on voit une lambda comme du sucre syntaxique pour créer un functor (= une classe qui surcharge `operator()` pour pouvoir être appelée comme une fonction) ; sauf erreur, c'est d'ailleurs ce que fait le compilateur pour compiler une lambda.

Ce mindset aide à y voir plus clair sur les arguments capturés.

Par exemple : si une lambda capture un objet par référence, elle pourra le muter : **les références capturées ne sont pas const**

- :warning: MAIS il faut alors faire très attention à ce que la lambda ait une durée de vie plus petite que l'objet référencé !
- sinon on a un **use-after-free** : la lambda utilisera — via la référence capturée — un objet qui aura été détruit

Keyword `mutable` et quand l'utiliser :

- par défaut, les lambdas n'ont pas le droit de muter leur état capturé : tout se passe comme si l' `operator()` du functor équivalent était `const`
- la raison "théorique" derrière ça est que exécuter une lambda deux fois de suite avec les mêmes arguments devrait donner le même résultat (ce qui ne sera pas le cas si l'état capturé est modifié)
- c'est particulièrement visible pour les lambda qui capturent un argument par copie : de façon contre-intuitive, la lambda n'aura pas le droit de muter l'objet capturé par copie !

Mais alors... Comment une lambda non-mutable peut elle muter une référence capturée, si `operator()` est `const` ?!

- parce qu'une méthode `const` a bien le droit de muter un attribut de type référence non-const !
- confirmé [ici](https://stackoverflow.com/questions/2431596/modifying-reference-member-from-const-member-function-in-c), et vérifié dans ma POC
- même si je trouve ça un peu contre-intuitif, c'est assez logique quand on raisonne avec des pointeurs plutôt que des références : la méthode const n'aura pas le droit de RÉASSIGNER un attribut de type `T * const`, mais elle pourra muter l'objet pointé (qui 1. n'est pas un attribut de la struct const et 2. n'est pas const, on a bien un `T*` et non un `T const*`)


