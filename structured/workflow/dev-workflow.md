Notes sur mon workflow de dev

* [neovim](#neovim)
   * [Notes d'installation](#notes-dinstallation)
   * [Tips](#tips)
   * [Plugins (neo)vim possiblement intéressants, mais que j'ai choisi de ne pas utiliser](#plugins-neovim-possiblement-intéressants-mais-que-jai-choisi-de-ne-pas-utiliser)
   * [Plugins (neo)vim possiblement intéressants, à regarder](#plugins-neovim-possiblement-intéressants-à-regarder)
* [Telescope](#telescope)
   * [Cheatsheet](#cheatsheet)
   * [Installation](#installation)
   * [Notes à l'usage](#notes-à-lusage)
* [LSP aka Language Server Protocol](#lsp-aka-language-server-protocol)
   * [LSP server — python](#lsp-server--python)
   * [LSP server — C++](#lsp-server--c)
      * [compile_commands.json](#compile_commandsjson)
      * [construction de l'index clangd](#construction-de-lindex-clangd)
   * [LSP client — native neovim client](#lsp-client--native-neovim-client)
   * [LSP client — ALE DEPRECATED](#lsp-client--ale-deprecated)
      * [Installation](#installation-1)
      * [Features et commandes](#features-et-commandes)
      * [Configuration](#configuration)
* [JOURNAL](#journal)
   * [Vrac1](#vrac1)
   * [Utilisation de telescope avec nvim](#utilisation-de-telescope-avec-nvim)
   * [Installation sur le vieux PC](#installation-sur-le-vieux-pc)
   * [Vrac2](#vrac2)
   * [Vrac3](#vrac3)
* [FUTUR](#futur)

# neovim

## Notes d'installation

- je choisis d'installer [en compilant depuis les sources](https://github.com/neovim/neovim/wiki/Installing-Neovim#install-from-source), il y a [une doc qui explique tout ça](https://github.com/neovim/neovim/wiki/Building-Neovim).
- j'installe la version `stable` :
    ```
    git clone https://github.com/neovim/neovim --depth 1 --branch stable
    cd neovim
    ```
- je choisis d'installer sous `~/neovim` plutôt que sous `/usr/local/bin` :
    ```
    rm -r build/  # clear the CMake cache
    make CMAKE_BUILD_TYPE=Release CMAKE_EXTRA_FLAGS="-DCMAKE_INSTALL_PREFIX=$HOME/neovim"
    make install

    # zshrc :
    export PATH="$HOME/neovim/bin:$PATH"
    ```
- derrière, [la doc](https://neovim.io/doc/user/nvim.html#nvim-from-vim) indique comment configurer neovim pour sourcer la config de vim
- truc cool pas si fréquent = la doc indique comment _déinstaller_ neovim :
    ```
    # documenté :
    sudo rm /usr/local/bin/nvim
    sudo rm -r /usr/local/share/nvim/

    # mais dans mon cas, plutôt :
    rm -rf ~/neovim
    ```

## Tips

- on dirait que `Shift+up` et `Shift+down` sont équivalents à `PageUp` / `PageDown`
- on peut switcher entre les différents modes visuels (par ligne, par caractère, par colonne) sans annuler la sélection courant e.g. en tapant `V` quand on est dans le mode `v`, ou le contraire
- pour connaître le cwd :
    ```
    :lua print(vim.fn.getcwd())
    :echo getcwd()
    ```
- pour rediriger le résultats de commandes qui occupent l'écran (comme `:digraphs`) :
    ```
    :redir @a
    :digraphs
    :redir END
    # le buffer "a contient maintenant la liste des digraphs.
    ```
- pour faire un alias de fonctions, il suffit de :
    ```
    :lua lsp_rename = vim.lsp.buf.rename
    ```
- pour la complétion, il y a une différence entre :
    - `Ctrl-n` (ou `Ctrl-p`)directement (qui trigge la complétion non-intelligente)
    - `Ctrl-x` suivi de `Ctrl-o` (qui trigge la complétion intelligente utilisant le lsp)
    - `Ctrl-x` suivi de `Ctrl-l` (qui trigge la complétion de ligne)

## Plugins (neo)vim possiblement intéressants, mais que j'ai choisi de ne pas utiliser

- https://github.com/norcalli/nvim-colorizer.lua : pour colorier les couleurs RGB avec leur couleur (EDIT : pas vraiment réussi à le faire marcher (il faut termguicolors), et de toutes façons je fais pas assez de frontend pour avoir un franc besoin...)
- https://github.com/tommcdo/vim-lion : une alternative à tabular pour aligner des colonnes (mais comme tabular me convient, je ne change pas)
- https://github.com/farmergreg/vim-lastplace : mémoriser l'emplacement des derniers fichiers édités pour les rouvrir au même endroit
- https://github.com/matze/vim-move : pour déplacer plusieurs lignes de code d'un coup. J'ai pas de bon shortcut pour ça : `A-j` / `A-k` sont pris par `ALENext/Previous`,  `C-k` est important pour les digraphes... Dans les deux cas, je pourrais changer la signification du shortcut juste en visual, mais je suis pas fan... De plus, le chemin des écoliers pour déplacer des lignes (= cut/paste) reste relativement facile.
- https://github.com/p00f/nvim-ts-rainbow : pour colorier les parenthèses. L'installation dépend de nvim-treesitter et a l'air compliqué + je suis pas super fan de colorier les parenthèse, je trouve que ça rend le fichier difficilement lisible.
- https://github.com/suy/vim-context-commentstring : permet d'avoir plusieurs types de commentaires différents en fonction du contexte (e.g. un fichier python qui génère du html pourra commenter en python ou en html). Pour le moment, mon besoin est trop faible (mais ça pourrait changer car j'ai un cas d'usage assez utile = les configs lua inlinées dans mon vimrc)
- https://github.com/JamshedVesuna/vim-markdown-preview : pouvoir facilement ouvrir un browser sur un markdown en cours d'édition. Les dépendances sont trop contraingnantes, et mon besoin est suffisamment faible pour ne pas vouloir ajouter un plugin supplémentaire (si le besoin se fait plus pressant, je passerai sans doute par un outil externe plutôt que par un plugin vim)
- https://github.com/tpope/vim-projectionist : pour configurer certains trucs par projet (e.g. les types d'alternate files) -> bon complément au LSP. Mais a l'air compliqué et pas ce que je veux.
- https://github.com/mhinz/vim-sayonara : pour fermer proprement les buffers/windows (tss tss, c'est triste d'avoir besoin d'un plugin pour ça). J'ai pas le besoin au point de vouloir m'encombrer d'un plugin supplémentaire pour ça.
- https://github.com/wellle/targets.vim : ajoute de nouvelles targets pour des motions ciblant des "intérieurs" (e.g. intérieurs de parenthèses, intérieurs de virgules, etc.). Par rapport à l'utilisation classique des intérieurs, ça semble rajouter trop peu de trucs pour que je veuille ajouter un plugin pour ça.
- https://github.com/mg979/vim-visual-multi : pouvoir travailler sur plusieurs mots en même temps. Semble trop complexe pour valoir la peine d'utiliser ce plugin plutôt que répéter une commande ou faire une macro...
- https://github.com/liuchengxu/vim-which-key : un plugin qui semble pouvoir lister automatiquement les raccourcis. L'idée est excellente, mais la mise en oeuvre trop clumsy.
- https://github.com/tommcdo/vim-exchange : permet d'échanger deux textes entre eux. J'ai pas le besoin au point de vouloir m'encombrer d'un plugin supplémentaire pour ça.
- https://github.com/unblevable/quick-scope : permet d'aider les mouvements en highlightant des caractères uniques dont on pourrait vouloir se servir pour des motions. J'ai pas le besoin au point de vouloir m'encombrer d'un plugin supplémentaire pour ça.
- https://github.com/folke/trouble.nvim : lister tout ce qui ne va pas dans un fichier ; je vois pas trop le besoin...
- https://github.com/nvim-lua/lsp-status.nvim : indique les erreurs LSP dans la statusline. Ça ne m'intéresse pas plus que ça.
- https://github.com/ethanholz/nvim-lastplace : rouvrir un fichier là où on l'avait laissé. Je pressens que ça pourrait être plus confusant qu'autre chose, mieux vaut me concentrer sur le fait de naviguer efficacement dans une codebase.

## Plugins (neo)vim possiblement intéressants, à regarder

Ressource pour trouver des plugins neovim = https://github.com/rockerBOO/awesome-neovim

- https://github.com/nvim-lualine/lualine.nvim : statusline
- https://github.com/folke/which-key.nvim : ouvrir une popup pour terminer une commande que j'ai commencé à taper
- https://github.com/tanvirtin/vgit.nvim : visualiser les diffs/blame/commits git

# Telescope

**C'est quoi ?** Un plugin (exclusif à neovim) diablement efficace permettant de "choisir des items dans une liste", qui sert à plein plein de trucs (e.g. fuzzy file opener, live grep dans une codebase, recherche dans les symboles LSP, etc.)

https://github.com/nvim-telescope/telescope.nvim

## Cheatsheet

Lancement sans préciser le picker = `:Telescope` (on accède alors à une liste des pickers disponibles).

Sinon, lancement direct sur le picket :

```
:Telescope live_grep
:Telescope lsp_references
:Telescope man_pages
:Telescope help_tags
```

Une fois dans le picker, deux modes vim sont possibles :

- en mode normal, les commandes de déplacement classiques fonctionnent
- en mode insertion, on peut utiliser `Ctrl-n` / `Ctrl-p`

Aide en ligne (sur les shortcuts utilisables dans telescope) :

- `?` en mode normal
- `Ctrl-/` en mode insertion

`Ctrl-c` pour fermer telescope (plus intuitif que `Escape`, qui peut soit fermer Telescope, soit quitter le insert mode)

La doc des mapings est [ici](https://github.com/nvim-telescope/telescope.nvim#default-mappings).

Note : le texte cherché par `live_grep` est un match EXACT ; par contre il accepte les regex : `closed.*real` (du coup, les parenthèses doivent être échappées).

## Installation

Apparemment il faut préciser le tag :

```
Plug 'nvim-telescope/telescope.nvim', { 'tag': '0.1.0' }
```

De plus, il faut installer les dépendances, [fd](https://github.com/sharkdp/fd) (qui permet notamment d'ignorer les fichiers binaires ou de respecter le gitignore) et [ripgrep](https://github.com/BurntSushi/ripgrep) (sans lequel `live_grep` ne fonctionnera pas) :

```sh
sudo apt install fd-find
sudo apt install ripgrep
```

Si l'une des deux dépendances est absentes, telescope peut nous prévenir :

```
:checkhealth Telescope
10   - WARNING: fd: not found. Install sharkdp/fd for extended capabilities
```

Par ailleurs, pour `live_grep`, j'utilise un plugin ([project.nvim](https://github.com/ahmedkhalf/project.nvim)) pour définir le cwd automatiquement à la racine git, sans quoi telescope ne parcourt pas tout le projet s'il a été ouvert depuis un sous-répertoire.

Au final, le plus casse-bonbons reste encore de définir les bindings pour lancer telescope...


## Notes à l'usage

- +++ la preview de fichiers est top de chez top (e.g. `live_grep` ou recherche dans les symboles), elle aide à choisir l'item
- +++ IHM intuitive
- +++ on peut cocher plusieurs fichiers (avec Tab) pour p.ex. ouvrir 3 tabs d'un coup. EDIT : ah non, je peux bien cocher plusieurs fichiers, mais je ne sais pas comment ouvrir plusieurs tabs d'un coup... On dirait que ça sert à envoyer les fichiers dans la quickfix-list, mais je ne sais pas quoi en faire derrière... On dirait qu'il y a p.ex. moyen de faire des [fonctions custom pour tout ouvrir dans des tabs](https://vimrcfu.com/snippet/143).
- +++ plein de pickers utiles
- +++ notamment, utilisation avec LSP très très puissante
- --- tous les bindings à faire à l'installation
- --- limité à neovim
- --- uniquement compatible avec le lsp-client builtin (mais qui est très bien en fait)

Les principes de fonctionnement :

- on dirait qu'un picker, c'est une "commande telescope" ; e.g. find_files est un picker qui permet d'utiliser telescope pour trouver des fichiers
- dans l'idée, telescope permet de "choisir dans une liste" de façon efficace, fuzzy, et avec preview
- les "listes" peuvent être beaucoup, beaucoup de choses :
    - fichiers
    - commits git
    - symboles LSP
    - tags
    - buffers vim
    - etc.
- il y a plein de pickers, y compris utiles pour d'autres choses que neovim (e.g. man_pages, ou git_commits)
- les concepts semblent être :
    - picker = ce qui remplit la liste (e.g. "git_commits" va remplir la liste avec des commits git ; lsb_incoming_calls va remplir la liste avec des symboles LSP)
    - previewer = ce qui permet de preview un item de la liste (on ne preview pas de la même façon un fichier de code et un commit git, I guess)
    - sorter = ce qui permet d'ordonner les items de la liste en fonction de la chaîne fuzzy que l'utilisateur a tapée

On peut rouvrir telescope dans l'état dans lequel on l'avait laissé : `:lua require('telescope.builtin').resume()`

# LSP aka Language Server Protocol

**C'est quoi ?** : un serveur LSP analyse le code et fournit des infos dessus à un client LSP :

- `clangd` analyse le code
- après analyse, il sait que la fonction `f` est appelée à tels 3 endroits, et définie à tel autre endroit
- il communique ça à un client LSP
- le client LSP expose des commandes du genre "jumper à la définition de f".

## LSP server — python

J'ai testé `pyright` qui fait le taf :

```
pipx install pyright
```

Il ne gère pas le formattage, par contre...

Je peux formatter l'ensemble d'un fichier python avec black en ajoutant ceci aux sources null-ls (mais sans pouvoir formatter une seule ligne comme en C++) :

```
null_ls.builtins.formatting.black,
```

## LSP server — C++

J'utilise `clangd` :

```
sudo apt-install clangd-12
```

Mais ça ne suffit pas, clangd a besoin d'un `compile_commands.json` à la racine du projet :

```
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# si compile_commands.json est généré par cmake dans un répertoire de build :
ln -s build/compile_commands.json .
```

**caveat** = si les temps de recompilation (surtout pour des gros fichiers sources mal découpés, comme ceux d'ULTRA) sont assez longs, le temps de réaction du linter pourra être assez lent...

### compile_commands.json

**C'est quoi ?** Problématique = les linters C++ ont besoin de compiler le fichier en cours d'édition **exactement** comme la toolchain de mon projet le fait (e.g. en incluant les bons chemins vers les headers, ou en passant exactement les bonnes options de compilation, telles que le standard C++).

Pour cela, les toolchains peuvent générer une base de données des commandes permettant de compiler chaque fichier = `compile_commands.json`. Exemple de contenu :

```
{
   directory :  /media/DATA/git_projects/ULTRA/_build ,
   command :  /usr/bin/clang++   -I/home/myself/.conan/data/rapidjson/1.1.0/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include -I/home/myself/.conan/data/fast-cpp-csv-parser/cci.20200830/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include -I/home/myself/.conan/data/fast-cpp-csv-parser/cci.20200830/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include/fast-cpp-csv-parser -I/media/DATA/git_projects/ULTRA/Investigations  -Wall -Wextra -Werror -Wunused-parameter -Wno-unused-parameter -Wno-infinite-recursion -Wno-unused-variable -Wno-sign-compare   -O3 -DNDEBUG    -fopenmp -pipe -march=native -O3 -ffast-math -ftree-vectorize -Wfatal-errors -DNDEBUG -std=gnu++17 -o CMakeFiles/build-transfer-graph.dir/media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp.o -c /media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp ,
   file :  /media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp
},
```

Pour cmake, on peut facilement le configurer pour qu'il génère ce `compile_commands.json`, avec une ligne dans le `CMakeLists.txt` :

```
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

**Caveat 1** = les linters C++ attendent le fichier à la racine du projet ; or, il est généré par cmake dans le répertoire de build → ne pas hésiter à faire un lien symbolique vers le répertoire de build :

```
ln -s build/compile_commands.json .
```

**Caveat 2** = il faut avoir lancé au moins une fois le build pour que `compile_commands.json` existe et puisse être lu par clangd ; tant que ça n'est pas fait pas d'utilisation correcte des features clangd possible.

**Caveat 3** = la plus grosse limitation semble être qu'il gère mal les header-files, car comme ils ne sont pas compilés, il n'y a pas d'entrée les concernant dans `compile_commands.json`.

### construction de l'index clangd

**Question** : l'index de clangd est-il recalculé à chaque nouvelle session, ou réutilisé ?

**Réponse** : l'index est caché et réutilisé, cf. [la doc](https://clangd.llvm.org/design/indexing) :

> Before indexing each file, the index checks for a cached xxx.idx file on disk. After indexing, it writes this file. This avoids reindexing on startup if nothing changed since last time. These files are located in .cache/clangd/index/ next to compile_commands.json. For headers with no CDB, such as the standard library, they are in clangd/index under the user’s cache directory

Je confirme sur mon poste :

```sh
find /path/to/myproject/.cache/clangd/index -name \*.idx | wc -l
# 4276
```

## LSP client — native neovim client

En deux mots, j'ai switché sur le client LSP natif de neovim (cf. mon vimrc).

Quelques notes vrac :

Pour avoir les infos du client LSP :

```
:LspInfo
:LspLog

Éventuellement précédé de :
:lua vim.lsp.set_log_level("debug")
```

Pour appeler une feature LSP (e.g. `hover`) sans la binder à une commande vim :

```
:lua vim.lsp.buf.hover()
```

Pour avoir l'aide sur une fonction LSP :
```
:help vim.lsp.buf.rename()
```

Pour lancer une fonction LSP de façon synchrone e.g. pour attendre que le formattage soit terminé avant de rendre la main lors d'un save :

```
- Q: How do I run a request synchronously (e.g. for formatting on file save)?
  A: Use the `_sync` variant of the function provided by |lsp-buf|, if it exists.
  E.g. code formatting: " Auto-format *.rs (rust) files prior to saving them
autocmd BufWritePre *.rs lua vim.lsp.buf.formatting_sync(nil, 1000)
```

## LSP client — ALE DEPRECATED

**EDIT** : j'ai switché sur le client LSP natif de neovim, et je n'utilise donc plus ALE.

**C'est quoi ?** En gros, un plugin vim pour lancer plein de linters sur un fichier de code, et interpréter les résultats. Il joue le rôle de LSP client.

### Installation

Avec vim-plug :

```
# vimrc :
Plug 'https://github.com/dense-analysis/ale'

# au runtime :
:PlugInstall
```

Une fois ALE installé, il utilisera les linters automatiquement, à condition que ceux-ci soient installés.

Dit autrement : il suffit d'installer un linter pour qu'ALE l'utilise ; par exemple, comme mon poste dispose d'un compilateur, out-of-the-box, ALE compile le fichier en cours de frappe et m'indique les erreurs dynamiquement \o/. En revanche, je ne pourrais utiliser `ALEGoToDefinition` que si j'installe un LSP server C++ comme `clangd`.

### Features et commandes

Note = la plupart des features nécessitent un LSP server. Par exemple, en C++, même si ALE est utile sans serveur LSP (car même dans ce cas, la compilation dynamique fait qu'ALE prévient des erreurs en cours de frappe), on ne pourra utiliser `ALEGoToDefinition` que si on installe un LSP server.

- complétion intelligente avec `Ctrl+n` et `Ctrl+p` en cours de frappe (inactive par défaut)
- `ALEGoToDefinition` et `Ctrl+t` pour en revenir si besoin
    - en alternative, on peut utiliser les tags
    - (avec ma config, `F5` pour générer les tags ; `Ctrl+]` pour jumper, et toujours `Ctrl+t` pour en revenir)
- `ALEHover` pour jeter un oeil à la définition :
    - par contre ça ouvre une fenêtre plutôt qu'un balloon, c'est [un problème connu](https://github.com/dense-analysis/ale/issues/2442)
- `ALEFindReferences` pour lister les appelants d'une fonction
- `ALESymbolSearch` pour trouver un symbole dans l'index buildé par le LSP
    - la recherche est fuzzy : si le symbole est `add_costs`, il sera matché par :
        ```
        :ALESymbolSearch add_costs
        :ALESymbolSearch add_co
        :ALESymbolSearch adco
        ```
    - on dirait qu'on ne peut pas utiliser de wildcard (mais c'est inutile car la recherche est fuzzy)
    - comme ça ne recherche que dans les symboles de l'index, ça ne matche pas avec les contenus des strings, les noms des variables ou les fichiers markdown :+1:
    - c'est bien plus puissant qu'un grep classique grâce au combo "ça ne regarde que dans les symboles + la recherche est fuzzy"
- `ALEInfo`  = la commande utilisée pour débugger l'application d'un linter
    - _pour le fichier courant_, liste les linters disponibles, les linters actifs
    - logge ce qui va pas dans l'exécution des linters, et la commande utilisée pour linter :
    ```
    (executable check - failure) flake8
    (executable check - failure) pylint
    (executable check - failure) pyright-langserver
    ```
- `ALEDetail` pour avoir plus de détails sur le message de retour du linter
- `ALERename` pour renommer intelligemment un symbole (plus puissant qu'un sed car ne renomme que les symboles)
    - attention que les buffers modifiés ne semblent pas enregistrés
    - en python, ça a l'air de bien fonctionner avec pyright
    - mais en C++, ça échoue une fois sur deux avec l'erreur `No rename result received from server`
- `ALEFix` pour laisser ALE fixer du code (le plus souvent : le formatter):
    - le principe est qu'ALE hardcode des fixers avec lesquels il sait interagir
    - il faut les activer (possiblement langage par langage) avec `g:ale_fixers`
    - mais je n'ai pas vraiment réussi à l'utiliser pour C++ ou python cf. le journal
    - notamment, 1. confirmer que le fixer se lance correctement est pas très intuitif et surtout 2. il faut utiliser la config vim pour passer les options au formatter (ce qui oblige de facto à avoir une config par projet)
- pour naviguer parmi les erreurs remontées par ALE :
    ```
    :help ale-navigation-commands
    :ALENext     / :ALEPrevious
    :ALENextWrap / :ALEPreviousWrap
    :ALEFirst    / :ALELast
    ```
- `ALEToggle` pour activer/désactiver ALE, utile car ALE peut parfois être un peu intrusif

### Configuration

Le lint ne se déclenche pas tout de suite mais après un petit délai :

```
:let g:ale_lint_delay=200    <-- valeur par défaut du delay = 200 ms
```

La complétion n'est pas active par défaut ; pour l'activer :

```
let g:ale_completion_enabled = 1
```



# JOURNAL

## Vrac1

- j'installe depuis les sources (cf. les notes dédiées) en utilisant un `INSTALL_PREFIX custom = ~/neovim`
- je suis [l'aide pour transitionner de vim à neovim](https://neovim.io/doc/user/nvim.html#nvim-from-vim)
- la conf est dans :
    ```
    ~/.config/nvim/init.vim
    ```
- nvim se lance sans erreur suite à ça (mais j'ai pas encore testé toutes mes configs, notamment les plugins)
- installation de vimplug ; je suis [la doc](https://github.com/junegunn/vim-plug#neovim) :
    ```
    sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
    ```
- sous neovim, j'appelle `:PlugInstall` :
    - tout s'installe correctement
    - je vois fugitivement un message d'erreur (trop rapide pour savoir ce dont il s'agit)
    - EDIT : sur le petit PC, j'arrive à copier-coller l'erreur (qui n'a pas l'air critique) :
        ```
        Erreur détectée en traitant /home/moi/neovim/share/nvim/runtime/filetype.vim :
        ligne 2558 :
        E122: La fonction TestFiletypeFuncs existe déjà (ajoutez ! pour la remplacer)
        Appuyez sur ENTRÉE ou tapez une commande pour continuer
        ```
- je prends le temps de tester un à un tous mes plugins, ce qui ne marche pas :
    - déplacement entre les tabs avec Alt+gauche / Alt+droite
    - le plugin alternate fonctionne, mais mon raccourci avec Shift+F12 ne marche plus
    - le plugin markdown (avec les foldings) n'a l'air de marcher ni pour vim ni pour neovim : _comme le problème se pose aussi bien pour vim que pour neovim, je délègue à plus tard_
    - le plugin asciidoctor a l'air chargé, mais les foldings n'ont pas l'air de marcher (ni pour vim ni pour neovim) : _comme le problème se pose aussi bien pour vim que pour neovim, je délègue à plus tard_
- tentative de refaire marcher mes raccourcis `A-Right` pour changer de tabs :
    - je constate que les raccourcis `C-Up / C-Down` fonctionnent toujours
    - je constate que la commande entrée manuellement (`:tabn<CR>`) fonctionne aussi
    - la commande suivante fonctionne : `:noremap k :echo 'pouet'<CR>`
    - mais la commande suivante ne fonctionne pas : `:noremap <A-Right> :echo 'pouet'<CR>`
    - on dirait qu'aucun de mes mappings avec Alt ne fonctionne... (mais leurs équivalents avec Shift ou Control fonctionnent)
    - j'investigue plusieurs pages d'aide sans succès : [lien1](https://github.com/neovim/neovim/issues/5792), [lien2](https://www.reddit.com/r/neovim/comments/aovgfm/how_to_use_alt_with_arrow_keys_on_macos/)
    - AH ! J'avance : ma config normale `<A-Right>` fonctionne correctement en dehors de tmux, c'est sous tmux qu'elle ne marche pas...
    - Résolu mon problème ha ha : en fait, j'avais ma conf tmux codé un truc custom pour permettre l'utilisation de Alt avec vim, et qu'il fallait que je l'adapte à neovim (c'est chose faite).
- résolu mon problème avec `Shift+F12` :
    - https://github.com/neovim/neovim/issues/7384
    - contourné : si je veux mapper `Shift+Fxx`, il faut en fait que je mappe `Fyy` où `yy = xx + 12` :
        ```
        if has("nvim")
            noremap <F24>    :execute ':A'<CR>
        else
            noremap <S-F12>  :execute ':A'<CR>
        endif
        ```
- également installé sans souci neovim stable sur mon petit PC \o/
- résolu mon warning d'incompatibilité gcc/clang en mettant à jour clangd depuis la version 10 vers la version 12 :
    - PROBLÈME = j'avais des warnings sur des options qui sont propres à gcc :
        ```
        [clang] Unknown warning option '-Wuseless-cast'
        ```
    - (note que ces warnings ne semblent pas empêcher les features de fonctionner e.g. détecter les erreurs de compilation à la volée)
    - solution = mise à jour de ma version de clangd sur mon ubuntu 20.04 :
        ```
        # Quelle est ma version de clangd :
        clangd --version
        # clangd version 10.0.0-4ubuntu1

        # Installation de clangd-12 pour résoudre mon problème :
        sudo apt install clangd-12

        # Mais par défaut, la version utilisée reste clangd10 :
        clangd --version
        # clangd version 10.0.0-4ubuntu1

        # Pour corriger :
        sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-12 100
        clangd --version
        # Ubuntu clangd version 12.0.0-3ubuntu1~20.04.5
        ```
- comme je continue malgré moi d'utiliser vim, j'ai ajouté un alias `vim` vers `nvim -p`...
- Mais pour corriger cette habitude petit à petit, j'ai plutôt fait :
    ```
    alias vim="echo '' ; echo 'UTILISER PLUTÔT NVIM !!!' ; echo '' ; sleep 1 ; nvim -p"
    ```
- comme neovim contient naturellement treesitter, regardé si je pouvais me passer de tagbar ; [on ne dirait pas](https://github.com/preservim/tagbar/issues/751)
- tentative (infructueuse, pour le moment) d'utiliser les fixers ALE pour du code C++ ou python :
    - mon objectif = lancer `clang-format` sur du code C++ ou `black` sur du code python
    - la config pour ça :
        ```
        let g:ale_fixers = {
                    \    'cpp': ['clang-format'],
                    \    'python': ['black'],
                    \}
        ```
    - déjà, je galère à être certain que les fixers sont bien lancés :
        - `ALEInfo` n'indique pas les enabled fixers (alors qu'il le fait pour les linters)
        - même correctement configuré, pour voir le linter un tant soit peu apparaître dans `ALEInfo`, il faut enchaîner `ALEFix` puis `ALEInfo`
        - en revanche, je peux voir dans `ALEInfo` le contenu de la variable `g:ale_fixers` (pour confirmer au moins que ma conf arrive correctement jusqu'à ALE)
    - problème python = black n'utilise pas les options de mon pre-commit (c'est assez logique, où irait-il les trouver) ; aiguillé par [cette issue](https://github.com/dense-analysis/ale/issues/659), j'ai vérifié qu'un contournement possible était : `let g:ale_python_black_options = '--line-length=120'`
    - problème C++ = c'est `clang-format-11` que j'utilise (alors qu'ALE lance `clang-format`) ; contournement possible en faisant un alias : `alias clang-format=clang-format-11`
    - problème C++ = la commande `clang-format` semble échouer, pas clair pourquoi :
        ```
        (finished - exit code 127) ['/usr/bin/zsh', '-c', '''clang-format'' --assume-filename=''src/path/to/pouet.cpp'' < ''/tmp/nvimtGbOgQ/15/pouet.cpp''']
        ```
    - [cette issue](https://github.com/dense-analysis/ale/issues/1960) est peut-être liée ? Elle dit que le fichier sur lequel travaille `clang-format` disparaît peut-être avant de pouvoir être lu par clang-format... (ça m'étonnerait parce qu'en cas de fichier inexistant, 1. clang-format produit une erreur sur stdout que ALE devrait remonter et 2. l'exit-code semble plutôt être 1 que 127)
    - conclusion python = tant que je ne peux pas facilement définir de vimrc locaux aux projets, utiliser `ALEFix` paraît une mauvaise idée
    - conclusion C++ = tant que j'aurais pas débuggé cette histoire de `clang-format`, `ALEFix` ne marche pas

## Utilisation de telescope avec nvim

- Installation de telescope, apparemment il faut préciser le tag :
    ```
    Plug 'nvim-telescope/telescope.nvim', { 'tag': '0.1.0' }
    ```
- Problème telescope = il n'utilise pas les features LSP avec ALE, car il ne semble capable de s'interfacer qu'avec le client LSP builtin avec neovim
- Du coup, j'utilise le client LSP builtin en remplacement d'ALE
    - corrigé un problème de coloration de certaines erreurs (invisibles avec mon thème) : contourné en overwrittant la couleur par défaut utilisée par le builtin LSP pour ces erreurs invisibles
    - problème = pas de flake8 dispo dans le builtin LSP, et pas de flake8 avec pyright...
    - je tente le contournement d'utiliser jedi, mais il n'a pas l'air de détecter les erreurs (pourtant, LspInfo semble indiquer qu'il tourne correctement...)
- Du coup, je contourne en utilisant [null-lsp](https://github.com/jose-elias-alvarez/null-ls.nvim) (aka null-ls) = un outil qui joue le rôle de LSPServer pour plein de tools non-intégrés aux LSP-servers, dont flake8 ou black ; ça fonctionne, j'ai bien flake8 d'actif
- on dirait que les linters de LSP natif se lancent correctement
- reste à essayer mes bindings...
- ok pour la plupart des bindings
- je modifie la conf pour ouvrir les jumps LSP dans des nouveaux tabs
- Si je me résume : j'ai un autre client LSP qu'ALE, j'ai un contournement pour lancer flake8 aussi, et j'ai remis la plupart de mes bindings
- le seul que j'ai pas encore remis (car je vais plutôt utiliser Telescope pour ça) = `ALESymbolSearch`
- Du coup, je supprime l'installation d'ALE et les bindings que j'avais fait
- La commande de formatting semble fonctionner avec clangd, mais pas avec python :
    - QUESTION = d'où clangd tire-t-il ses infos de formatting ? Est-il capable de lire mon `.clang-format` ?
    - en faisant des tests, je confirme que clangd arrive à lire le fichier `.clang-format` à la racine du projet :+1:
    - c'est le cas même si je lance nvim depuis un sous-répertoire (différent du répertoire contenant `.clang-format`)
- Ceci permettrait d'avoir le formatting avec black : https://github.com/neovim/nvim-lspconfig/issues/903#issuecomment-843820972
- Sinon, ceci est intéressant : https://www.reddit.com/r/neovim/comments/gm4ir3/comment/hj9zvh9/
    ```
    :'<,'>lua vim.lsp.buf.range_formatting()
    ```
- Et [ici](https://vi.stackexchange.com/questions/36946/keymap-for-lsp-code-formatting-in-visual-mode), je trouve une astuce pour que le formattage fonctionne en visuel (qui m'indique également comment utiliser des anciens bindings : avec une simple string) :
    ```
    vim.keymap.set('v', '=', '<ESC><cmd>lua vim.lsp.buf.range_formatting()<CR>', bufopts)
    ```
- `LspInfo` + `LspStop` pour arrêter un client (mais ça marche pas de ouf en python, à cause de null-lsp)

## Installation sur le vieux PC

Ce journal correspond à mon travail sur le petit PC pour avoir une utilisation de neovim cohérente avec mes autres PC.

- installation clangd :
    - À la base, je veux installer un clangd récent pour utilisation comme serveur LSP.
    - Comme il n'y a pas de clangd suffisamment récent dans les dépôts APT, je compile clangd moi-même :
    - Comilation clangd ([lien](https://github.com/llvm/llvm-project/tree/main/clang-tools-extra/clangd#building-and-testing-clangd)) :
        ```
        cd /tmp
        git clone --depth=1 https://github.com/llvm/llvm-project.git
        cd llvm-project
        export LLVM_ROOT=/tmp/llvm-project
        mkdir build
        cd build
        cmake $LLVM_ROOT/llvm/ -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra"
        ```
    - Comme j'ai une erreur de compilateur trop vieux, j'essayer d'installer un clang récent :
        ```
        apt-cache search clang
        sudo apt install clang-8
        type clang
        clang --version
        type clang-8
        sudo update-alternatives --install /usr/bin/cc cc /usr/bin/clang-8 100
        sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang-8 100
        sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++-8 100
        sudo update-alternatives --config c++
        c++ --version
        ```
    - Malheureusement, ça ne suffit pas :
        ```
        CMake Error at cmake/modules/CheckCompilerVersion.cmake:114 (message):
          libstdc++ version should be at least 7 because LLVM will soon use new C++ features which your toolchain version doesn't support.  You can temporarily
          opt out using LLVM_TEMPORARILY_ALLOW_OLD_TOOLCHAIN, but very soon your toolchain won't be supported.
        ```
    - Du coup j'autorise explicitement l'utilisation d'un ancien compilo :
        ```
        pyenv shell 3.9.12  # nécessaire pour la compilation, je l'avais déjà sur le petit PC
        cmake $LLVM_ROOT/llvm/ -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" -DLLVM_TEMPORARILY_ALLOW_OLD_TOOLCHAIN=1
        cmake --build $LLVM_ROOT/build --target clangd
        ```
    - (NOTE : la compilation a pris plusieurs heures)
- Je poursuis avec l'installation de mes plugins vim sur le petit PC :
    - je commence par désactiver les LSP server (clangd est en train de compiler + on verra plus tard pour eux)
    - un plugin semble poser problème = fugitive, c'est sans doute à cause d'une version de git trop ancienne :
        ```
        git --version
        # git version 2.7.4
        ```
    - sur le PC wework, j'ai :
        ```
        git --version
        # git version 2.25.1
        ```
    - ça me conduit à vouloir compiler une version récente de git...
    - (et tant qu'à faire ça, autant compiler également une version récente de tmux, faut que je prévoie de faire ça...)
- Compilation d'une version récente de git ([lien](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)) :
    ```
    sudo apt install dh-autoreconf libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev

    cd /tmp
    wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.37.1.tar.gz
    tar -xvzf git-2.37.1.tar.gz
    cd git-2.37.1
    make configure
    ./configure --prefix=/usr
    make all doc info  # la cible info échoue, sans doute car j'ai pas téléchargé ses dépendances (je me suis limité au minimum)
    make all doc
    sudo make install install-doc

    git --version
    # git version 2.37.1
    ```
    - Après un `PlugUpdate`, je n'ai plus de souci git \o/
- J'ai également des soucis avec d'autres plugins :
    ```
    x null-ls.nvim:
        fatal: référence invalide : master
    x nvim-comment:
        fatal: référence invalide : master
    ```
    - Après analyse, je confirme que null-ls utilise la branche "main"
    - À froid, je pense qu'il faut que j'update vim-plug : `:PlugUpgrade`
    - je confirme que je n'ai plus le souci après un `:PlugUpgrade` :-)
- Point d'étape so far :
    - ma conf neovim fonctionne sur le petit PC, à l'exception des LSP que je n'ai pas encore essayés
    - j'ai compilé une version récente de git manuellement
    - j'ai installé une version "récente" de clang++ (la 8) comme compilo par défaut du système
    - je suis en train de compiler clangd
- Allez, je tente un LSP = pyright :
    ```
    pipx install pyright
    ```
    - Bon ben pyright s'installe bien, mais nvim n'arrive pas à l'utiliser sur le petit PC (sans doute des versions trop anciennes de js/npm) :
        ```
        [START][2022-07-23 10:29:26] LSP logging initiated
        [START][2022-07-23 10:30:44] LSP logging initiated
        [START][2022-07-23 10:30:56] LSP logging initiated
        [START][2022-07-23 11:44:36] LSP logging initiated
        [START][2022-07-23 11:44:42] LSP logging initiated
        [START][2022-07-23 11:45:18] LSP logging initiated
        [START][2022-07-23 11:46:10] LSP logging initiated
        [START][2022-07-23 11:49:18] LSP logging initiated
        [START][2022-07-23 11:49:32] LSP logging initiated
        [START][2022-07-23 11:50:13] LSP logging initiated
        [START][2022-07-23 11:50:24] LSP logging initiated
        [START][2022-07-23 11:48:24] LSP logging initiated
        [START][2022-07-23 11:54:39] LSP logging initiated
        [START][2022-07-23 11:55:10] LSP logging initiated
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"WARN"
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	" "
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"engine"
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	' pyright@1.1.263: wanted: {"node":">=12.0.0"} (current: {"node":"4.2.6","npm":"3.5.2"})\n'
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"WARN"
        [ERROR][2022-07-23 11:55:16] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	' engine pyright@1.1.263: wanted: {"node":">=12.0.0"} (current: {"node":"4.2.6","npm":"3.5.2"})\n'
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"npm"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	" "
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"WARN "
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"enoent"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	" ENOENT: no such file or directory, open '/tmp/pyright-python-langserver.myself/package.json'\n"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"npm "
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"WARN pyright-python-langserver.myself No description\nnpm WARN"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	" pyright-python-langserver.myself No repository field.\n"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"npm WARN pyright-python-langserver.myself No README data\n"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	"npm WARN"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	" pyright-python-langserver.myself No license field.\n"
        [ERROR][2022-07-23 11:55:24] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	'/tmp/pyright-python-langserver.myself/node_modules/pyright/dist/pyright-langserver.js:1\n(function (exports, require, module, __filename, __dirname) { (()=>{"use strict";var e,r,t,o={5822:(e,r,t)=>{(0,t(6334).main)()},9491:e=>{e.exports=require("assert")},2081:e=>{e.exports=require("child_process")},6113:e=>{e.exports=require("crypto")},2361:e=>{e.exports=require("events")},7147:e=>{e.exports=require("fs")},1808:e=>{e.exports=require("net")},2037:e=>{e.exports=require("os")},1017:e=>{e.exports=require("path")},2781:e=>{e.exports=require("stream")},7310:e=>{e.exports=require("url")},3837:e=>{e.exports=require("util")},4655:e=>{e.exports=require("v8")},1267:e=>{e.exports=require("worker_threads")},9796:e=>{e.exports=require("zlib")}},i={};function s(e){var r=i[e];if(void 0!==r)return r.exports;var t=i[e]={id:e,loaded:!1,exports:{}};return o[e].call(t.exports,t,t.exports,s),t.loaded=!0,t.exports}s.m=o,s.x=()=>{var e=s.O(void 0,[736,194],(()=>s(5822)));return s.O(e)},e=[],s.O=(r,t,o,i)=>{if(!t){var u=1/0;for(l=\n\nSyntaxError: Unexpected token [\n    at exports.runInThisContext (vm.js:53:16)\n    at Module._compile (module.js:374:25)\n    at Object.Module._extensions..js (module.js:417:10)\n    at Module.load (module.js:344:32)\n    at Function.Module._load (module.js:301:12)\n    at Module.require (module.js:354:17)\n    at require (internal/module.js:12:17)\n    at Object.<anonymous> (/tmp/pyright-python-langserver.myself/node_modules/pyright/langserver.index.js:8:1)\n    at Module._compile (module.js:410:26)\n    at Object.Module._extensions..js (module.js:417:10)\n'
        [INFO][2022-07-23 11:58:25] .../vim/lsp/rpc.lua:261	"Starting RPC client"	{  args = { "--stdio" },  cmd = "pyright-langserver",  extra = {    cwd = "/media/truecrypt1/myproject"  }}
        [ERROR][2022-07-23 11:58:26] .../vim/lsp/rpc.lua:420	"rpc"	"pyright-langserver"	"stderr"	'/tmp/pyright-python-langserver.myself/node_modules/pyright/dist/pyright-langserver.js:1\n(function (exports, require, module, __filename, __dirname) { (()=>{"use strict";var e,r,t,o={5822:(e,r,t)=>{(0,t(6334).main)()},9491:e=>{e.exports=require("assert")},2081:e=>{e.exports=require("child_process")},6113:e=>{e.exports=require("crypto")},2361:e=>{e.exports=require("events")},7147:e=>{e.exports=require("fs")},1808:e=>{e.exports=require("net")},2037:e=>{e.exports=require("os")},1017:e=>{e.exports=require("path")},2781:e=>{e.exports=require("stream")},7310:e=>{e.exports=require("url")},3837:e=>{e.exports=require("util")},4655:e=>{e.exports=require("v8")},1267:e=>{e.exports=require("worker_threads")},9796:e=>{e.exports=require("zlib")}},i={};function s(e){var r=i[e];if(void 0!==r)return r.exports;var t=i[e]={id:e,loaded:!1,exports:{}};return o[e].call(t.exports,t,t.exports,s),t.loaded=!0,t.exports}s.m=o,s.x=()=>{var e=s.O(void 0,[736,194],(()=>s(5822)));return s.O(e)},e=[],s.O=(r,t,o,i)=>{if(!t){var u=1/0;for(l=\n\nSyntaxError: Unexpected token [\n    at exports.runInThisContext (vm.js:53:16)\n    at Module._compile (module.js:374:25)\n    at Object.Module._extensions..js (module.js:417:10)\n    at Module.load (module.js:344:32)\n    at Function.Module._load (module.js:301:12)\n    at Module.require (module.js:354:17)\n    at require (internal/module.js:12:17)\n    at Object.<anonymous> (/tmp/pyright-python-langserver.myself/node_modules/pyright/langserver.index.js:8:1)\n    at Module._compile (module.js:410:26)\n    at Object.Module._extensions..js (module.js:417:10)\n'
        ```
    - Le simple lancement de pyright dans un répertoire vide dans mon shell échoue... (alors qu'il passe bien sur le PC WeWork)

## Vrac2

- je finis de tester + configurer des bindings pour telescope :
    - ouvrir un fichier (du coup je dégage le plugin Ctrl+p, mais je garde le raccourci)
    - live grep
    - rechercher dans les symboles LSP
- TELESCOPE> Pas de chance, le raccourci vscode pour "search symbols" est Ctrl-t , déjà utilisé pour vim ; du coup, j'essaye de le configurer à Ctrl+Alt+t
- Problème = il a l'air de ne PAS utiliser le gitignore (et il trouve des NOGIT)
- Problème = j'aimerais qu'il ne matche pas les fichiers binaires
- Solution aux deux = installer `fd` :
    ```
    :checkhealth Telescope
    10   - WARNING: fd: not found. Install sharkdp/fd for extended capabilities
    ```
- Installation de fd ([lien](https://github.com/sharkdp/fd)) :
    ```
    sudo apt install fd-find
    ```
- Quel raccourci vscode pour le live_grep dans les fichiers ?
    - `https://code.visualstudio.com/docs/editor/codebasics#_search-across-files`
    - Ctrl+Shift+F pour rechercher dans tous les fichiers du répertoire courant
    - je ne peux pas reprendre le raccourci (ni Ctrl+Alt+F que j'utilise également pour ouvrir un browser)
    - du coup, je prends `Ctrl+Alt+g` (comme [G]rep)
- je tente comme je peux de permettre les foldings avec la syntaxe markdown, mais comme je n'y arrive pas, je finis par désactiver markdown pour polyglot, au profit du plugin vim-markdown-folding
    - il y a bien une option native pour fold le markdown `let g:markdown_folding = 1` mais elle est inutilisable car elle interprète mal les commentaires dans les code-blocks...
- toggle des diagnostics LSP :
    - à la base, le problème et que null-ls ne répond pas à `:LspStop` / `:LspStart`
    - il existe bien ceci, mais c'est pas pratique :
        ```
        :lua require("null-ls").toggle({})
        ```
    - je me débrouille pour pouvoir toggle les diagnostics LSP, avec une fonction custom inspirée de [ce lien](https://www.reddit.com/r/neovim/comments/ng0dj0/lsp_diagnostics_query_is_there_an_way_to_toggle/)
- j'utilise un plugin ([project.nvim](https://github.com/ahmedkhalf/project.nvim) pour définir le cwd automatiquement à git (sans quoi telescope ne parcourt pas tout le projet)
- lien éclusé = [lunarvim](https://www.lunarvim.org/), pas vraiment un plugin mais un outil buildé sur neovim (pour transformer vim en IDE)

## Vrac3

J'abandonne le fait que mon workflow soit 100% utilisable sur le petit PC portable, qui est vraiment trop vieux pour que le jeu en vaille la chandelle...

Notamment, j'abandonne le fait d'installer pyright.

# FUTUR

- parcourir toutes les [options qui sont actives par défaut avec neovim](https://neovim.io/doc/user/vim_diff.html#vim-differences) pour les encadrer dans ma conf vim d'un `if has("nvim")`
- le chargement avec nvim -p Nx.otl est beaucoup plus long qu'avec vim...
    - plus de 2 fois plus long !
    - c'est visible même sur un seul fichier otl
    - EDIT : aha, j'avais oublié de builder en mode `Release`, j'ai donc un neovim de debug ! Hum... non : après recompilation en mode release, j'ai réduit l'écart, mais nvim -p est toujours sensiblement plus long que vim... (moins de 2 fois plus long, though)
- plus surprenant, le fait de QUITTER neovim est beaucoup plus long qu'avec vim
- j'aimerais visualiser quand l'index clangd est en cours de build vs. quand il a fini de calculer...
    - en effet, j'aimerais savoir que s'il lagge, c'est que l'index n'a pas fini d'être construit (OU que s'il ne réagit pas, c'est qu'il n'a rien trouvé)
    - possiblement, utilisable avec [cette référence](https://github.com/dense-analysis/ale/pull/1203)
- on dirait qu'ALE utilise la loclist → me renseigner là-dessus ?
    - `:help location-list`
    - https://medium.com/@lakshmankumar12/quickfix-and-location-list-in-vim-ca0292ac894d
    - on dirait que c'est comme la quickfix-list, mais local au fichier de la fenêtre → c'est plutôt ça que je veux !
        ```
        :lopen
        :lcl
        :lnext
        :lprev
        ```
- réussir à utiliser `ALEFix` avec `clang-format` pour le C++ et `black` pour python (cf. mon journal)
- profiler le lancement de zsh, qui est assez lent : https://stevenvanbael.com/profiling-zsh-startup (EDIT : c'est en partie dû à nvm)
- quand le besoin reviendra, essayer de trouver une solution pour avoir une config (neo)vim propre à un projet ([exemple ?](https://github.com/neovim/nvim-lspconfig/wiki/Project-local-settings))
- essayer d'ajouter clang-tidy comme linter ALE ?
- Trouver des serveurs LSP pour des langages additionnels ?
    - Shell
    - Cmake
    - JavaScript
    - HTML
    - CSS
- ALE : [ce post](https://github.com/liuchengxu/vista.vim) montre comment intégrer à la barre de status le nombre d'erreurs/warnings (si je dis pas de bêtises, c'est également le cas du readme d'ALE)
- TMUX : éventuellement, installer un tmux récent sur mon vieux PC portable ?
    - sur mon PC fixe :
        ```
        tmux -Version
        tmux 3.0a
        ```
    - https://github.com/tmux/tmux/releases/tag/3.0a
    - https://bogdanvlviv.com/posts/tmux/how-to-install-the-latest-tmux-on-ubuntu-16_04.html
    - https://jdhao.github.io/2018/10/16/tmux_build_without_root_priviledge/
- neovim/LSP : quand j'utilise l'omnicompletion avec lsp, ça m'ouvre une fenêtre en bas de mon écran, qu'il faut que je referme manuellement derrière...
    - pour corriger, il faudra que je comprenne mieux la complétion sous vim
    - notamment, quelle différence entre :
        - taper directement `Ctrl+n` (actuellement, ça trigge la complétion "dumb")
        - taper `Ctrl+x` puis `Ctrl+o` (actuellement, ça trigge la complétion "smart")
- dev C++/python : pouvoir compiler/exécuter/débugger via vim ? (est-ce vraiment une feature pertinente ? le seul intérêt serait de jumper directement aux localisations des erreurs et breakpoints dans le code)
- Divers liens à écluser :
    - [Vim for Python in 2020 | Vim From Scratch](https://www.vimfromscratch.com/articles/vim-for-python) : Contient quelques plugins à essayer qui ont l'air intéressants
    - [Your ultimate VIM setup for Python](https://casas-alejandro.medium.com/your-ultimate-vim-setup-for-python-b43a522b1152) : Contient quelques plugins à essayer qui ont l'air intéressants + une config
    - https://www.incredibuild.com/blog/vim-c-there-is-such-a-thing-tricks-to-use-vim-in-c : Pas fou... Quelques plugins à regarder though
    - https://dane-bulat.medium.com/vim-setting-up-a-build-system-and-code-completion-for-c-and-c-eb263c0a19a1 : Pas inintéressant car mixe vim et cmake très concrètement, mais rien de nouveau pour vim
    - https://hackingcpp.com/dev/vim_plugins.html : Plein plein plein de plugins qui ont l'air cool !
    - [https://idie.ru/posts/vim-modern-cpp/](lien1) + [lien2](https://chmanie.com/post/2020/07/17/modern-c-development-in-neovim/) : pour développer en C++ avec neovim
    - liens sur le profiling de vim : [lien1](https://thoughtbot.com/blog/profiling-vim), [lien2](https://stackoverflow.com/questions/12213597/how-to-see-which-plugins-are-making-vim-slow)
    - [lien](https://superuser.com/questions/77800/vims-autocomplete-how-to-prevent-vim-to-read-some-include-files) pour avoir un vim fonctionnel avec le C++, mieux gérer les includes dans l'autocomplétion :
    - [cet article](https://medium.com/@lakshmankumar12/c-source-code-browsing-in-vim-d0afee82b688) donne quelques outils intéressants pour le C++ : [lien1](https://github.com/deoplete-plugins/deoplete-clang), [lien2](https://github.com/Andersbakken/rtags)
    - Voir également [ce post](https://www.danielfranklin.id.au/extending-a-shared-vim-configuration/) sur le fait d'étendre une config vim partagée (entre plusieurs machines)
    - Autres références à lire : [lien1](https://realpython.com/vim-and-python-a-match-made-in-heaven/), [lien2](https://www.vimfromscratch.com/articles/vim-and-language-server-protocol/)
    - sur [une vidéo cmake et ses dérivées](https://youtu.be/Y_UubM5eYAM), j'ai choppé quelques trucs cools :
        - Au moins dans ses vidéos (donc avec neovim + coc), il y a des messages indiquant que l'indexation est en cours... ce serait top d'avoir la même chose avec le client LSP natif
        - Google test + intégration avec vim
    - (N)VIM = Regarder d'un peu plus près les splits vim (utiles pour garder un morceau de code sous le coude) : [lien](https://thoughtbot.com/blog/vim-splits-move-faster-and-more-naturally)
    - https://stackoverflow.com/questions/24232354/vim-set-color-for-listchars-tabs-and-spaces
- Splitter mon usage de neovim en deux :
    - éditeur de texte (la config vim doit fonctionner avec neovim AINSI QU'AVEC vim legacy)
    - IDE (je n'utiliserai que neovim, et m'autorise donc à ce que la config ne soit pas compatible avec vim legacy)
- Du coup, discriminer mes plugins (neo)vim :
    - les plugins essentiels à l'édition de texte (e.g. NERDTree ?)
    - les plugins essentiels au dev (e.g. LSP, telescope)
    - les plugins non-essentiels
- python : utiliser [isort](https://github.com/PyCQA/isort) pour les imports.
- TELESCOPE> lsp_references est top (bien mieux que la loclist) par contre par défaut l'affichage du nom de fichier est trop petit (comme il est mangé par la répétition de la ligne greppée, que j'ai de toutes façons via la preview, je peux customizer la config pour faire sauter le panel central...)
- TELESCOPE> question : comment tuner la recherche ?
    - toggle une recherche exacte/fuzzy
    - toggle case sensitive ou non
    - limiter la recherche à mon workspace (et ne pas recherche dans /usr/include/boost, p.ex.)
- TELESCOPE> une feature qui me manque = Ctrl-D pour matcher avec le nom du fichier uniquement (sans son chemin)
- TELESCOPE> est-ce que j'ai moyen de limiter le grep/find_files au répertoire local ?
- TELESCOPE> comment ne PAS avoir les symboles de /usr/include comme boost renvoyés par LSP ?
- TELESCOPE> Le dynamic_workspace_symbol de telescope n'est pas très pratique car dans le panel de gauche, les noms des fichiers sont tronqués par le nom du symbole...
    - à la limite, comme le match est fuzzy, le nom du symbole reste utile
    - mais peut-être que son type non ?
    - et idéalement, quand le chemin du fichier doit être tronqué, mieux que le CHEMIN (et non le nom du fichier) soit tronqué
- TELESCOPE> jeter un oeil aux extensions ([lien](https://github.com/nvim-telescope/telescope.nvim/wiki/Extensions)). Par exemple = une extension pour visualiser les variables d'environnement = https://github.com/LinArcX/telescope-env.nvim
- LSP> voir ce que permettent les code actions ? Notamment, utiliser [ce plugin](https://github.com/kosayoda/nvim-lightbulb) pour les rendre visible ? Exemple de code-action :
    - ajouter automatiquement un import manquant
    - renommer automatiquement une fonction pour être cohérent avec la convention de nommage
    - ajouter automatiquement de la doc
    - créer des accesseurs
    - plus généralement : fixer une erreur (en fait, les code-actions sont peut-être l'équivalent de :ALEFix)
    - à noter qu'en décembre 2021, pyright ne supportait rien d'autre que le tri des imports ([lien](https://stackoverflow.com/questions/70162438/neovim-built-in-lsp-shows-no-code-actions-available-for-python-files/70192996#70192996))
- LSP> permettre le formattage de code python avec black (quitte à hard-coder la line-length à 120) ?
- LSP> être capable d'arrêter les LSP null-ls avec `LspStop` (LspStop arrête correctement pyright et clangd, mais pas null-ls). cf. https://github.com/jose-elias-alvarez/null-ls.nvim/issues/896  :
    ```
    :lua vim.diagnostic.show()
    :lua vim.diagnostic.hide()
    (à noter que ça n'arrête pas vraiment les LSP)
    ```
- LSP> utiliser d'autres linters avec null-ls ; la liste des tools gérés par null-ls est [ici](https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md) :
    - [markdownlint](https://github.com/DavidAnson/markdownlint) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#markdownlint
    - [pydocstyle](http://www.pydocstyle.org/en/stable/) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#pydocstyle
    - [shellcheck](https://www.shellcheck.net/) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#shellcheck-1
    - [vulture](https://github.com/jendrikseipp/vulture) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#vulture
    - [autopep8](https://github.com/hhatto/autopep8) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#autopep8
    - [beautysh](https://github.com/lovesegfault/beautysh) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#beautysh
    - [black](https://github.com/psf/black) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#black
    - [clang-format](https://www.kernel.org/doc/html/latest/process/clang-format.html) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#clang_format
    - [cmake-format](https://github.com/cheshirekow/cmake_format) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#cmake_format
    - [isort](https://github.com/PyCQA/isort) = https://github.com/jose-elias-alvarez/null-ls.nvim/blob/main/doc/BUILTINS.md#isort
- LSP> trouver comment mapper les commandes neovim LSP à l'ouverture de tabs plutôt que de buffers ? (peut-être avec telescope plutôt ?) Les références LSP sont ouvertes dans des buffers. Pour les ouvrir dans des newtabs :
    ```
    :set switchbuf+=usetab,newtab
    ```
- LSP> besoin qui pourrait être utile = ouvrir la définition/implémentation/déclaration dans un split vertical (ce serait l'équivalent du "peek" de vscode)
- LSP> [ceci](https://github.com/neovim/nvim-lspconfig/issues/903#issuecomment-843820972) permettrait d'utiliser directement black pour le formatting LSP
- LSP> sinon, [ceci](https://www.reddit.com/r/neovim/comments/gm4ir3/comment/hj9zvh9/) est intéressant pour ne formatter que partiellement (plutôt que tout le fichier) :
    ```
    :'<,'>lua vim.lsp.buf.range_formatting()
    ```
