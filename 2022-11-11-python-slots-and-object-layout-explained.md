# Python slots and object layout explained


- **url** = https://www.youtube.com/watch?v=Iwf17zsDAnY
- **type** = vidéo
- **auteur** = [James MURPHY](https://mcoding.io/about-james-murphy) = mathématicien et codeur C++/python avec un background dans la finance
- **date de publication** = 2021-10-23
- **source** = [sa chaîne youtube](https://www.youtube.com/c/mCodingWithJamesMurphy/about)
- **tags** = language>python ; topic>slots ; level>intermediate

**TL;DR** : explications sur `__slots__` et sur le layout en mémoire des instances de classes en python :

- pour qu'une classe soit une _slotted-class_, il faut qu'elle ait un class-attribute nommé `__slots__` qui doit être un itérable de strings.
- avec une classe classique (non-slotted), les attributs sont dynamiques, gérés dans un attribut `__dict__` dynamique
- avec une classe slotted, les attributs sont définis statiquement par le contenu de `__slots__` (et elle n'a plus d'attribut `__dict__`)
- les attributs définis restent mutables : ce qui est statiquement défini et immutable, c'est bien la liste des attributs et pas les attributs eux-mêmes
- l'intérêt principal est : les instances consomment moins de mémoire, car les slotted-attributes sont inlinés dans le layout mémoire de l'objet (plutôt que d'avoir à gérer un coûteux dict)
- la contrainte principale est : il faut connaître à l'avance la liste exacte des attributs (on perd le côté dynamique)


Notes complémentaires :

- l'instance de classe n'a plus d'attribut `__dict__` (car ses attributs sont statiques), mais la classe elle-même (i.e. l'instance de méta-classe) a bien un attribut `__dict__`, on peut donc ajouter dynamiquement des attributs à la classe elle-même
- `getsizeof` montre que les instances slotted consomment moins de mémoire
- les slotted classes sont aussi un poil plus rapide, mais de façon tellement mineure qu'il est inutile de redesigner son code juste pour ce petit gain
- dès qu'on a défini les slots (et avant même leur initialisation), les attributs slotted existent sur la classe et sont des descripteurs
- 06:00 : _how python objects are laid out in memory_
    - tous les objets commencent avec un `size_t` pour le refcount, et un pointeur vers le type de l'objet
    - (il y a également un préfixe de deux pointeurs next et prev qui servent au garbage-collector, je suppose que chaque objet fait partie d'une double linked list pour intérer sur tous les objets pour la partie sweep de Mark-And-Sweep, la vidéo laisse ça de côté)
    - dans une slotted-class, derrière, on a autant de pointeurs que de slots
    - a contrario, dans une classe classique, les deux membres suivants sont un pointeur vers le `__dict__`, et un pointeur vers un `weakref`
    - (NdM : il n'explique pas le weakref dans la vidéo, mais [cette page](https://stackoverflow.com/questions/36787603/what-exactly-is-weakref-in-python/36789779#36789779) explique ça très bien : `__weakref__` _is just an opaque object that references all the weak references to the current object. It's just an implementation detail that allows the garbage collector to inform weak references that its referent has been collected, and to not allow access to its underlying pointer anymore._ )
- le fait d'être slotted n'est pas hérité
- Un commentaire de la vidéo ajoute un truc intéressant :
    > It's not just the memory savings. Restricting the ability to add new members is a feature itself. If I make a typo when assigning to an attribute, I want to get an error. I don't want it to be silently ignored
