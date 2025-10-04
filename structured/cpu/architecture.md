* [C'est quoi l'architecture d'un processeur](#cest-quoi-larchitecture-dun-processeur)
* [RISC vs. CISC](#risc-vs-cisc)
* [Instructions SIMD](#instructions-simd)
* [Registres](#registres)
* [Mode d'adressage](#mode-dadressage)
* [x86, ARM, et les différentes architectures](#x86-arm-et-les-différentes-architectures)
* [amd64 vs. x86-64 vs. intel-64](#amd64-vs-x86-64-vs-intel-64)


# C'est quoi l'architecture d'un processeur

https://fr.wikipedia.org/wiki/Architecture_de_processeur

Une architecture (externe) de processeur, c'est l'interface EXTERNE du processeur, i.e. ce qu'en voit un programmeur qui code en assembleur.

Le jeu d'instructions est très important, mais il n'est pas le seul élément constituant l'architecture (i.e. intéressant un dev qui code en assembleur) :

- quels sont les registres ?
- quels sont les types de données manipulées ?
- quelles sont les IO ?
- Comment adresse-t-on la mémoire ? Quels modes d'adressage ?
- Comment sont gérées les erreurs ou exceptions ?
- Si plusieurs cores ou processeurs, comment communiquent-ils ?
- Quels sont les caches existants ? (e.g. le TLB), car il arrive que les programmes doivent interagir avec les caches, qui ne peuvent donc pas être considérés comme des détails d'implémentation.

# RISC vs. CISC

Une architecture CISC propose de nombreux modes d'adressage, dont certains sont souvent complexes.

Le jeu d'instructions comporte souvent de nombreuses instructions complexes qui seront réalisées en plusieurs cycles.

Une architecture RISC propose un jeu d'instructions relativement réduit. Chacune de ces instructions est censée être exécutée en un seul cycle. Les modes d'adressage sont plus simples que dans une architecture CISC. L'architecture propose en général un nombre important de registres généraux. Ces caractéristiques favorisent une utilisation optimale du pipeline au niveau de la microarchitecture.

# Instructions SIMD

TL;DR = ce sont des instructions propres à une architecture donnée, capables de traiter plusieurs données en une seule instruction.

SIMD : Afin d'améliorer les performances en calcul vectoriel (lié à l'algèbre linéaire) des processeurs traditionnels, on peut adjoindre à leur architecture de base une unité dédiée aux traitements SIMD, effectuant des calculs sur plusieurs données en une seule instruction.

On compte notamment les technologies suivantes :

- dans le monde PowerPC : AltiVec ;
- dans le monde ARM : NEON ;
- dans le monde x86 : MMX (multimedia extensions), SSE (streaming SIMD extensions), AVX (advanced vector extensions).

# Registres

https://fr.wikipedia.org/wiki/Registre_de_processeur

Une architecture externe de processeur définit un ensemble de registres, dits architecturaux, qui sont accessibles par son jeu d'instructions. Ils constituent l'état externe (architectural) du processeur. Cependant, une réalisation donnée d'une architecture interne (microarchitecture) peut contenir un ensemble différent de registres, qui sont en général plus nombreux que les registres architecturaux. Ils stockent non seulement l'état externe du processeur, mais aussi celui de sa microarchitecture : valeurs opérandes, indicateurs, etc. Ce dernier état est utilisé exclusivement par la microarchitecture, et n'est pas visible par le jeu d'instructions (architecture)[

Sur de nombreux processeurs, les registres sont spécialisés et ne peuvent contenir qu'un type bien précis de données.

On rencontre souvent les classes de registres suivantes :

- les registres entiers, chargés de stocker des nombres entiers (et éventuellement des adresses) ;
- les registres flottants, qui stockent des nombres à virgule flottante ;
- les registres d'adresses : sur certains processeurs, les adresses mémoires à manipuler sont placées dans ces registres dédiés ;

# Mode d'adressage

https://fr.wikipedia.org/wiki/Mode_d%27adressage

Les modes d'adressage définis dans une architecture régissent la façon dont les instructions en langage machine identifient leurs opérandes. Un mode d'adressage spécifie la façon dont est calculée l'adresse mémoire effective d'un opérande à partir de valeurs contenues dans des registres et de constantes contenues dans l'instruction ou ailleurs dans la machine.

En programmation informatique, les personnes concernées par les modes d'adressage sont principalement celles qui programment en assembleur et les auteurs de compilateurs.

# x86, ARM, et les différentes architectures

`x86` est le nom de l''architecture originale, elle a été renommée en `IA-32` à partir du pentium. Derrière, elle a encore évolué en `x64`.

NDM : quand on compile un programme vers une architecture cible, on le compile pour une **famille** de processeurs donnée = tous ceux qui suivent la même architecture.

Exemples d'architectures :

- `x86` étendue en `IA-32`, elle-même étendue en `x64`
- Motorola 680x0
- SPARC
- POWER
- PowerPC
- MIPS
- ARM
- ...

https://fr.wikipedia.org/wiki/X86

> Un constructeur de microprocesseur pour compatible PC doit maintenir une compatibilité descendante avec ce jeu d'instructions s'il veut que les logiciels déjà écrits fonctionnent sur les nouveaux microprocesseurs.

^ si un nouveau processeur se contente d'ajouter des nouveaux trucs à une archi existante, il pourra exécuter des programmes compilés pour l'ancien processeur (pas besoin de les recompiler)

# amd64 vs. x86-64 vs. intel-64

https://fr.wikipedia.org/wiki/AMD64

AMD64 est l'évolution à 64 bits de l'architecture de processeur x86. AMD64 a été inventée par Advanced Micro Devices (AMD). Le nom x86-64 est généralement utilisé lorsque l'on parle de ce jeu d'instructions sans faire référence à la marque AMD.

Nomenclature Industrielle

> Puisque les architectures AMD64 et Intel 64 sont relativement similaires, beaucoup de produits logiciels et matériels utilisent un terme commercial neutre pour indiquer leur compatibilité avec les deux implémentations. La désignation d'origine d'AMD pour l'architecture de ce processeur, "x86-64", est encore parfois utilisée dans ce but, tout comme "x86_64"[1]. D'autres entreprises, comme Microsoft et Sun Microsystems, utilisent la contraction "x64" au niveau marketing.

Beaucoup de systèmes d'exploitation et de produits, spécialement ceux qui introduisirent la prise en charge de x86-64 avant la venue d'Intel sur cette architecture, utilisent le terme "AMD64" ou "amd64" pour se référer à la fois à AMD64 et à Intel 64.

- les systèmes BSD comme FreeBSD, NetBSD et OpenBSD incluent AMD64 et Intel 64 sous le nom "amd64".
- Debian, Ubuntu, et Gentoo incluent AMD64 et Intel 64 sous le nom "amd64".
- Fedora, PackageKit, openSUSE, et Arch Linux la nomment "x86_64".
- Java Development Kit (JDK): Le nom "amd64" est utilisé pour les noms de répertoires contenant des fichiers x86-64.
- Mac OS X: Apple parle de "x86_64" notamment dans la commande de Terminal arch et dans la documentation[2].
- Microsoft Windows: Les versions x64 de Windows utilisent le AMD64 moniker pour désigner divers composants qui utilisent ou sont compatibles avec cette architecture. Par exemple, le dossier système dans un CD-ROM d'installation du « Windows x64 Edition » est nommé "AMD64", pour le différencier de ceux des versions 32 bits nommés "i386".

https://fr.wikipedia.org/wiki/Intel_64

> Intel 64 autrefois appelé Extended Memory 64-bit Technology (ou EM64T en abrégé) est l'implémentation Intel de l'architecture x86-64, une extension 64-bit de l'architecture IA-32.

Différences entre l'AMD64 et l'Intel 64

Bien qu'étant largement compatible au niveau binaire, Intel ne peut pas intégrer une technologie sous licence AMD sans en payer le prix. C'est pourquoi il existe de subtiles différences entre les deux implémentations, notamment en termes d'instructions (par exemple, CMPXCHG16B fut ajouté par Intel). Toutefois, les compilateurs savent s'en accommoder et produire des exécutables qui fonctionnent sans modification sur les deux architectures

https://www.reddit.com/r/linuxquestions/comments/ra4vjj/difference_between_x86_x8664_amd64_and_x64/

^ explications sur les différents archis 64
