**META** : faute de mieux, je mets des trucs ici ; mais ne pas hésiter à déménager des notes a posteriori vers des emplacements plus appropriés.


* [Includes de headers avec guillemets ou chevrons](#includes-de-headers-avec-guillemets-ou-chevrons)


----

# Includes de headers avec guillemets ou chevrons

```cpp
// faut-il faire ceci :
#include "myheader.h"

// ou cela :
#include <myheader.h>
```

Parmi les différences, les guillemets donnent priorité aux headers qui sont dans le même répertoire que le `.cpp` source (alors que les chevrons ignorent le dossier local).

Dit autrement :

- les chevrons permettent de s'assurer qu'on passe bien par les chemins d'includes du projet
- les guillemets permettent de bypasser ces chemins d'includes pour inclure un header frère

Comme c'est plutôt une bonne pratique de respecter les chemins d'includes du projet, une heuristique permettant de choisir le moyen d'inclure pourrait être :

- dans la majorité (totalité ?) des cas, utiliser les chevrons
- exception = uniquement si je veux inclure un header situé dans le même répertoire (et uniquement si je veux l'inclure de façon relative à ce répertoire plutôt que par un chemin défini à partir des includes du projet)
