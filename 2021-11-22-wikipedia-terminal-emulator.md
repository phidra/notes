# Terminal emulator

- **url** = https://en.wikipedia.org/wiki/Terminal_emulator
- **type** = page wikipedia
- **auteur** = contributeurs wikipedia
- **date de publication** = 2021-11-04 pour la dernière édition (mais ça a assez peu de pertinence)
- **source** = [https://en.wikipedia.org/](wikipedia)
- **tags** = language>none ; topic>terminal ; topic>terminal-emulator ; level>beginner

**TL;DR** : en complément de plusieurs notes précédentes sur les terminaux, les teletype, etc. ; cet article wikipedia a des infos intéressantes.

> A terminal emulator, terminal application, or term, is a computer program that emulates a video terminal within some other display architecture. 

Point important : un émulateur de terminal émule ledit terminal **au sein d'une architecture permettant d'afficher ce qui est émulé**.

----

> A terminal emulator inside a graphical user interface is often called a terminal window.

Dans mon usage quotidien, l'architecture d'affichage est un environnement graphique X.

----

> Most terminals were connected to minicomputers or mainframe computers 

Initialement, les terminaux servaient d'IHM à des ordinateurs déportés.

----

> Typically terminals communicate with the computer via a serial port via a null modem cable, often using an EIA RS-232

La liaison entre le terminal et l'ordinateur était souvent une lisaison RS-232 (mais l'article parle d'autres liaisons).

----

> the single factor that classed a terminal as "intelligent" was its ability to process user-input within the terminal—not interrupting the main computer at each keystroke—and send a block of data at a time (for example: when the user has finished a whole field or form)
>
> Some dumb terminals had been able to respond to a few escape sequences without needing microprocessors: 
>
> Most terminals in the early 1980s [...] despite the introduction of ANSI terminals in 1978, were essentially "dumb" terminals, although some of them (such as the later ADM and TVI models) did have a primitive block-send capability. 

La notion de _dumb terminal_ correspond à la possibilité ou non pour le terminal de traiter les séquences entrées par l'utilisateur (sinon, un dumb terminal envoie CHAQUE touche enfoncée à l'ordinateur, ce que je trouve assez fou !).

----

> The advance in microprocessors and lower memory costs made it possible for the terminal to handle editing operations such as inserting characters within a field that may have previously required a full screen-full of characters to be re-sent from the computer, possibly over a slow modem line. Around the mid 1980s most intelligent terminals, costing less than most dumb terminals would have a few years earlier, could provide enough user-friendly local editing of data and send the completed form to the main computer. 

Dans les années 1980, les _dumb terminal_ ont laissé la place à des terminaux "intelligents" (i.e. capable de traiter des blocs de texte localement pour n'envoyer une ligne définitive à l'ordinateur déporté qu'après confirmation/correction par l'utilisateur).

----

> Providing even more processing possibilities, workstations like the Televideo TS-800 could run CP/M-86, blurring the distinction between terminal and Personal Computer.

Point intéressant concernant la distinction entre terminal (i.e. dispositif servant d'IHM à un ordinateur déporté) et PC=Personal-Computer (où l'ordinateur n'est plus déporté, mais devant l'utilisateur) : plus les terminaux deviennent intelligents et capables de calculs, plus cette distinction est floue.

----

> Virtual consoles, also called virtual terminals, are emulated text terminals, using the keyboard and monitor of a personal computer or workstation. The word "text" is key since virtual consoles are not GUI terminals and they do not run inside a graphical interface. 

À la différence des _terminal emulators_, les _virtual terminals_ sont utilisables dans un environnement non-graphique.

----

> In the past, Unix and Unix-like systems used serial port devices such as RS-232 ports, and provided /dev/* device files for them.
>
> With terminal emulators those device files are emulated by using a pair of pseudoterminal devices. This pair is used to emulate a physical ports/connection to the host computing endpoint - computer's hardware provided by operating system APIs, some other software like rlogin, telnet or SSH or else.[12] For example, in Linux systems these would be /dev/ptyp0 (for the master side) and /dev/ttyp0 (for the slave side) pseudoterminal devices respectively.

Lorsque les UNIX se connectaient à de "vrais" terminaux, la communication se faisait avec une liaison RS-232, qui apparaissait comme _device-file_ dans le filesystem sous `/dev/`.

Maintenant, pour les _terminal emulators_ (et, je suppose, également pour les _virtual terminal_), un **pseudo-terminal** émule ces _device-files_.
