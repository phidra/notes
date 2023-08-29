# PyConFR Talence

- **url** = https://www.pycon.fr/2023/
- **type** = conférence
- **auteur** = N/A
- **date de publication** = 2023-02-18
- **source** = N/A
- **tags** = language>python ; topic>multiple ; level>intermediate


**TL;DR** = notes brutes sur les conférences auxquelles j'ai pu assister à la PyConFR 2023 à Talence.

* [PyConFR Talence](#pyconfr-talence)
   * [Driving down the Memray lane - Profiling your data science work](#driving-down-the-memray-lane---profiling-your-data-science-work)
   * [Je suis nul·le !](#je-suis-nulle)
   * [Portage Python sur Webassembly](#portage-python-sur-webassembly)
   * [Continuous performance analysis for Python](#continuous-performance-analysis-for-python)
   * [Fear the mutants. Love the mutants.](#fear-the-mutants-love-the-mutants)
   * [Python moderne et fonctionnel pour des logiciels robustes](#python-moderne-et-fonctionnel-pour-des-logiciels-robustes)
   * [Domain-driven design: what can Python do for you?](#domain-driven-design-what-can-python-do-for-you)
   * [On a testé pour vous... méthodologie, outils](#on-a-testé-pour-vous-méthodologie-outils)
   * [Traitement de données géographiques avec Rasterio, NumPy, Fiona et Shapely](#traitement-de-données-géographiques-avec-rasterio-numpy-fiona-et-shapely)
   * [Cerveau, Biomarqueurs et Deep Learning](#cerveau-biomarqueurs-et-deep-learning)
   * [Apport du langage Python dans un service de recherche hospitalière pour mener des analyses de deep learning](#apport-du-langage-python-dans-un-service-de-recherche-hospitalière-pour-mener-des-analyses-de-deep-learning)

Par ailleurs, les confs qui ont retenu mon attention, mais auxquelles je n'ai pas pu assister (donc à voir en replay), dans le désordre :

- [Django Admin comme framework pour développer des outils internes](https://www.pycon.fr/2023/fr/talks/30m.html#django-admin-comme-framework-p) ([replay](https://indymotion.fr/w/3HP9UdwPybvC8CLaJXLE2i))
- [Python web performance 101: uncovering the root causes](https://www.pycon.fr/2023/fr/talks/30m.html#python-web-performance-101-unc) ([replay](https://indymotion.fr/w/bJytAHj2sab5ENhj771Wq7))
- [Giving and receiving great feedback through PRs](https://www.pycon.fr/2023/fr/talks/30m.html#giving-and-receiving-great-fee) ([replay](https://indymotion.fr/w/udj9K5VQrKasNaH6GC3sCW))
- [Monorepo Python avec environnements de développement reproductibles et CI scalable](https://www.pycon.fr/2023/fr/talks/30m.html#monorepo-python-avec-environne) ([replay](https://indymotion.fr/w/9VWSjLge45KqiRgitZ7obC))
- [Déployer du backend en 2023](https://www.pycon.fr/2023/fr/talks/1h.html#deployer-du-backend-en-2023) ([replay](https://indymotion.fr/w/orJRgHsst8MwNEfpdsF6cA))
- [Uncovering Python’s surprises: a deep dive into gotchas](https://www.pycon.fr/2023/fr/talks/30m.html#uncovering-pythons-surprises-a) ([replay](https://indymotion.fr/w/dy8du4pKjXE6umxcmAdLmT))
- [Développement VRAIMENT cross-platform avec Python](https://www.pycon.fr/2023/fr/talks/1h.html#developpement-vraiment-cross-p) ([slides](https://v.gd/ZA3XLv), [replay](https://indymotion.fr/w/811dRxXUMHeRk3TzFMUfuL))
- [Writing Great Test Documentation](https://www.pycon.fr/2023/fr/talks/30m.html#writing-great-test-documentati) ([replay](https://indymotion.fr/w/wpaDkJzENJ1vdQmn4pzTgQ))
- [ZFS: un stockage fiable, puissant et accessible.](https://www.pycon.fr/2023/fr/talks/1h.html#zfs-un-stockage-fiable-puissan) ([replay](https://indymotion.fr/w/j3C47dzkfT9bBhY11CzHzG))
- [« Fixed bugs » n’est peut-être pas le meilleur message de commit](https://www.pycon.fr/2023/fr/talks/1h.html#fixed-bugs-nest-peut-etre-pas) ([replay](https://indymotion.fr/w/iFJthhnVyNfjNZ2cLfAQSK))
- [Psycopg, troisième du nom](https://www.pycon.fr/2023/fr/talks/30m.html#psycopg-troisieme-du-nom) ([replay](https://indymotion.fr/w/eA4SFNYbfF4du4J3t5ocjc))
- [Geographic visualization using Streamlit](https://www.pycon.fr/2023/fr/talks/30m.html#geographic-visualization-using) ([replay](https://indymotion.fr/w/fNPWF1iiuZyzd1K6rNTR5z))
- [Monitorez vos applications Python (et pas uniquement votre infra)](https://www.pycon.fr/2023/fr/talks/30m.html#monitorez-vos-applications-pyt) ([replay](https://indymotion.fr/w/jWPJYRnUh8BttPbmgdnqfV))
- [Cloud infrastructure from Python code: how far could we go?](https://www.pycon.fr/2023/fr/talks/30m.html#cloud-infrastructure-from-pyth) ([replay](https://indymotion.fr/w/sVTpHHactff28Zyc1PwDuW))
- [GEMSEO : une bibliothèque pour l’optimisation multi-disciplinaire](https://www.pycon.fr/2023/fr/talks/30m.html#gemseo-une-bibliotheque-pour-l) ([replay](https://indymotion.fr/w/by8dBRA1i4EjAdhf9EP1Hj))


## Driving down the Memray lane - Profiling your data science work

- speaker = [Cheuk Ting-Ho](https://cheuk.dev), bosse à Anaconda
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#driving-down-the-memray-lane-p), prez en anglais, durée = 30 min ; [slides](slides.com/cheukting_ho/memray-lane) + [replay](https://indymotion.fr/w/oyNNJHcMLhpyVqiexYKuo9)

Gentle introduction au memory-profiling.

- presque 15 min de retard (sur une prez de 30 min) à cause de problèmes techniques, du coup, la prez est écourtée...
- what is memory profiling
    - composante dynamique obligatoire (pour voir en conditions réelles, ce que ne montre pas la static-analysis)
    - nécessaire car les python apps ne sont pas très bonnes à gérer la mémoire
    - (de plus, les apps python sont utilisées pour setup un truc rapidement, donc c'est rarement très optimisé)
- heap vs stack
- resident vs virtual
    - resident = actual memory that is used physically on RAM (not an accurate measure for total used — NdM : car une partie peut être swappée)
    - virtual = total memory that the process needs (not an accurate measure on how much consumed at a time — NdM : car on n'utilise pas toute la virtual memory, une partie peut ne pas être backée par de la RAM physique)
- how python manages memory
    - ce slide est intéressant pour savoir comment python gère la mémoire
- pymalloc
    - idem slide précédent = intéressant + donne des pistes à creuser
    - seuil au delà duquel on fallback sur `PyMem_RawMalloc` pour les gros objets
    - les petits objets sont gérés par pymalloc
- demo time = dummy case
    - memray = memory profiler
    - sur la démo, elle montre des flamegraphs de la mémoire occupée par différents appels de fonctions
    - (la démo est faite avec jupyter, ce qui fait sens !)
- demo time = data science = ice_cream products = vraies data
- question intéressante = overhead du profiler ?


## Je suis nul·le !

- speaker = [Guillaume AYOUB](https://www.yabz.fr/)
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#je-suis-nul-le), durée = 30 min ; [replay](https://indymotion.fr/w/kKFA25V4bNSUirH6SmKkY7)

Prez rafraîchissante qui aborde l'aspect humain du métier et défend un rapport positif à son travail et aux autres devs.

- j'avais initialement l'intention de voir la prez sur les gotchas, mais les soucis techniques de l'autre amphi m'ont conduit à plutôt assister à celle-ci.
- bien menée, le mec sait mener des présentations, et le public est conquis
- voir les choses autrement : reformuler "je suis nul"

## Portage Python sur Webassembly

- speakers = [Paul PENY](https://github.com/pmp-p), spécialiste de la conversion en wasm + [François GUTHERZ](https://fr.linkedin.com/in/astrofra?trk=pulse-article_main-author-card), développe un moteur 3D = Harfang
- [description de la prez](https://www.pycon.fr/2023/fr/talks/1h.html#portage-python-sur-webassembly), durée = 1h ; [replay](https://indymotion.fr/w/rEV1qrdYoTa4FGqQTTAGAt)

Retour d'expérience sur le portage d'un moteur 3D en python/C++ vers wasm, pour que les apps qui l'utilisent puissent être utilisées dans le navigateur.

- de lourdes difficultés techniques (micro, mauvais ordi, démo qui foire, ...) rendent la prez très très laborieuse (malgré les promesses du sujet)
- historique et contexte :
    - anatomie des applications non-web (car on veut porter des applications non-web vers le web)
    - utilisateurs variés, machines variées
    - portage natif trop difficile (e.g. itch.io prévient que cerrtains jeux considèrent des jeux comme des virus)
    - déjà des langages intermédiaires orientés machine dès 1979 (e.g. le jeu Zork)
    - soution = compromis entre encombrement/performance/sécurité (on ne pourra pas avoir les trois)
    - pourtant, il y a eu des VM intermédiaires pour les langages de haut-niveau (perl, python, ruby, java, llvm-ir, parrotvm ...)
    - mention spéciale à llvm-ir, mais pas prévu pour être lu par les humains
    - flash = tentative dans le même genre (qui plante aussi)
    - 2009 = nodejs (une VM), pas mal, très performant, mais ne comprend que le js
    - asm.js = compilateur C/C++ vers javascript + machine virtuelle écrite en javascript
    - ça a validé le concept et on pousse avec WebAssembly = on intègre cette machine virtuelle, et on l'intègre dans TOUS les navigateurs
- pourquoi cette adoption ?
    - sécurité + simplicité
    - bonnes performances
    - pas de licence
    - standard simple + batterie de tests
    - les GAFAM en ont besoin !
- les PME/ETI en ont aussi besoin → l'autre présentateur présente un cas réel
    - application pour industriels
    - outils de formation
    - certains cas d'usages ont besoin de la 3D (e.g. surveiller des robots à distance : on modélise le robot avec de la 3D temps-réel)
    - François a la compétence pour faire de la 3D
    - Paul prend du C++ et du python et l'amène dans un navigateur
- Démo 1 = réalité virtuelle en python apour un simpulateur de vol
- Démo 2 = monitoring d'un robot réel via un modèle 3D ("jumeau numérique")
- démo 3 = modèle 3D pour analyser pourquoi les gens se font percuter par les trains en traversant les voix
- objectif = échapper à l'enfer du portage
- un module py3 est portable avec (un tool?) en webassembly
- démo pour faire tourner un jumeau numérique 3D (en C++) en python
    - comment ? quelles adaptations ?
    - mettre un peu d'asynchrone
    - modifier les shaders
- Paul a dévloppé un paquet, pygbag, pour convertir un paquet python en WebAsm
- (NdM : leurs démos marchent pas, pour des raisons techniques ; leur portable prévu a cassé, ils ont fait la prez sur un ordi prêté, qui n'a pas les dépendances pour faire tourner le truc, à savoir Chrome)
- note : l'objectif est de prendre du python qui fait de la 3D, et de mettre ça en ligne
- ce qui ne marche pas quand on porte en Wasm : threads / boucles infinies / tout ce qui est input() (qui va générer une boîte de dialogue dans le navigateur)
- il faut utiliser le shell asyncio pour tester son dev, plutôt que le REPL classique (car le web est un système asynchrone)
- si on mets plus de temps qu'une frame (1/60s), le navigateur va finir par killer le script
- il faut prendre le code, et le mettre dans une fonction asynchrone, et appeler ce "main asynchrone"
- certains modules binaires de la stdlib n'ont pas encore été portés vers WasM
- NDM : si ce genre d'initiative n'est pas tractée, le portage n'est que local dans le temps, et finira par être dépassé : le vrai challenge, c'est qu'il faut qu'il soit maintenu, donc qu'il ait de la traction
- ah, on finit par avoir la démo dans le navigateur :+1:
- il y a un mode terminal à l'intérieur du web (qui est plus universel que des modules 2D ou 3D) ; ce terminal peut afficher des liens, des images, et des caractères semi-graphiques
- Paul a porté pygame sur webassembly (en 3 mois)
- son conseil = ne pas utiliser pygame pour faire de la 3D (on va avoir des soucis de compatibilité avec les différentes plateformes), mieux vaut utiliser harfang ou un autre moteur 3D qui sont habitués à gérer ces problèmes de multiples plateformes
- il donne d'autres conseils, e.g. préférer ogg à mp3 (qui n'est pas lu partout)
- import de moteur physique en 2D importé par ctypes

## Continuous performance analysis for Python

- speaker = [Arthur PASTEL](https://github.com/art049), il a créé codspeed
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#continuous-performance-analysi), durée = 30 min ; [replay](https://indymotion.fr/w/9iwwEkFv6XqZjhHvLZQ5ag)

Ma prez préférée de l'event, elle introduit [codspeed](https://codspeed.io/) = l'outil qu'il développe qui utilise valgrind pour disposer de tests de perfs reproductibles et les intégrer aux pipelines de CI.

- monitoring des perfs en continu
- performances = vitesse d'exécution ? throughput ? (= débit) ; ici, on se concentre sur speed
- APM = application performance monitoring platform (datadog, sentry, ...)
- Elles sont cools, mais ne permettent que de mesurer la prod (donc arrivent tard = on utilise les users comme des cobayes)
- autre conséquence de déployer un truc avec de mauvaises perfs = coûts + friction entre devs (responsables de la mauvaise perf) et ops (qui en subissent les effets)
- l'objectif du talk est d'avoir le performance feedback plus tôt, au moment du testing, plutôt qu'après le deployment
- idéalement, on pourrait voir une trend de la performance
- problèmes = répétabilité, hardware-agnostic, bloquant, etc.
- exmeple avec fibo
- approche basique = `time.time` (horloge système) / `time.perf_counter` (timer de haute-résolution)
    - le deuxième est à privilégier quand on veut mesurer des temps d'exécution
- problème = dépendant de l'activité sur le système
    - "solution" = faire une moyenne statistique
- problème = dépendant d'autres facteurs influençant : warmup, outliers, GC, ...
- des frameworks aident à faire ça sans qu'on ait à le coder à la main
    - pytest-benchmark / airspeed velocity / pyperformance
    - les résultats à utiliser ces outils sont plus stables
    - mais tout de même trop instables pour être utilisés dans une CI
- du coup, codspeed = autre approche, pour essayer de mesurer de façon déterministe la perf d'un système ; perf_events
- problème du cache L3 = dans un environnement virtualisé, l'exécution n'est pas déterministe à cause du cache L3 qui est partagé avec le monde extérieur
    - (e.g. un programme tierce va massivement utilisé le cache L3 pour un run, mais pas pour un autre)
    - (NdM = noisy neighbour)
- idée = valgrind = CPU "simulé" → on va simuler des caches dans un environnement sandboxé
- usecase démontré = pydantic (qui utilise codspeed)
    - (note : V2 rewritten in rust)
    - permet de "faire des tests" avec le code, et de voir les impacts sur les perfs
    - côté UX, on wrappe l'utilisation de pytest par codspeed
    - ça montre vraiment le côté shift-left = on a le résultat du test de perf au même titre qu'un résultat de pipeline de pre-merge
- il montre son dashboard
- QUESTION = codspeed = équivalent de valgrind ? ou utilise valgrind sous le capot ?
    - réponse = il utilise valgrind sous le capot, et permet de l'intégrer plus facilement à un process CI

## Fear the mutants. Love the mutants.

- speaker = [Max KAHAN](https://uk.linkedin.com/in/maxkahan), python developer advocate chez [vonage](https://www.vonage.fr/developer-center/) = communication's API (SMS API / 2 factor auth / ...)
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#fear-the-mutants-love-the-muta), durée = 30 min ; [replay](https://indymotion.fr/w/j7sNkSW1NfXE3ciL9SBTB9)

Présentation des principes du mutation testing, avec des exemples concrets.

- conférencier chevronné
- dans la salle pas grand monde n'a utilisé le mutation testing
- testing → code coverage → mutation testing
- why unit tests
- problem = project grows
- code coverage = quelle proportion du code est exécutée quand on lance les tests
    - +++ montre ce qu'on ne teste pas (autres avantages dans les slides)
    - --- misleading (c'est pas parce qu'on passe dans une LOC qu'on la teste ; e.g. on peut vérifier qu'on passe bien dans un `requests.get`, sans pour autant tester que la réponse est valide)
- Goodhart's law (= _lorsqu'une mesure devient un objectif, elle cesse d'être une bonne mesure_) s'applique bien au code coverage
- comment comprendre ce qu'un test fait exactement ?
- comment savoir si on peut faire confiance à nos tests ?
    - who watches the watchers ?
- réponse = mutation testing
    - e.g. changer légèrement le code-source d'une fonction
    - suite à cette mutation, on souhaite que le test échoue
- mutation testing frameworks
    - [mutmut](https://mutmut.readthedocs.io/en/latest/) (préféré par le conférencier)
    - [cosmic-ray](https://cosmic-ray.readthedocs.io/en/latest/)
- démo live avec mumut, qui 1. lance les tests normalement 2. lance les mutation testing, et vérifient les mutants : tués (good !) ou survived (bad : il faut améliorer le test)
- viser à 100% n'a pas de sens : certains tests continuent de passer même après mutation, et c'est normal ou pas bien grave :
    - `max_retry=4` au lieu de `max_retry=3`
    - `getLogger("XXpouetXX")` au lieu de `getLogger("pouet")`
- son conseil = commencer petit (en excluant des trucs)
- désavantage = ça prend du temps
- key takeaways = mutation testing tests your tests + smart small

## Python moderne et fonctionnel pour des logiciels robustes

- speaker = [Guillaume Desforges](https://github.com/GuillaumeDesforges) , data/software engineer chez tweag
- [description de la prez](https://www.pycon.fr/2023/fr/talks/1h.html#python-moderne-et-fonctionnel), durée = 1h ; [replay](https://indymotion.fr/w/mWB9g1Z6dUt8iVdm8LEF6F)

Introduction en douceur à la FP et ses concepts (fonctions pures/impures, immutabilité, types) pour le pythonista.

- il recommande fortement [Functional architecture - The pits of success](https://www.youtube.com/watch?v=US8QG9I1XW0) de Mark SEEMAN
- un logiciel robuste = si on introduit une feature, les outils nous poussent à revenir vers des états stables, genre "puits de potentiel"
- première partie du talk = fonctions pures et impures :
    - les "fonctions" classiques sont en fait des routines = des bouts de codes réutilisables (avec side effects)
    - les "vraies" fonctions sont des fonctions mathématiques
    - si une "fonction" utilise un état global caché, le test unitaire peut passer, mais le main échouer (car le main et le test unitaire n'utilisaient pas le même état global)
    - fonctions impures indispensables (mais interdites d'appel par une fonction pure)
    - exemple d'effet de bord qui embête = on checke la config au chargement du module (alors qu'on peut avoir besoin du module sans avoir de config)
    - le chargement du module gagnerait à être pur = ne pas avoir de side-effect
    - le domaine métier s'exprime généralement très bien de façon pure
    - démo concrète intéressante pour montrer comment on peut introduire une dose de pureté dans un programme impur = on fait remonter les side-effects à l'appelant de plus haut-niveau (et les appelées sont des fonctions pures)
        - architecture en oignon
        - le fait de penser en fonctions pures/impures aide à définir les couches (coucher métier > couche applicative > échanges avec la machine)
- deuxième partie du talk = concept d'immutabilité
    - en FP, on adore ça
    - "les variables mutables devraient être un choix conscient, lié à une activité spécifique"
    - démo concrète sur des variables mutables et immutables
- troisième partie = types
    - python = fortement typé, dynamiquement
    - typage statique = restreindre le champ des possibles
    - en vrai, type-hint plutôt que typage statique, car on peut faire ce qu'on veut (y compris n'importe quoi) avec le typage, et python ne râlera pas
    - présentation des ABC vs. Protocols
    - un Protocol avec la méthode `__call__` permet de typer une fonction qui serait autrement compliquée à annoter

## Domain-driven design: what can Python do for you?

- speaker = [David KREMER](https://github.com/drake-mer)
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#domain-driven-design-what-can), durée = 30 min ; [replay](https://indymotion.fr/w/wAA4GdKJF3XorcsnxkAvZ7)

Intro au DDD = Domain-Driven Design, sans entrer dans les détails.

- keys :
    - domain = tout peut se résumer à un domaine
    - driven = on essaye de définir un langage permettant de communiquer dans toute l'entreprise, et c'est ça qui drive, plutôt que des considérations techniques genre la DB
    - design = on laisse ça de côté pour la prez, ça s'intéresse aux techniques pour traduire DDD dans le design du code, mais la prez se concentre plutôt sur les concepts
- cadre conceptuel (= essentiellement, du naming) commun pour explorer le domaine ; langage non-ambigu pour adresser les business-case/domain
- les avantages sont d'autant plus importants qu'on dépasse une certaine taille d'entreprise (car ceci multiplie les risque de miscommunication)
- favorise une archi qui suit le domaine
- favorise la communication globale dans l'entreprise
- communication un peu hésitante, ce qui nuit au message ; c'est d'autant plus dommage que c'est plus une "prez à message" plutôt qu'une "prez à contenu technique"
- il donne un exemple sur un puzzle advent of code, avec l'expression du problème grâce au typage de python
- plus facile de modifier, étendre du code si le modèle a été fait correctement (NdM : oh que je plussoie...)
- il montre que même si les résultats sont identiques, deux façons différentes de coder ne se valent pas !

## On a testé pour vous... méthodologie, outils

Sous-titré = "je doute que tester soit douter"

- speaker = [Nicolas LEDEZ](https://nicolas.ledez.net)
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#on-a-teste-pour-vous-methodolo), durée = 30 min ; [replay](https://indymotion.fr/w/18YPF5QZ7QJcA9T6nSo87E)

Manifeste pour convaincre d'utiliser TDD et expliquer comment le faire.

Notes brutes :

- au pifomètre, 150 lignes de test pour 50 lignes de code
- pourquoi TDD ?
    - jamais de temps pour écrire les test après
    - ça permet de penser à l'utilisation du code en amont
    - tu n'implémentes que les tests dont tu auras besoin (au lieu de suivre son réflexe naturel = tester toute la matrice de possibilités, même si elle est inutile)
    - on est sûr que le test est faux s'il n'y a pas d'implémentation en face
- comment :
    - définir un problème simple à résoudre
    - si on ne peut pas tester, peut-être qu'on comprend mal le problème, ou qu'on ne s'y prend pas correctement ; le problème n'est pe pas assez simple (e.g. on comprend après coup qu'il fallait découper l'input qui fournit les données)
    - (ne pas commencer sur un vrai projet ; son conseil = commencer sur des coding dojo)
- avantage
    - plus facile à maintenir
    - meilleur code coverage
    - refactos du code plus faciles (grâce aux tests) + refactos des tests plus faciles aussi
- mise en oeuvre
    - on écrit le test rouge
    - on écrite le code qui fait passer le test vert, EN MODE KISS
    - on n'oublie pas le refacto (du code ou du test) tout en faisant en sorte que ça reste vert ET que la coverage reste à 100%, AVANT de passer au test suivant
- il donne quelques conseils concrets
    - (entre autres, garder les tests propres est un investissement intéressant vu qu'il auto-documente le code)
    - (entre autres, le test ne doit faillir que d'une seule manière possible)
- définitions :
    - tests unitaires = très rapides = on maîtrise ce qui rentre et ce qui sort, mais on mocke tous les autres composants
    - tests intégration = dialoguer avec des libs / interagir avec les BDD / ...
    - tests fonctionnels
    - tests manuels (il en faut, mais très peu car très cher)
- tips

## Traitement de données géographiques avec Rasterio, NumPy, Fiona et Shapely

- speaker = [Arnaud MORVAN](https://github.com/arnaud-morvan), [camptocamp](https://www.camptocamp.com/fr), ils mettent en œuvre des SIG
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#traitement-de-donnees-geograph), durée = 30 min ; [replay](https://indymotion.fr/w/vhhRGdaSEVJW5kQnKa4Wzz)

Cette prez est une gentle introduction aux outils SIG, elle est pour toi si :

- tu veux un exemple de stack pour travailler avec des données géographiques
- tu veux des exemples concrets d'utilisations de ces outils

Notes brutes :

- présentation de quelques bibliothèques utilisées en géospatial
- données raster (un peu comme des images, mais les "pixels" contiennent des données plutôt que des couleurs) vs. données vecteur (points + lignes + polygones + ...)
- "shapefile préhistorique" ha ha
- démo live avec qgis
- GDAL/OGR très utile, supporte tout un tas de formats de données
    - il y a des bindings en C, mais c'est pas très pratique à utiliser
    - du coup, il y a des bibliothèques au dessus des bindings python de GDAL/OGR
- (la prez est faite partiellement avec jupyter)
- gdalinfo / ogrinfo
- lib rasterio = interface en python sur GDAL, qui fournit les données en numpy
- lib fiona = pour ouvrir des formats géographiques
- lib shapely = manipulation des géométries
- quelques cas concrets
- geoalchemy est à sqlalchemy ce que postgis est à postgres
- mieux vaut utiliser les plugins natifs de qgis que des dépendances externes (mais comme tous ne sont pas intégrés, malgré une volonté d'interaction dans ce sens, on peut s'autoriser à utiliser directement les libs)


## Cerveau, Biomarqueurs et Deep Learning

- speaker = [Facundo CALCAGNO](https://twitter.com/fmcalcagno), techlead chez Servier
- [description de la prez](https://www.pycon.fr/2023/fr/talks/30m.html#cerveau-biomarqueurs-et-deep-l), durée = 30 min ; [replay](https://indymotion.fr/w/v9Epbfk6vsqFxsK7Z1y3Jd)

Prez non-technique qui présente de façon très abstraite quelques pipelines d'analyse de données médicales.

Notes brutes :

- "ça le fait se lever le matin motivé"
- commence par faire la promotion de la fondation internationale de recherche SERVIER (alors que le public est sans doute plutôt là pour la prez technique / le REX)
- data factory
- partenariat avec différents consortiums de données nécessaire, car avoir de la donnée patient n'est pas facile
- maladie ataxia3 neurodégénérative, très rare
- types d'images : IRM vs. radio
- donne une image de l'archi de son pipeline de données
- cerveau et kubernetes = comment transformer les images en quelque chose d'utile aux chercheurs
    - indique les outils/formats de données, mais il n'y a rien de très concret
    - (à la différence de la prez d'avant, p.ex. qui ne se contentait pas de donner des listes d'outils, mais montrait leur utilisation concrète et le résultat des traitements ; là, il dit à l'oral ce qu'il fait, sans rien nous montrer)
    - GKE = Google Kubernetes Engine
- deep-learning sur les images du cerveau, en deux parties :
    - calcul volumétrique de chaque partie du cerveau
    - tractographie du cerveau (= comment les connexions au cerveau du patient sont gérées)
- calcul volumétrique : il montre le pipeline
    - réseau de neurone cerebnet pour discriminer les 92 parties du cerveau
- tractographie
    - deep learning = TractSeg (github MIC-DKFZ)
- visualisation des résultats
    - objectif = enrichier les images d'entrées
    - OHIF = visualisateur open-source d'images médicales
- 5 dernières minutes = démo live d'OHIF, y compris l'enrichissement des images résultant de l'algo de deep-learning : sympa (et impressionnante !)

## Apport du langage Python dans un service de recherche hospitalière pour mener des analyses de deep learning

- speaker = [Clément BENOIST](https://fr.linkedin.com/in/clementbenoist?original_referer=https%3A%2F%2Fwww.google.com%2F), ingénieur en statistiques au CHU de Limoges ; son activité = data science sur la transplantation rénale
- [description de la prez](https://www.pycon.fr/2023/fr/talks/1h.html#apport-du-langage-python-dans), durée = 1 h ; [replay](https://indymotion.fr/w/ddjm63CWj4tSUZ4psA1Qyt)

Présentation dune application de machine-learning en python pour prévoir les rejets de greffes de rein.

Notes brutes :

- 3 logiciels utilisés en recherche hospitalière :
    - R (gratuit et open-source)
    - SAS (payant)
    - MsOffice
- analyse statistique en plusieurs étapes :
    - analyse descriptive
    - nettoyage de données
    - preprocessing
    - analyse statistique
    - traitement des résultats
- contre-intuitivement, ce sont le nettoyage de données + preprocessing le plus compliqué
- comparatif R / python
    - R orienté statistiques, très facile d'utilisation, CRAN = dépôt officiel des paquets R ; package standards pas forcément très complets en ML ou Neural Network
    - python plus généraliste, excellent en ML / NN ; inconvénient = peu de packages spécifiques à la pharmacologie (e.g. pas d'équivalent du package R mrgsolve)
    - le workflow du présentateur : R pour data management / data cleaning / stats classiques, et python pour ML et NN ; idéal = maîtriser les deux langages
- problématique : si rein défaillant alors transplantation / dialyse
    - intéressante présentation (non liée au dev) du problème, et des contraintes
- ABIS = tool (site internet) pour adapter la posologie des immunosuppresseurs (post-greffe) à la concentration dans le sang, par estimateurs bayésiens
    - plutôt que de beaucoup prélever, ils ne font que 3 prélèvements, et en déduisent la courbe de concentration
    - meilleur qualité de vie pour les patients + moins de travail pour les hopitaux
    - les patients suivis par ABIS font moins de rejets que ceux non-suivis par ABIS
    - objectif = connaître efficacité à long-terme (sur d'autres résultats que les rejets)
- objectif = aider le clinicien à identifier précocément les patients à risques (pour essayer d'éviter le rejet)
    - prédire à partir des données antérieures, la qualité de la fonction rénale (objectivée par débit de filtration glomérulaire = DFG : en dessous d'un certain seuil, ça signifie retour en dialyse)
    - objectif = prédire le futur DFG à partir des DFG passés, et d'autres variables
- pourquoi les stats et le ML ? Car pas de modèle clinique d'évolution du DFG
- courbe de DFG = données non-structurées (d'où le besoin de ML adaptées aux données non-structurées)
- les NN ont besoin de grands jeux de données ; plus le NN est profond, plus il est performant, mais plus il a besoin de données
- les NN ont besoin de beaucoup de puissance de calcul (serveurs locaux, serveurs mutualisés, serveur national Jean Zay ; tous en linux ; SLURM = ordonnanceur de travaux sur le serveur ; tensorflow + pytorch)
- le NN donne une précision sur son estimation (et l'objectif est d'avoir la valeur la plus basse)
- résultats = la performance est bonne, la prédiction est bonne ; mais difficile à utiliser en clinique, car comme les conséquences sont graves, on ne l'utilise pas
    - en résumé : performances bonnes, mais pas utilisé -> c'est de la recherche à long terme
- ajout d'une couche (valeurs de Shapley) qui permet D'EXPLIQUER la décision du NN !
    - NdM : c'est intéressant, car ça permet d'éviter l'aspect "boîte noire" des NN/ML
- conclusion :
    - python apport considérable sur ML/NN
    - grosse capacité de calcul
    - deep-learning donnent bons résultats
