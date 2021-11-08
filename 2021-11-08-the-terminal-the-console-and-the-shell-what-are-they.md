# The terminal, the console and the shell - what are they?

- **url** = https://unixsheikh.com/articles/the-terminal-the-console-and-the-shell-what-are-they.html
- **type** = post
- **auteur** = [unixsheikh](https://unixsheikh.com), je ne trouve pas de nom, il/elle semble être un dev Danois avec de la bouteille
- **date de publication** = 2021-01-13
- **source** = [son blog](https://unixsheikh.com/)
- **tags** = language>none ; topic>terminal ; topic>console ; topic>shell ; level>beginner

**TL;DR** : en complément [du précédent](./2021-11-07-anatomy-of-a-terminal-emulator.md), un bon article pour comprendre le rôle relatif entre terminal, console, et shell :

- historiquement, la console était le panneau de contrôle-commande des whatmille armoires qui constituaient un ordinateur
- le terminal était le dispositif permettant de commander l'ordinateur (via le clavier pour entrer ses commandes) et de recevoir ses retours (via le TeleTYpewriter = `TTY` qui imprimait les outputs de l'ordinateur)
- quand la technologie l'a permis, la console a intégré un terminal vidéo (dont l'écran était plus pratique que d'avoir à imprimer les outputs de l'ordinateurs), et les deux sont devenus synonymes
- différence entre _virtual terminal_ et _terminal eumlator_ : le second est un programme externe qui tourne dans une interface graphique, alors que le premier est intégré à l'OS

