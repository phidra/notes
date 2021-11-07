# Anatomy of a Terminal Emulator

- **url** = https://www.poor.dev/blog/terminal-anatomy/
- **type** = post
- **auteur** = [Aram DREVEKENIN](https://www.poor.dev/), dev rust
- **date de publication** = 2021-11-02
- **source** = [son blog](https://www.poor.dev/blog/)
- **tags** = language>none ; topic>terminal ; topic>pty ; topic>shell ; level>beginner

**TL;DR** : un excellent article pour comprendre le rôle relatif entre terminal, shell, et pty.

* [Anatomy of a Terminal Emulator](#anatomy-of-a-terminal-emulator)
   * [Rôles respectifs](#rôles-respectifs)
   * [Ce qui se passe au démarrage d'un terminal](#ce-qui-se-passe-au-démarrage-dun-terminal)
   * [ANSI escape-codes](#ansi-escape-codes)
   * [Manipuler directement le pty d'un terminal](#manipuler-directement-le-pty-dun-terminal)

## Rôles respectifs

- **shell** = programme, qui est un wrapper autour des syscalls exposés par l'os (+basic scripting capabilities)
- **terminal emulator** = programme qui reçoit en entrée des données en provenance du shell (ou de tout autre programme qu'il "contient", comme vim), et qui est chargé de savoir comment les afficher à l'écran. En gros, c'est un "displayer".
- **pty** = pseudo-terminal =  bidirectionnal asynchronous communication channel

Ce que [wikipedia](https://en.wikipedia.org/wiki/Pseudoterminal) en dit :

> In some operating systems, including Unix and Linux, a pseudoterminal, pseudotty, or PTY is a pair of pseudo-device endpoints (files) which establish asynchronous, bidirectional communication (IPC) channel (with two ports) between two or more processes.

Ce qu'en dit [la page de man](https://man7.org/linux/man-pages/man7/pty.7.html) :

> A pseudoterminal (sometimes abbreviated "pty") is a pair of virtual character devices that provide a bidirectional communication channel.  One end of the channel is called the master; the other end is called the slave.

## Ce qui se passe au démarrage d'un terminal

Il a un schéma animé hyper hyper bien :
- un terminal emulator démarre
- il spawne un pty avec un shell à l'autre bout (et ç'aurait pu être directement un autre programme comme vim)
- il le shell envoie des données, que le terminal emulator affiche
- en retour, le terminal emulator envoie au shell ce que l'utilisateur lui a entré

Note : le pty bufferise stdin, ce qui est confirmé plus bas dans le post : stdin is line-buffered.

Note : d'après le code-source du post, on dirait que l'outil utlisé pour cette chouette animation est https://greensock.com/

## ANSI escape-codes

Ce que le shell envoie au pty (et donc au terminal emulator) n'est pas que du texte ASCII : ça peut également être des ANSI escape-codes

Pour visualiser la donnée binaire que le shell envoie, il écrit un programme rust (en fait, un terminal emulator basique !) qui spawne un pty avec un shell à l'autre bout, et voit passer ces fameux ANSI escape-codes.

Note : l'[ascii escape character](https://en.m.wikipedia.org/wiki/Escape_character#ASCII_escape_character) est représenté par `^[` mais ça n'est qu'une représentation : il ne s'agit pas des deux caractères consécutifs `^` puis`[` mais plutôt d'un seul caractère ASCII (celui n°27 en décimal, où `\033` en octal).

La page wikipédia des ANSI escape séquences : https://en.m.wikipedia.org/wiki/ANSI_escape_code

**Charset** : une instruction (ou plutôt, une ANSI sequence code) permet de setter le charest à ASCII. Une autre permet de le setter à "spécial characters" (NdM : comment le terminal les connait il ? C'est shell-dependent ? Ou standardisé ?)

**Redimensionnement** : sigwinch est triggered à chaque fois que le terminal est resizé.

## Manipuler directement le pty d'un terminal

Rigolo : on peut écrire directement sur le pty d'un terminal, il traitera le texte comme s'il provenait de son process esclave (i.e. du shell). NdM : pour connaître le pty du terminal courant, c'est la commande `tty`. Pour jouer avec :

- ouvrir un premier terminal n°1
- récupérer le fichier du pseudo-terminal connecté à ce terminal n°1 avec la commande `tty`, ça nous donne par exemple `/dev/pts/6`
- ouvrir un second terminal n°2 (on peut utiliser `tty` ici aussi, pour se convaincre qu'il utilise un pseudo-terminal différent, p.ex. `/dev/pts/7`)
- dans le **second** terminal, écrire du texte sur le pty du **premier** terminal :
  ```sh
  # le premier terminal a pour pty :  /dev/pts/6
  # le second  terminal a pour pty :  /dev/pts/7
  # depuis le SECOND terminal, on écrit sur le pty du PREMIER :
  echo "pouet" >> /dev/pts/6
  ```
- constater que le texte apparaît sur le **premier** terminal \o/

