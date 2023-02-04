# C++ Basic Concepts

- **url** = https://learn.microsoft.com/en-us/cpp/cpp/basic-concepts-cpp
- **type** = tuto
- **auteur** = N/A
- **date de publication** = 2021-03-08
- **source** = la [C++ language documentation](https://learn.microsoft.com/en-us/cpp/cpp/) de Microsoft
- **tags** = language>cpp ; topic>concepts ; level>beginner



Doc microsoft sur des concepts basiques du C++ ; je regroupe plusieurs pages indépendantes de la doc sous un seul fichier de notes, et pour l'essentiel, je me contente de citer les passages particulièrement pertinents.

* [C++ Basic Concepts](#c-basic-concepts)
   * [C++ type system](#c-type-system)
   * [Scope](#scope)
   * [Translation units and linkage](#translation-units-and-linkage)
   * [main function and command-line arguments](#main-function-and-command-line-arguments)
   * [Program termination](#program-termination)
   * [Lvalues and rvalues](#lvalues-and-rvalues)
   * [Temporary objects](#temporary-objects)
   * [Alignment](#alignment)
   * [Trivial, standard-layout and POD types](#trivial-standard-layout-and-pod-types)

## C++ type system

https://learn.microsoft.com/en-us/cpp/cpp/cpp-type-system-modern-cpp

> The type specifies the amount of memory that's allocated for the variable (or expression result). The type also specifies the kinds of values that may be stored, how the compiler interprets the bit patterns in those values, and the operations you can perform on them.

^ Définition d'un **type**.

> Compound type: A type that isn't a scalar type. Compound types include array types, function types, class (or struct) types, union types, enumerations, references, and pointers to non-static class members.

^ Compound type = le contraire d'un scalar-type.

> When you declare a variable of POD type, we strongly recommend you initialize it, which means to give it an initial value. Until you initialize a variable, it has a "garbage" value that consists of whatever bits happened to be in that memory location previously. It's an important aspect of C++ to remember, especially if you're coming from another language that handles initialization for you. When you declare a variable of non-POD class type, the constructor handles initialization.

^ POD (voir aussi [ma page spécifique sur le sujet](./structured/cpp/pod_structures.md)), il est très important d'initialiser explicitement les POD, sinon ils contiendront du garbage.

> Raw pointers : However, in modern C++, it's no longer required (or recommended) to use raw pointers for object ownership at all [...] It's still useful and safe to use raw pointers for observing objects.

^ Ce point est intéressant et la précision est importante : dans le C++ "moderne", on entend souvent qu'il faut proscrire les raw-pointers ; mais ce qu'il faut proscrire, ce sont les OWNING raw-pointers : des non-owning raw-pointers peuvent tout à fait être utilisés !

## Scope

https://learn.microsoft.com/en-us/cpp/cpp/scope-visual-cpp

> There are six kinds of scope:
>
> - **Global scope** A global name is one that is declared outside of any class, function, or namespace. [...] \
> - **Namespace scope** [...] \
> - **Local scope** A name declared within a function or lambda, including the parameter names, have local scope. They are often referred to as "locals". [...] \
> - **Class scope** Names of class members have class scope, which extends throughout the class definition regardless of the point of declaration. [...] \
> - **Statement scope** Names declared in a for, if, while, or switch statement are visible until the end of the statement block. \
> - **Function scope** A label has function scope, which means it is visible throughout a function body even before its point of declaration. Function scope makes it possible to write statements like goto cleanup before the cleanup label is declared.

NDM : différence entre function scope et local scope : function scope est un petit truc propre aux goto (qui fonctionne un peu comme le hoisting en javascript).

> For global names, visibility is also governed by the rules of linkage which determine whether the name is visible in other files in the program.

^ cf. partie suivante, dans le cas très particulier du namespace global, le linkage joue sur la visibilité des noms par d'autres translation units.

> A namespace may be defined in multiple blocks across different files.

## Translation units and linkage

https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp

> In a C++ program, a symbol, for example a variable or function name, can be declared any number of times within its scope. However, it can only be defined once. This rule is the "One Definition Rule" (ODR). A declaration introduces (or reintroduces) a name into the program, along with enough information to later associate the name with a definition. A definition introduces a name and provides all the information needed to create it

^ ODR

> If the name represents a variable, a definition explicitly creates storage and initializes it. A function definition consists of the signature plus the function body. A class definition consists of the class name followed by a block that lists all the class members. (The bodies of member functions may optionally be defined separately in another file.

^ La définition d'une variable entraîne son "allocation" + son initialization.

> A program consists of one or more translation units. A translation unit consists of an implementation file and all the headers that it includes directly or indirectly.
>
> [...] Each translation unit is compiled independently by the compiler. After the compilation is complete, the linker merges the compiled translation units into a single program.

^ translation units

> In C++20, modules are introduced as an improved alternative to header files.

^ modules !

> In some cases, it may be necessary to declare a global variable or class in a .cpp file. In those cases, you need a way to tell the compiler and linker what kind of linkage the name has. The type of linkage specifies whether the name of the object is visible only in one file, or in all files. The concept of linkage applies only to global names. The concept of linkage doesn't apply to names that are declared within a scope. A scope is specified by a set of enclosing braces such as in function or class definitions
>
> A free function is a function that is defined at global or namespace scope. Non-const global variables and free functions by default have external linkage; they're visible from any translation unit in the program. No other global object can have that name. A symbol with internal linkage or no linkage is visible only within the translation unit in which it's declared. When a name has internal linkage, the same name may exist in another translation unit. Variables declared within class definitions or function bodies have no linkage

^ Again, il s'agit d'un cas particulier = les variables globales déclarées dans un cpp ; leur visibilité par d'autres translation-unit est appelée *linkage*.

> You can force a global name to have internal linkage by explicitly declaring it as static.

^ `static` pour des variables globales / free-floating functions.

## main function and command-line arguments

https://learn.microsoft.com/en-us/cpp/cpp/main-function-command-line-args

Rien de particulier à annoter ici

## Program termination

https://learn.microsoft.com/en-us/cpp/cpp/program-termination

> In C++, you can exit a program in these ways:
>
> - Call the exit function. \
> - Call the abort function. \
> - Execute a return statement from main.
>
> The difference between exit and abort is that exit allows the C++ runtime termination processing to take place (global object destructors get called). abort terminates the program immediately. The abort function bypasses the normal destruction process for initialized global static objects. It also bypasses any special processing that was specified using the atexit function.

^ C'est important : `exit` termine proprement, tandis que `abort` est brutal (et possiblement ne ferme pas des ressources telles qu'un lock sur une base de données).

> A return statement in main first acts like any other return statement. Any automatic variables are destroyed. Then, main invokes exit with the return value as a parameter.

^ De ce point de vue, un `return` depuis la fonction `main` n'a rien de particulier, en dehors du fait que le runtime appelle automatiquement `exit` après coup.

## Lvalues and rvalues

https://learn.microsoft.com/en-us/cpp/cpp/lvalues-and-rvalues-visual-cpp

> Every C++ expression has a type, and belongs to a value category. The value categories are the basis for rules that compilers must follow when creating, copying, and moving temporary objects during expression evaluation.
>
> The C++17 standard defines expression value categories as follows:

L'article détaille les différents types de values (mais ça ne vaut pas le coup d'annoter) :

- glvalue
- prvalue
- xvalue
- lvalue
- rvalue

## Temporary objects

https://learn.microsoft.com/en-us/cpp/cpp/temporary-objects

> A temporary object is an unnamed object created by the compiler to store a temporary value.
>
> Any expression that creates more than one temporary object eventually destroys them in reverse order of creation.

^ Les temporary sont créés par le compiler.

## Alignment

https://learn.microsoft.com/en-us/cpp/cpp/alignment-cpp-declarations

> One of the low-level features of C++ is the ability to specify the precise alignment of objects in memory to take maximum advantage of a specific hardware architecture. By default, the compiler aligns class and struct members on their size value: bool and char on 1-byte boundaries, short on 2-byte boundaries, int, long, and float on 4-byte boundaries, and long long, double, and long double on 8-byte boundaries.

^ L'alignement par défaut est sur la taille de la structure.

> In most scenarios, you never have to be concerned with alignment because the default alignment is already optimal. In some cases, however, you can achieve significant performance improvements, or memory savings, by specifying a custom alignment for your data structures. 

^ Dans la plupart des cas, l'alignement ne m'intéressera pas.

> you should use the C++11 standard keywords alignof and alignas for maximum code portability.
>
> Use the aligned_storage class for memory allocation of data structures with custom alignments.

Keywords pour travailler avec l'alignement.

> Alignment is a property of a memory address, expressed as the numeric address modulo a power of 2. For example, the address 0x0001103F modulo 4 is 3. That address is said to be aligned to 4n+3, where 4 indicates the chosen power of 2. The alignment of an address depends on the chosen power of 2. The same address modulo 8 is 7. An address is said to be aligned to X if its alignment is Xn+0.

^ Excellente définition, qui devrait d'ailleurs plutôt être en tête de page.

> CPUs execute instructions that operate on data stored in memory. The data are identified by their addresses in memory. A single datum also has a size. We call a datum naturally aligned if its address is aligned to its size. It's called misaligned otherwise. For example, an 8-byte floating-point datum is naturally aligned if the address used to identify it has an 8-byte alignment.
>
> Compilers attempt to make data allocations in a way that prevents data misalignment.
>
> For simple data types, the compiler assigns addresses that are multiples of the size in bytes of the data type. For example, the compiler assigns addresses to variables of type long that are multiples of 4, setting the bottom 2 bits of the address to zero.
>
> The compiler also pads structures in a way that naturally aligns each element of the structure.

^ Comment le compilo bosse avec l'alignement.

> The alignas type specifier is a portable, C++ standard way to specify custom alignment of variables and user defined types. The alignof operator is likewise a standard, portable way to obtain the alignment of a specified type or variable.

## Trivial, standard-layout and POD types

https://learn.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types

J'ai creusé le sujet il y a quelques temps, notamment en utilisant cette page ; mes notes sont [ici](./structured/cpp/pod_structures.md).
