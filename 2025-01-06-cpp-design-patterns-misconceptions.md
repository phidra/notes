# C++ Design Patterns - The Most Common Misconceptions (2 of N)

- **url** = https://www.youtube.com/watch?v=pmdwAf6hCWg
- **type** = vidéo
- **auteur** = [Klaus IGLBERGER](https://github.com/igl42) = consultant C++
- **date de publication** = 2024-12-27
- **source** = [CppCon 2024](https://www.youtube.com/playlist?list=PLHTh1InhhwT6U7t1yP2K8AtTEKmcM3XU_)
- **tags** = language>cpp ; topic>design-patterns ; level>advanced

**TL;DR** : très bonne explication du CRTP + pourquoi le CRTP et les `std::variant` ne sont pas des remplaçants aux interfaces à base de fonctions virtuelles pour faire du polymorphisme.

- 10 premières minutes = explication du CRTP (suite à la quelle j'ai fait [une POC](https://github.com/phidra/pocs/tree/c0bc88897483301a238e8ecd9ebb1eb7c1d1ec99/cpp/CATEGORY_patterns/crtp)).
    - Même utilisation qu'interface = manipuler plusieurs objets de type différents de façon homogène (EDIT : à la différence des templates, l'interface statique est ici *explicite*)
    - Une fonction (templatée sur un type génétique Derived) peut travailler sur n'importe quel objet hétérogène
    - Inconvénient 1 = à la différence de l'héritage, les objets hétérogènes héritent d'une classe de base différente (e.g. on peut pas les mettre dans un Vector
    - Inconvénient 2= on ne peut manipuler ces objets que via une fonction templatée , et pire, c'est viral : notre code devient une lib template header-only
- 11m = on peut simplifier le CRTP avec `deducing this` introduit en c++23
- 18m = deux usages différents du CRTP :
    - 1 = avoir une interface statique _explicite_ pour manipuler des objets hétérogènes d'une façon homogène
    - 2 = ajouter des fonctionnalités à la la classe dérivée (NDM je suppose que l'intérêt est dans la mutualisation dans le parent des fonctionnalités ajoutées : un genre de mixin — EDIT : ha ha, d'ailleurs, juste après, il propose le nom "mixin" :muscle:)
- 22m : le duck typing (via les concepts ou les templates classiques) n'est PAS un remplaçant pour CRTP, car le CRTP (utilisé comme interface statique) ne se contente pas d'accepter n'importe quel type qui satisfait les contraintes templates : il définit **explicitement** une _famille_ de types. Les usages sont donc subtilement différents.
- 23m du coup, on rajoute la contrainte de faire partie de la famille de type (en héritant d'un type de marquage) : grâce à ça, les types enfants doivent choisir explicitement de faire partie de la famille (en héritant de la classe parente)
- 37m = il explore le fait d'utiliser un `std::variant` pour remplacer une interface à base de fonction virtuelle (avec sa solution à base de variant, tout variant est utilisé comme une interface)
- 44m = avec les variants, l'interface (le variant) doit avoir connaissance des types qui l'implémentent (qui constituent le variant) ; i.e. il en dépend. Architecturalement, ça ne va pas, on veut la dépendance dans l'autre sens.
- 47m = on peut s'en sortir en saupoudrant de templates, mais on en revient alors à une header-only template library. Sur une grosse codebase, devoir compiler la lib partout où on l'utilise peut être rhedibitoire.
- 51m = Au final, ni les variants , ni le CRTP ne sont des remplaçants des virtual functions pour faire du polymorphisme.
- 52m20 = les guidelines de design :
    > - Think about your design/architecture first and about implementation details second.
    > - Consider only the patterns/abstractions that fit your design.
    > - Don't design based on performance requirements.


