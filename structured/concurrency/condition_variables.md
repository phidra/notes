* [C'est quoi une condition-variable aka CV ?](#cest-quoi-une-condition-variable-aka-cv-)
* [Ça sert à quoi ?](#ça-sert-à-quoi-)
* [À quoi est-ce que ça ne sert PAS ?](#à-quoi-est-ce-que-ça-ne-sert-pas-)
* [Caveat = spurious wakeup / lost wakeup](#caveat--spurious-wakeup--lost-wakeup)
   * [spurious wakeup](#spurious-wakeup)
   * [lost wakeup](#lost-wakeup)
* [Utilisation standard pour éviter les problèmes](#utilisation-standard-pour-éviter-les-problèmes)
* [Infos complémentaires](#infos-complémentaires)


# C'est quoi une condition-variable aka CV ?

Mécanisme de bas-niveau permettant de "réveiller" un autre thread.

Description donnée [dans le code python](https://github.com/python/cpython/blob/c7a79bb036b42f96b7379b95efa643ee27df2168/Lib/threading.py#L224) (voir aussi [la doc python de threading.Condition](https://docs.python.org/3/library/threading.html#condition-objects)) :

> A condition variable allows one or more threads to wait until they are notified by another thread.

Définition donnée sur [cppreference](https://en.cppreference.com/w/cpp/thread/condition_variable) :

> The condition_variable class is a synchronization primitive that can be used to block a thread [...] until another thread both modifies a shared variable (the condition), and notifies the condition_variable.

# Ça sert à quoi ?

Cas principal :

- un thread T1 s'endort, et ne souhaite se réveiller que lorsqu'il se passe quelque chose qui l'intéresse (e.g. un évènement `E` s'est produit)
- en parallèle, un thread T2 fait du travail ; à un moment donné il se rend compte que `E` s'est produit → il veut réveiller T1
- pour que T2 puisse réveiller T1 au bon moment, T1 et T2 utilisent une condition-variable pour se synchroniser

En pratique, l'utilisation d'une CV est obligatoirement couplée à un flag protégé par un lock (pour éviter les spurious wakeup, cf. ci-dessous), donc un cas supplémentaire est celui-ci où T2 veut non seulement réveiller T1, mais également lui **transmettre** de la donnée à traiter de façon threadsafe :

- un thread T1 de conversion vidéo s'endort, et ne souhaite se réveiller que lorsqu'il a une vidéo à convertir
- en parallèle, un thread T2 télécharge des vidéos ; à un moment donné, il a fini de télécharger la vidéo `lolcat.avi`
- T2 souhaite réveiller T1 (pour qu'il puisse la convertir en `lolcat.mp4`) et lui transmettre la donnée à convertir
    > "réveille-toi, j'ai fini de préparer ta donnée d'entrée, la voici, protégée par notre lock partagé"
- T1 et T2 utilisent un couple {CV + lock} pour se synchroniser, et pour échanger la donnée sans se marcher sur les pieds.

# À quoi est-ce que ça ne sert PAS ?

Si on est dans un modèle producteur+consommateur (e.g. un thread T1 produit de la donnée, et un ou plusieurs threads T2 consomment cette donnée), la bonne façon d'implémenter ce modèle est plutôt d'utiliser une queue threadsafe sur laquelle T1 `put` et T2 `get`, et non de faire de la synchro à la main avec une CV !

(possiblement, la queue threadsafe utilise elle-même une CV comme mécanisme de synchronisation, mais c'est un détail qui nous est abstrait par l'API de la queue)

# Caveat = spurious wakeup / lost wakeup

## spurious wakeup

En résumé, lorsqu'un thread utilise une CV et attend d'être réveillé, il se peut qu'il soit réveillé "à tort".

https://en.wikipedia.org/wiki/Spurious_wakeup :

> A spurious wakeup happens when a thread wakes up from waiting on a condition variable that's been signaled, only to discover that the condition it was waiting for isn't satisfied.
>
> condition variables may also be allowed to return from a wait even if not signaled, though it is not clear how many implementations actually do that
>
> Because spurious wakeups can happen whenever there's a race and possibly even in the absence of a race or a signal, when a thread wakes on a condition variable, it should always check that the condition it sought is satisfied. If it is not, it should go back to sleeping on the condition variable, waiting for another opportunity.

Quelques infos dans [cette réponse stackoverflow](https://stackoverflow.com/questions/8594591/why-does-pthread-cond-wait-have-spurious-wakeups/8594964#8594964) :

> There are at least two things 'spurious wakeup' could mean:
> -  A thread blocked in pthread_cond_wait can return from the call even though no call to pthread_call_signal or pthread_cond_broadcast on the condition occurred.
> -  A thread blocked in pthread_cond_wait returns because of a call to pthread_cond_signal or pthread_cond_broadcast, however after reacquiring the mutex the underlying predicate is found to no longer be true.

Le premier point dépend des implémentations, mais le deuxième point est toujours vrai → un thread réveillé doit TOUJOURS vérifier (via un flag booléen setté par le thread réveillant p.ex.) s'il y a du travail à faire ou si le réveil était spurious.

Dit autrement, c'est une ERREUR de considérer que si on a été réveillé, c'est pour une bonne raison !

## lost wakeup

https://www.modernescpp.com/index.php/c-core-guidelines-be-aware-of-the-traps-of-condition-variables

> Lost wakeup: The phenomenon of the lost wakeup is that the sender sends its notification before the receiver gets to its wait state. The consequence is that the notification is lost.

# Utilisation standard pour éviter les problèmes

Du coup, l'utilisation canonique côté consommateur est de 1. `wait` de façon bloquante, et 2. aussitôt qu'on est réveillé, checker (e.g. via un flag booléen) si le réveil est spurious (auquel cas on se rendort aussitôt), ou réel (auquel cas on peut travailler) :

Exemple [en python](https://docs.python.org/3/library/threading.html#condition-objects) :

```python
with cv:
    # on wait en boucle jusqu'à ce que la condition soit vraie :
    while not an_item_is_available():
        cv.wait()
    do_the_work_with_the_item()
```

Exemple [en C++](https://en.cppreference.com/w/cpp/thread/condition_variable/wait) :

```cpp
// on wait en boucle jusqu'à ce que la condition soit vraie :
while (!stop_waiting()) {
    wait(lock);
}
```

Comme on doit obligatoirement vérifier un flag (ou une donnée) partagé(e) entre le thread producteur et le thread consommateur — ce flag représentant la **condition** qui doit être à `True` pour que le réveil soit réel — les CV sont **obligatoirement** utilisées avec un lock permettant de protéger ce flag partagé.

# Infos complémentaires

En python, le lock peut être caché dans l'implémentation (un lock par défaut est créé à la construction si on ne lui en a pas passé explicitement).

On pourrait se dire qu'en python, `threading.Event` sert justement à réveiller les threads qui dorment, donc quel intérêt d'utiliser une CV ? En fait, les `threading.Event` python sont justement implémentés avec des CV : [lien1](https://github.com/python/cpython/blob/c7a79bb036b42f96b7379b95efa643ee27df2168/Lib/threading.py#L554), [lien2](https://github.com/python/cpython/blob/c7a79bb036b42f96b7379b95efa643ee27df2168/Lib/threading.py#L224).

D'ailleurs, c'est pas hyper-clair si les `threading.Event` peuvent avoir des spurious wakeups : d'un côté la doc n'en parle pas, pas même dans son exemple de base ; de l'autre, d'après le code, on se contente d'un `wait` sur une CV, donc je pense qu'on devrait avoir un risque de spurious-wakeup aussi... [Cette question stackoverflow](https://stackoverflow.com/questions/57399474/does-the-threading-event-object-suffer-from-spurious-wakeup) pose la même question... mais n'a pas de réponse :-/

Quelques liens :

- [un exemple de cours utilisant les CV, en java](https://web.stanford.edu/~ouster/cgi-bin/cs140-spring14/lecture.php?topic=locks)
- [le man pthread_cond_signal](https://linux.die.net/man/3/pthread_cond_signal) ; sous Linux, les CV utilisent pthread
- [la core-guidelines C++ sur l'utilisation des CV](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-wait), ainsi qu'[un blogpost la détaillant](https://www.modernescpp.com/index.php/c-core-guidelines-be-aware-of-the-traps-of-condition-variables)
- [le couple CV+lock semble parfois être appelé Monitor](https://en.wikipedia.org/wiki/Monitor_(synchronization))
