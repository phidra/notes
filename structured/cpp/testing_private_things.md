Préambule : la question de savoir si c'est une bonne idée ou pas de tester des trucs privés est pertinente (d'ailleurs, j'ai des notes transverses sur le sujet), mais hors-scope des présentes notes.

* [Takeaway](#takeaway)
* [Astuces](#astuces)
   * [define private public](#define-private-public)
   * [passer en protected](#passer-en-protected)
   * [friend](#friend)
   * [faire une sous-classe](#faire-une-sous-classe)
   * [polluer l'API publique](#polluer-lapi-publique)
   * [n'utiliser QUE la visibilité public](#nutiliser-que-la-visibilité-public)
   * [abandonner le testing de trucs privés](#abandonner-le-testing-de-trucs-privés)


# Takeaway

En C++, comment tester des fonctions privées ?

Différentes astuces ci-dessous ([source1](https://wiki.c2.com/?UnitTestingNonPublicMemberFunctions), [source2](https://www.codeproject.com/Tips/5249547/How-to-Unit-Test-a-Private-Function-in-Cplusplus)), avec différents degrés d'ugliness, d'impact sur le code de production, de contournement du fonctionnement "normal", etc.

Mes deux takeaways :

- il n'est PAS recommandé de polluer son API publique juste pour le testing (car l'API publique a beaucoup de valeur)
- sortir dans une sous-classe à part est certes un peu lourd mais assez propre : on peut tester l'API publique de cette sous-classe, et ne pas modifier l'API publique de sa lib (la sous-classe est un membre privé de la classe initiale)

# Astuces

## define private public

```cpp
#define protected public
#define private   public
#include "headeroftheclassbeingtested.h"
#undef protected
#undef private
```

## passer en protected

Passer la fonction à tester en `protected` et faire hériter le test de la classe à tester (ne marche que pour ce qui est `protected`).

## friend

Déclarer le test comme `friend` de la classe à tester (variante = conditional compilation pour supprimer cette friendliness en production) :

```cpp
class MyClassToTest {
   #ifdef UNIT_TEST
     friend TesterClass;
   #endif
}
```

## faire une sous-classe

(NDM : mon moyen préféré, s'il est faisable)

Sortir la fonction privée à tester dans une classe à part, et la tester indépendamment :

```cpp
// AVANT :
class ToBeTested {
private:
   int calculate() { /* ... */ }
};


// APRÈS :
class Calculator {
public:
    int get_result() { /* ... */ }
};
class ToBeTested
{
private:
    Calculator calculator;
};
```

## polluer l'API publique

Intégrer la fonction à tester à l'API publique de la classe (NDM : et polluer cette dernière...).

## n'utiliser QUE la visibilité public

Tout passer en `public`, et utiliser des conventions de nommage pour identifier ce qui est vraiment public ou pas (un peu comme ce que fait python).

## abandonner le testing de trucs privés

Ne tester que l'API publique de la classe = abandonner l'idée de tester des trucs privés.
