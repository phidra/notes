# Eliminating Data Races in Firefox – A Technical Report – Mozilla Hacks - the Web developer blog

- **url** = https://hacks.mozilla.org/2021/04/eliminating-data-races-in-firefox-a-technical-report/
- **type** = post
- **auteurs** = Christian HOLLER (Security Engineer), Alexis BEINGESSNER (dev rust), Kris WRIGHT (crash reporting peer), tous trois chez mozilla
- **date de publication** = 2021-04-06
- **source** = https://hacks.mozilla.org/, le blog tech de mozilla

**TL;DR** : mozilla a utilisé ThreadSanitizer sur le projet firefox, voici leur retour d'expérience :
- la partie difficile était plutôt la partie humaine :
    - convaincre les gens de l'intérêt (les utilisateurs pensaient que la plupart des data-races étaient bénins)
    - et que ce ne serait pas trop lourd à l'usage (il faut pas être bloqué par la grande quantité d'alertes dûes au legacy)
- un élément qui a joué sur ceci est le fait que TSan ne produise pas de faux-positif (sous conditions de bonne utilisation, que l'article précise)
- donne des stats sur la ventilation des alertes remontées et leur fix (e.g. utiliser des atomics, supprimer du code inutile, ...)
- donne les principales causes de data-races dans le code
- parle de rust :
    - rust n'est pas infaillible, mais ses erreurs sont moins nombreuses, et plus simples à corriger
    - en revanche, des data-races C++ peuvent être "obfuscées" car passées à rust
    - de plus, la jeunesse de rust le rend moins compatible avec un outil comme TSan (mais je ne comprends pas bien pourquoi)


## La mise en place

> What is ThreadSanitizer? ThreadSanitizer (TSan) is compile-time instrumentation to detect data races according to the C/C++ memory model on Linux.
>
> One important property of TSan is that, when properly deployed, the data race detection does not produce false positives.
>
> TSan is built into Clang and can be used with any recent Clang/LLVM toolchain. If your C/C++ project already uses e.g. AddressSanitizer (which we also highly recommend), deploying ThreadSanitizer will be very straightforward from a toolchain perspective.

Le point qui leur a le plus posé problème est un point "humain" :

> The most significant issue we faced was that it is really difficult to prove that data races are actually harmful at all and that they impact the everyday use of Firefox. In particular, the term “benign” came up often. Benign data races acknowledge that a particular data race is actually a race, but assume that it does not have any negative side effects.
>
> While benign data races do exist, we found (in agreement with previous work on this subject
>
> The reasons for this are clear: It is hard to reason about what compilers can and will optimize, and confirmation for certain “benign” data races requires you to look at the assembler code that the compiler finally produces

Du coup, ils choisissent de ne pas tolérer les data-races :

> As a result, we decided that the ultimate goal should be a “no data races” policy that declares even benign data races as undesirable due to their risk of misclassification
> 
> This is where TSan’s suppression list came in handy: We knew we had to stop the influx of new data races but at the same time get the tool usable without fixing all legacy issues.

Je suppose que ça permet d'ignorer des alertes volontairement.

## Les fix

> we found that the majority of these fixes were trivial and/or improved code quality.
>
> The trivial fixes were mostly turning non-atomic variables into atomics (20%), adding permanent suppressions for upstream issues that we couldn’t address immediately (15%), or removing overly complicated code (20%). Only 45% of the benign fixes actually required some sort of more elaborate patch (as in, the diff was larger than just a few lines of code and did not just remove code).

Ils ont tout de même dû contourner un peu :

> TSan does not produce false positive data race reports when properly deployed, which includes instrumenting all code that is loaded into the process and avoiding primitives that TSan doesn’t understand (such as atomic fences)

## Les causes

Les bitfields gagnent de la mémoire (?) mais sont dangereux :

> For the most part this works fine, but it has one nasty consequence: different pieces of data now alias. This means that accessing “neighboring” bitfields is actually accessing the same memory, and therefore a potential data race.
>
> We find this bug particularly interesting because it demonstrates how hard it is to diagnose data races without appropriate tooling

Autre cause :

> Oops That Wasn’t Supposed To Be Multithreaded

Autre cause :

> intentionally racily reading a value, but then later doing checks that properly validate it

Et son exemple de code est très parlant :

```cpp
static bool sInitialized = false;
static Mutex sMutex; // guards sInitialized

// racily check if initialized (gotta go fast!)
if (!sInitialized) {

  // now properly check
  auto lock = sMutex.Lock();

  if (!sInitialized) {
    /* initialization code */
    sInitialized = true;
  }
}
```

Ils ne recommandent pas :

> Please Don’t Do This. These patterns are really fragile and they’re ultimately undefined behavior, even if they generally work right. Just write proper atomic code

## Au sujet de  rust

Le manque de maturité de rust est une difficulté (mais je ne vois pas bien pourquoi) :

> Another difficulty that we had to solve during TSan deployment was due to part of our codebase now being written in Rust, which has much less mature support for sanitizers.

Intéressant : même si rust n'est pas un problème en soi, il "pose problème" malgré tout indirectement, en masquant des data-races C++ :

> We weren’t particularly concerned with our Rust code having a lot of races, but rather races in C++ code being obfuscated by passing through Rust. In fact, we strongly recommend writing new projects entirely in Rust to avoid data races altogether.

Ceci dit, rust n'est pas infaillible :

> We did however find two pure Rust races:

... mais en plus de limiter drastiquement les erreurs, celles-ci sont plus faciles à corriger:

> Overall Rust appears to be fulfilling one of its original design goals: allowing us to write more concurrent code safely.
>
> What issues we did find were mistakes in the implementations of low-level and explicitly unsafe multithreading abstractions — and those mistakes were simple to fix.
> This is in contrast to many of our C++ races, which often involved things being randomly accessed on different threads with unclear semantics, necessitating non-trivial refactorings of the code.

La conclusion de l'article est intéressante, car l'article ne porte pas tant sur "est-ce que TSan apporte quelque chose" (c'est tenu pour acquis), mais plutôt sur "ce qui est dur, c'est de le mettre en place. Mais croyez nous, c'est faisable" :

> ThreadSanitizer has proven to be not just effective in locating data races and providing adequate debug information, but also to be practical even on a project as large as Firefox.
