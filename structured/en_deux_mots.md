(ces notes correspondent à une sorte de glossaire tech, initialement [sur mon blog](https://phidra.github.io/blog/) ; mais je préfère les gérer comme markdown public)

----

L'objectif de cette page est de "documenter" toutes les notions que je croise, liées de près ou de loin à mon métier.

Je mets des guillemets autour de "documenter", car il s'agit ici d'être concis : juste expliquer la notion en quelques mots, et éventuellement un ou deux pointeurs pour en apprendre plus.

D'autres ressources sur internet ont la même vocation, et sont de bien meilleure qualité que cette page : elle a surtout de l'intérêt pour MOI, vu que c'est les notions que j'ai croisées MOI que je reporte ici, et que je les décris avec MES mots à moi, en y joignant les références que JE trouve pertinentes.

- **Edge Computing** : les calculs sont faits, et les décisions sont prises directement sur le dispositif (données traitées au plus proche de leur source : IoT, sonde curiosity, etc.), par opposition aux cas où le dispositif déporte ses calculs+décision sur un serveur externe, centralisé ([lien](https://blog.octo.com/quest-ce-que-ledge-computing/))
- **Argument-Dependent Lookup** (ADL) : si `f` est une free-floating function, l'appel `f(a, b)` va être résolu par un compilo C++ :
    -  en recherchant une fonction `f` qui accepte un `A` et un `B` dans le namespace courant
    -  en recherchant une fonction `f` qui accepte un `A` et un `B` dans le namespace de `A`
    -  en recherchant une fonction `f` qui accepte un `A` et un `B` dans le namespace de `B`
- **Goal-Directed Approaches (routing)** : _direct the search towards the target t by preferring edges that shorten the distance to t and by excluding edges that cannot possibly belong to a shortest path to t — such decisions are usually made by relying on preprocessed data._ cf. [le survey de 2016 sur le calcul d'itinéraire](https://arxiv.org/abs/1504.05140)
- **Hierarchical Approaches (routing)** : _try to exploit the hierarchical structure of the given network. In a preprocessing step, a hierarchy is extracted, which can be used to accelerate all subsequent queries._ cf. [le survey de 2016 sur le calcul d'itinéraire](https://arxiv.org/abs/1504.05140)
- **Semipredicate problem** : situation où une fonction qui ne retourne qu'un élément (en situation nominale : son résultat) veut également pouvoir renvoyer une erreur, et on se retrouve donc embêté à vouloir renvoyer deux types d'information dans une seule valeur de retour ([lien wikipedia](https://en.wikipedia.org/wiki/Semipredicate_problem))
- **H-tree** : structure fractale recouvrant une surface, présentant la propriété intéressante que tous les noeuds terminaux sont à la même distance de la racine. Utilisé pour [distribuer uniformément le signal d'horloge](https://www.techspot.com/article/1830-how-cpus-are-designed-and-built-part-2/) dans un CPU ([lien wikipedia](https://en.wikipedia.org/wiki/H_tree))
- **consistent hashing** : algorithme de hashing permettant de ne pas jeter tout les hashs à la poubelle lorsqu'on ajoute/enlève une instance (les hashs invalidés par cette redistribution sont limités à une fraction) : [lien octo](https://blog.octo.com/consistent-hashing-ou-l%E2%80%99art-de-distribuer-les-donnees/)
- **single compilation unit / unity build** (SCU) : idiome C/C++ consistant à fusionner plusieurs fichiers source au sein d'une même translation-unit, pouvant parfois accélérer les builds, ou permettre des optimisations du même genre que la LTO. [lien wikipedia](https://en.wikipedia.org/wiki/Single_Compilation_Unit)
