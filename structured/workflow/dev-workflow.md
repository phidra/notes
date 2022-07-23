Notes sur mon workflow de dev

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

# ALE

C'est quoi ? En gros, un plugin vim pour lancer plein de linters sur un fichier de code, et interpréter les résultats. Il joue le rôle de LSP client.

## Installation

Avec vim-plug :

```
# vimrc :
Plug 'https://github.com/dense-analysis/ale'

# au runtime :
:PlugInstall
```

Une fois ALE installé, il utilisera les linters automatiquement, à condition que ceux-ci soient installés.

Dit autrement : il suffit d'installer un linter pour qu'ALE l'utilise ; par exemple, comme mon poste dispose d'un compilateur, out-of-the-box, ALE compile le fichier en cours de frappe et m'indique les erreurs dynamiquement \o/. En revanche, je ne pourrais utiliser `ALEGoToDefinition` que si j'installe un LSP server C++ comme `clangd`.

## Setup python

Du coup, il suffit d'installer un LSP server pour python pour profiter d'ALE en python. Par exemple `pyright` :

```
pipx install pyright
```

## Setup C++

Installer `clangd` pour qu'ALE puisse l'utiliser comme LSP server C++ :

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

C'est quoi, ça sert à quoi ? Problématique = les linters utilisés par ALE ont besoin de compiler le fichier en cours d'édition **exactement** comme la toolchain de mon projet le fait (e.g. en incluant les bons chemins vers les headers, ou en passant exactement les bonnes options de compilation, telles que le standard C++).

Pour cela, les toolchains peuvent générer une base de données des commandes permettant de compiler chaque fichier = `compile_commands.json`. Exemple de contenu :

```
{
   directory :  /media/DATA/git_projects/ULTRA/_build ,
   command :  /usr/bin/clang++   -I/home/phijul/.conan/data/rapidjson/1.1.0/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include -I/home/phijul/.conan/data/fast-cpp-csv-parser/cci.20200830/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include -I/home/phijul/.conan/data/fast-cpp-csv-parser/cci.20200830/_/_/package/5ab84d6acfe1f23c4fae0ab88f26e3a396351ac9/include/fast-cpp-csv-parser -I/media/DATA/git_projects/ULTRA/Investigations  -Wall -Wextra -Werror -Wunused-parameter -Wno-unused-parameter -Wno-infinite-recursion -Wno-unused-variable -Wno-sign-compare   -O3 -DNDEBUG    -fopenmp -pipe -march=native -O3 -ffast-math -ftree-vectorize -Wfatal-errors -DNDEBUG -std=gnu++17 -o CMakeFiles/build-transfer-graph.dir/media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp.o -c /media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp ,
   file :  /media/DATA/git_projects/ULTRA/Custom/Common/polygon.cpp
},
```

Pour cmake, on peut facilement le configurer pour qu'il génère ce `compile_commands.json`, avec une ligne dans le `CMakeLists.txt` :

```
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

**Caveat 1** = les linters lancés par ALE attendent le fichier à la racine du projet ; or, il est généré par cmake dans le répertoire de build → ne pas hésiter à faire un lien symbolique vers le répertoire de build :

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

## Features et commandes

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

## Configuration

Le lint ne se déclenche pas tout de suite mais après un petit délai :

```
:let g:ale_lint_delay=200    <-- valeur par défaut du delay = 200 ms
```

La complétion n'est pas active par défaut ; pour l'activer :

```
let g:ale_completion_enabled = 1
```

# Journal

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

# Futur

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
- TMUX : éventuellement, installer un tmux récent sur mon PC portable creanov ?
    - sur mon PC fixe :
        ```
        tmux -Version
        tmux 3.0a
        ```
    - https://github.com/tmux/tmux/releases/tag/3.0a
    - https://bogdanvlviv.com/posts/tmux/how-to-install-the-latest-tmux-on-ubuntu-16_04.html
    - https://jdhao.github.io/2018/10/16/tmux_build_without_root_priviledge/
- retrouver + ajouter à ma doc vim la commande pour copier dans un buffer le contenu de commandes dans le genre de :
    ```
    :digraph
    :ALEInfo
    ```
