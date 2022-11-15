Contexte : je m'intéresse à la notion de POD = Plain Old Datatype ; C'est quoi ? Ça sert à quoi ? Je consulte plusieurs ressources sur le sujet.

**TL;DR :**

- initialement, une structure POD était une structure suffisamment simple pour mirrorer celles du C → on pouvait passer la donnée binaire de cette structure à un programme C sans souci.
- mais en vrai, ça recouvre plutôt deux concepts :
    - **standard layout** = le layout en mémoire de la structure est le même que celui du C
    - **trivial** = la structure peut être initialisée (resp. copiée) simplement en `memcpy` son contenu
- cf. les règles détaillées pour obtenir ces deux comportements
- il y a des fonctions C++ spéciales pour tester ces propriétés : `is_standard_layout`, `is_trivial`, `is_trivially_copyable`, `is_pod`
- quelques exemples illustratifs aidant à comprendre les notions :
    ```cpp
    // trivial, mais pas standard-layout car mix public/private :
    struct NonStandardLayout
    {
       int a;
    private:
       int b;
    };
    
    // standard-layout, mais pas trivial car constructeur évolué :
    struct NonTrivial
    {
       int a;
       int b;
       NonTrivial() { std::cout << "non-trivial default construction" << std::endl; }
    };
    
    // trivial + standard-layout = POD :
    struct POD
    {
       int a;
       int b;
    };
    ```

---

Autre réflexion perso = pourquoi une classe n'est-elle pas triviale si elle a un default-constructeur custom ?

Prenons un exemple concret :


```cpp
#include <ctime>
struct Person {
    int age;
    time_t constructed_at;  // time_t est un unsigned long

    Person() { constructed_at = time(0); }
    Person(Person const& other) : age{other.age} { constructed_at = time(0); }
};
```

La classe n'hérite de personne, elle n'a que deux membres, tous deux entiers, et ayant la même visibilité : elle a bien un layout standard !

Elle a un constructeur par défaut custom, elle n'est donc pas triviale (donc pas POD).

Pourtant, on peut bien `memcpy` les données, et le layout obtenu après ce `memcpy` sera pourtant bien valide ?! Alors pourquoi ne serait-elle pas triviale ?!

En fait, même si le layout est valide du point de vue binaire, _l'objet n'est pas construit_ : en C++, si un objet est correctement construit, c'est qu'on a exécuté un constructeur, et ici le fait d'exécuter un constructeur sette systématiquement le champ `constructed_at`.

Dit autrement, ici, une `Person` correctement construite stocke forcément sa date de consruction dans `constructed_at`. Or, ce n'est PAS le cas après un simple `memcpy` (vu que dans ce cas, on a plutôt cloné le `constructed_at` de la classe qui a servi de modèle, au lieu de setter la bonne date = la date à laquelle on a appelé le constructeur de copie), on ne peut donc pas considérer que faire un `memcpy` est suffisant pour construire une `Person` : il nous manquerait alors une étape, celle d'initialiser le `constructed_at`.

Par conséquent, cette classe n'est pas triviale.

Bonus : on pourrait considérer qu'avoir un constructeur custom n'empêche pas forcément d'être trivial, par exemple :

```cpp
#include <ctime>
struct Person {
    int age;
    time_t constructed_at;

    Person() : constructed_at{0} {}
    Person(Person const& other) : age{other.age}, constructed_at{other.constructed_at} {}
};
```

^ Tout comme précédemment, cette classe n'est pas triviale en théorie (vu qu'elle a un default-constructeur custom), mais elle est triviale "en pratique" : le résultat d'un `memcpy` est (en pratique) strictement équivalent à une copie.

Pourquoi ne pas marquer comme "non-trivial" que les classes dont le constructeur custom empêche la trivialité en pratique ?

Probablement parce que si on commence à essayer de vérifier si le travail custom que fait le constructeur empêche la copie/construction triviale ou non, on ne s'en sortira pas...

Dit autrement, un constructeur custom n'empêche pas forcément d'avoir la propriété voulue (i.e. de construction triviale) mais comme on peut pas le garantir, `is_trivial` renvoie false de façon conservative dès qu'on a un constructeur custom.

---

https://www.cs.technion.ac.il/users/yechiel/c++-faq/pod-types.html

> A POD type is a C++ type that has an equivalent in C, and that uses the same rules as C uses for initialization, copying, layout, and addressing.

^ c'est la définition d'un POD : c'est une structure qui se comporte comme une structure C

> As an example, the C declaration struct Fred x; does not initialize the members of the Fred variable x. To make this same behavior happen in C++, Fred would need to not have any constructors.

