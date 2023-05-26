# Simple Made Easy

- **url** = https://www.youtube.com/watch?v=SxdOUGdseq4
- **type** = vidéo
- **auteur** = [Rich HICKEY](https://github.com/richhickey), créateur de clojure, no less
- **date de publication** = 2011-09-19
- **source** = [chaîne youtube de TheStrangeLoop](https://www.youtube.com/@StrangeLoopConf) = conférences tech : https://thestrangeloop.com/
- **tags** = language>clojure ; topic>best-practice ; level>intermediate


TL;DR = talk parlant des notions de **simple** vs. **easy**, du fait que le premier est préférable au second, et comment atteindre un état où le code et le soft est simple.

Définitions :

- **simple** = un unique fold, un unique braid, un unique quelque chose
- **easy** = proche, accessible, facile

4:45 simple = un unique rôle/tâche, mais possiblement plusieurs fonctions/instances :

> [simplicity is] about lack of interleaving, not cardinality

^ ce qu'on veut, c'est éviter de mélanger les trucs (et non pas garder un compteur quelconque à 1)

6:20 easy = near = facile à attraper, facile à comprendre, familier

C'est pas parce que quelque chose n'est pas familier qu'il n'est pas simple : galérer au début avec quelque chose ne veut pas automatiquement dire que ce quelque chose est complexe.

La notion de easy est donc corrélée à nos capabilities. Elle est donc RELATIVE (à une personne donnée) ; a contrario, la notion de simplicité est objective, absolue.

12:00 ce qui l'intéresse, ce sont les mêmes propriétés qui sont pertinentes pour l'architecture logicielle = maintenabilité.

(Le message du slide est de dire qu'à cause de nos egos, la façon dont on construit le code ne nous intéresse à tort que comme propriété intrinsèque (e.g. "écrire du beau code") plutôt que pour ses bénéfices = maintenabilité)

14:00 la complexité empêche la compréhension (car on doit garder en tête plus de choses en même temps, vues qu'elles sont liées), ce qui empêche la fiabilité.

15:20 pour pouvoir changer un software (= le maintenir), il fait pouvoir l'analyser et le comprendre, pour prévoir l'impact de ces changements.

17:00 les tests (ou le type-checker) ne suffisent pas, selon lui : il faut d'abord être capable de raisonner sur son programme pour pouvoir le debugger.

19:00 son message = sur des projets non-triviaux, si on ne garde pas la complexité sous contrôle, ça finira par nous tuer, i.e. par tuer la vitesse de développement.

NDM : je comprends maintenant mon goût pour les outils de visualisation : c'est pour permettre facilement (au sens easy) de raisonner au sujet de son programme.

22:20 bénéfices de la simplicité :

- ease understanding
- ease of change
- easier debugging
- flexibility

27:30 intéressant exemple des parenthèses en LISP : le fait que les parenthèses entourent tout n'est pas easy (car on n'y est pas habitué) ni simple (car les parenthèses sont overloadées : elles servent à d'autres trucs que de wrapper des méthodes, par exemple à définir des vector, il y a donc interleaving ce qui est la définition de la complexité). Si on utilisait autre chose que des parenthèses pour définir des vector (p.ex. une structure), alors on atteint l'état souhaité où l'usage des parenthèses est simple (car on a supprimé l'interleaving). De façon intéressante, l'usage des parenthèses, bien que simple, reste hard (on n'y est toujours pas habitué), mais ça n'est pas le plus important.

30:10 _meaning of simple is unentangled_ = simple ne veut PAS dire "je sais déjà ce que ça veut dire"

33:00 il oppose ce qu'il appelle complecter des trucs (i.e. les mélanger de façon complexe) à la composition = juxtaposer de façon simple des trucs simples en eux-mêmes.

35:00 laïus intéressant sur le fait qu'on peut avoir du code proprement découpé, et pourtant couplé et complexe.

36:00 _stateful programming is easy, but complex_ (notamment, les méthodes, où deux appels successifs d'une même fonction n'ont pas le même résultat)

Pas mal de références dans ce qui suit à des points de détail sur des langages fonctionnels du type lisp / Haskell / Clojure. (le contexte du talk semble être les langages fonctionnels, et sachant que c'est l'auteur de Clojure, langage fonctionnel, qui parle...)

42:50 **the complexity toolkit** = liste des trucs (parfois basiques !) qui ajoutent de la complexité à un programme.

47:00 **the simplicity toolkit** = liste des trucs simples à utiliser.

^ Sa présentation de ces deux slides sont les parties les plus importantes de la vidéo (à revoir, p.ex.)

Abstraction :

> I don't know, I don't want to know

52:00 ne pas complexifier le what (= l'interface) avec le how (= l'implémentation)

57:00 _represent data as data_

57:45 _simplifying is disentangling_

59:10 des trucs intéressants dans sa conclusion :

- il faut développer un entanglement radar
- de façon intéressante, les outils pour la fiabilité (testing, refactoring, type systems) sont secondaires, car ils ne visent pas à réduire la complexité !

> Simplicity often means making more things, not fewer !
