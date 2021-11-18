# What Are Teletypes, and Why Were They Used with Computers ?

- **url** = https://www.howtogeek.com/727213/what-are-teletypes-and-why-were-they-used-with-computers/
- **type** = article
- **auteur** = [Benj EDWARDS](https://www.howtogeek.com/author/benjedwards/), associate editor for How-To Geek
- **date de publication** = 2021-08-24
- **source** = [How-To-Geek](https://www.howtogeek.com/), magazine tech
- **tags** = language>none ; topic>computers-history ; topic>teletype ; level>beginner

**TL;DR** : très intéressant article retraçant l'histoire de l'utilisation des teletypes en informatique.


* [What Are Teletypes, and Why Were They Used with Computers ?](#what-are-teletypes-and-why-were-they-used-with-computers-)
   * [Montée en puissance des teletypes](#montée-en-puissance-des-teletypes)
   * [Déclin des teletypes](#déclin-des-teletypes)

## Montée en puissance des teletypes

> For a few decades, many computer system operators used devices called teletypes to interact with computers using a typewriter-style keyboard and output printed on spools of paper.

Historiquement, les interactions avec les ordinateurs se faisaient :

- en entrée avec un clavier du même type que ceux des machines à écrire
- en sortie en imprimant sur des bobines de papier

> The term “teletype” originated as a trademarked term for a brand of teleprinters created by the Teletype Corporation in 1928.

Terminologie : **teleprinter** (en français : téléscripteur ou téléimprimeur) est le terme officiel, **teletype** en est devenu synonyme par [antonomase](https://fr.wikipedia.org/wiki/Antonomase), suite au succès du [Teletype Model 33](https://en.wikipedia.org/wiki/Teletype_Model_33), sorti en 1963.

[L'article suivant](./2021-11-18-forgotten-world-of-dumb-terminals.md) en a une définition que je trouve intéressante = teletype = _mechanical typewriter_ = machines à écrire automatiques.

Les teletype prédatent les ordinateurs ! L'utilisation des teletype en tant que I/O pour communiquer avec un ordinateur n'est qu'un de leurs usages (un autre étant le [Télex](https://fr.wikipedia.org/wiki/T%C3%A9lex) = réseau de communication entre téléscripteurs, mis en place à partir des années 1930 et encore en service au début du XXIe siècle [...] Orange clôture ses derniers abonnements français le 31 janvier 2017.).

> Whatever you type on one typewriter gets automatically printed out on the other.

Dans le contexte du début du 20e siècle, les teletype permettaient des communications instantanées entre deux teletypes.

> Instead of communicating with a remote teleprinter, you’re sending and receiving human-readable text to and from a computer.
> 
> The computer could be in the same room, in another part of a building, or even halfway across the world when linked by a telephone network.

Utilisation pour l'informatique.


> Many early large computer systems (especially those sold by IBM) were batch operated, which meant that a program would be typed onto punched cards, the punched cards would be fed into the machine with other programs (in a batch), and then the results would be written onto another stack of punched cards. The output stack would then be fed into a tabulating machine or a printer that would print the results in human-readable form.

Intéressant : au début, les ordinateurs n'étaient pas interactifs. De plus, la sortie était produite sur des cartes perforées, et un agent tiers faisait la traduction entre des trous dans les cartes perforées, et des résultats lisibles imprimés.

> Alongside batch computing in the mid-1950s, engineers began to experiment with interactive computing, where a computer operator could provide input and get results back in almost real-time in a sort of interactive “conversation” with the machine.

À partir du milieu des années 1950, les ordis deviennent "interactifs" (par opposition au mode batch). Ces ordis interactifs utilisent des teleprinters pour l'interaction.

L'article parle de 1954 pour la sortie de l'IBM 610 (qui disposait d'un clavier pour ses inputs), mais [la page wikipedia](https://en.wikipedia.org/wiki/IBM_610) mentionne plutôt 1957.

> The invention of time-sharing in 1959 allowed multiple users to share an interactive computer system at the same time, making low-cost, single-personal terminals like teletypes desirable for computer use. As time-sharing became more common in the 1960s, organizations with mainframe computers began to buy off-the-shelf commercial teletype machines to use as terminals more frequently.

Jusqu'ici, les teletypes n'étaient pas forcément des teletypes commerciaux indépendant (mais étaient, je suppose, des teletypes custom, livrés avec les ordinateurs).

Au début des années 1960, grâce au [time-sharing](https://web.stanford.edu/~learnest/nets/timesharing.htm), les ordinateurs deviennent multi-utilisateurs, il devient possible de "partager" un mainframe, et on avait donc besoin de plusieurs terminaux par ordinateur -> on s'est mis à utiliser des teletypes commerciaux.

> Popular minicomputers of the late 1960s and early ’70s, such as the PDP-8, PDP-11, and the Data General Nova, supported ASCII encoding, making the Model 33 an ideal low-cost (relatively speaking) input/output (I/O) terminal for them.

C'est parce qu'il utilisait ASCII que le *Teletype Model 33* a eu son succès, et que **teletype** est devenu synonyme de **teleprinter**.

> When you used a teletype with a mainframe computer like these, you’d see your own local input on paper as you typed, and then you’d receive a response from the computer printed below it as the teletype printed to a continuous feed of rolled paper stored within the unit.
>
> In 1970, Dennis Ritchie and Ken Thompson developed the UNIX operating system on a PDP-11 using Model 33 teletypes as interfaces, and some of the teletype-related design choices that they made are still with us today

Fonctionnement d'un teletype, et utilisation pour créer UNIX !

À ce stade, les teletypes jouent le rôle de terminal, i.e. de "bout terminal d'une liaison de communication avec l'ordinateur".

## Déclin des teletypes

> While popular for a time, Teletypes did have some significant drawbacks as computer terminals.\
> They were very noisy due to the mechanical action of the impact printhead rapidly hitting the paper.\
> They were also slow, often limited to about 10 characters per second.\
> And finally, you had to use a lot of paper.

Les teletypes en tant que terminal, c'était mieux que le mode batch, mais c'était pas fi-fou non plus...

> In the 1960s, companies such as IBM began experimenting with computer terminals that used CRT displays instead of paper for output.\
> Still, many computer operators often stuck with teletypes throughout the 1970s due to their lower cost.\
> While at least three manufacturers produced video terminals by 1970, each cost significantly more than a Teletype Model 33.

Point de repère temporel = années 1960, premiers terminaux video (CRT). Pourtant, même dans les années 1970, les teletypes continuent d'être utilisés, car ils sont environ 3 fois moins chers.

> Once video terminals dropped in price and exceeded the capabilities of teletypes, teletypes quickly fell out of favor.\
> Compared to teletypes, video terminals were silent and had no moving parts other than the keyboard, making them more reliable and pleasant to use.\
> Their display speed also wasn’t limited to the mechanical action of a printhead, so they could display more information much faster than a teletype could.

Le prix des terminaux video chute au cours des années 1970, pour devenir complètement abordable en 1980. Comme leur prix était leur seul désavantage, ils supplantent complètement les teletypes.

> Also, in the mid-1970s, personal computers like the Apple II began to integrate input and output functionality directly into the computer itself.

En plus des terminaux pour les mainframes, les ordinateurs personnels comme l'Apple II intégraient leur propre écran vidéo CRT. D'après [wikipedia](https://fr.wikipedia.org/wiki/Apple_II), l'Apple II, sorti en 1977, avait un écran de 24 lignes x 40 colonnes.
