Notes sur mon workflow de dev

* [neovim](#neovim)
   * [Notes d'installation](#notes-dinstallation)
   * [Plugin manager](#plugin-manager)
   * [Configuration lua](#configuration-lua)
* [Telescope](#telescope)
   * [Cheatsheet](#cheatsheet)
   * [Installation](#installation)
* [LSP aka Language Server Protocol](#lsp-aka-language-server-protocol)
   * [LSP server — python](#lsp-server--python)
   * [LSP server — C++](#lsp-server--c)
      * [compile_commands.json](#compile_commandsjson)
      * [construction de l'index clangd](#construction-de-lindex-clangd)
   * [LSP client — native neovim client](#lsp-client--native-neovim-client)
* [Debugging cpp](#debugging-cpp)
   * [Les DAP](#les-dap)
   * [Avec vimspector](#avec-vimspector)
      * [Installation et dépendance à python](#installation-et-dépendance-à-python)
      * [Gadgets](#gadgets)
      * [Fichier de config](#fichier-de-config)
      * [Choix du programme à débugger](#choix-du-programme-à-débugger)
      * [Notes à l'usage](#notes-à-lusage)
      * [Playground](#playground)
   * [Avec nvim-dap et nvim-dap-ui](#avec-nvim-dap-et-nvim-dap-ui)
* [rust](#rust)
   * [Utilisation avec neovim](#utilisation-avec-neovim)
* [pyenv](#pyenv)
* [javascript et typescript](#javascript-et-typescript)
* [git](#git)
   * [diff-so-fancy](#diff-so-fancy)
* [just](#just)
   * [C'est quoi](#cest-quoi)
   * [Installation](#installation-1)
   * [Notes vrac d'utilisation](#notes-vrac-dutilisation)
* [NerdFonts](#nerdfonts)
   * [Que fait NerdFont](#que-fait-nerdfont)
   * [Quelle police mon terminal utilise-t-il ?](#quelle-police-mon-terminal-utilise-t-il-)
   * [Installation d'une NerdFont](#installation-dune-nerdfont)
* [JOURNAL](#journal)
   * [Vrac1](#vrac1)
   * [Utilisation de telescope avec nvim](#utilisation-de-telescope-avec-nvim)
   * [Installation sur le vieux PC](#installation-sur-le-vieux-pc)
   * [Vrac2](#vrac2)
   * [Vrac3](#vrac3)
   * [Nouveau problème avec clangd = il ne trouve pas libc++](#nouveau-problème-avec-clangd--il-ne-trouve-pas-libc)
   * [Nouveau problème avec clangd = le standard utilisé n'est pas le bon](#nouveau-problème-avec-clangd--le-standard-utilisé-nest-pas-le-bon)
      * [Manifestation de mon problème](#manifestation-de-mon-problème)
      * [TL;DR](#tldr)
      * [Analyse](#analyse)
         * [STEP 1 = en première approche, les logs LSP montrent que clangd ne trouve pas de compile_commands.json](#step-1--en-première-approche-les-logs-lsp-montrent-que-clangd-ne-trouve-pas-de-compile_commandsjson)
         * [STEP 2 = même avec un compile_commands.json ET le standrad en C++17 dans le CMakeLists.txt, clangd continue d'utiliser le standard C++14](#step-2--même-avec-un-compile_commandsjson-et-le-standrad-en-c17-dans-le-cmakeliststxt-clangd-continue-dutiliser-le-standard-c14)
         * [STEP 3 = je force cmake à préciser explicitement le standard dans compile_commands.json](#step-3--je-force-cmake-à-préciser-explicitement-le-standard-dans-compile_commandsjson)
      * [Questions](#questions)
* [FUTUR](#futur)


# neovim

## Notes d'installation

- je choisis d'installer [en compilant depuis les sources](https://github.com/neovim/neovim/wiki/Installing-Neovim#install-from-source), il y a [une doc qui explique tout ça](https://github.com/neovim/neovim/wiki/Building-Neovim).
- je n'oublie pas d'installer les dépendances (la commande ci-dessous n'est indiquée qu'à titre indicatif, mieux vaut se référer à [la doc](https://github.com/neovim/neovim/wiki/Building-Neovim#build-prerequisites)) :
    ```
    sudo apt install ninja-build gettext cmake unzip curl
    ```
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

## Plugin manager

En novembre 2023, je passe à [lazy.nvim](https://github.com/folke/lazy.nvim) ; [ce post reddit](https://www.reddit.com/r/neovim/comments/w0ijm3/should_i_use_a_plugin_manager/) de juillet 2022 recommande d'utiliser [packer](https://github.com/wbthomason/packer.nvim), mais à date packer, recommande d'utiliser lazy :

> lazy.nvim: Most stable and maintained plugin manager for Nvim.

Toute la config est dans le vimrc ; il y a un panel d'UI qui permet de tout faire, pour l'afficher :

```
:Lazy
```

## Configuration lua

La config est chargée depuis `~/.config/nvim/init.lua ` ; avec ma config (en novembre 2023), ce fichier se contente presque de sourcer mon `~/.vimrc`.

(si un jour je veux découper ma config en plusieurs modules lua, on peut faire tout une hiérarchie de modules à loader au startup, cf. https://neovim.io/doc/user/lua-guide.html#lua-guide-modules)

J'ai splitté mon vimrc en deux : une section vimscript (config de base, utilisable depuis vim) et une section lua (config poweruser, utilisable depuis neovim uniquement).

[La doc](https://neovim.io/doc/user/lua-guide.html) sur comment configurer vim en lua est indispensable ; j'en donne quelques infos importantes :

- il y a plusieurs APIs :
    - `vim.api` = utiliser l'API officielle neovim
    - `vim.opt` = permet de définir des options (i.e. équivalent de `:set` en vim classique)
    - `vim.fn` = permet d'appeler des fonctions vim depuis du lua
- on peut toujours inliner n'importe quelle commande vimscript dans du lua avec `vim.cmd` :
    ```lua
   vim.cmd([[
       whatever vim commands here
       even on multiple lines
   ]])
    ```
- quelques équivalents de commandes "directes" vim :
    ```
    :highlight                      = vim.cmd.highlight()  (il n'y a pas de strict équivalent pour TOUTES les commandes, mais ça fait le taf pour mes besoins simples)
    :set expandtab                  = vim.opt.myvar = true
    :set shiftwidth=4               = vim.opt.shiftwidth=4
    :let g:loaded_matchparen = 0    = vim.g.loaded_matchparen = 0
    ```
- les auto-commandes sont un peu plus verbeuses :
    ```
    # en vimscript :
    autocmd FileType markdown setlocal foldexpr=NestedMarkdownFolds()

    # en lua :
    vim.api.nvim_create_autocmd({"FileType"}, {
        pattern = "markdown",
        callback = function (opts)
            vim.opt_local.foldexpr = "NestedMarkdownFolds()"
        end,
    })
    ```

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

Équivalents lua :

```
:lua require('telescope.builtin').live_grep()
:lua require('telescope.builtin').lsp_references()
:lua require('telescope.builtin').man_pages()
:lua require('telescope.builtin').help_tags()
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

# LSP aka Language Server Protocol

**C'est quoi ?** : un serveur LSP analyse le code et fournit des infos dessus à un client LSP :

- `clangd` analyse le code
- après analyse, il sait que la fonction `f` est appelée à tels 3 endroits, et définie à tel autre endroit
- il communique ça à un client LSP
- le client LSP expose des commandes du genre "jumper à la définition de f".

## LSP server — python

J'ai testé `pyright` qui fait le taf :

```
pipx install flake8
pipx install black
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

**caveat** = pour être sûr que clangd utilise le bon standard de compilation, il faut le préciser explicitement dans `CMakeLists.txt` afin d'être sûr qu'il apparaisse dans la ligne de commande de `compile_commands.json` :

```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# malheureusement, ne pas utiliser la version moderne = target_compile_features
# en effet, elle ne permet pas de forcer le standard...
```

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

**Question** : où est l'index clangd ?

**Réponse** : j'en ai trouvé dans de multiples endroits :

```
~/.cache/clangd
myproject/src/.cache/clangd
myproject/.cache/clangd
myproject/clangd
myproject/src/.clangd
```

Voir aussi [cette doc](https://releases.llvm.org/10.0.0/tools/clang/tools/extra/docs/clangd/Installation.html#background-indexing), qui dit :
> the index is saved to the .clangd/index in the project root
>
> index shards for common headers e.g. STL will be stored in $HOME/.clangd/index

## LSP client — native neovim client

En deux mots, j'ai switché sur le client LSP natif de neovim (cf. mon vimrc).

Quelques notes vrac :

Pour avoir les infos du client LSP :

```
:LspInfo

:LspLog
NOTE : les logs LSP sont tous conservés (au lieu d'être clear régulièrement).
On se mange donc un énorme fichier dans :
    ~/.cache/nvim/lsp.log


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

# Debugging cpp

**Contexte** : novembre 2023, je veux utiliser vimspector sous neovim pour débugger mon programme.

Au préalable : vérifier que le debugging hors neovim (i.e. directement avec gdb) fonctionne :

- vérifier dans le `compile_commands.json` qu'on est en `-O0 -g`
- sous meson, c'est le cas car `buildtype=debug` par défaut ([source](https://mesonbuild.com/Builtin-options.html#core-options))
- sous gdb : `r ; info functions PATTERN ; b myfunction`

## Les DAP

DAP (= [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/)) est un peu l'équivalent de LSP (= Language Server Protocol) pour le debugging :

- LSP :
    - un outil (e.g. clangd) est capable d'analyser le code-source
    - un LSP-server wrappe clangd pour analyser le code-source, et sait convetir ce que dit clangd en language LSP
    - un LSP-client (e.g. le client natif nvim-lsp) est capable de discuter avec les différents LSP-servers pour les utiliser
    - intérêt = les LSP-server se concentrent sur leur métier (analyser le code-source) sans avoir à s'intéresser à l'UI ou au client + on peut n'utiliser qu'un seul même client pour parler différents langages
- DAP :
    - un outil (e.g. gdb) est capable de débugger le programme
    - un DAP-server wrappe gdb pour débugger le programme, et sait convertir ce que dit gdb en language DAP
    - un DAP-client (e.g. nvim-dap ou vimspector) est capable de discuter avec les différents DAP-servers pour les utiliser

## Avec vimspector

### Installation et dépendance à python

D'après [la doc](https://github.com/puremourning/vimspector#dependencies), il faut un support de python :

> Neovim 0.8 with Python 3.10 or later (experimental)

Pour vérifier le support de python par neovim (et plein d'autres choses utiles) :

```sh
nvim +checkhealth  # ou alternativement, une fois neovim lancé :checkhealth
```

Le check a l'air d'essayer de loader tout un tas de versions de python hardcodée (de 3.12 à 3.7, avec ma version de neovim), et pour chacune, il semble essayer d'importer le module neovim, en faisant quelque chose comme :

```
python3.11 -c 'import neovim'
```

Ce module s'installe avec `pynvim`, donc au total avec pyenv :

```sh
pyenv install 3.11.6
pyenv global  # system 3.9.13
pyenv global system 3.9.13 3.11.6
python3.11 -m pip install --user --upgrade pynvim
```

### Gadgets

vimspector appelle les debug adapters des "gadgets", qu'il faut installer :

- la liste des gadgets est [ici](https://github.com/puremourning/vimspector#supported-languages)
- si besoin, la commande autocomplète :
   ```
   :VimspectorInstall CodeLLDB
   ```
- les gadgets sont installés à cet emplacement :
   ```
   ~/.local/share/nvim/lazy/vimspector/gadgets
   ```
- et leur fichier de config est ici :
   ```
   ~/.local/share/nvim/lazy/vimspector/gadgets/linux/.gadgets.json
   ```

### Fichier de config

Le plugin est paramétré par un fichier caché `.vimspector.json` (ce qui est triste).

<details>
  <summary>Cliquer pour voir un sample de fichier de config qui fait le taf</summary>

```json
{
  "$schema": "https://puremourning.github.io/vimspector/schema/vimspector.schema.json",
  "configurations": {
  "base": {
    "default": true,
    "adapter": "vscode-cpptools",
    "configuration": {
    "targetArchitecture": "x86_64",
    "request": "launch",
    "program": "${workspaceRoot}/path/to/my/binary",
    "externalConsole": false,
    "stopAtEntry": false,
    "stopOnEntry": false,
    "MIMode": "gdb",
    "breakpoints": {
      "exception": {
        "all": "",
        "cpp_catch": "",
        "cpp_throw": "",
        "objc_catch": "",
        "objc_throw": "",
        "swift_catch": "",
        "swift_throw": ""
      }
    },
    "setupCommands": [
      {
      "description": "Enable pretty-printing for gdb",
      "text": "-enable-pretty-printing",
      "ignoreFailures": true
      }
    ]
    }
  }
  },
  "adapters": {
  "lldb-vscode": {
    "variables": {
    "LLVM": {
      "shell": "brew --prefix llvm"
    }
    },
    "attach": {
    "pidProperty": "pid",
    "pidSelect": "ask"
    },
    "command": [
    "${LLVM}/bin/lldb-vscode"
    ],
    "env": {
    "LLDB_LAUNCH_FLAG_LAUNCH_IN_TTY": "YES"
    },
    "name": "lldb"
  }
  }
}
```

</details>

Quelques notes vrac :

- Le mini-projet de test donne un autre exemple de fichier de config.
- le schema du fichier de config est [documenté ici](https://puremourning.github.io/vimspector/schema/vimspector.schema.json)
- la clé racine `configurations` décrit le programme à débugger (il peut y en avoir plusieurs, et elles peuvent hériter les unes des autres)
- `stopAtEntry` / `stopOnEntry` = le debugger break dès l'entrée dans le `main`
- avec gdb, le snippet pour le pretty-printing des conteneurs de la STL est indispensable :
    ```json
    "setupCommands": [
        {
            "description": "Enable pretty-printing for gdb",
            "text": "-enable-pretty-printing",
            "ignoreFailures": true
        }
    ]
    ```

### Choix du programme à débugger

Je me serais attendu à ce qu'on puisse choisir dynamiquement le programme à débugger (e.g. retrouver le binaire dans un file-browser), mais on dirait malheureusement que le programme débuggé est défini statiquement dans la config du projet (= le fichier `.vimspector.json` à la racine du projet).

(on peut jouer un peu en définissant une variable `prog` qui sera promptée dynamiquement, mais c'est vraiment pas pratique...)

### Notes à l'usage

Cf. ma config nvim pour les commandes.

Les différentes vues de l'UI vimspector sont des fenêtres vim classiques → on s'y déplace normalement, avec `Ctrl+W`.

Comment choisir la configuration à utiliser (parmi les différentes clés dans le fichier de config) quand on lance vimspector ? [L'intégration avec telescope](https://github.com/nvim-telescope/telescope-vimspector.nvim) semble permettre ça.

Il semblerait que la config pour break sur les exceptions est cassée : il me demande quand même à chaque démarrage ce que je veux faire, même si je renseigne les clés de config...

Pour voir les breakpoints (et leurs conditions), la commande `<Plug>VimspectorBreakpoints` ouvre une fenêtre pour ça, mais ça semble ruiner le layout des fenêtres...

J'aimerais bien visualiser dynamiquement les breakpoints, plutôt que de devoir cliquer... Il faut que je voie s'il y a une intégration avec telescope pour ça.


### Playground

Le plugin est packagé avec un mini-projet qui peut être utile pour tester :

```sh
cd ~/.local/share/nvim/lazy/vimspector/tests/testdata/cpp/simple
make
nvim  -Nu  ~/.local/share/nvim/lazy/vimspector/tests/vimrc --cmd "let g:vimspector_enable_mappings='HUMAN'" simple.cpp
```

Attention, à la différence de ce qu'indique la doc, il faut éditer un fichier de test (ici `simple.cpp`) AVANT de lancer vimspector avec `F5` (car la config vimspector de ce projet de test renseigne que le programme à débugger a le même nom que le fichier source, sans l'extension).

Une fois nvim lancé, on peut placer un breakpoint avec `F9` puis lancer le debugger avec `F5`.

Le plugin prompte deux variables qui sont absente de la config, je trouve quoi répondre sur [cette page](https://github.com/puremourning/vimspector/blob/master/run_tests) :

```
Enter value for VIMSPECTOR_ARCH:
mettre le résultat de `uname -m` (e.g. `x86_64`)

Enter value for VIMSPECTOR_MIMODE:
j'ai mis `gdb`
```

Pour clore la fenêtre :

```
:VimspectorReset
```

## Avec nvim-dap et nvim-dap-ui

STILL TO DO :

- plus moderne
- en lua
- possibilité d'intégration avec telescope


# rust

J'ai suivi [la recommandation officielle](https://doc.rust-lang.org/book/ch01-01-installation.html#installing-rustup-on-linux-or-macos) :

```sh
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh

# [...]

rustc --version
# rustc 1.69.0 (84c898d65 2023-04-16)
```

J'ai conservé les chemins par défaut, ils sont indiqués dans la sortie de l'installation :

<details>
  <summary>Cliquer pour voir l'output</summary>

```
info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /home/myself/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory is located at:

  /home/myself/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /home/myself/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /home/myself/.profile
  /home/myself/.bashrc
  /home/myself/.zshenv

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: x86_64-unknown-linux-gnu
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes
```

</details>

## Utilisation avec neovim

J'utilise [rust-tools](https://github.com/simrat39/rust-tools.nvim#setup).

Dans le `.vimrc` (

```lua
require('rust-tools').setup{
   on_attach = on_attach,
   flags = lsp_flags,
}
```

Attention que le `require` est un poil différent des autres lspconfig, ce qui est rappelé explicitement dans [la doc](https://github.com/simrat39/rust-tools.nvim#setup) :

> This plugin automatically sets up nvim-lspconfig for rust_analyzer for you, so don't do that manually, as it causes conflicts.

(on trouve aussi le setup de rust-analyzer, légèrement différent, [ici](https://rust-analyzer.github.io/manual.html#nvim-lsp)).

Chez moi, après une fresh installation de rust avec rustup, `rust-analyzer` n'est pas utilisable :

- sous neovim, quand je tape `:LspLog` :
    ```
    [START][2023-05-12 11:48:00] LSP logging initiated
    [START][2023-05-12 11:48:45] LSP logging initiated
    [ERROR][2023-05-12 11:48:46] .../vim/lsp/rpc.lua:420▶   "rpc"▶  "rust-analyzer"▶"stderr"▶   "error: 'rust-analyzer' is not installed for the toolchain 'stable-x86_64-unknown-linux-gnu'\n"
    ```
- et effectivement, dans un terminal, le binaire `rust-analyzer` a beau exister, il n'est pas utilisable :
    ```
    rust-analyzer --version
    # error: 'rust-analyzer' is not installed for the toolchain 'stable-x86_64-unknown-linux-gnu'
    ```

Du coup, je suis [la doc](https://rust-analyzer.github.io/manual.html#rust-analyzer-language-server-binary) de `rust-analyzer` pour l'installer :

```sh
curl -L https://github.com/rust-lang/rust-analyzer/releases/latest/download/rust-analyzer-x86_64-unknown-linux-gnu.gz | gunzip -c - > ~/.local/bin/rust-analyzer
chmod +x ~/.local/bin/rust-analyzer
```

Mais comme j'ai maintenant deux versions concurrentes de `rust-analyzer`, il faut que je désactive celle qui ne marche pas → ça fonctionne :

```sh
mv ~/.cargo/bin/rust-analyzer ~/.cargo/bin/REMOVED_rust-analyzer

rust-analyzer --version
# rust-analyzer 0.3.1506-standalone
```

Note : on dirait que `rust-analyzer` nécessite un projet cargo pour fonctionner, et qu'il ne peut pas compiler un code-source standalone :

```
[ERROR][2023-05-12 11:58:39] .../vim/lsp/rpc.lua:420▶   "rpc"▶  "rust-analyzer"▶"stderr"▶   '[ERROR rust_analyzer::config] failed to find any projects in [AbsPathBuf("/path/to/my/projects")]\n'
[ERROR][2023-05-12 11:58:39] .../vim/lsp/rpc.lua:420▶   "rpc"▶  "rust-analyzer"▶"stderr"▶   "[ERROR rust_analyzer::main_loop] FetchWorkspaceError:\nrust-analyzer failed to discover workspace\n"
```


# pyenv

cf. [mes notes spécifiques sur le sujet](../python/pyenv.md)

# javascript et typescript

Regarder quelle est la dernière version de node sur https://nodejs.org/en

Installer la dernière version disponible :

```sh
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs
sudo apt autoremove
npm config set prefix '~/.local/'  # en tant que REGULARUSER
```

Installation de typescript + LSP :

```sh
npm install -g typescript typescript-language-server
```

# git

## diff-so-fancy

https://github.com/so-fancy/diff-so-fancy

Installation manuelle de `diff-so-fancy` = un simple clone :

```sh
cd ~/.local
git clone https://github.com/so-fancy/diff-so-fancy
cd bin
ln -s ../diff-so-fancy/diff-so-fancy
```

Configuration de git pour l'utiliser :

```sh
git config --global core.pager "diff-so-fancy | less --tabs=4 -RFX"
git config --global interactive.diffFilter "diff-so-fancy --patch"
```

Configuration des couleurs (j'ai un poil modifié ce que la page officielle suggère pour `color.diff.meta` et `color.diff.commit`, car le yellow ne se marie pas bien avec mon thème clair)

```sh
git config --global color.ui true
git config --global color.diff-highlight.oldNormal    "red bold"
git config --global color.diff-highlight.oldHighlight "red bold 52"
git config --global color.diff-highlight.newNormal    "green bold"
git config --global color.diff-highlight.newHighlight "green bold 22"
git config --global color.diff.meta       "blue"
git config --global color.diff.frag       "magenta bold"
git config --global color.diff.func       "146 bold"
git config --global color.diff.commit     "blue bold"
git config --global color.diff.old        "red bold"
git config --global color.diff.new        "green bold"
git config --global color.diff.whitespace "red reverse"
```

Rétro-pédalage dans le philenv où j'avais setté le `GIT_PAGER` à `cat` :

```sh
unset GIT_PAGER
```

# just

## C'est quoi

`just` ([github](https://github.com/casey/just), [site officiel](https://just.systems/), [manuel](https://just.systems/man/en/)) est un command-runner = un équivalent moderne à `make`.

Il permet 1. de définir des shortcuts per-project, et surtout 2. d'auto-documenter les commandes utiles propres à un projet, plutôt que de les définir dans le README (et les commandes étant documentées, elles restent lançables manuellement si on n'a pas installé `just`).

## Installation

Le tool n'est pas packagé pour ubuntu, j'ai donc suivi la doc pour installer manuellement :

```sh
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to /home/myself/.local/bin
# install: Repository:  https://github.com/casey/just
# install: Crate:       just
# install: Tag:         1.13.0
# install: Target:      x86_64-unknown-linux-musl
# install: Destination: /home/myself/.local/bin
# install: Archive:     https://github.com/casey/just/releases/download/1.13.0/just-1.13.0-x86_64-unknown-linux-musl.tar.gz

just --version
# just 1.13.0
```

Syntaxe des `justfile` dans le vimrc :

```
call plug#begin()
Plug 'https://github.com/NoahTheDuke/vim-just'
call plug#end()
```

Éditer le zshrc pour avoir un alias `j` plus court à taper + activer la complétion des recipes avec zsh :



```sh
# dans un terminal :
mkdir ~/.local/custom-zsh-completions && just --completions zsh > ~/.local/custom-zsh-completions/_just

# dans le zshrc :
alias j=just

# dans le zshrc, modifier le fpath AVANT de sourcer oh-my-zsh.sh :
fpath=(~/.local/custom-zsh-completions $fpath)
source $ZSH/oh-my-zsh.sh
```

## Notes vrac d'utilisation

- Le tool est plus puissant qu'il n'y paraît (e.g. on peut charger des variables depuis un dotenv) mais [le manuel](https://just.systems/man/en/) est bien fait → si j'ai des besoins particuliers, ne pas hésiter à aller y jeter un coup d'œil.
- Par défaut, `just` printe la commande exécutée avant de l'exécuter (pour disabler, il faut préfixer la commande par `@`)
- Par défaut, `just` arrête une recipe dès qu'une de ses commandes échoue (pour forcer à continuer malgré les erreurs, il faut préfixer la commande par `-`)
- Si on ne précise pas de recipe à `just`, il exécutera la première (une bonne astuce est donc de définir `just --list` comme première recipe du `justfile`).
- Les lignes de commentaires dans le justfile sont printées à l'exécution (ici aussi on peut les préfixer par `@` pour ne pas les printer à l'exécution) ; c'est un peu déroutant, mais à l'usage c'est pratique, et c'est un besoin que j'ai déjà eu = commenter à la fois le code et ce qui est affiché.
- On peut définir des dépendances (un peu comme les makefiles : on résout le DAG, et les dépendances sont exécutées d'abord) :
    ```
    task1:
      echo "TASK 1"

    task2: task1
      @echo "TASK 2"
    ```
- On peut définir des alias pour raccourcir la commande `just` à utiliser :
    ```
    alias b := build

    build:
      echo 'Building!'
    ```
- Un commentaire juste avant une recipe fait office de docstring, qui apparaîtra comme doc de la recipe dans un `just --list` :
    ```
    # this is the docstring of the recipe
    slides:
      echo "slides"
    ```
- On peut définir des variables pour les réutiliser dans plusieurs recipes :
    ```
    name := "Luke"

    jedi:
      @echo "Use the force {{ name }}"
    ```
- `just` fournit quelques fonctions pour faire de l'introspection ou de la manipulation de paths ou de strings, cf. https://just.systems/man/en/chapter_30.html
- modularité : on peut splitter ses justfiles et inclure des fichiers externes


# NerdFonts

Contexte : je veux utiliser [NerdFonts](https://www.nerdfonts.com/) ([github](https://github.com/ryanoasis/nerd-fonts)) pour disposer d'icônes sous neovim : principalement avec le file-browser nvim-tree et dans telescope.

À noter : au moins pour nvim-tree, il semblerait qu'en plus de la font, il faille installer le plugin [nvim-web-devicons](https://github.com/nvim-tree/nvim-web-devicons). Et le plugin seul n'est pas suffisant pour avoir les icônes, vu qu'il se contente d'utiliser des NerdFonts, qui doivent donc être préalablement installées.

## Que fait NerdFont

[La doc](https://github.com/ryanoasis/nerd-fonts#font-installation) de nerdfonts est assez vague, elle parle de "patcher" des polices, et recommande vaguement de simplement de télécharger une police (mais où exactement ?) ou d'utiliser une commande pour installer une police spécifique `./install.sh <FontName>`.

En pratique, [cette description](https://docs.rockylinux.org/books/nvchad/nerd_fonts/) m'éclaire un peu plus :

> Nerd Fonts takes the most popular programming fonts and modifies them by adding a group of glyphs (icons).

Donc les NerdFonts **modifient** la police que j'utilise pour lui ajouter des icônes.

Il faut donc que je trouve la police que mon terminal utilise.

## Quelle police mon terminal utilise-t-il ?

De façon surprenant, je n'ai pas trouvé de réponse définitive à cette question, j'ai dû me rabattre sur des moyens détournés pour trouver l'info : mon terminal utilise **DejaVu Sans Mono**.

Déjà, mon terminal est :

- PC fixe `Gnome Terminal 3.44.0` (Préférences > police personnalisée = `Monospace 8`)
- PC pro `Gnome Terminal 3.36.2` (Préférences > police personnalisée = `Monospace Regular 9`)

Les polices listées par gnome-terminal sont peu nombreuses (beaucoup moins nombreuses que celles contenues dans `/usr/share/fonts`, et elles ont des noms différents, simplifiés, je ne sais pas d'où ils sortent.

Et je n'ai initialement rien sous `~/.local/share/fonts`.

Pour creuser les polices de mon système :

- le gestionnaire de polices qui vient avec ubuntu (gnome-font-manager) refuse de se lancer ou plante...
- j'installe et utilise `font-manager` :
    - il accède déjà à beaucoup plus de fonts que Gnome Terminal
    - je filtre par `monospace`, et je n'ai que 17 fonts possibles
    - je compare manuellement les caractères de mon terminal avec ceux que montre font-manager pour trouver ma police
    - j'affiche notamment plein de caractères spéciaux, qui ont plus de chances d'être rendus différemment :
        ```
        ! " # $ % & 0 1 2 3 4 5 6 7 8 9 @ ß ™ ∫ ‰ ﬁ ﬂ ∂ Ʊ њ ؇ ⍙
        ```
    - au final, je réduis les candidats à :
        - **Bitstream Vera Sans Mono**
        - **DejaVu Sans Mono**
    - mais je n'ai pas d'information sur l'origine du fichier fournissant la police...
- avec LibreOffice Writer :
    - Avec LibreOffice Writer, j'ai accès à beaucoup de polices (encore plsu que font-manager... du coup ça fait 3 programmes différents qui listent 3 sets de polices différentes...)
    - En ouvrant mon texte d'essai sous LibreOffice Writer, je peux alterner facilement entre **Bitstream Vera Sans Mono** et **DejaVu Sans Mono**
    - Et je constate des différences ! Notamment, ce caractère `∫` (qui s'écrit avec le digraphe vim `In`) me permet de trancher : mon terminal utilise **DejaVu Sans Mono** !

À noter que la page de download des NerdFonts indique que DejaVu Sans est basée sur Bitstream Vera, ce qui explique leur similitude.

À noter enfin que `fc-list` semble pouvoir lister les fonts (mais je ne sais toujours pas d'où Gnome Terminal associe `Monospace Regular` à `DejaVu Sans Mono`).


## Installation d'une NerdFont

La doc indique beaucoup de moyens d'installation, mais comme je ne savais initialement pas où télécharger les fonts, j'ai retenu :

```
cd /tmp
git clone --depth 1 https://github.com/ryanoasis/nerd-fonts.git  # très long !
cd nerd-fonts
./install.sh DejaVuSansMono
# Nerd Fonts installer -- Version 0.7
# mkdir: création du répertoire '/home/myself/.local/share/fonts/NerdFonts'
# '/tmp/nerd-fonts/patched-fonts/DejaVuSansMono/Bold-Italic/DejaVuSansMNerdFont-BoldOblique.ttf' -> '/home/myself/.local/share/fonts/NerdFonts/DejaVuSansMNerdFont-BoldOblique.ttf'
# '/tmp/nerd-fonts/patched-fonts/DejaVuSansMono/Bold/DejaVuSansMNerdFont-Bold.ttf' -> '/home/myself/.local/share/fonts/NerdFonts/DejaVuSansMNerdFont-Bold.ttf'
# '/tmp/nerd-fonts/patched-fonts/DejaVuSansMono/Italic/DejaVuSansMNerdFont-Oblique.ttf' -> '/home/myself/.local/share/fonts/NerdFonts/DejaVuSansMNerdFont-Oblique.ttf'
# '/tmp/nerd-fonts/patched-fonts/DejaVuSansMono/Regular/DejaVuSansMNerdFont-Regular.ttf' -> '/home/myself/.local/share/fonts/NerdFonts/DejaVuSansMNerdFont-Regular.ttf'
# /home/myself/.local/share/fonts/NerdFonts: caching, new cache contents: 4 fonts, 0 dirs
# /var/cache/fontconfig: not cleaning unwritable cache directory
# /home/myself/.cache/fontconfig: cleaning cache directory
# /home/myself/.fontconfig: not cleaning non-existent cache directory
# fc-cache: succeeded
```

^ ceci résout le problème sur le PC pro, et j'ai pu vérifier que ce qui résout est d'avoir la font sous `~/.local/share/fonts/NerdFonts` (quand je déplace le répertoire, je n'ai plus les icônes ; `fc-cache` n'a pas l'air d'avoir un impact).

Mais le problème n'est que partiellement résolu sur le PC fixe : il reste quelques mauvaises icônes.

Du coup j'ai contourné comme une brutasse : j'installe toutes les nerd-fonts (`./install.sh` sans argument) et je choisis manuellement sous Gnome Terminal une autre font NerdFont, car les NerdFonts apparaissent dans les fonts disponibles sous Gnome Terminal (j'en essaye plusieurs, pour finir sur `Bitstrom Wera Nerd Font`)


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

## Nouveau problème avec clangd = il ne trouve pas libc++

Problème : en gros, clangd n'arrive pas à utiliser libc++ (et soulève donc des erreurs sur de simples inclusions de headers standards comme `<ctime>`).

Pourquoi a-t-elle disparu ? mystère... peut-être parce que j'ai upgradé des trucs ? peut-être qu'elle n'a jamais vraiment marché depuis que je suis passé en 22.04 ? Par ailleurs, clang++ arrivait très bien à compiler mon code alors même que je n'avais pas de libc++ sur mon système (peut-être que clang++ arrive à utiliser libstdc++, mais pas clangd ?)

Note qui peut servir = on a tous les outils LLVM dans ce package : https://apt.llvm.org/

Analyse = qu'est-ce que j'ai sur mon poste ?

```sh
dpkg -l|grep -i clangd
# ii  clangd:amd64                              1:14.0-55~exp2                             amd64        Language server that provides IDE-like features to editors
# ii  clangd-12                                 1:12.0.1-19ubuntu3                         amd64        Language server that provides IDE-like features to editors
# ii  clangd-14                                 1:14.0.0-1ubuntu1                          amd64        Language server that provides IDE-like features to editors

clangd --version
# Ubuntu clangd version 14.0.0-1ubuntu1
# Features: linux+grpc
# Platform: x86_64-pc-linux-gnu

apt-cache policy clangd-14
# clangd-14:
#   Installé : 1:14.0.0-1ubuntu1
#   Candidat : 1:14.0.0-1ubuntu1
#  Table de version :
#  *** 1:14.0.0-1ubuntu1 500
# 		500 http://fr.archive.ubuntu.com/ubuntu jammy/universe amd64 Packages
# 		100 /var/lib/dpkg/status

lsb_release -a
# No LSB modules are available.
# Distributor ID:	Ubuntu
# Description:	Ubuntu 22.04.1 LTS
# Release:	22.04
# Codename:	jammy
```

Tenté ceci, sans que ça fasse de différence (a priori, car toutes les versions avaient une priorité équivalente) :

```sh
clangd --version
#Ubuntu clangd version 14.0.0-1ubuntu1
#Features: linux+grpc
#Platform: x86_64-pc-linux-gnu

sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-14 100
# update-alternatives: avertissement: forçage de la réinstallation de l'alternative /usr/bin/clangd-12 car le groupe de liens clangd est cassé
# clangd --version
# Ubuntu clangd version 12.0.1-19ubuntu3

sudo apt install libc++-14-dev libc++1-14 libc++abi-14-dev libc++abi1-14
# [sudo] Mot de passe de myself :
# Lecture des listes de paquets... Fait
# Construction de l'arbre des dépendances... Fait
# Lecture des informations d'état... Fait
# Les paquets suivants ont été installés automatiquement et ne sont plus nécessaires :
#   libflashrom1 libftdi1-2 libgoogle-perftools4 libllvm13:i386 libtcmalloc-minimal4
# Veuillez utiliser « sudo apt autoremove » pour les supprimer.
# Les paquets supplémentaires suivants seront installés :
#   libunwind-14 libunwind-14-dev
# Les paquets suivants seront ENLEVÉS :
#   libgoogle-perftools-dev libunwind-dev
# Les NOUVEAUX paquets suivants seront installés :
#   libc++-14-dev libc++1-14 libc++abi-14-dev libc++abi1-14 libunwind-14 libunwind-14-dev
# 0 mis à jour, 6 nouvellement installés, 2 à enlever et 6 non mis à jour.
# Il est nécessaire de prendre 1 469 ko dans les archives.
# Après cette opération, 1 635 ko d'espace disque supplémentaires seront utilisés.
# Souhaitez-vous continuer ? [O/n]
```

Au final, sans que j'en sois sûr à 100% car j'ai essayé plein de trucs, ce qui semble corriger le problème, c'est de choisir manuellement la version 14 de clangd :


```sh
sudo update-alternatives --config clangd
# _Il existe 3 choix pour l'alternative clangd (qui fournit /usr/bin/clangd).
# _  Sélection   Chemin              Priorité  État
# _------------------------------------------------------------
# _  0            /usr/bin/clangd-14   100       mode automatique
# _  1            /usr/bin/clangd-12   100       mode manuel
# _* 2            /usr/bin/clangd-14   100       mode manuel
# _  3            /usr/bin/clangd-15   100       mode manuel
# _Appuyez sur <Entrée> pour conserver la valeur par défaut[*] ou choisissez le numéro sélectionné :
```

Toutes les versions étaient à une priorité de 100, j'ai choisi manuellement clangd-14 comme prioritaire.

## Nouveau problème avec clangd = le standard utilisé n'est pas le bon

(possiblement une conséquence du paragraphe précédent où j'ai forcé clangd-14)


### Manifestation de mon problème

- j'ai fait une POC sur `std::optional`, qui n'est disponible qu'à partir de C++17
- je compile avec cmake en précisant le standard à C++17 :
    + ça compile bien
- pourtant, lorsque j'ouvre le code dans nvim, les erreurs du LSP montrent qu'il analyse le code en C++14 (qui ne connaît pas `std::optional`) :
    ```
    std::optional<unsigned int> halve_if_even(unsigned int value) {     ■ No template named 'optional' in namespace 'std'
    ```

### TL;DR

**EXPLICATION** = le standard n'était pas indiqué dans `compile_commands.json` (car mon compilo était compatible) ET j'utilisais un clangd/clang++ avec un standard par défaut différent.

**RÉSOLUTION** :

- ne pas oublier d'activer le `compile_commands.json` :
    ```
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
    ```
- définir le standard avec `CMAKE_CXX_STANDARD` (et non pas avec `target_compile_features`) :
    ```
    set(CMAKE_CXX_STANDARD 17)
    ```
    - malheureusement, il ne faut PAS utiliser la commande "moderne" de cmake permettant de définir le standard = `target_compile_features(mytarget PRIVATE cxx_std_17)`
- ne pas oublier de forcer cmake à préciser explicitement le standard sur la ligne de commande de compilation dans `compile_commands.json` :
    ```
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    ```

Note = pour débugger le client LSP de neovim :

- `:LspRestart` pour redémarrer le Lsp (et forcer la réanalyse du code, e.g. après modification du `compile_commands.json`)
- `:LspLog` pour voir ce que dit clangd (se passe avec clangd redémarrer le client LSP (et forcer la réanalyse du code)
- éventuellement, pour cleaner le log LSP préalablement  =  `rm ~/.cache/nvim/lsp.log`

### Analyse

#### STEP 1 = en première approche, les logs LSP montrent que clangd ne trouve pas de `compile_commands.json`

`:LspLog`  (ou `bat ~/.cache/nvim/lsp.log`) :

```
Failed to find compilation database for /path/to/main.cpp
ASTWorker building file /path/to/main.cpp version 0 with command clangd fallback
/usr/bin/clang -resource-dir=/usr/lib/llvm-14/lib/clang/14.0.0 -- /path/to/main.cpp
```

Du coup, on voit qu'il utilise clang14 pour compiler, sans lui préciser le standard (et clang14 ne semble pas être en C++17 par défaut).

#### STEP 2 = même avec un `compile_commands.json` ET le standrad en C++17 dans le CMakeLists.txt, clangd continue d'utiliser le standard C++14

Je m'assure d'avoir un `compile_commands.json` :

```
# dans le CMakeLists.txt :
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# lien obligatoire à la racine du projet
ln -s NOGIT_build/`compile_commands.json`
```

Je vérifie également que le projet est compilé en C++17 :

```
# dans le CMakeLists.txt :
set(CMAKE_CXX_STANDARD 17)
# (et je vérifie que le build compile correctement, donc qu'il est en C++17)
```

Pourtant, LspLog montre que clangd, même s'il utilise `compile_commands.json`, compile toujours en C++ 14 :

```
Loaded compilation database from /path/to/`compile_commands.json`
ASTWorker building file /path/to/main.cpp version 0 with command
usr/bin/c++ --driver-mode=g++ -Wall -Wextra -pedantic -Werror -o CMakeFiles/pouet.dir/main.cpp.o -c -resource-dir=/usr/lib/llvm-14/lib/clang/14.0.0 -- /path/to/main.cpp
```

Et effectivement, `compile_commands.json` n'indique PAS le standard :

```
"directory": "/path/to/NOGIT_build",
"command": "/usr/lib/ccache/c++   -Wall -Wextra -pedantic -Werror -o CMakeFiles/pouet.dir/main.cpp.o -c /path/to/main.cpp",
"file": "/path/to/main.cpp"
```

Du coup, clangd (qui utilise la commande de compilation, mais avec clang14 !) utilise le standard par défaut de clang-14... qui se trouve être C++14 ! (cf. QUESTIONS ci-dessous)

#### STEP 3 = je force cmake à préciser explicitement le standard dans `compile_commands.json`

En fait, cmake ne passe pas forcément le standard C++ dans la ligne de commande (cf. QUESTIONS ci-dessous). Pour forcer explicitement, il faut une option :

```
# dans le CMakeLists.txt :
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

Si on fait ça, le `compile_commands.json` précise ENFIN le standard explicitement :

```
"directory": "/path/to/NOGIT_build",
"command": "/usr/lib/ccache/c++   -Wall -Wextra -pedantic -Werror -std=gnu++17 -o CMakeFiles/pouet.dir/main.cpp.o -c /path/to/main.cpp",
"file": "/path/to/main.cpp"
```

Du coup, LspLog montre ENFIN que clangd le prend en compte :

```
Loaded compilation database from /path/to/`compile_commands.json`\n"
ASTWorker building file /path/to/main.cpp version 0 with command
/usr/bin/c++ --driver-mode=g++ -Wall -Wextra -pedantic -Werror -std=gnu++17 -o CMakeFiles/pouet.dir/main.cpp.o -c -resource-dir=/usr/lib/llvm-14/lib/clang/14.0.0 -- /path/to/main.cpp
```

Et les warnings disparaissent, victory !

### Questions

Q = quel est le standard C++ utilisé par mon code ?

- en théorie, je peux aller observer la ligne de compilation dans `compile_commands.json`
- en pratique, le standard n'y apparaît pas...
- pourtant, j'ai bien `CMAKE_CXX_STANDARD` dans mon CMakeLists.txt !

Q = quel est le standard C++ par défaut utilisé par clangd 14.0.0 ?

-  j'ai eu du mal à y arriver, mais la question est mal posée
-  s'il n'y a pas de database compilation, clangd redirige sur clang pour compiler
-  c'est donc le standard C++ par défaut de clang qui compte

Q = quel est le standard C++ utilisé par clang 14 ?
- EDIT : finalement, l'info est dans le man de clang (pas de clangd, mais bien de clang) :
   > The default C++ language standard is gnu++14.
- de façon surpenante, c'est difficile d'obtenir une réponse définitive à cette question
- je ne trouve pas d'option dans clangd pour lui dire "dis-moi quel standard par défaut tu utilises"
- LspLog m'indique la version de clangd utilisée (c'est bien celle qui est installée sur mon poste) :
    ```
    Ubuntu clangd version 14.0.0-1ubuntu
    ```
- sans trouver d'info officielle, je trouve ceci sur [cette page](https://www.phoronix.com/news/LLVM-Clang-16-Default-GNU17), qui montre que ce n'est qu'à partir de clang-16 que clang est en C++17 par défaut :
    > GNU++17 (C++17 with GNU extensions) is now the default C++ standard targeted if no other version is explicitly set for the compiler.
    >
    > This is a bump from GNU++14 as the current C++ default up to LLVM/Clang 15.
- indirectement, l'info est dans [la page officielle](https://clang.llvm.org/cxx_status.html) :
    > By default, Clang 16 or later builds C++ code according to the C++17 standard.
- Mais comme c'est au milieu d'autres phrases un peu contradictoire, c'est un peu confusant :
    > Clang 5 and later implement all the features of the ISO C++ 2017 standard. \
    > By default, Clang 16 or later builds C++ code according to the C++17 standard. \
    > You can use Clang in C++17 mode with the -std=c++17 option (use -std=c++1z in Clang 4 and earlier).

Q = comment modifier le standard utilisé par clangd ?

   - en s'assurant qu'il utilise `compile_commands.json` + en forçant cmake à préciser le standard dans la ligne de commande de compilation

Q = pourquoi le standard C++17 n'apparaît pas dans la ligne de compilation de `compile_commands.json` ?

- TL;DR = car le compilo détecté par cmake est compatible avec C++17, donc il ne juge pas nécessaire de le préciser.
- Du coup, clangd utilise son standard par défaut à sa propre version, qui peut être différent du standard par défaut du compilo utilisé pour builder.
- Pour forcer le standard explicitement :
    ```
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    ```
- (la doc est pas hyper-claire, mais en pratique ça a l'air de marcher)

Q = comment indiquer à cmake le standard à utiliser ?

- Notamment, quelle différence entre :
    ```
    set(CMAKE_CXX_STANDARD 17)
    target_compile_features(pouet PUBLIC cxx_std_17)
    ```
- réponse : le deuxième est la façon moderne
- MAIS [la doc](https://cmake.org/cmake/help/latest/manual/cmake-compile-features.7.html) indique que le flag n'est pas obligatoirement passé :
    > Note If the compiler's default standard level is at least that of the requested feature, CMake may omit the -std= flag. The flag may still be added if the compiler's default extensions mode does not match the <LANG>_EXTENSIONS target property, or if the <LANG>_STANDARD target property is set.
- du coup, j'ai l'impression que si je veux forcer le fait de passer le standard, pour le moment, je dois rester à l'ancienne façon de passer le standard + préciser le REQUIRED...
- note : [cette issue cmake](https://gitlab.kitware.com/cmake/cmake/-/issues/23397) présente exactement mon problème :
    > If you want to control the standard explicitly, use CMAKE_CXX_STANDARD.  That mode will be used unless a compile feature demands a higher level.
    - TL;DR : c'est by design, donc pour obtenir ce que je veux (= utiliser EXACTEMENT ce standard et non "au moins" ce standard); il faut utiliser `CMAKE_CXX_STANDARD` et non `target_compile_features`


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
- profiler le lancement de zsh, qui est assez lent : https://stevenvanbael.com/profiling-zsh-startup (EDIT : c'est en partie dû à nvm)
- Trouver des serveurs LSP pour des langages additionnels ?
    - Shell
    - Cmake
    - JavaScript
    - HTML
    - CSS
- neovim/LSP : quand j'utilise l'omnicompletion avec lsp, ça m'ouvre une fenêtre en bas de mon écran, qu'il faut que je referme manuellement derrière...
    - pour corriger, il faudra que je comprenne mieux la complétion sous vim
    - notamment, quelle différence entre :
        - taper directement `Ctrl+n` (actuellement, ça trigge la complétion "dumb")
        - taper `Ctrl+x` puis `Ctrl+o` (actuellement, ça trigge la complétion "smart")
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
- python : utiliser [isort](https://github.com/PyCQA/isort) pour les imports.
- TELESCOPE> question : comment tuner la recherche ?
    - toggle une recherche exacte/fuzzy
    - toggle case sensitive ou non
    - limiter la recherche à mon workspace (et ne pas recherche dans /usr/include/boost, p.ex.)
- TELESCOPE> une feature qui me manque = Ctrl-D pour matcher avec le nom du fichier uniquement (sans son chemin)
- TELESCOPE> est-ce que j'ai moyen de limiter le grep/find_files au répertoire local ?
- TELESCOPE> comment ne PAS avoir les symboles de /usr/include comme boost renvoyés par LSP ?
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
