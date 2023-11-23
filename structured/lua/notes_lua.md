**Contexte** : novembre 2023 = je tune un peu ma config neovim, et j'en convertis une partie de vimscript à lua.

[Ce primer](https://www.geeks3d.com/20130516/lua-primer-for-the-impatient/) est une bonne synthèse.

Il y a [une doc nvim sur lua](https://neovim.io/doc/user/lua.html#lua-concepts) (et même plusieurs).

Quelques notes vrac :

- les listes et dict sont gérés avec la même structure = les **tables** = `{...}`
- tous les scopes sont locaux
- les rawstrings entre `[[...]`] :
    ```lua
    local myrawstring = [[coucou "pouet"]]
    ```
- le fait qu'on peut se passer de parenthèses pour les cas particulieres où on appelle une fonction à un seul argument string ou dict est à connaître car un peu contre-intuitif. On peut s'en servir pour avoir une syntaxe proche des arguments nommés, les deux lignes ci-dessous sont équivalentes :
    ```lua
    setup{opt1="pouet", opt2="youpi")
    setup({opt1="pouet", opt2="youpi"))
    ```
