* [C'est quoi une condition-variable aka CV ?](#cest-quoi-une-condition-variable-aka-cv-)
* [Ça sert à quoi ?](#ça-sert-à-quoi-)
* [À quoi est-ce que ça ne sert PAS ?](#à-quoi-est-ce-que-ça-ne-sert-pas-)
* [Caveat = spurious wakeup / lost wakeup](#caveat--spurious-wakeup--lost-wakeup)
   * [spurious wakeup](#spurious-wakeup)
   * [lost wakeup](#lost-wakeup)
* [Utilisation standard pour éviter les problèmes](#utilisation-standard-pour-éviter-les-problèmes)
   * [Précisions additionnelles sur l'utilisation en C++](#précisions-additionnelles-sur-lutilisation-en-c)
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

## Précisions additionnelles sur l'utilisation en C++

Contexte de ces précisions a posteriori : en décembre 2022, je m'intéresse aux mécanismes de communication IPC, et notamment à [boost::interprocess](https://www.boost.org/doc/libs/1_80_0/doc/html/interprocess/synchronization_mechanisms.html), qui permet d'utiliser des condition-variables entre plusieurs process.

En préambule, on suppose que les deux process se partagent :

- `the_cv` = une [condition-variable](https://www.boost.org/doc/libs/1_80_0/doc/html/boost/interprocess/named_condition.html) pour envoyer/recevoir les notifications
- `is_notified` = un bool (partagé en shared-memory !) indiquant qu'une notification a été envoyée :
    - initialisé à `false`
    - l'envoyeur de la notif le passera à `true` lorsqu'il enverra la notif
    - le récepteur de la notif le resettera à `false` lorqu'il aura reçu la notif
- `the_mutex` = un [mutex](https://www.boost.org/doc/libs/1_80_0/doc/html/boost/interprocess/named_mutex.html) empêchant les race-condition (notamment entre le check du bool et le `wait` de la CV)

```cpp
using namespace boost::interprocess;
named_mutex the_mutex(open_or_create, "mutex_name");
named_condition the_cv(open_or_create, "cv_name");
// is_notified est un bool partagé en shared-memory, initialisé à false
```

L'utilisation canonique est alors, **pour le process qui attend la notification** :

```cpp
{
    scoped_lock<named_mutex> lock(the_mutex);
    while (!is_notified) {
        the_cv.wait(lock);
    }
    is_notified = false;  // pas indispensable = "reset" pour être prêt à recevoir d'autres notifications
}
// lorsqu'on atteint ce stade, on a été notifié par l'envoyeur
```

Et **pour le process qui envoie la notification** :

```cpp
{
    scoped_lock<named_mutex> lock(the_mutex);
    is_notified = true;
    the_cv.notify_one();
}
// lorsqu'on atteint ce stade, on a notifié le récepteur
```

En fait, le bon modèle mental à avoir, c'est plutôt de considérer que c'est `is_notified` qui indique si oui ou non on est notifié (et non pas le réveil ou encore l'appel à `notify_one`, vu qu'il peut y avoir des spurious wakeup... voire pas de wakeup du tout si `notify_one` a été appelé avant `wait`).

Et ce `is_notified` est protégé par un mutex, ce qui garantit qu'on ne loupe pas l'info :

- soit le récepteur est en train de checker le flag (dans ce cas, le mutex force le notifieur à attendre avant de muter le flag)
- soit le notifieur est en train de muter le flag (dans ce cas, le mutex force le récepteur à attendre avant de constater que le flag a été muté)

Côté récepteur, cf. [la doc de condition_variable.wait](https://en.cppreference.com/w/cpp/thread/condition_variable/wait) :

- on commence par locker le mutex, pour deux raisons :
    - pour éviter les data-race en accédant à `is_notified` (l'intérêt n'apparaît pas de façon flagrante quand ce flag est un simple bool, mais le flag pourrait être plus complexe)
    - pour empêcher la modification de `is_notified` entre sa vérification par le `while`, et l'appel à `wait`
    - en effet, sinon, on pourrait avoir l'enchaînement suivant :
        - le récepteur checke `is_notified` dans le `while` ; comme il est à `false`, il rentre dans la boucle
        - entretemps, le notifieur passe `is_notified` à `true`, puis appelle `notify_one`
        - le récepteur appelle `wait`, et commence à attendre un `notify_one`... qui n'arrivera jamais, puisqu'il a **déjà** été effectué entre le check du flag et le `wait`
    - (et si jamais le mutex est déjà locké quand on essaye de le locker nous-même, c'est que le notifieur est en train de notifier — ou bien qu'un autre récepteur est en train de checker ; l'unlock va arriver vite)
- dans le cas le plus simple, `is_notified` est à `false` au moment du check, et on appelle `wait` ; à ce stade, on est toujours sous la protection du mutex locké
- `wait` unlock le mutex et endort le process courant (en une seule opération atomique) → c'est un keypoint : tant que le process `wait`, il ne garde pas le mutex locké
- plus tard, quand `wait` reçoit la notification (ou un spurious wakeup), il se réveille et locke le mutex (toujours en une seule opération atomique)
- on repasse alors dans le while :
    - si le réveil est spurious, `is_notified` est à `false`, et on recommence : le `wait` nous rendort et unlock le mutex
    - si le réveil est réel, `is_notified` est obligatoirement à `true`, on sort de la boucle ; la destruction du `scoped_lock` unlocke le mutex fraîchiement réacquis, et on continue notre traitement : **on a été notifié**


Côté envoyeur, c'est plus simple :

- on locke le mutex (si jamais le mutex est déjà locké, c'est que le récepteur est en train de checker ; ça devrait être rapide)
- sous la protection du mutex, on switch le flag `is_notified` à `true` (ce qui est le "vrai" signal indiquant qu'on notifie, mais comme à ce stade, on n'a pas encore unlocké le mutex, personne ne peut le savoir)
- toujours sous la protection du mutex, on `notify_one` : ça débloque le `wait`, mais comme celui-ci essaye immédiatement (et même atomiquement) de locker le mutex, il va mécaniquement attendre que le notifieur ait unlocké le mutex
- on a bien effectué notre couple {switcher le flag + notifier} "en même temps" (i.e. dans la même section critique) → on délocke le mutex (ce qui permet au récepteur de repasser dans son while, de checker le flag, et de constater qu'il est notifié)
- note : quel est l'intérêt du lock côté notifieur ?
    - principal intérêt = s'assurer qu'on ne peut muter `is_notified` que quand le récepteur est dans de bonnes conditions pour le savoir (notamment, il ne faut surtout pas muter `is_notified` quand le récepteur est entre le check de `is_notified` et le `wait`)
    - de plus, éviter les data-race quand on mute `is_notified` (même si c'est pas flagrant dans le cas simple où le flag est un simple bool...)
    - enfin, s'assurer que switcher le flag et appeler `notify_one` ont lieu en même temps, cf. détails ci-dessous

Le point important, en lien avec le modèle-mental suggéré plus haut, c'est donc :

> on n'appelle pas `notifiy_one` sans avoir passé `is_notified` à `true`

_Question_ : quid des spurious-wakeups ?

- pas grave : si `wait` est réveillé, même à tort, il a réacquis le mutex
- le notifieur ne peut donc pas notifier (car le couple "je mute `is_notified` + j'appelle `notify_one`" est protégé par le mutex)
- le récepteur va donc se contenter de repasser dans la boucle `while`, et se rendormir immédiatement (en unlockant le mutex)

_Question_ : faut-il de la synchro additionnelle entre les process pour s'assurer de n'appeler `notify_one` qu'APRÈS qu'on ait appelé `wait` préalablement ?

- non ! Même si le notifieur fait tout son cycle avant même que le récepteur débute le sien, le système fonctionne toujours parfaitement
- en effet, côté notifieur, on va :
    - locker le mutex
    - passer `is_notified` à `true`
    - appeler `notify_one` (ce qui sera sans effet, car aucun `wait` n'aura encore été appelé)
    - unlocker le mutex
- et côté récepteur, on va :
    - locker le mutex
    - constater que `is_notified` est _déjà_ à `true`
    - considérer qu'on a été notifiés... sans même avoir à utiliser la CV pour appeler `wait` (d'où le fait que la "vraie" notification, c'est plutôt le flag)
- du coup, la seule chose qui se passe dans ce cas, c'est que le notifieur appelle `notify_one` pour rien (vu que personne ne `wait`) et que le récepteur n'a pas l'occasion d'appeler `wait`

_Question_ : pourquoi l'appel à `notify_one` est-il protégé par le mutex ?

- le TL;DR, c'est qu'on pourrait se permettre de sortir `notify_one` de la section critique sans provoquer de race-condition, mais qu'il vaut mieux éviter car ça peut poser problèmes dans certains as.
- plus en détail, [cette réponse stack-overflow](https://stackoverflow.com/questions/17101922/do-i-have-to-acquire-lock-before-calling-condition-variable-notify-one/17102100#17102100) indique pourquoi on peut notifier en dehors de la section critique :
    - en préambule, si l'appel à `notify_one` (en dehors de la section critique, donc) a lieu avant le réveil du `wait`, tout se passe comme s'il avait été fait dans la section critique, donc ça ne change rien
    - si `wait` a été réveillé (par un spurious-wakeup, donc) AVANT appel à `notify_one` (mais après unlock du mutex par le notifieur), alors le récepteur va passer dans le while, constater que le flag `is_notified` a été switché
    - le récepteur va donc shunter le prochain wait, et considérer qu'il a été notifié (alors même qu'à ce stade, `notify_one` n'a pas encore été appelé)
    - côté notifieur, quand finalement on appelle `notify_one`, ça ne sert plus à rien (pas bien grave), car le récepteur se considère déjà notifié + il n'y a plus personne qui `wait`
- et [cette autre réponse à la même question](https://stackoverflow.com/questions/17101922/do-i-have-to-acquire-lock-before-calling-condition-variable-notify-one/52950485#52950485) donne un excellent exemple des problèmes qui peuvent arriver si on sort `notify_one` de la section critique :
    - si `notify_one` n'est pas dans la section-critique, il peut être appelé alors que le thread récepteur se considère déjà notifié
    - or, peut-être que le thread récepteur, une fois notifié, va détruire la condition-variable `the_cv`
    - du coup, au moment où finalement on appelle `the_cv.notifiy_one`, l'objet `the_cv` a été détruit !
    - (et ce problème ne peut pas arriver si on garde `the_cv.notify_one()` dans la section critique, car le récepteur ne peut vérifier s'il a été notifié qu'en lockant le mutex pour accéder à `is_notified`)
- comme ça ne mange (presque) pas de pain de notifier dans la section critique, autant garder la rule-of-thumb de le faire...
    - ("presque", car la première réponse dit que ça peut être un peu moins efficace, et pointe vers [ce lien](https://stackoverflow.com/questions/16889063/pthread-mutex-pthread-mutex-unlock-consumes-lots-of-time) que je n'ai pas regardé)

_Question_ : du coup, comme le `wait` et le `notify_one` peuvent être appelés dans n'importe quel ordre, pas besoin de synchroniser les process pour qu'ils communiquent via une CV ?

- j'ai l'impression que si : on doit obligatoirement partager le flag `is_notified` entre les process en shared-memory avant toute utilisation d'une CV (vu que c'est ce flag la "vraie" notification)
- or, pour partager des trucs en shared-memory, on dirait qu'il faut de la synchro préalable entre les process, e.g. pour construire un payload dans une shared-memory, et ne débloquer un process récepteur que quand le payload est construit
- (dans mes POCs, j'ai utilisé une message queue posix pour cette synchro, mais on peut aussi ne démarrer le notifieur et le récepteur qu'une fois le payload construit, ou bien se reposer sur des fichiers sur disque)

# Infos complémentaires

En python, le lock peut être caché dans l'implémentation (un lock par défaut est créé à la construction si on ne lui en a pas passé explicitement).

On pourrait se dire qu'en python, `threading.Event` sert justement à réveiller les threads qui dorment, donc quel intérêt d'utiliser une CV ? En fait, les `threading.Event` python sont justement implémentés avec des CV : [lien1](https://github.com/python/cpython/blob/c7a79bb036b42f96b7379b95efa643ee27df2168/Lib/threading.py#L554), [lien2](https://github.com/python/cpython/blob/c7a79bb036b42f96b7379b95efa643ee27df2168/Lib/threading.py#L224).

D'ailleurs, c'est pas hyper-clair si les `threading.Event` peuvent avoir des spurious wakeups : d'un côté la doc n'en parle pas, pas même dans son exemple de base ; de l'autre, d'après le code, on se contente d'un `wait` sur une CV, donc je pense qu'on devrait avoir un risque de spurious-wakeup aussi... [Cette question stackoverflow](https://stackoverflow.com/questions/57399474/does-the-threading-event-object-suffer-from-spurious-wakeup) pose la même question... mais n'a pas de réponse :-/

Quelques liens :

- [un exemple de cours utilisant les CV, en java](https://web.stanford.edu/~ouster/cgi-bin/cs140-spring14/lecture.php?topic=locks)
- [le man pthread_cond_signal](https://linux.die.net/man/3/pthread_cond_signal) ; sous Linux, les CV utilisent pthread
- [la core-guidelines C++ sur l'utilisation des CV](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rconc-wait), ainsi qu'[un blogpost la détaillant](https://www.modernescpp.com/index.php/c-core-guidelines-be-aware-of-the-traps-of-condition-variables)
- [le couple CV+lock semble parfois être appelé Monitor](https://en.wikipedia.org/wiki/Monitor_(synchronization))
