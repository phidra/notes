**Contexte** : novembre 2023 = je tune un peu ma config neovim, et j'en convertis une partie de vimscript à lua.

[Ce primer](https://www.geeks3d.com/20130516/lua-primer-for-the-impatient/) est une excellente synthèse en quelques minutes.

Il y a [une doc nvim sur lua](https://neovim.io/doc/user/lua.html#lua-concepts) (et même plusieurs).

Et la [doc officielle lua](https://www.lua.org/pil/contents.html) est plus longue, mais bien faite.

Si besoin, [ce REPL en ligne](https://www.lua.org/cgi-bin/demo) est utile pour tester facilement des trucs.

----

Quelques notes vrac :

- les listes et dict sont gérés avec la même structure = les **tables** = `{...}` ([doc officielle](https://www.lua.org/pil/2.5.html))
- tous les scopes sont locaux
- les rawstrings entre `[[...]`] :
    ```lua
    local myrawstring = [[coucou "pouet"]]
    local myotherrawstring = [[
        ça fonctionne aussi
        pour les multiline
        strings
    ]]
    ```
- un peu contre-intuitif donc à connaître = lorsqu'on appelle une fonction, les parenthèses sont optionnelles dans le cas particulier où la fonction n'attend qu'un unique argument, et que celui-ci est de type `string` ou `dict` ([doc officielle](https://www.lua.org/pil/5.html)) :
    ```
    print "Hello World"     <-->     print("Hello World")
    dofile 'a.lua'          <-->     dofile ('a.lua')
    print [[a multi-line    <-->     print([[a multi-line
     message]]                        message]])
    f{x=10, y=20}           <-->     f({x=10, y=20})
    type{}                  <-->     type({})
    ```
    - on peut se servir de cette syntaxe particulière pour avoir une syntaxe proche des arguments nommés ; les deux lignes ci-dessous sont équivalentes :
    ```lua
    setup{opt1="pouet", opt2="youpi")
    setup({opt1="pouet", opt2="youpi"))
    ```
- toujours dans la [doc sur les fonctions](https://www.lua.org/pil/5.html), appeler une fonction avec le mauvais nombre de paramètres est (malheureusement, IMO...) valide :
    > Lua adjusts the number of arguments to the number of parameters, as it does in a multiple assignment: Extra arguments are thrown away; extra parameters get nil.
- la syntaxe pour utiliser une méthode est `obj:f()` ([doc officielle](https://www.lua.org/pil/16.html)) :
    > o:foo(x) is just another way to write o.foo(o, x)


- construction des tables avec des clés complexes ([doc officielle](https://www.lua.org/pil/3.6.html)) :
    - les clés des tables peuvent prendre n'importe quelle valeur (y compris des mots-clés réservés du langage, comme `-1`), mais les constructors des tables ne peuvent pas forcément les gérer :
        ```lua
        -- identifiant "simple" :
        a = {pouet = 42}
        print(a.pouet)

        -- identifiant non-supporté : ceci provoque une erreur :
        b = {-1 = 42}
        print(c.-1)
        ```
    - lua propose une syntaxe alternative :
        > a.name as syntactic sugar for a["name"]
    - donc le contournement est d'utiliser cette syntaxe ; elle est plus longue, mais fonctionne sur des mots-clés :
        ```lua
        c = {["-1"] = 42}
        print(c["-1"])
        ```
    - c'est par exemple utilisé dans [valhalla](https://github.com/valhalla/valhalla/blob/6a83b61acca88582d9a5be8ef53e535f78dce00e/lua/graph.lua), et comme ça peut surprendre, mieux vaut connaître la syntaxe
