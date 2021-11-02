# The End Of Object Inheritance & The Beginning Of A New Modularity

- **url** = https://pyvideo.org/pycon-us-2013/the-end-of-object-inheritance-the-beginning-of.html
- **type** = vidéo
- **auteur** = [Augie FACKLER](https://www.linkedin.com/in/augiefackler) et [Nathaniel MANISTA](https://github.com/nathanielmanistaatgoogle), tous deux devs chez Google
- **date de publication** = 2013-03-15
- **source** = [les vidéos de la PYCON US 2013](https://pyvideo.org/events/pycon-us-2013.html), j'ai trouvé le lien sur [cet excellent article](https://glyph.twistedmatrix.com/2020/07/new-duck.html)
- **tags** = language>python ; topic>python>interfaces ; topic>composition topic>inheritance ; level>intermediate

**TL;DR** : notes en vrac sur la vidéo :

- 00:40 premise #1 types for nouns
- 01:10 premise #2 express code structurally (subdirectories/visibility/pas de goto/...)
- 02:20 premise #3 programming is parametrized
- 03:15 les conventions suivies par les diagrammes de leurs slides : creux=abstrait plein=concret
- 07:45 une autre lecture de la premise #1 = quand on décrit quelque chose avec un nom, il faut lui donner un type
- Grosso modo, les deux premières minutes exposent le problème : trois étages d'un traitement = trois méthodes d'une même classe, le deuxième étage est abstrait (i.e. défini par l'implémentation concrète donc par une classe fille), les deux autres étages sont définis dans l'AbstractBaseClass parente
- 13:00 une erreur doit attirer l'attention (un contre-exemple de ça = si un requirement n'est décrit que dans une docstring, et n'est pas rendu apparent/obligatoire par le code).  Dit autrement, comment le système réagit-il à un PROGRAMMING defect (on parle bien de defect au development-time, et pas au runtime : par exemple si le développeur utilise mal une classe). En pratique, on veut que le système soit fault-intolerant, i.e. qu'il empêche l'utilisateur d'introduire un tel defect.
- inheritance = pas très fault-intolerant : on peut violer dans une classe fille un requirement de la classe parente
- 15:15 **make illegal behavioral interactions impossible**
- 16:16 comment découper du code en plusieurs morceaux ?
    - Règle n°1 = découper le code de sorte que les relations entre les morceaux découpés soient minimisées (mon interprétation = pour réduire les dépendances)
    - Règle n°2 = découper pour essayer d'obtenir des relations unidirectionnelles (et minimiser les relations bidirectionnelles)
    - Quand on essaye de faire ça, on finit naturellement par définir des types (= des interfaces abstraites) partout, et c'est une bonne chose !
    - Les clients vont utiliser les interfaces, et non les classes concrètes
- 19:20 les constructions vont se transformer en factory functions (NdM : pas bien compris pourquoi... parce qu'on utilise les types abstraits ?)
- 20:20 les modules se transforment en des collections de {types+fonctions}
- 22:30 position (d'un autre) problème : quand on cherche à faire parler ensemble différents modules, tous les types sont différents, vu que les modules ont été conçus indépendamment ? Une mauvaise solution serait de modifier un module A pour qu'il fournisse son résultat dans le type qu'attend le module B.
- 24:10 c'est une mauvaise solution : la business logic du code doit être écrite en exprimant les dépendances qu'on aimerait voir (plutôt que celles qu'on a réellement). La bonne solution est plutôt de garder les modules A et B inchangés, et d'écrire un module A2B qui permet la transition.
- 24:45 **stateless adapters are cheap and provide clearer higher level code**
- Combiné avec la note précédente, j'en déduis : écrire les modules utilisant des types en entrée et en sortie "idéaux" (i.e. parfaits pour le job du module pris isolément), et écrire en plus des adapters, pour que la combinaison de tout ça forme mon application (plutôt que d'écrire des modules avec des types pas terrrribles, mais nécessaires pour l'application)
