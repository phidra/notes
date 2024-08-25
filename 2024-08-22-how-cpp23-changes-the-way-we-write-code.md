# How C++23 Changes the Way We Write Code

- **url** = https://www.youtube.com/watch?v=eD-ceG-oByA
- **type** = talk
- **auteur** = [Timur DOUMLER](https://timur.audio/about), dev C++ anciennement spécialisé dans l'audio et les tools pour développeurs, maintenant chez JetBrains = la boîte qui fait CLion et plein d'autres tools pour le C++
- **date de publication** = 2023-03-11
- **source** = [CppCon 2022](https://www.youtube.com/@CppCon)
- **tags** = language>C++ ; topic>C++23 ; level>intermediate

TL;DR = il présente les 4 features de C++23 qui changent le plus la façon dont on code (il avait déjà fait [la même chose](./2021-03-xx-how-cpp20-changes-the-way-we-write-code.md) pour C++20). NDM : seul `std::expected` me paraît vraiment important ; `std::print` est vraiment sympa quand même, les deux autres me paraissent niche.


Les 17 premières minutes, il écréme toutes les nouvelles features du language et de la lib standard pour finir par retenir les 4 features qui vont changer la façon dont on fait du C++.

# deducing this

17:00

Ça ajoute une syntaxe spéciale pour définir une méthode, en indiquant un `this` spécial qui permet d'utiliser le `this` implicite à chaque appel de méthode.

Ça a plusieurs avantages :

- évite la duplication (en permettant d'utiliser un seul template pour overloader différents méthodes d'un objet selon que l'objet est const ou non (ainsi, je l'apprends, selon que la référence vers l'objet est & ou &&), et le template va déduire le type exact du this de la nouvelle syntaxe)
- simplifie le CRTP (qui sert à faire une classe mixin permettant d'ajouter un comportement à une classe en la faisant hériter de la classe mixin)
- permettre des lambdas récursives (ce qu'on ne peut pas faire simplement, car on ne peut pas nommer la lambda à l'intérieur de la lambda vu qu'elle n'existe pas encore), ce qui peut être utile pour visiter des variants.

# std::expected

37:00

Rien de nouveau pour moi ici. En résumé, c'est une façon supplémentaire possible de gérer les erreurs parmi celles que le C++ propose déjà (exception, code de retour, paire de valeur+erreur, etc.)

Note : `std::optional` n'est pas ouf pour représenter une erreur, car il ne contient que si la raison pour laquelle on n'a pas de valeur normale est obvious ; souvent, ça ne convient pas pour gérer des erreurs, car on veut renvoyer une erreur précise (possiblement parmi plusieurs erreurs possibles), plutôt que simplement dire "pas de valeur".

Par ailleurs, on peut renvoyer un `std::expected<T, std::variant<Err1, Err2>>` pour renvoyer plusieurs types d'erreurs. Comme avec tous les variants, le compilo préviendra si des clients n'ont pas géré l'une des erreurs possibles, ce qui est chouette !

52:30 le code appelant ressemble beaucoup à l'utilisation d'exceptions, mais sans le surcoût, et avec la vérif par le compilo.


# std::mdspan

54:00

TL;DR = `std::mdspan` est un array à plusieurs dimensions, plus ergonomique et standard que ce qu'on ferait à la main, notamment pour gérer le type de données contenue dans l'array, et pour gérer le nombre de dimensions.

Note : le mdspan est non-owning (en C++26, il y aura une version owning)

Le nombre d'éléments dans chaque dimension est défini soit au runtime, soit au compile-time ; la façon dont les éléments sont stockés en mémoire est configurable

# std::print

1:15:00

TL;DR = `std::print` est la combinaison de `std::format` qui permet de formater des chaînes, et d'un affichage de cette chaîne formattée sur `std::cout`.
