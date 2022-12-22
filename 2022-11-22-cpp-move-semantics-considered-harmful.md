# C++ Move Semantics Considered Harmful (Rust is better)

- **url** = https://www.thecodedmessage.com/posts/cpp-move/
- **type** = article
- **auteur** = [Jimmy HARTZELL](https://www.thecodedmessage.com/about/), dev multi-langage, essentiellement rust
- **date de publication** = 2021-11-03 
- **source** = [son site](https://www.thecodedmessage.com/)
- **tags** = language>cpp ; language>rust ; topic>move-semantics ; level>intermediate

TL;DR : une comparaison des move semantics de rust et C++ ; l'article est très intéressant pour trois raisons :

- il explique très bien le POURQUOI de la move sémantique = à quel problème ça répond
- il montre que l'implémentation de la move sémantique en C++ est pas terrible, car elle répond indirectement au problème, et laisse de la place aux erreurs humaines (du coup, ça explique que l'apprentissage de la move-semantique soit compliquée)
- il montre le parallèle avec rust et les destructive move, qui répondent de façon plus directe au problème

Plus en détail, à quoi sert la move-semantique ?

- ça ne sert que pour des objets qui gèrent des ressources (e.g. une `std::string` qui gère un buffer heap-allocated)
- si on veut les passer dans une fonction qui doit outlive la string, sans move sémantique, on n'a pas d'autre choix que de copier le buffer, puis supprimer l'original
- la move-sémantique sert précisément à éviter cette opération sous-optimale = "copier un buffer puis détruire l'original"
- à la place, on veut "move" le buffer

Implémentation C++ pourrie ?

- pas en accord avec le besoin
- force les types qu'on veut move à avoir un état "moved-from", valide mais "vide" (i.e. force les types qui gèrent les ressources à avoir un état "toujours valide, mais sans ressource")
- (c'est dû au fait que le destructeur va obligatoirement être appelé sur cet objet moved-from, il faut donc que le destructeur soit un noop, i.e. qu'il ne cleane pas la ressource — vu que celle-ci a été movée vers son nouveau conteneur)
- pour les conteneurs (dont la ressource gérée est leur contenu), c'est pas critique, car il suffit que cet état moved-from soit l'état "empty"
- mais c'est déjà plus embêtant p.ex. pour `unique_ptr` :
    - on aurait aimé que le design de `unique_ptr` ne permette QUE de contenir un pointeur
    - or, comme on DOIT avoir un état "valide mais vide", ça force le design de `unique_ptr` à accepter aussi un pointeur nul (ce dont on se serait bien passé)

Meilleure implémentation rust = destructive move :

- a contrario, en rust, le move transfère la ressource au nouvel objet, ET MARQUE l'ancien objet comme "PSA = ne pas appeler le destructeur sur cet objet car il a été mové"
- du coup, on n'a pas besoin de gérer artificiellement un état "toujours valide mais sans ressource"
- (d'où le "destructive move" : le move "détruit" l'objet, en quelque sorte)
