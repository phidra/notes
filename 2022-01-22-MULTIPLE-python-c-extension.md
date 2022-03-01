# Notes déstructurées sur les extensions C en python issues de MULTIPLES sources

- **url** = multiples
- **type** = multiple docs
- **auteur** = multiples
- **date de publication** = multiples
- **source** = multiples
- **tags** = language>python ; topic>c-extension ; level>intermediate

**TL;DR** : agrégats de notes sur les extensions python codées en C

* [Notes déstructurées sur les extensions C en python issues de MULTIPLES sources](#notes-déstructurées-sur-les-extensions-c-en-python-issues-de-multiples-sources)
   * [Writing the Setup Script](#writing-the-setup-script)
   * [C++ python bindings in 5 minutes](#c-python-bindings-in-5-minutes)
   * [What are (c)python extension modules?](#what-are-cpython-extension-modules)
   * [Writing cpython extension modules using C++](#writing-cpython-extension-modules-using-c)
   * [Extending and Embedding the Python Interpreter](#extending-and-embedding-the-python-interpreter)
      * [Extending Python with C or C++](#extending-python-with-c-or-c)
         * [Reference count](#reference-count)
      * [Defining Extension Types: Tutorial](#defining-extension-types-tutorial)
      * [Defining Extension Types: Assorted Topics](#defining-extension-types-assorted-topics)
      * [Building C and C++ Extensions](#building-c-and-c-extensions)
   * [Python/C API Reference Manual](#pythonc-api-reference-manual)
      * [Introduction](#introduction)
   * [Coding Patterns for Python Extensions](#coding-patterns-for-python-extensions)
   * [Building a Python C Extension Module](#building-a-python-c-extension-module)
   * [Python Bindings: Calling C or C++ From Python](#python-bindings-calling-c-or-c-from-python)

## Writing the Setup Script

**url** = https://docs.python.org/3/distutils/setupscript.html

Cette page est la doc de distuils (et normalement, je n'utilise _jamais_ distutils directement, je passe par setuptools) ; en pratique, l'une de ses sections traite des extensions C et donne des infos qui restent utiles = https://docs.python.org/3/distutils/setupscript.html#describing-extension-modules

L'argument `ext_modules` du `setup(...)` permet d'indiquer les extensions, c'est une liste d'instances d'objets `Extension`.

Le code minimal décrit juste le nom du module et sa source C :

```python
ext_modules=[Extension('foo', ['foo.c'])],
```

La doc de `Extension` est ici = https://docs.python.org/3/distutils/apiref.html#distutils.core.Extension ; il y a plein de keywords et de trucs intéressants !

> The Extension class describes a single C or C++ extension module in a setup script.

On peut passer des indications de configuration au build-system qui sera utilisé pour builder l'extension :

- on peut configurer la compilation :
    - `include_dirs`
    - `define_macros`
    - `undef_macros`
- on peut configurer les librairies à utiliser au link :
    - `library_dirs`
    - `libraries`
- on peut même configurer les librairies à utiliser au runtime !
    - `runtime_library_dirs`

Attention : si on est trop précis sur ce qu'on met comme répertoire d'include, on ne pourra builder sur toutes les plateformes... Par exemple, la config suivante ne buildera que sur un unix-like et pas sous windows :

```python
p.ex. include_dirs=['/usr/include/X11']  
```

## C++ python bindings in 5 minutes

**url** = https://www.blopig.com/blog/2021/03/c-python-bindings-in-5-minutes/

Cette page donne un exemple complet et fonctionnel utilisant [pybind11](https://pybind11.readthedocs.io/en/stable/index.html) = un tool pour facilement définir des extensions python en C++ ; j'ai fait une [POC dessus](https://github.com/phidra/pocs/tree/dcf6b818d19e47a8fee655b389ca34ea914e8314/python/LIB_pybind11_01_basics).


## What are (c)python extension modules?

**url** = https://thomasnyberg.com/what_are_extension_modules.html

L'article est assez ancien (2017 — les exemples sont donnés avec python 3.6), mais intéressant : il explique ce qu'est une extension C du point de vue de python = un simple shared-object que l'interpréteur python va _dlopen_ :

> The upshot of this article is that a C extension module (at least one compiled for cpython on a debian/Ubuntu OS) is a shared library which (correctly) makes use of the cpython interpreter C api and which exports one specifically named initialization function.

Du coup, on peut builder une extension sans aucun outil, et à partir de n'importe quel langgage, potentiellement : tant qu'on produit un shared-object avec les bons symboles (en fait, uniquement `PyInit_spam` où `spam` est le nom de mon module), ça sera une extension python valide.

## Writing cpython extension modules using C++

**url** = https://thomasnyberg.com/cpp_extension_modules.html

L'article est un peu plus récent que le précédent, mais reste ancien (2018 — les exemples sont donnés avec python 3.6) : il traite des extensions python codées en C++.

> a C extension module is a shared library which makes use of the cpython interpreter C api and which exports one specifically named initialization function.
>
> That means that as long as you can produce a shared library of that form, it does not really matter what language you use to do so.

Il donne un code minimal pour builder une extension C++ pour python, ça a l'air d'être possible de base avec python (i.e. sans third-party tool).

Notamment, la macro `PyMODINIT_FUNC` ajoute un `extern "C"` pour éviter que C++ mangle le nom.

Le reste de l'article suit son utilisation de l'API python pour faire une extension C++ et donne des infos spécifiques au C++ (e.g. exceptions).

## Extending and Embedding the Python Interpreter

**url** = https://docs.python.org/3/extending/index.html

Le doc et ses sous-pages décrivent deux usages (qui n'ont rien à voir du point de vue utilisation... mais sont très proches du point de vue implémentation) :

- écrire des modules en C/C++ (pour les charger dynamiquement dans l'interpréteur)
- embarquer l'interpréteur python dans une application C/C++ = pouvoir utiliser du code python dans un main C++ (c'est pas ce que je veux aujourd'hui, mais ça pourra faire l'objet d'une POC intéressante)

Notamment, un point intéressant est qu'on peut tout à fait écrire des extensions C/C++ sans aucun third-party tools tels que pybind11 (même s'il reste recommandé d'un utiliser) :

> This guide only covers the basic tools for creating extensions provided as part of this version of CPython. Third party tools like Cython, cffi, SWIG and Numba offer both simpler and more sophisticated approaches to creating C and C++ extensions for Python. This section of the guide covers creating C and C++ extensions without assistance from third party tools. It is intended primarily for creators of those tools, rather than being a recommended way to create your own C extensions.

### Extending Python with C or C++

**url** =  https://docs.python.org/3/extending/extending.html

Un truc que seules les extensions C peuvent faire = créer de nouveaux builtin objects (i.e. au même niveau que `str` ou `dict`), et utiliser la libc ou les syscalls.

C'est en fait via l'API C python qu'on écrit l'extension (ce qui est cohérent avec ce qu'en disait un article ci-dessus = _a C extension module is a shared library which makes use of the cpython interpreter C api (and which exports one specifically named initialization function)_) :

> To support extensions, the Python API (Application Programmers Interface) defines a set of functions, macros and variables that provide access to most aspects of the Python run-time system. The Python API is incorporated in a C source file by including the header "Python.h".

On utilise pas une éventuelle API python "générale" stable et bien définie, mais bien l'API CPython (et même celle propre à UNE version de python) ; les third-party tools comme cffi permettent à l'inverse d'écrire des extensions plus robustes aux différentes versions de python :

> The C extension interface is specific to CPython, and extension modules do not work on other Python implementations.

L'exemple bateau, fil rouge de la page, est celui d'un module `spam` codé en C, qui expose une fonction `spam.system` qui wrappe la fonction C `system` (qui prend une string en argument).

Convention : si le module est `spam`, alors le fichier C s'appelle `spammodule.c`.

Les deux lignes suivantes doivent être inclues AVANT les autres includes :

```c
#define PY_SSIZE_T_CLEAN
#include <Python.h>
```

Nommage des symboles :

> All user-visible symbols defined by Python.h have a prefix of Py or PY

Lorsqu'on appelle `spam.system(string)`, c'est une fonction C de notre code C qui sera appelée :

> The next thing we add to our module file is the C function that will be called when the Python expression spam.system(string) is evaluated (we’ll see shortly how it ends up being called)

Les fonctions C correspondant aux fonctions python du module suivent une convention pour le passage des arguments :

> The C function always has two arguments, conventionally named self and args.

    static PyObject* spam_system(PyObject *self, PyObject *args)
> The self argument points to the module object for module-level functions; for a method it would point to the object instance.
>
> The args argument will be a pointer to a Python tuple object containing the arguments.
>
> The arguments are Python objects — in order to do anything with them in our C function we have to convert them to C values

Il y a une sous-section dédiée à la gestion des erreurs = https://docs.python.org/3/extending/extending.html#intermezzo-errors-and-exceptions

> An important convention throughout the Python interpreter is the following: when a function fails, it should set an exception condition and return an error value (usually -1 or a NULL pointer).
>
> When a function f that calls another function g detects that the latter fails, f should itself return an error value (usually NULL or -1). It should not call one of the PyErr_* functions — one has already been called by g.

Donc :

- pour signaler une erreur dans son propre code, il faut définir un état d'erreur et renvoyer `NULL`
- pour signaler une erreur dans une fonction qu'on appelle, on renvoie `NULL` sans toucher à l'état d'erreur (déjà défini par la fonction appelée), c'est ce qui se passe avec `PyArg_ParseTuple` dans le fil-rouge

En résumé sur le passage d'arguments :

- mon code C ne sait travailler qu'avec des structures C
- mais l'interpréteur python ne manipule que des structures python (les `PyObject`)
- du coup, à l'entrée et à la sortie, il faut convertir
- note : pour renvoyer `None`, il faut incrémenter le compteur de référence avant de le renvoyer

Puis, on déclare statiquement la liste des méthodes de notre module :

```c
static PyMethodDef SpamMethods[] = {
    ...
    {"system",  spam_system, METH_VARARGS, "Execute a shell command."},
    ...
    {NULL, NULL, 0, NULL}        /* Sentinel */
};
```

Ce que cette déclaration dit, c'est que quand EN PYTHON, on appelle `spam.system`, ça redirige sur la fonction C `spam_system`.

De même, on déclare statiquement notre module et ses infos (dont la liste des méthodes déclarée juste au dessus, mais également d'autres choses, comme sa docstring, p.ex.)

Enfin, on déclare la **SEULE FONCTION NON-STATIQUE** du fichier C = une fonction avec un nom bien précis, qui va servir à initialiser le module (en utilisant les déclarations statiques précédentes) :

```c
PyMODINIT_FUNC
PyInit_spam(void)
{
    return PyModule_Create(&spammodule);
}
```

La macro `PyMODINIT_FUNC` s'occupe de gérer `extern "C"` si le compilo est C++.

Derrière, le process suivi par l'interpréteur python utilise la fonction publique ci-dessus pour initialiser le module :

> When the Python program imports module spam for the first time, PyInit_spam() is called. It calls PyModule_Create(), which returns a module object, and inserts built-in function objects into the newly created module based upon the table (an array of PyMethodDef structures) found in the module definition. PyModule_Create() returns a pointer to the module object that it creates. [...] The init function must return the module object to its caller, so that it then gets inserted into sys.modules.

À noter que plutôt que d'en faire une extension chargeable dynamiquement, on peut aussi en faire une extension de l'interpréteur lui-même (mais il faut alors rebuilder l'interpréteur) :

> If you can’t use dynamic loading, or if you want to make your module a permanent part of the Python interpreter, you will have to change the configuration setup and rebuild the interpreter.

Il y a une sous-section dédiée à l'inverse = au fait d'appeler des fonctions python depuis le code C = https://docs.python.org/3/extending/extending.html#calling-python-functions-from-c

Un cas où c'est utile est le cas où ma fonction python codée en C (`spam.system` ci-dessus)) accepte une callback python en paramètre. Dans ce cas, mon code C a besoin d'appeler la callback python, cf. `PyObject_CallObject`.

Il y a une sous-section dédiée à `PyArg_ParseTuple()` = https://docs.python.org/3/extending/extending.html#extracting-parameters-in-extension-functions

Il y a une sous-section dédiée à `Py_BuildValue` (qui fait le contraire de `PyArg_ParseTuple` = elle construit un objet python à partir d'un objet C et d'une chaîne de formattage) = https://docs.python.org/3/extending/extending.html#building-arbitrary-values

Il y a une sous-section dédiée au cas où on écrit une extension C qui n'a pas pour objectif d'être utilisé en python, mais plutôt par d'autres extensions C = https://docs.python.org/3/extending/extending.html#providing-a-c-api-for-an-extension-module (dans ce cas, il y a l'air d'avoir des trucs spéciaux, mais je saute la section)

Il y a une longue sous-section dédiée aux règles qui gouvernent le refcount en python = https://docs.python.org/3/extending/extending.html#reference-counts :

#### Reference count

> Py_DECREF() also frees the object when the count reaches zero. For flexibility, it doesn’t call free() directly — rather, it makes a call through a function pointer in the object’s type object. For this purpose (and others), every object also contains a pointer to its type object.

Le type de l'objet est stocké avec l'objet (et c'est le type qui appelle `free()`).

>  Nobody “owns” an object; however, you can own a reference to an object. An object’s reference count is now defined as the number of owned references to it. The owner of a reference is responsible for calling Py_DECREF() when the reference is no longer needed. Ownership of a reference can be transferred.

Le code client créée des références à un objet avec `Py_INCREF`, et s'en débarasse avec `Py_DECREF`.

> It is also possible to borrow a reference to an object. The borrower of a reference should not call Py_DECREF(). The borrower must not hold on to the object longer than the owner from which it was borrowed. Using a borrowed reference after the owner has disposed of it risks using freed memory and should be avoided completely.
>
> The advantage of borrowing over owning a reference is that you don’t need to take care of disposing of the reference on all possible paths through the code — in other words, with a borrowed reference you don’t run the risk of leaking when a premature exit is taken. The disadvantage of borrowing over owning is that there are some subtle situations where in seemingly correct code a borrowed reference can be used after the owner from which it was borrowed has in fact disposed of it.
>
> A borrowed reference can be changed into an owned reference by calling Py_INCREF(). This does not affect the status of the owner from which the reference was borrowed 

On peut emprunter des références, avec les caveats habituels du type use-after-free.

> Whenever an object reference is passed into or out of a function, it is part of the function’s interface specification whether ownership is transferred with the reference or not.

Ce point est important : l'interface d'une fonction définit implicitement si les références qu'elles reçoit ou retourne sont transférées (auquel cas elle doit `Py_DECREF`) ou empruntées.

> Most functions that return a reference to an object pass on ownership with the reference. In particular, all functions whose function it is to create a new object, such as PyLong_FromLong() and Py_BuildValue(), pass ownership to the receiver.Even if the object is not actually new, you still receive ownership of a new reference to that object.
>
> When you pass an object reference into another function, in general, the function borrows the reference from you — if it needs to store it, it will use Py_INCREF() to become an independent owner. There are exactly two important exceptions to this rule: PyTuple_SetItem() and PyList_SetItem(). These functions take over ownership of the item passed to them — even if they fail!
>
> When a C function is called from Python, it borrows references to its arguments from the caller. The caller owns a reference to the object, so the borrowed reference’s lifetime is guaranteed until the function returns. Only when such a borrowed reference must be stored or passed on, it must be turned into an owned reference by calling Py_INCREF().

Quelques règles habituelles.

> The solution, once you know the source of the problem, is easy: temporarily increment the reference count. The correct version of the function reads:

Un pattern classique lorsqu'on reçoit une référence empruntée est d'augmenter temporairement son refcount.

> Functions that return object references generally return NULL only to indicate that an exception occurred.

Gestion d'erreur usuelle = null pointer.

> The reason for not testing for NULL arguments is that functions often pass the objects they receive on to other function — if each function were to test for NULL, there would be a lot of redundant tests and the code would run more slowly.
>
> It is better to test for NULL only at the “source:” when a pointer that may be NULL is received, for example, from malloc() or from a function that may raise an exception
>
> It is a severe error to ever let a NULL pointer “escape” to the Python user.

### Defining Extension Types: Tutorial

**url** = https://docs.python.org/3/extending/newtypes_tutorial.html

Il y a une page dédiée à la définition de nouveaux types de base, équivalents à `str` et `dict`, je saute pour le moment.

> Python allows the writer of a C extension module to define new types that can be manipulated from Python code, much like the built-in str and list types.

### Defining Extension Types: Assorted Topics

**url** = https://docs.python.org/3/extending/newtypes.html

Je saute également cette page, qui concerne le même sujet.

### Building C and C++ Extensions

**url** = https://docs.python.org/3/extending/building.html

Cette page donne quelques infos sur comment builder notre module codé en C.

De nouveau, les extensions C ne sont que des shared-objects qui exportent une fonction particulière :

> A C extension for CPython is a shared library (e.g. a .so file on Linux, .pyd on Windows), which exports an initialization function.

Il y a des conventions à suivre sur le nommage du shared-object file pour que l'interpréteur python le "trouve" :

> To be importable, the shared library must be available on PYTHONPATH, and must be named after the module name, with an appropriate extension.

Il y a d'autres conventions à suivre sur le nommage de la fonction d'initilaisation du module pour que l'interpréteur python la "trouve" :

> The initialization function has the signature: PyObject *PyInit_modulename(void)

## Python/C API Reference Manual

**url** = https://docs.python.org/3/c-api/

C'est cette page qui documente l'API C python, utilisée pour les extensions (et probablement au sein de l'interpréteur aussi). Elle contient beaucoup de choses intéressantes, et beaucoup de sous-pages intéressantes.

Je viendrai sans doute incrémenter ces notes au fur et à mesure de mes lectures.

### Introduction

**url** = https://docs.python.org/3/c-api/intro.html

> To include the headers, place both directories (if different) on your compiler’s search path for includes. Do not place the parent directories on the search path and then use `#include <pythonX.Y/Python.h>;`

À vérifier, mais c'est dans doute une option `-I` passée par `setup.py build_ext` qui inclut `/usr/include/python3.8`

Il y a plein de macros utiles, par exemple : `Py_STRINGIFY(x)` = _Convert x to a C string. E.g. Py_STRINGIFY(123) returns "123"._ ou `Py_UNUSED(arg)` = _Use this for unused arguments in a function definition to silence compiler warnings. Example: int func(int a, int Py_UNUSED(b)) { return a; }._

Les objets sont tous instanciés dynamiquement, et associés à un type :

> Most Python/C API functions have one or more arguments as well as a return value of type PyObject*. This type is a pointer to an opaque data type representing an arbitrary Python object.
>
> Almost all Python objects live on the heap: you never declare an automatic or static variable of type PyObject, only pointer variables of type PyObject* can be declared. The sole exception are the type objects;
>
> All Python objects (even Python integers) have a type and a reference count. An object’s type determines what kind of object it is (e.g., an integer, a list, or a user-defined function
>
> For each of the well-known types there is a macro to check whether an object is of that type; for instance, PyList_Check(a) is true if (and only if) the object pointed to by a is a Python list.

Gestion du refcount, qui doit être géré manuellement :

> When an object’s reference count becomes zero, the object is deallocated. If it contains references to other objects, their reference count is decremented. Those other objects may be deallocated in turn, if this decrement makes their reference count become zero, and so on. 
>
> Reference counts are always manipulated explicitly. The normal way is to use the macro Py_INCREF() to increment an object’s reference count by one, and Py_DECREF() to decrement it by one. 
>
> The type-specific deallocator takes care of decrementing the reference counts for other objects contained in the object if this is a compound object type, such as a list, as well as performing any additional finalization that’s needed.

Pour les objets locaux, c'est pas indispensable d'incrémenter systématiquement :

> It is not necessary to increment an object’s reference count for every local variable that contains a pointer to an object.  [...] If we know that there is at least one other reference to the object that lives at least as long as our variable, there is no need to increment the reference count temporarily. An important situation where this arises is in objects that are passed as arguments to C functions in an extension module that are called from Python; the call mechanism guarantees to hold a reference to every argument for the duration of the call.

Mais attention à ce que l'objet pointé continuent de vivre pendant la durée du scope local — pour être safe, on incrémente temporairement :

> However, a common pitfall is to extract an object from a list and hold on to it for a while without incrementing its reference count. Some other operation might conceivably remove the object from the list, decrementing its reference count and possibly deallocating it.
>
> A safe approach is to always use the generic operations (functions whose name begins with PyObject_, PyNumber_, PySequence_ or PyMapping_). These operations always increment the reference count of the object they return. This leaves the caller with the responsibility to call Py_DECREF() when they are done with the result; this soon becomes second nature.

Un point important : le comportement vis-à-vis de la référence qu'une fonction nous renvoie (soit possédée, soit empruntée) dépend de la fonction !

> It is important to realize that whether you own a reference returned by a function depends on which function you call only

---

REPRISE = d'autres pages qui ont l'air intéressantes :

- https://docs.python.org/3/c-api/stable.html
- https://docs.python.org/3/c-api/intro.html
- https://docs.python.org/3/c-api/exceptions.html
- https://docs.python.org/3/c-api/abstract.html
- https://docs.python.org/3/c-api/init.html
- https://docs.python.org/3/c-api/memory.html
- https://docs.python.org/3/c-api/objimpl.html
- https://docs.python.org/3/c-api/apiabiversion.html
- https://docs.python.org/3/reference/datamodel.html#types

## Coding Patterns for Python Extensions

**url** = https://pythonextensionpatterns.readthedocs.io/en/latest/

La ressource est facile d'accès, même si elle n'est pas introductive (l'auteur suppose une préconnaissance du développement d'extensions C pour python) ; elle regroupe des bonnes pratiques poussées par l'auteur.

Quelques notes très vrac :

- [Chapitre 9](https://pythonextensionpatterns.readthedocs.io/en/latest/compiler_flags.html) = les flags de compilateur à utiliser.
- Pour un usage avancé, [la section 10](https://pythonextensionpatterns.readthedocs.io/en/latest/debugging/debug.html) donne des conseils de debugging très intéressants
   - Notamment, pour utiliser un IDE pour debugger l'extension, on peut créer un main C qui utilise l'extension.
- La [section 13](https://pythonextensionpatterns.readthedocs.io/en/latest/code_layout.html) donne également des infos sur ce sujet du testing
- La [section 14](https://pythonextensionpatterns.readthedocs.io/en/latest/cpp.html) a plein de trucs chouette pour se simplifier la vie avec C++, notamment gestion du refcount par RAII, et conversion depuis et vers les conteneurs de la STL
- La [section 17](https://pythonextensionpatterns.readthedocs.io/en/latest/further_reading.html) donne une ressource qui a l'air intéressante

## Building a Python C Extension Module

**url** = https://realpython.com/build-python-c-extension-module/

Cette page est un tuto alternatif à celui de la doc officielle python pour construire (et packager) une extension C à python.

Difficile d'avoir la date du post, mais d'après les commentaires les plus anciens, ça remonte à octobre 2019.

## Python Bindings: Calling C or C++ From Python

**url** = https://realpython.com/python-bindings-overview/

Complément de la précédente, cette page revoit des alternatives third-party possibles à l'écriture d'une extension python directement en C, avec tout le boilerplate :

- ctypes
- CFFI
- pybind11
- cython
- et d'autres...

Difficile d'avoir la date du post, mais d'après les commentaires les plus anciens, ça remonte à juillet 2020.
