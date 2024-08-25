# The Zen of Erlang

- **url** = https://ferd.ca/the-zen-of-erlang.html
- **type** = blogpost
- **auteur** = [Fred HEBERT](https://leaddev.com/community/fred-hebert) = SRE + system engineer
- **date de publication** = 2016-02-08
- **source** = [son site](https://ferd.ca/)
- **tags** = language>Erlang ; topic>concepts ; level>intermediate


TL;DR = très très bonnes explications sur les concepts de base d'Erlang, permettant d'appliquer le motto **"Let it crash"**. Ça donne envie de bosser avec Erlang !

L'innovation concerne la façon de structurer un programme :

- les différents morceaux d'un programme sont des "process" isolés
- ils communiquent par passage de message
- on les structure en arbre, et certains process (= les supervisors) sont en charge de (re)démarrer les process, selon certaines stratégies/configurations.
- du coup, en cas de Heisenbugs (ceux qui sont difficiles à détecter avant la prod, et les plus fréquents en prod), on laisse le process crasher et son supervisor le redémarrer, en supposant que les conditions ayant conduit à l'Heisenbug ne seront plus là au redémarrage, ce qui est souvent vrai pour les Heisenbugs

# Les briques de base de Let It crash

- Processes légers, isolés
- Message passing, asynchrone, copie de données
- Chaque process a une mailbox

Pour gérer un process qui meurt, deux solutions : monitor et links.

> **Monitor** = You decide to keep an eye on a process, and if it dies for whatever reason, you get a message in your mailbox telling you about it. You can then react to this and make decisions with your newly found information. The other process will never have had an idea you were doing all of this
>
> **Links** are bidirectional, and setting one up binds the destiny of the two processes between which they are established. Whenever a process dies, all the linked processes receive an exit signal. That exit signal will in turn kill the other processes.
>
> I can use monitors to quickly detect failures, and I can use links as an architectural construct letting me tie multiple processes together so they fail as a unit

(...)

> Links are a tool letting developers ensure that in the end, when a thing fails, it fails entirely and leaves behind a clean slate, still without impacting components that are not involved in the exercise.

(...)

> Erlang instead lets you specify that some processes are special and can be flagged with a trap_exit option. They can then take the exit signals sent over links and transform them into messages. This lets them recover faults and possibly boot a new process to do the work of the former one

Les process erlang peuvent se préempter si besoin, selon leur priorité

> This may not seem like a big or common requirement, and people still ship really successful projects only with cooperative scheduling of concurrent tasks, but it certainly is extremely valuable because it protects you against the mistakes of others, and also against your own mistakes. It also opens up the door to mechanisms like automated load-balancing, punishing or rewarding good and bad processes or giving higher priorities to those with a lot of work waiting for them. Those things can end up giving you systems that are fairly adaptive to production loads and unforeseen events

Dernier ingrédient avant de parler de Let it crash = s'intéresser dans le langage à la façon dont une application est déployée (les autres langages s'en fichent) :

> Erlang acknowledges the reality of distribution and gives you an implementation for it, which is documented and transparent. This lets people set up fancy logic for failing over or taking over applications that crash to provide more fault tolerance
>
> So those are all the basic ingredients in the recipe for Erlang Zen. The whole language is built with the purpose of taking crashes and failures, and making them so manageable it becomes possible to use them as a tool

# Supervision trees

> They start with a simple concept, a supervisor, whose only job is to start processes, look at them, and restart them when they fail
>
> The objective of doing that is to create a hierarchy, where all the important stuff that must be very solid accumulate closer to the root of the tree, and all the fickle stuff, the moving parts, accumulate at the leaves of the tree (...) That means that when you structure Erlang programs, everything you feel is fragile and should be allowed to fail has to move deeper into the hierarchy, and what is stable and critical needs to be reliable is higher up

Je reproduis les quelques paragraphes suivants à l'identique car très clairs et synthétiques :

> Supervisors can do that through usage of links and trapping exits. Their job begins with starting their children in order, depth-first, from left to right. Only once a child is fully started does it go back up a level and start the next one. Each child is automatically linked.
>
> Whenever a child dies, one of three strategies is chosen. The first one on the slide is 'one for one', enacted by only replacing the child process that died. This is a strategy to use whenever the children of that supervisor are independent from each other.
>
> The second strategy is 'one for all'. This one is to be used when the children depend on each other. When any of them dies, the supervisor then kills the other children before starting them all back. You would use this when losing a specific child would leave the other processes in an uncertain state. Let's imagine a conversation between three processes that ends with a vote. If one of the process dies during the vote, it is possible that we have not programmed any code to handle that. Replacing that dead process with a new one would now bring a new peer to a table that has no idea what is going on either!
>
> This inconsistent state is possibly dangerous to be in if we haven't really defined what goes on when a process wreaks havoc through a voting procedure. It is probably safer to just kill all processes, and start afresh from a known stable state. By doing so, we're limiting the scope of errors: it is better to crash early and suddenly than to slowly corrupt data on a long-term basis.
>
> The last strategy happens whenever there is a dependency between processes according to their booting order. Its name is 'rest for one' and if a child process dies, only those booted after it are killed. Processes are then restarted as expected

----

"but if my configuration file is corrupted, restarting won't fix anything!"

^ Oui mais...

Les bugs **repétables** sur des features importantes seront de toutes façons détectés avant d'arriver en prod.

Les bugs repétables sur des features mineures peuvent arriver en prod, mais c'est pas bien grave (voire parfois souhaitable : éradiquer TOUS les bugs même les plus mineurs coûte trop cher — NDM : c'est une posture que je trouve saine, mais qui ne m'est pas naturelle...)

Les bugs qui posent problème sont donc les bugs non repétables en prod, et ce sont ceux qui arrivent le plus souvent.

^ c'est pour les bugs non repétables (Heisenbugs) que redémarrer un process qui a crashé est efficace.

> Rolling back to a known stable state and trying again is unlikely to hit the same weird context that causes them. And just like that, what could have been a catastrophe has become little more than a hiccup for the system, something users quickly learn to live with.
>
> You can then make use of logging, tracing, or a variety of introspection tools (which all come out of the box in Erlang) to later find, understand, and fix the issues so they stop happening. Or you could just decide to tolerate them were the effort required to fix the issues too large.

Derrière, il donne un intéressant exemple (à revenir revoir si besoin car il illustre les concepts présentés précédemment) de programme erlang pour montrer les dépendances entre les différents parties d'un programme :

> The supervision strategies I mentioned earlier let us encode these requirements in the program structure, and they are still respected at run time, not just at boot time.

^ la structure du programme reflète la façon dont il sera opéré en prod !

Au passage, l'architecture erlang peut être utilisé avec des process erlang représentant des programmes dans d'autres langages :

> The OCR process itself here could be just monitoring code written in C, as a standalone agent, and be linked to it. This would further isolate the faults of that C code from the VM, for better isolation or parallelisation.

Chaque process/supervisor peut avoir ses propres seuils de tolérance, configurables.

NDM : toute cette description d'erlang a plus l'air liée à la façon de faire tourner un runtime en environnement plutôt qu'à un langage de programmation.

Et on en arrive à **Let it crash** :

> There's enormous value in structuring the system this way because error handling is baked into its structure. This means I can stop writing outrageously defensive code in the edge nodes — if something goes wrong, let someone else (or the program's structure) dictate how to react. If I know how to handle an error, fine, I can do that for that specific error. Otherwise, just let it crash!