^ e.g. dès qu'on a un constructeur, on n'est donc plus POD, vu que la structure fait des trucs particuliers à l'initialisation (et ne se comporte donc pas comme une structure C)

Plus généralement, les 4 points mentionnés dans la définition sont importants, et c'est d'eux que dérivent les "règles" qui font un type POD :

- pas de constructeur
- pas de destructor (pour les mêmes raisons : le code du destructeur serait appelé en C++, mais pas en C)
- pas d'assignment-operator
- pas de fonctions virtuelles (sinon, la vtable rend le type non-POD)
- pas de classe de base
- pas de membre private ou protected

La définition complète est un chouille plus complexe et récursive

---

https://en.cppreference.com/w/cpp/named_req/PODType

> (POD) means the type is compatible with the types used in the C programming language, that is, can be exchanged with C libraries directly, in its binary form.

---

https://en.wikipedia.org/wiki/Passive_data_structure

> In computer science and object-oriented programming, a passive data structure (PDS, also termed a plain old data structure, or plain old data, POD) is a term for a record, to contrast with objects. It is a data structure that is represented only as passive collections of field values (instance variables), without using object-oriented features

^ En gros, ça rejoint le fait qu'en C les structs sont des simples aggrégats de champs, sans aucun code associé (alors qu'en C++, les classes permettent d'associer du code aux données, notamment par le biais des constructeurs/destructeurs)

> Passive data structures are appropriate when there is a part of a system where it should be clearly indicated that the detailed logic for data manipulation and integrity are elsewhere.
>
> PDSs are often found at the boundaries of a system, where information is being moved to and from other systems or persistent storage (...) For example, PDS would be convenient for representing the field values of objects that are being constructed from external data

^ intéressant, les POD ont leur utilité en tant que format d'échange binaire.

> A PDS type in C++, or Plain Old C++ Object, is defined as either a scalar type or a PDS class.[2] A PDS class has no user-defined copy assignment operator, no user-defined destructor, and no non-static data members that are not themselves PDS. Moreover, a PDS class must be an aggregate, meaning it has no user-declared constructors, no private nor protected non-static data, no virtual base classes[a] and no virtual functions.[

^ règles des POD en C++

---

https://stackoverflow.com/questions/2293796/pods-non-pods-rvalue-and-lvalues/2293926#2293926

> POD means Plain Old Data and is a collection of requirements that specify using memcpy is equivalent to copying. Non-POD is any type for which you cannot use memcpy to copy (the natural opposite of POD, nothing hidden here), which tends to be most types you'll write in C++.

^ avec les POD, utiliser `memcpy` est équivalent à copier (et donc construire proprement) la donnée ; ça n'est pas le cas dans le cas général.

---

https://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/4178176#4178176

C'est de loin la réponse la plus complète ! (à noter que la réponse est splittée en plusieurs morceaux selon le standard, mais l'auteur donne des liens)

> Aggregate : We should understand that memberwise initialization with braces implies that the class is nothing more than the sum of its members. If a user-defined constructor is present, it means that the user needs to do some extra work to initialize the members therefore brace initialization would be incorrect.

^ ça rejoint ce que je disais ailleurs : dès qu'on a un user-defined constructor, c'est qu'on veut faire quelque chose de spécial à l'initialization, et que la structure n'est pas un simple aggrégat de données binaires...

> If virtual functions are present, it means that the objects of this class have (on most implementations) a pointer to the so-called vtable of the class, which is set in the constructor, so brace-initialization would be insufficient.

^ ici aussi j'avais bon : on ne veut pas de virtual functions car ça ajoute un membre particulier (absent des structures C) = le pointeur sur la vtable

**POD — avant C++11 :**

Un POD est un aggregate particulier, avec des règles encore plus strictes :

> POD-classes are the closest to C structs. Unlike them, PODs can have member functions and arbitrary static members, but neither of these two change the memory layout of the object.
>
> So if you want to write a more or less portable dynamic library that can be used from C and even .NET, you should try to make all your exported functions take and return only parameters of POD-types.
>
> The lifetime of objects of non-POD class type begins when the constructor has finished and ends when the destructor has finished. For POD classes, the lifetime begins when storage for the object is occupied and finishes when that storage is released or reused.

^ Ce point est très intéressant, et lié à `is_trivially_copyable` : comme les constructeurs/destructeurs ne font rien (pour être iso-C), dès que la mémoire est remplie avec les membres de la structure, l'objet est directement utilisable (alors que dans le cas général, en C++, ça n'est pas vrai : il faut exécuter le constructeur sur cette mémoire).

