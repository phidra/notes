# Systems design explains the world: volume 1

- **url** = https://apenwarr.ca/log/20201227
- **type** = post
- **auteur** = Avery PENNARUN, dev canadien, [ancien de chez Google et Microsoft](https://www.usenix.org/conference/srecon18europe/speaker-or-organizer/avery-pennarun-google), actuellement co-fondateur et CEO de https://tailscale.com/ , boîte qui fait des VPN
- **date de publication** = 2020-12-27
- **tags** = language>agnostic ; topic>system-design ; essay


**TL;DR** : article intéressant sur le system-design.

Il ne donne pas de définition claire du system-design, mais indique qu'il traite d'abstractions, de patterns qui se répètent souvent : raisonner au niveau de l'abstraction permet de mieux identifier et gérer les situations.

Le gros intérêt à mes yeux de l'article, c'est surtout que j'en ressors sensibilisé aux 4 patterns qu'il donne en exemple :
- **Systems of control: hierarchies and decentralization** : avoir un système réellement décentralisé est difficile, on peut se retrouver à obfusquer l'autorité centrale, et dans ce cas, mieux vaut la laisser explicite.
- **chicken-egg problems** : les problèmes pour lequels pour qu'un système apporte de la valeur, il faut des utilisateurs, *mais* pour que les utilisateurs viennent utiliser le système, il faut que celui-ci apporte de la valeur.
- **second-system effect** : le cas où on veut réécrire un système de zéro en repartant d'un état vierge.
- **innovator's dilemma** : les situations où une entreprise se fait grignoter petit à petit par un concurrent à la technologie différente, sur laquelle il n'est jamais intéressant de travailler... jusqu'au moment où elle prend le dessus !


## Au sujet du system design en général

"Systems design" is a branch of study that tries to find universal architectural patterns that are valid across disciplines.

Intéressant exemple (qui résonne, actuellement...) sur l'évolution de carrière "classique" des ingénieurs. Deux profils ne cadrent pas dans le système classique :
- ceux qui, une fois senior, veulent rester experts à débugger/comprendre/construire des choses (alors que l'évolution classique est plutôt de gérer des problèmes business de plus en plus importants)
- ceux qui, encore junior, sont pourtant bons avec les problèmes business, mais pas très bons à débugger/comprendre/construire des choses

> "Glue work is expected when you're senior... and risky when you're not."
>
> What she calls glue work, I'm going to call systems design
>
> People who are naturally excellent at glue work often stall out early in the prescribed engineering pipeline, even when they'd be great in later stages (staff engineers, directors, and executives) that traditional engineers struggle at.

Sa définition du system design est tout sauf précise, mais il donne l'idée générale :
- _It's the thing that will eventually kill your project if you do it wrong, but probably not right away._
- _It's macroeconomics instead of microeconomics._
- _It's fixing which promotion ladders your company even has, rather than trying to climb the ladders._
- _It's knowing when a distributed system is or isn't appropriate, not just knowing how to build one._


Caractère invisible, flou, non-rigoureux du system-design :

> Most of all, systems design is invisible to people who don't know how to look for it. At least with code, you can measure output by the line or the bug, and you can hire more programmers to get more code. With systems design, the key insight might be a one-sentence explanation given at the right time to the right person, that affects the next 5 years of work, or is the difference between hypergrowth and steady growth.

### Digression sur les commentaires de l'article sur HackerNews

**Digression** : ce manque de rigueur est ce qui est le plus critiqué dans les différentes commentaires du post sur hacker-news, cf. [ici](https://news.ycombinator.com/item?id=25552267) ou [là](https://news.ycombinator.com/item?id=25553288).

Une critique intéressante (celle du deuxième lien) = les gens ont pas de problème avec les boxes and arrows, mais critiquent le fait que les boxes et arrows en question ont une définition trop vague.

En effet, l'article ne donne pas de définition de system design (ni de system d'ailleurs). Extraits d'un commentaire :

> Let me put it to you this way. Anytime you see the word "Design" it refers to a field where we have little knowledge of how things truly work.
>
> Design is another word for random intuitive guess. 
>
> If someone has the job title "System architect" or "System Designer" you know it's complete BS. The guy is just making stuff up from intuition and experience, there is no formal science here. He is much closer to an "artist" then he is to a "mathematician", physicist" or "scientist"
> The lack of formal legitimacy leaves a lot of room for BS.
>
> Overall My point is two things:
>
> 1. Often people don't know the difference between intuitive knowledge and formalized knowledge and attribute way too much legitimacy to "systems design." People think of these guys as scientists when they're really just making stuff up.
>
> 2. Informal intuition can as a result devolve into a trap where we endlessly come up with new frameworks, new metaphors and new "designs" without improving anything because there's no formal way of verifying what is optimal. I am arguing that the author of this article and articles like this is just the latest iteration of an endless circle. He introduces nothing new and instead provides a new metaphor for us to ponder about. Literally read it. Did anything change did you actually learn anything new? Was anything actually improved? No.

**NdM** : de mon côté, oui, j'ai appris quelque chose à la lecture de l'article : j'aurais bien plus tendance qu'avant à identifier les 4 patterns qu'il a donnés en exemple. En quelque sorte, je suis "sensibilisé".

## Exemple de pattern n°1 = Systems of control: hierarchies and decentralization

> The truth is, nearly every attempt to design a hierarchy-free, "flat" control system just moves the central control around until you can't see it anymore.

TL;DR : tout le monde dit que décentralisé c'est mieux, mais c'est *très* dur car en pratique, on se contente souvent de reporter la centralisation ailleurs.

Exemple de systèmes vraiment décentralisés :
- écosystème terrestre
- git

Exemple de systèmes pas vraiment décentralisés :
- démocratie (car il faut bien une autorité cenrale qui garantit que le pouvoir appartient au peuple)
- DNS / TLS / routing (qui dépendent d'autorités centrales)

Sa suggestion sur ce sujet :

> my rule of thumb is to do exactly what Jo Freeman suggested: at least make sure the control structure is explicit. When it's explicit, you can debug it.

## Exemple de pattern n°2 = chicken-egg problems

Sa définition = un cas où il faut des utilisateurs pour qu'un système ait de l'intérêt, et où il est donc difficile d'attirer les premiers utilisateurs. Exemples :
- social network (qui va aller s'inscrire sur un réseau social sans utilisateurs ? comment attirer les utilisateurs si justement personne n'est dessus ?)
- le téléphone (qui va faire installer le téléphone s'il ne peut appeler personne ?)
- consoles (quel studio va développer des jeux pour une console sans utilisateurs ? Quels utilisateurs vont aller acheter une console sans jeux disponibles ?)

> The defining characteristic of a chicken-egg technology or product is that it's not useful to you unless other people use it.
>
> Since adopting new technology isn't free (in dollars, or time, or both), people aren't likely to adopt it unless they can see some value, but until they do, the value isn't there, so they don't.

Sa suggestion sur ce sujet :

> Just like with real chickens and real eggs, there's a way to do it by bootstrapping from something smaller.
>
> The main techniques are to lower the cost of adoption, and to deliver more value even when there are fewer users.

Exemple de Nintendo, qui, comme les autres développeurs de console, fait bien les choses sur ce sujet :
- _Subsidizing the cost of early console sales._ (NdM : incite à l'achat de la console car elle ne coûte pas cher)
- _Backward compatibility, so people who buy can use older games even before there's much native content._ (NdM : incite à l'achat de la nouvelle console car même nouvelle, elle dispose de jeux = ses anciens jeux)
- _Games that are "mostly the same" but "look better" on the new console._ (NdM : incite au développement d'un jeu, car c'est plus facile pour le studio, qui se contente de refaire le même)
- _Compatible gamepads between generations, so developers can port old games more easily._ (NdM : incite à l'achat de la console car on peut réutiliser le gamepad, et incite au développement d'un jeu car le studio peut plus facilement porter un ancien jeu)
- _"Exclusive launch titles": co-marketing that ensures there's value up front for consumers (new games!) and for content producers (subsidies, free advertising, higher prices)._ (NdM : incite à l'achat de la console, car on a une bonne raison de l'acheter, et incite au développement des jeux car le studio a plus de chances d'avoir des acheteurs si jeu exclusif)

Note : en plus d'inciter les utilisateurs à switcher, on peut inciter "l'autre parti" (dans l'exemple de Nintendo : le studio de jeu vidéo = celui qui a besoin d'utilisateurs) à switcher sur le "nouveau truc" même s'il y a peu d'utilisateur : dans l'exemple, ça revenait à prendre des mesures pour réduire le coût de dev.

Il donne quelques exemples de cas où les gens ont IGNORÉ le chicken-egg problem :
- Firefox and Ubuntu phones,
- distributed open source social networks,
- alternative app stores,
- Linux on the desktop,
- Netflix competitors.

NdM : ma compréhension des choses est que ce sont des situations où on s'est contenté de balancer un produit nouveau alors qu'il y a déjà de la concurrence, sans prendre en compte le fait que les gens n'auraient a priori aucune raison de switcher sur le truc nouveau (aucun "facteur différenciant").

Autre exemple = IPV6 : pas d'intérêt (d'incitation) à switcher, au contraire : c'est du coût de maintenance en plus.

NdM : tout ceci m'évoque ÉNORMÉMENT python3.

NdM : une autre façon de présenter les choses est que sans incitation forte, c'est le statu quo qui demeurera.

Une autre caractéristique du chicken-egg problem : plus il y a d'utilisateurs, plus chaque utilisateur tire de la valeur des autres utilisateurs (la valeur est exponentielle en le nombre d'utilisateur).

> If your product or company has a chicken-egg problem, and you can't clearly spell out your concrete plan for solving it, then investors definitely should not invest in your company.

Version encore plus avancée = le cas où on a DEUX types d'utilisateurs, et où aucun utilisateur de type 1 n'a de raison d'utiliser le système s'il n'y a pas encore d'utilisateur de type2 (et vice-versa). Exemple d'Uber :
- il faut des clients = des passagers (mais il n'y en aura pas s'il n'y a pas de chauffeurs).
- il faut des chauffeurs (mais personne ne voudra être chauffeur s'il n'y a pas assez de travail car pas assez de chauffeur)

Version ENCORE plus avancée = ubereats :
- il faut des clients = des mangeurs (qui ont besoin à la fois de chauffeurs, et à la fois de restaurants)
- il faut des chauffeurs (qui ont besoin à la fois de mangeurs et de restaurants)
- il faut des restaurants pour produire la nourriture (qui ont besoin à la fois de mangeurs et de chauffeurs)

Note : l'un des avantages d'ubereats sur le sujet, c'est qu'ils avaient déjà des chauffeurs et des clients : c'était plus facile pour eux de se contenter d'ajouter les restaurants.

## Exemple de pattern n°3 = second-system effect

Sa définition :
- on commence un produit petit
- le produit gagne en popularité, on lui ajoute des features, verrue sur verrue, les tradeoffs initiaux (même justifiés) deviennent limitants
- pour corriger ces limites, on invente un design qui adresse les problèmes
- on est prêt à investir pour tout réécrire de zéro selon ce nouveau design

Avis de Joël Spolsky sur ce qu'a fait netscape mozilla :
> "They did it by making the single worst strategic mistake that any software company can make: they decided to rewrite the code from scratch."

Autres exemples :
- IPV6
- python3 (NdM : mais je suis pas tout à fait d'accord avec lui : un changement non-rétrocompatible n'est pas la même chose qu'une réécriture de zéro)
- plan9

Les problèmes = ce qui se passe quand on essaye de cosntruire ce "second system" :
- ça prend plus de temps que prévu
- le nouveau design créée de nouveau problèmes (et ce, même s'il adresse correctement les problèmes qui étaient identifiés)
- on doit partager les ressources entre 1. faire évoluer l'ancien système et 2. développer le nouveau système
- afin de forcer la migration, on a tendance à éteindre de force l'ancien système, même s'il est toujours utilisé/désiré

Sa suggestion : pas grand chose, à part essayer à tout prix de l'éviter, par exemple en refactorant le code plutôt que de le réécrire.

## Exemple de pattern n°4 = innovator's dilemma

C'est le cas le moins facile à décrire. En deux mots, une entreprise se retrouve distancé par une technologie concurrente, alors même qu'à aucun moment il n'était intéressant pour l'entreprise de travailler sur cette techno.

> You (Intel in this case) make an awesome product in a highly profitable industry.
>
> Some crappy startup appears (ARM in this case) and makes a crappy competing product with crappy specs. The only thing they seem to have going for them is they can make some low-end garbage for cheap.
>
> As a big successful company, your whole business is optimized for improving profits and margins. Your hard-working employees realize that if they cede the ultra-low-end garbage portion of the market to this competitor, they'll have more time to spend on high-valued customers. As a bonus, your average margin goes up! Genius.
>
> The next year, your competitor's product gets just a little bit better, and you give up the new bottom of your market, and your margins and profits further improve. This cycle repeats, year after year. (We call this "retreating upmarket.")
>
> The crappy competitor has some kind of structural technical advantage that allows their performance (however you define performance; something relevant to your market) to improve, year over year, at a higher percentage rate than your product can. And/or their product can do something yours can't do at all (in ARM's case: power efficiency).
>
> Eventually, one year, the crappy competitor's product finally exceeds the performance metrics of your own product, and promptly blows your entire fucking company instantly to smithereens.


Au final, non seulement le concurrent finit par faire jeu égal avec toi voire même te dépasser, mais surtout la différence de sa techno fait que tu peux pas le rattraper. Le coeur du truc repose sur le fait que :
1. à la base, la techno différente est pas intéressante pour toi : il est LOGIQUE pour toi d'abandonner la R&D sur le sujet car elle est moins rentable que ta propre techno.
2. le concurrent, lui, ne l'abandonne pas, et beaucoup (beaucoup) plus tard, elle finit par devenir intéressante.

En quelque sorte, à aucun moment, Intel n'aurait eu intérêt à investir dans la même techno qu'ARM : ç'aurait été perdre de l'argent, donc contre-productif, que de le transférer depuis x86 (vache à lait, grosses marges) vers ARM. Pourtant, plusieurs années plus tard, c'est ARM qui devient intéressant (car consommant moins d'eléctricité). Et là, c'est trop tard : ARM a dépassé intel, sur une techno (devenue) plus intéressante : ils sont devant, et progressent plus vite. Et même : si après quelques années, Intel a un super pressentiment de ce qui va arriver et essayer de concurrencer ARM, ça ne sera pas simple, car ARM a déjà une longueur d'avance.

NdM : ma compréhension = pour éviter cet effet, il faut accepter d'investir de la R&D dans des projets qui ne sont pas rentables, ce qui est contre-productif (on ne peut pas investir à tort et à travers : sur 100 projets non-rentables, seul 1 sera peut-être couronné de succès). D'où le "dilemma" : soit on perd de l'argent en investissant dans des trucs pas intéressants et pas rentables, soit on se fera concurrencer beaucoup plus tard.

> The dilemma comes from the fact that all large companies are heavily optimized to discard ideas that aren't as profitable as their existing core business. Any company that doesn't optimize like this fails.
>
> But this optimization creates a corporate political environment where, for example, Intel could never create a product like ARM. A successful low-priced chip would take time, energy, and profitability away from the high-priced chips, and literally would have made Intel less successful for years of its history.

Sa suggestion : il redirige sur ce livre : [The Innovator's Dilemma](https://www.amazon.com/Innovators-Dilemma-Technologies-Management-Innovation-ebook/dp/B00E257S86) by Clayton M. Christensen

Le livre (qui a créé le terme "disruptive innocation") dit qu'il y a deux types d'innovation :
- _sustaining innovation_ = améliorer ses processeurs x86
- _disruptive innovation_ = créer un processeur ARM : _the kind where an entirely new thing sucks for a very long time, and then suddenly and instantly blows you away_

Conséquence amusante : si tu as une disruptive innovation, ne t'inquiète pas trop qu'on te la pique : par définition, les grandes entreprises n'investiront pas dedans car ce sera pas rentable.

Autre exemple concret :
- les DVCS ne concurrençaient pas sérieusement les VCS centralisés... tant que le réseau et le disque étaient limitants !
- une fois que cette limitation a été dépassée, les DVCS ont tués d'un coup les VCS centralisés
- et le dilemme, c'est qu'il était CONTRE-PRODUCTIF pour les VCS centralisés de partir sur la voie du décentralisé plus tôt
