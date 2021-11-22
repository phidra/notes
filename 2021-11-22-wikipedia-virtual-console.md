# Virtual console

- **url** = https://en.wikipedia.org/wiki/Virtual_console
- **type** = page wikipedia
- **auteur** = contributeurs wikipedia
- **date de publication** = 2021-10-04 pour la dernière édition (mais ça a assez peu de pertinence)
- **source** = [https://en.wikipedia.org/](wikipedia)
- **tags** = language>none ; topic>terminal ; topic>virtual-terminal ; topic>virtual-console ; level>beginner

**TL;DR** : en complément de plusieurs notes précédentes sur les terminaux, les teletype, etc. ; cet article wikipedia a quelques petites infos.

> A virtual console (VC) – also known as a virtual terminal (VT) – is a conceptual combination of the keyboard and display for a computer user interface.

Console = user-interface = combinaison de :

- **display** = quelque chose pour afficher les outputs de l'ordinateur
- **keyboard** = quelque chose pour accepter les inputs de l'utilisateur

----

> It is a feature of some Unix-like operating systems such as Linux, BSD, [...] in which the system console of the computer can be used to switch between multiple virtual consoles to access unrelated user interfaces.

La console système (celle de `init`) permet de switcher entre plusieurs consoles virtuelles, chacune se comportant comme une UI indépendante des autres ; d'où le fait que UNIX est un système multi-utilisateur.

----

> In the Linux console and other platforms, usually the first six virtual consoles provide a text terminal with a login prompt to a Unix shell. The graphical X Window System starts in the seventh virtual console.

Rien de nouveau.