OTP (le framework généraliste d'erlang) permet même de découper la structure de son programme en des composants réutilisables.

# Conclusion intermédiaire

> With all of this done, our Erlang system now has all of the following properties defined:
>
> - what is critical or not to the survival of the system
> - what is allowed to fail or not, and at which frequency it can do so before it is no longer sustainable
> - how software should boot according to which guarantees, and in what order
> - how software should fail, meaning it defines the legal states of partial failures you find yourself in, and how to roll back to a known stable state when this happens
> - how software is upgraded (because it can be upgraded live, based on the supervision structure)
> - how components interdepend on each other
>
> This is all extremely valuable. What's more valuable is forcing every developer to think in such terms from early on. You have less defensive code, and when bad things happen, the system keeps running. All you have to do is go look at the logs or introspect the live system state and take your time to fix things, if you feel it's worth the time.

NDM : en side-note, l'un des messages implicites du talk est qu'il est vain (et même contre productif !) de vouloir corriger tous les bugs :

> They're all things we know of, but are impact-free. Two years later and we haven't bothered to fix it because the system works fine despite that.

Les autres trucs qu'on gagne gratos en utilisant Erlang :

- pattern matching
- functional programming
- optional type checking
- property-based testing tools
- hot code loading
- an entire community using the same principles

# Synthèse

> In a nutshell, the Zen of Erlang and 'let it crash' is really all about figuring out how components interact with each other, figuring out what is critical and what is not, what state can be saved, kept, recomputed, or lost. In all cases, you have to come up with a worst-case scenario and how to survive it. By using fail-fast mechanisms with isolation, links & monitors, and supervisors to give boundaries to all of these worst-case scenarios' scale and propagation, you make it a really well-understood regular failure case

(...)

> That’s the Zen of Erlang: building interactions first, making sure the worst that can happen is still okay. Then there will be few faults or failures in your system to make you nervous (and when it happens, you can introspect everything at run time!) You can sit back and relax.