> For objects of POD types it is guaranteed by the standard that when you memcpy the contents of your object into an array of char or unsigned char, and then memcpy the contents back into your object, the object will hold its original value.

^ cette propriété intéressante (on peut memcpy un POD vers un char* et vice-versa) découle du fait que la structure est équivalente à l'aggrégat de ses données binaires.

Exemple de code, pour "sauvegarder" l'état d'un objet POD, simplement en le memcopiant dans un buffer :

```cpp
#define N sizeof(T)
char buf[N];
T obj;
use_it(obj);
memcpy(buf, &obj, N);   // on "sauvegarde" l'état de l'objet
use_it_even_more(obj);  // on modifie l'objet
memcpy(&obj, buf, N);   // on "restaure" l'état de l'objet
```

**POD — en C++11 :**

> PODs went through a lot of changes. Lots of previous rules about PODs were relaxed in this new standard, and the way the definition is provided in the standard was radically changed. The idea of a POD is to capture basically two distinct properties:
>    - It supports static initialization, and
>    - Compiling a POD in C++ gives you the same memory layout as a struct compiled in C.
>
> Because of this, the definition has been split into two distinct concepts: trivial classes and standard-layout classes, because these are more useful than POD. The new definition basically says that a POD is a class that is both trivial and has standard-layout, and this property must hold recursively for all non-static data members

^ ce point est important : ce qu'on appelait POD jusqu'ici est le regroupement de deux concepts différents :

- classe triviale = la structure est initialisable statiquement = il suffit de `memcpy` le contenu de la classe pour l'utiliser
- standard layout = le layout mémoire de la classe sera le même qu'en C

> Trivial class : trivial classes support static initialization.
>
> If a class is trivially copyable (a superset of trivial classes), it is ok to copy its representation over the place with things like memcpy and expect the result to be the same.

Le caractère trivial d'un contructor (ou autre opérateur dans le genre) est :

> Basically this means that a copy or move constructor is trivial if it is not user-provided, the class has nothing virtual in it, and this property holds recursively for all the members of the class and for the base class.

Concept un peu moins fort = **trivially copyable class** :

> A trivially copyable class is a class that: \
> — has no non-trivial copy constructors (12.8), \
> — has no non-trivial move constructors (12.8), \
> — has no non-trivial copy assignment operators (13.5.3, 12.8), \
> — has no non-trivial move assignment operators (13.5.3, 12.8), and \
> — has a trivial destructor (12.4).

^ j'en infère qu'une classe trivially copyable, c'est une classe pour laquelle on ne fait rien de particulier au moment de la copie/move. On a le droit d'avoir un constructeur particulier, vu qu'une fois l'objet construit (y compris si le constructeur a fait un truc custom), alors la copie des membres n'ont plus de comportement custom (?)

Concept un peu plus fort = **trivial class** :

> A trivial class is a class that has a trivial default constructor (12.1) and is trivially copyable.

^ C'est donc idem que trivially copyable, mais avec en plus le fait que la construction ne fasse rien de custom : la classe n'est pas simplement trivialement _copiable_, elle est également triviallement _constructible_ !

Ma compréhension, c'est que si on attribue des valeurs valides pour chacun des emplacements mémoires des membres de la classe triviale, alors ça y est, la classe est construite de façon valide (il n'y a besoin de rien d'autre : sa construction est triviale).


**Standard Layout**

> Standard layout : The standard mentions that these are useful for communicating with other languages, and that's because a standard-layout class has the same memory layout of the equivalent C struct or union.

^ le layout mémoire est le même que celui en C

> This is another property that must hold recursively for members and all base classes.

^ c'est assez logique

> And as usual, no virtual functions or virtual base classes are allowed. That would make the layout incompatible with C.

^ logique aussi : le pointeur vers la vtable va tout foirer : C ne serait pas capable de l'utiliser.

> A relaxed rule here is that standard-layout classes must have all non-static data members with the same access control. Previously these had to be all public, but now you can make them private or protected, as long as they are all private or all protected.

^ pour une raison que je ne connais pas encore (peut-être liée à l'implémentation de la vérification d'access-type par les compilos ?), la visibilité des membres joue sur le layout mémoire !

> A standard-layout class is a class that: \
> — has no non-static data members of type non-standard-layout class (or array of such types) or reference, \
> — has no virtual functions (10.3) and no virtual base classes (10.1), \
> — has the same access control (Clause 11) for all non-static data members, \
> — has no non-standard-layout base classes, \
> — either has no non-static data members in the most derived class and at most one base class with non-static data members, or has no base classes with non-static data members, and \
> — has no base classes of the same type as the first non-static data member.

