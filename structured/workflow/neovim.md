**Contexte** : novembre 2023, je place ici les notes liées à neovim qui encombrent mon [dev-workflow.md](./dev-workflow.md).

* [Neovim](#neovim)
   * [Tips](#tips)
   * [Colorscheme et highlights](#colorscheme-et-highlights)
   * [Plugins (neo)vim possiblement intéressants, mais que j'ai choisi de ne pas utiliser](#plugins-neovim-possiblement-intéressants-mais-que-jai-choisi-de-ne-pas-utiliser)
   * [Plugins (neo)vim possiblement intéressants, à regarder](#plugins-neovim-possiblement-intéressants-à-regarder)
* [Telescope](#telescope)
   * [Notes à l'usage](#notes-à-lusage)


# Neovim

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
- commandes commençant par `<Plug>` (e.g. `<Plug>VimspectorContinue`, [lien](https://github.com/puremourning/vimspector#mappings)) :
    - cf. la doc :
    ```
    :help <Plug>
    :help using-<Plug>
    ```
    - le principe est de définir une chaîne de caractère (qui utilise `<Plug>`) qu'on ne matchera pas par hasard sans le vouloir
    - par ailleurs, on ne peut pas l'utiliser comme commande, l'idée est d'obligatoirement la mapper
    - on peut tout de même contourner, mais c'est compliqué :
    ```
   :execute "normal \<Plug>VimspectorContinue"
    ```
- vimdiff n'existe plus, mais l'équivalent est:
    ```
   nvim -d -O file1 file2
    ```
- pour afficher une variable lua, les deux syntaxes suivantes sont équivalentes, mais la deuxième est plus courte :
    ```
    :lua =myvar
    :lua vim.print(myvar)

    ```

## Colorscheme et highlights

Quand on a un souci de coloration, c'est pas évident de débugger ; quelques tips :

- en complément de lister les highlihts avec `highlight` (éventuellement couplé à `redir`), on peut utiliser Telescope :
    ```
    :Telescope highlights
    ```
- pour connaître le nom du groupe de couleurs sous le caractère courant (ne semble pas marcher tout le temps, je ne sais pas trop pourquoi) :
    ```
    :echo synIDattr(synID(line("."),col("."),1),"name")
    :echo synIDattr(synID(line("."),col("."),0),"name")
    ```
- pour connaître la provenance d'un highlight (ici, `Comment`) :
    ```
    :verbose highlight Comment
    ```

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