* [The terminal, the console and the shell - what are they?](#the-terminal-the-console-and-the-shell---what-are-they)
   * [Console vs. Terminal](#console-vs-terminal)
   * [virtual terminal vs. terminal emulator](#virtual-terminal-vs-terminal-emulator)
   * [la variable d'environnement TERM](#la-variable-denvironnement-term)
   * [Les ANSI escape sequences](#les-ansi-escape-sequences)
   * [Le shell](#le-shell)

## Console vs. Terminal

> Early computers where huge machines that consisted of multiple cabinets, e.g. one cabinet for the CPU, one cabinet for each disk drive, one cabinet for a punched card reader

NdM : _cabinet_ = armoire.

> The "console" was the "control console".
>
> The word terminal [indicates] that it's the terminating end or "terminal" end of a communications process.
>
> The teleprinter or TTY was the first kind of terminal. Rather than a monitor you would have a literal typewriter in front of you. When you typed on it, you would see the text on a piece of paper and that text would be send to the computer. When the computer replied, you would see the typewriter print on the paper.
>
> Later, as computers became much smaller, it was possible to integrate multiple components into one single unit, with both a video monitor and a keyboard put together inside it, and the "console" now became more or less synonymous to the "terminal". Both the "console" and the "terminal" now referred to the physical video terminal that had replaced both the teleprinter and the old control console.

Du coup, avant la console servait à contrôler l'ordinateur (i.e. l'ensemble des armoires) et le terminal était ce qui servait à écrire du texte, et à recevoir le retour de l'ordinateur.  PUIS, DANS UN SECOND TEMPS, la console servant à contrôler l'ordinateur a accueilli un terminal, et les deux sont devenus synonymes. Note : il s'agissait d'un terminal vidéo (qui pouvait afficher les infos envoyées par l'ordinateur sans avoir à imprimer des trucs papiers).

Points de repère temporel :

- VT100 = 1978 = l'un des premiers à supporter les ANSI escape codes
- (et son succès à conduit ce standard ANSI à être massivement adopté)
- l'article donne plusieurs images des années 1960 montrant qu'on avait encore des armoires, une console, et un teletypewriter, et idem pour le début des années 1970
- du coup, la transition vers une console tout intégrée (incluant notamment un terminal) semble avoir été fait dans le courant/fin des années 1970


## virtual terminal vs. terminal emulator

> A virtual terminal or virtual console is a program that simulates a physical terminal.

Avec cette terminologie, un terminal virtuel est un émulateur de terminal ? NdM : a priori pas vraiment, le distinguo est que le terminal virtuel n'est pas vraiment un programme externe (comme peut l'être `xterm`), mais est plutôt un mécanisme intégré à l'OS. Typiquement, si ma GUI est dans les choux, je dispose toujours des terminaux virtuels pour me connecter à mon OS, alors que je ne pourrais plus lancer de terminal emulator (ou en tout cas, pas depuis la GUI).

> The virtual terminal gives the impression that several independent terminals are running concurrently. Each virtual terminal can be logged in with a different user and it can run its own shell and have its own font settings.

C'est un point important (vu que c'est leur utilité) des terminaux virtuels aujourd'hui.

Notes :

- la commande `tty` qui `print the file name of the terminal connected to standard input` fait bien le distinguo entre _terminal emulator_ (pour lequel il renvoie `/dev/pts/5`) et _virtual terminal_ (pour lequel il renvoie `/dev/tty5`).
- on fait Ctrl+Alt+F4 pour accéder au terminal virtuel n°4. Je confirme d'ailleurs que la commande `tty` renvoie `/dev/tty4` lorsque je suis sous un terminal virtuel.
- par ailleurs, pour revenir à l'interface graphique, on fait Ctrl+Alt+F7, ce qui tend à confirmer que l'IHM de mon OS est simplement un "terminal virtuel graphique"
- question : si je suis sur le terminal virtuel non-graphique, puis-je lancer une session graphique ? réponse = non, on dirait que le serveur X refuse : `server is already active for display 0`

> A terminal emulator is a computer program that emulates a physical terminal within some other display architecture, such as the X Window System.

En complément de [l'article précédent](./2021-11-07-anatomy-of-a-terminal-emulator.md), Le point de vue du présent article sur le travail du terminal-emulator est donné :

> The terminal emulator takes the input you type at the keyboard and convert those to ASCII characters which it sends to the shell, or to a program running under the shell (more about the shell later). The terminal emulator also takes the stream of output characters from the various programs you run via the shell and displays them on the monitor.
>
> The purpose of the terminal emulator is to allow access to the command line while working in a graphical user interface, such as the X Window System. Since the shell is "expecting" to interface with a human through a terminal, and we don't use a physical terminal while in a graphical environment, we need the terminal emulator.

Le fichier `/lib/terminfo` contient les terminaux gérés sur mon système :

<details>
  <summary>Expand les types de terminaux</summary>

```
tree /lib/terminfo
    /lib/terminfo
    ├── a
    │   └── ansi
    ├── c
    │   ├── cons25
    │   ├── cons25-debian
    │   └── cygwin
    ├── d
    │   └── dumb
    ├── E
    │   ├── Eterm
    │   └── Eterm-color -> Eterm
    ├── h
    │   └── hurd
    ├── l
    │   └── linux
    ├── m
    │   ├── mach
    │   ├── mach-bold
    │   ├── mach-color
    │   ├── mach-gnu
    │   └── mach-gnu-color
    ├── p
    │   └── pcansi
    ├── r
    │   ├── rxvt
    │   ├── rxvt-basic
    │   ├── rxvt-m -> rxvt-basic
    │   ├── rxvt-unicode
    │   └── rxvt-unicode-256color
    ├── s
    │   ├── screen
    │   ├── screen-256color
    │   ├── screen-256color-bce
    │   ├── screen-bce
    │   ├── screen-s
    │   ├── screen-w
    │   ├── screen.xterm-256color
    │   └── sun
    ├── t
    │   ├── tmux
    │   └── tmux-256color
    ├── v
    │   ├── vt100
    │   ├── vt102
    │   ├── vt220
    │   └── vt52
    ├── w
    │   ├── wsvt25
    │   └── wsvt25m
    └── x
        ├── xterm
        ├── xterm-256color
        ├── xterm-color
        ├── xterm-debian -> xterm
        ├── xterm-mono
        ├── xterm-r5
        ├── xterm-r6
        ├── xterm-vt220
        └── xterm-xfree86
```

</details>

## la variable d'environnement TERM

> The environment variable TERM tells applications the name of a terminal description to read from the terminfo database (see man terminfo). Each description consists of a number of named capabilities which tell applications what to send to control the terminal. For example, the cup capability contains the escape sequence used to move the cursor up.

Ma compréhension : vim a besoin d'être capable de "dessiner" sur le terminal -> pour ce faire, il doit envoyer des escape codes particuliers -> pour connaître lesquels, il utilise `$TERM`

Note : sur mes systèmes, aussi bien dans tmux qu'en dehors, `echo $TERM` me renvoie `xterm-256color`.

## Les ANSI escape sequences


> ANSI escape sequences are a standard for in-band signaling to control cursor location, color, font styling, and other options on video text terminals and terminal emulators. Certain sequences of bytes, most starting with an ASCII escape character and a bracket character, are embedded into text. The terminal interprets these sequences as commands, rather than text to display verbatim.
>
> An escape sequence is a combination of characters that has a meaning other than the literal characters contained therein. It is marked by one or more preceding (and possibly terminating) characters.

Point de repère temporel :

> ANSI sequences were introduced in the 1970s to replace vendor-specific sequences and became widespread in the computer equipment market by the early 1980s.

Le point ci-dessous est intéressant : il n'existe plus de "vrais" terminaux, mais les ANSI escape sequences gardent leur importance pour les terminaux virtuels et les émulateurs de terminaux :

> Although hardware text terminals have become increasingly rare in the 21st century, the relevance of the ANSI standard persists because a great majority of terminal emulators and command consoles interpret at least a portion of the ANSI standard.

## Le shell

> The operating system is the interface between the user and the hardware. It performs a variety of tasks including controlling hardware devices, file handling, memory management, application process management and much more.
>
> The kernel is the "core" of the operating system that controls and handles all the tasks of the system while the shell is the "interface" that provides users access to communication with the kernel.
>
> A shell process is the program that prompts you for input, takes your commands, and runs them for you. It is a computer program that serves as a command-line interpreter. The shell implements a read-eval-print loop (REPL).

En gros, le shell est un REPL qui (en tant que programme capable d'utiliser la libc et de faire des syscalls) a accès à l'OS.

Si on veut avoir l'accès à l'OS, soit il faut passer par la libc (et donc être un programme compilé), soit il faut passer par un wrapper qui expose les syscalls (... comme le shell \o/)

Rôle du shell par rapport au terminal :

> The shell knows nothing about displaying characters on the monitor or about handling input keystroke codes from the keyboard - that is up to the hardware and software that is implementing the terminal. That is why we interact with the shell using the terminal

Point de repère temporel :

- première shell UNIX = `sh` = 1971
- The first Unix shell was the Thompson shell (sh), written by Ken Thompson at Bell Labs and distributed with Versions 1 through 6 of Unix, from 1971 to 1975.


