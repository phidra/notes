**TL;DR** :

- `man 2 open` :
    - ouvrir la man-page `open` de la section 2
    - `open` existe dans les sections (1) et (2)
    - par défaut, la première est ouverte -> si je veux la (2), je dois le préciser explicitement
- `LANG=en man nmap` :
    - forcer la consultation de la man-page en anglais
- `man -f close` :
    - lister toutes les sections contenant la man-page exacte `close`
    - utile pour savoir qu'une commande a des homonymes
- `man -k regex` :
    - lister toutes les man-pages dont le titre ou la description contient `regex`
    - exemple : `man -k '^close'`
- `man -s 2 -k .`  :
    - lister toutes les man-pages dans la section 2 (i.e. tous les appels système)

# Sections

- les pages de man sont classée par sections, consultables dans la doc :
    ```
    man man
    ```
- en pratique, les plus importantes sont :
    - 1. shell
    - 2. syscall
    - 3. fonctions de libs
    - 7. divers
- si une même entrée existe dans plusieurs sections, seule la première sera affichée, il faut donc préciser celle qui nous intéresse si nécessaire :
    ```
    man 3 printf
    ```
- pour avoir un descriptif d'une section donnée :
    ```
    man 3 intro
    ```

Les chiffres entre parenthèses derrière des pages correspondent à la section :

```
man printf
    SEE ALSO
    printf(3)  # <--- le printf actuellement consulté est la shell-builtin (1), mais printf existe aussi dans la libc (3)
man fopen
    SEE ALSO
    open(2), fclose(3), fileno(3), fmemopen(3), fopencookie(3)  # open est un syscall (2), les autres sont des fonctions de la libc (3)
```


# Divers

Le format est un format de texte avec des balises : **roff**

Les pages de man sont (gzippées) dans `/usr/share/man`

Les paquets installés ajoutent parfois les pages de man (p.ex. `postgresql` ajoute entre autres `man 7 CLOSE` et `man 7 CLUSTER`)