Donc en résumé, même si ma compréhension n'est pas encore absolument limpide :

- trivially copyable semble indiquer qu'on peut memcpy la mémoire de l'objet pour représenter l'objet
    - un bon contre-exemple serait la classe vector : si on veut copier un vector, il ne faut pas se contenter de copier les 3 pointeurs membres du vector
    - (sans quoi on se retrouverait avec DEUX vectors qui pointeraient vers le même buffer d'objets sous-jacents)
    - pour copier proprement un vector, il faut allouer un deuxième buffer de même taille que le premier, y copier les objets (éventuellement, les memcopier si T est trivially-copyable lui-aussi)
- standard layout semble indiquer que les membres d'une classe sont représentées en mémoire de la même façon que le C
    - pour reprendre le cas du vector, il pourrait très bien être standard-layout ! (au final, il ne comporte que 3 pointeurs, qui pourraient se manipuler comme en C)
    - les contre-exemples de standard-layout semblent liés soit aux fonctions virtuelles (logique : cf. vtable) soit à l'héritage multiple

**Changements introduits en C++14 :**

https://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/27511360#27511360

^ quelques petits changements (suffisamment mineurs pour ne pas influer sur ma compréhension de tous ces concepts)

**Changements introduits en C++17 :**

https://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/50786105#50786105

^ ici aussi, les changements sont suffisamment mineurs pour ne pas influer sur ma compréhension. (par exemple, on s'assure dorénavant de ne pas pouvoir std::memcpy une classe si son concepteur a explicitement empêché sa copie, en deletant ses copy/move-constructors/assignment-operators)

La nouvelle définition d'une classes trivialement copiable est :

> A trivially copyable class is a class: \
> — where each copy constructor, move constructor, copy assignment operator, and move assignment operator is either deleted or trivial, \
> — that has at least one non-deleted copy constructor, move constructor, copy assignment operator, or move assignment operator, and \
> — that has a trivial, non-deleted destructor.

NdM : trivialement copiable = une fois construite, la classe peut se copier avec `memcpy`

Et la nouvelle définition d'une classe triviale est :

> A trivial class is a class that is trivially copyable and has one or more default constructors, all of which are either trivial or deleted and at least one of which is not deleted.
>
> [Note: In particular, a trivially copyable or trivial class does not have virtual functions or virtual base classes.—end note]

NdM : trivial classe = trivialement copiable + trivialement constructible (le point qui m'intéresse le plus est sans doute plutôt l'aspect "trivialement copiable")

**Changements introduits en C++20 :**

https://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/53819762#53819762

^ ici aussi, les changements sont suffisamment mineurs pour ne pas influer sur ma compréhension

NdM : pour affiner ma compréhension, il me faut un exemple pour chacun des 4 cas :

- ni trivialement copiable, ni standard layout
- trivialement copiable mais pas standard layout
- pas trivialement copiable mais standard layout
- trivialement copiable ET standard layout

---

https://en.cppreference.com/w/cpp/types/is_standard_layout

> A pointer to a standard-layout class may be converted (with reinterpret_cast) to a pointer to its first non-static data member and vice versa.
>
> The macro offsetof is only guaranteed to be usable with standard-layout classes.

NdM : avec une classe standard_layout, ses membres en mémoire sont packés les uns à la suite des autres comme en C.

---

https://en.cppreference.com/w/cpp/types/is_trivially_copyable

> Objects of trivially-copyable types [...] are the only C++ objects that may be safely copied with std::memcpy or serialized to/from binary files with std::ofstream::write()/std::ifstream::read().

---

https://en.cppreference.com/w/cpp/types/offsetof

> The macro offsetof expands to an integral constant expression of type std::size_t, the value of which is the offset, in bytes, from the beginning of an object of specified type to its specified subobject, including padding if any. (offsetof cannot be implemented in standard C++ and requires compiler support)

C'est une macro C, donc c'est logique que ça ne soit utilisable qu'avec des standard-layout types, i.e. des types qui par définition ont le même layout qu'en C.

En gros, ça permet de récupérer l'adresse mémoire d'un membres d'une classe à partir de l'adresse de l'objet.

---

https://learn.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-170

Note : cette ressource est particulièrement intéressante car elle donne des exemples très illustratifs.

> The term layout refers to how the members of an object of class, struct or union type are arranged in memory. In some cases, the layout is well-defined by the language specification. But when a class or struct contains certain C++ language features such as virtual base classes, virtual functions, members with different access control, then the compiler is free to choose a layout.

^ Ce point est intéressant : avec certaines features du C++, le layout est implementation-defined (ça explique que le `is_standard_layout` soit false pour de telles classes).

> That layout may vary depending on what optimizations are being performed and in many cases the object might not even occupy a contiguous area of memory. For example, if a class has virtual functions, all the instances of that class might share a single virtual function table. Such types are very useful, but they also have limitations. Because the layout is undefined they cannot be passed to programs written in other languages, such as C, and because they might be non-contiguous they cannot be reliably copied with fast low-level functions such as memcopy, or serialized over a network.

**TRIVIAL TYPES :**

> When a class or struct in C++ has compiler-provided or explicitly defaulted special member functions, then it is a trivial type. It occupies a contiguous memory area. It can have members with different access specifiers. In C++, the compiler is free to choose how to order members in this situation. Therefore, you can memcopy such objects but you cannot reliably consume them from a C program.

Ça me donne un exemple de classe trivialement copiable mais pas standard layout : une classe qui est un simple agrégat de deux `int`, mais dont l'un est public l'autre privé :

```cpp
// trivial, mais pas standard-layout car mix public/private :
struct NonStandardLayout
{
   int a;
private:
   int b;
};
```

NdM : le contraire serait assez simple : une classe avec des membres simples, MAIS des opérateurs de construction ou copie évolués (e.g. simplement qui fait un cout dans le constructeur) :

```cpp
// standard-layout, mais pas trivial car constructeur évolué :
struct NonTrivial
{
   int a;
   int b;
   NonTrivial() { std::cout << "non-trivial default construction" << std::endl; }
};
```

Et du coup, une classe qui est à la fois triviale et standard-layout est POD :

```cpp
struct POD
{
   int a;
   int b;
};
```


> A trivial type T can be copied into an array of char or unsigned char, and safely copied back into a T variable. Note that because of alignment requirements, there might be padding bytes between type members.

^ le padding n'empêche pas le caractère copiable avec `memcpy`

> Trivial types have a trivial default constructor, trivial copy constructor, trivial copy assignment operator and trivial destructor. In each case, trivial means the constructor/operator/destructor is not user-provided and belongs to a class that has :
>    - no virtual functions or virtual base classes,
>    - no base classes with a corresponding non-trivial constructor/operator/destructor
>    - no data members of class type with a corresponding non-trivial constructor/operator/destructor

^ leur définition est simple \o/

> In Trivial2, the presence of the Trivial2(int a, int b) constructor requires that you provide a default constructor. For the type to qualify as trivial, you must explicitly default that constructor.

^ intéressant : il faut un default-constructeur trivial, mais il n'est pas INTERDIT d'avoir en plus des constructeurs évolués : l'existence d'un default-constructor trivial garantit qu'on *peut* construire le type trivialement.

**STANDARD LAYOUT TYPES**

> When a class or struct does not contain certain C++ language features such as virtual functions which are not found in the C language, and all members have the same access control, it is a standard-layout type. It is memcopy-able and the layout is sufficiently defined that it can be consumed by C programs.

^ standard-layout = on peut échanger la structure binaire avec du C.

> In addition, standard layout types have these characteristics:
> - no virtual functions or virtual base classes
> - all non-static data members have the same access control
> - all non-static members of class type are standard-layout
> - any base classes are standard-layout
> - has no base classes of the same type as the first non-static data member.
> - meets one of these conditions:
> - no non-static data member in the most-derived class and no more than one base class with non-static data members, or
> - has no base classes with non-static data members

NdM : les exemples donnés (que j'ai repris ci-dessus) sont très clairs.

**POD TYPES**

> When a class or struct is both trivial and standard-layout, it is a POD (Plain Old Data) type. The memory layout of POD types is therefore contiguous and each member has a higher address than the member that was declared before it, so that byte for byte copies and binary I/O can be performed on these types. Scalar types such as int are also POD types. POD types that are classes can have only POD types as non-static data members.

---

https://stackoverflow.com/questions/54067826/why-must-members-of-standard-layout-classes-have-the-same-access-control/54067841#54067841

Pourquoi le caractère `public` ou `private` joue sur le layout mémoire ? Parce que le standard C++ :

- garantit que des membres ayant la même visibilité auront un layout standard (i.e. ils apparaissent dans le même ordre qu'en C, avec la même taille et le même padding)
- ne garantit RIEN sur la disposition relative des membres ayant des visibilités différentes : chaque compilo fait ce qu'il préfère

Du coup, si on veut un layout mémoire déterministe (et identique à celui du C), il FAUT que tous les membres aient la même visibilité...

