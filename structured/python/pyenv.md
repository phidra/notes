Contexte = j'ai eu besoin de pyenv en janvier 2022 pour installer un vieux python 3.5 sur une machine relativement récente.

La doc est sur [le github du projet](https://github.com/pyenv/pyenv), et le [tuto de realpython](https://realpython.com/intro-to-pyenv/) est pas mal.

* [TL;DR](#tldr)
* [Principe de fonctionnement](#principe-de-fonctionnement)
* [Installation](#installation)
* [Utilisation](#utilisation)
   * [quelques essais](#quelques-essais)
   * [shims et rehash](#shims-et-rehash)
   * [pyenv local](#pyenv-local)
   * [pyenv virtualenv](#pyenv-virtualenv)
      * [utilisation TL;DR](#utilisation-tldr)
      * [comment ça marche](#comment-ça-marche)
      * [qu'apporte pyenv activate ?](#quapporte-pyenv-activate-)
      * [comment deactivate ?](#comment-deactivate-)
      * [virtualenv, entry_points, shims et rehash](#virtualenv-entry_points-shims-et-rehash)
   * [connexe = zsh precmd_functions](#connexe--zsh-precmd_functions)

# TL;DR

- `pyenv install` pour installer une version de python en particulier (dans le répertoire pyenv)
- `pyenv [global / local / shell]` pour utiliser une version gérée par pyenv en tant que python [système / du projet courant / du shell courant]
- `pyenv version` pour connaître la version actuellement utilisée (et `pyenv versions` pour connaître celles dispos sur la machine)
- quand on utilise pyenv, `which python` pointe _toujours_ sur le shim ; il faut utiliser `pyenv which python` pour voir sur quoi pointe le shim lui-même.
- `pyenv rehash` pour créer un shim inexistant (e.g. un `entry_point` créé dans un venv) et supprimer les shims inutiles

Autres commandes utiles

- `pyenv whence COMMAND` = savoir quelles versions de python (ou venvs) fournissent la COMMAND
- `pyenv update` = mettre à jour pyenv, afin qu'il sache installer des versions plus récentes de python
- `pyenv commands` = lister les commandes disponibles
- `pyenv shims` = lister les shims
- `pyenv versions` = lister les shims
- `pyenv root` = connaître le répertoire d'installation de pyenv
- `pyenv uninstall VERSION` = supprimer une VERSION de python

# Principe de fonctionnement

Le besoin auquel pyenv répond est de pouvoir installer simplement (puis jongler facilment avec) plusieurs versions concurrentes de python sur un même système. Les explications dans [le README du projet](https://github.com/pyenv/pyenv/blob/master/README.md) sont très claires, en voici un résumé :

- pyenv prepend au `PATH` son propre répertoire de binaires
- du coup, il intercepte les commandes python (e.g. `python`, `python3`, `pip`, `2to3`, etc.) et exécute à la place ses propres **shims**
    ```sh
    type python
    python is /home/username/.pyenv/shims/python
    ```
- un shim n'est qu'un passe-plat intelligent : il "choisit" la bonne version de python à utiliser, puis lui transmet la commande
    ```sh
    pyenv which python
    #  /usr/bin/python  (sera résolu différement si on active une autre version que 'system')
    ```
- la façon dont un shim choisit la version de python à utiliser suit le process suivant, dans l'ordre :
    - envvar `PYENV_VERSION` (cf. `pyenv shell`)
    - fichier `.python-version` dans le répertoire courant ou un de ses parents (cf. `pyenv local`)
    - fichier `/home/username/.pyenv/version` (cf. `pyenv global`)
    - fallback sur la commande "normale" (i.e. celle qui serait lancée si pyenv n'était pas dans le `PATH`)
- on utilise `pyenv rehash` pour créer de nouveaux shims ou mettre à jour ceux existants (cf. paragraphe ci-dessous)
- (il est possible d'activer _plusieurs_ versions de python au sein d'un projet, pour que le projet puisse utiliser plusieurs versions concurrentes de python ; c'est utile pour que tox puisse tester que le projet est correct avec différentes versions de python)

# Installation

- je suis la procédure du github = https://github.com/pyenv/pyenv/wiki#suggested-build-environment
    ```sh
    sudo apt install make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
    curl https://pyenv.run | bash
    ```
- je modifie les fichiers de conf pour prepend pyenv à mes PATH : `vim ~/.zshrc ~/.bashrc` (attention : j'ai dû tâtonner pour trouver les bonnes lignes, on dirait que ni la doc, ni l'aide en ligne ne sont à jour) :
    ```sh
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init --path)"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    ```
- in fine, voici l'état de mon PATH `echo $PATH` :
    ```sh
    /home/username/.pyenv/plugins/pyenv-virtualenv/shims:/home/username/.pyenv/shims:/home/username/.pyenv/bin:/usr/lib/ccache:.....
    ```
- on dirait que les versions de python installables par pyenv sont hard-codées, du coup, attention à distinguer :
    - les différentes versions de python qu'on peut installer avec pyenv
    - la version de pyenv lui-même (il faut mettre à jour pyenv pour lui faire "connaître" les versions les plus récentes de python)
- toutes les versions de python installées par pyenv semblent compilées from scratch, et installées dans :
    ```sh
    ~/.pyenv/versions/
    ```
- du coup, on dirait qu'il suffit de supprimer un de ces répertoires particuliers pour arrêter que pyenv gère une version :
    ```sh
    rm -rf ~/.pyenv/versions/2.7.15
    ```
- mais il existe aussi une commande uninstall :
    ```sh
    pyenv uninstall 2.7.15
    ```
- pyenv n'a pas l'air de nécessiter un python installé sur la machine (il semble codé en shell, et rebuilde lui-même les python) → il pourrait très bien wrapper d'autres langages (et le projet semble d'ailleurs émaner d'un équivalent en ruby)

# Utilisation

## quelques essais

- lister ce que pyenv peut installer comme version de python :
    ```sh
    pyenv install --list | wc -l  # 532 (!)
    pyenv install --list | grep 3\.5  # plein de py35 disponibles...
    ```
- installer (i.e. télécharger, et compiler depuis les sources) une version de python particulière :
    ```sh
    pyenv install 3.5.10
    ```
    - c'est assez long, car ça compile python depuis les sources
    - à l'issue de l'installation, je vérifie que `python3` continue d'être python 3.8 : installer py3.5 n'entraîne pas automatiquement son utilisation
    - en revanche, j'ai bien mon python 3.5.10 installé dans l'arborescence python de pyenv :
    ```sh
    ls -lh ~/.pyenv/versions/
    # 3.5.10
    ```
- je peux confirmer que mon python global est celui du système ; je peux changer de python global et revenir à celui du système :
    ```sh
    pyenv versions
    # * system (set by /home/username/.pyenv/version)
    #   3.5.10

    pyenv global 3.5.10
    pyenv versions
    #   system
    # * 3.5.10 (set by /home/username/.pyenv/version)

    pyenv global system
    pyenv versions
    # * system (set by /home/username/.pyenv/version)
    #   3.5.10
    ```
- je peux aussi modifier le python du shell courant (plus pratique que `local`, et moins invasif que `global`) :
    ```sh
    # STEP 1 = le python actuel est 'system' :
    pyenv version
    # system (set by /home/username/.pyenv/version)
    echo $PYENV_VERSION
    # [vide]

    # STEP 2 = je modifie le python du shell courant :
    pyenv shell 3.5.10
    pyenv version
    # 3.5.10 (set by PYENV_VERSION environment variable)
    echo $PYENV_VERSION
    # 3.5.10

    # STEP 3 = si besoin, retour manuel à la normale :
    PYENV_VERSION=""
    pyenv version
    # system (set by /home/username/.pyenv/version)
    ```
- dans tous les cas, *même quand pyenv est configuré pour utiliser le python système*, `python` pointe sur le shim pyenv (et il faut utiliser `pyenv which` pour savoir quelle est la commande réellement utilisée derrière) :
    ```sh
    pyenv versions
    # * system (set by /home/username/.pyenv/version)
    #   3.5.10

    which python
    # /home/username/.pyenv/shims/python
    pyenv which python
    # /usr/bin/python

    pyenv global 3.5.10
    which python
    # /home/username/.pyenv/shims/python  <--- ceci ne change pas
    pyenv which python
    # /home/username/.pyenv/versions/3.5.10/bin/python  <--- mais ceci change !
    ```

## shims et rehash

- on peut lister les shims (= les proxies qui permettent d'utiliser les exécutables de pyenv plutôt que système) connus de pyenv :
    ```sh
    pyenv shims
    # ... plein de trucs ...
    ```
- par exemple, `python3` est un shim, qui vient faire le proxy pour "remplacer" le python3 du système :
    ```sh
    which python3
    # /home/username/.pyenv/shims/python3
    pyenv shims
    # [...]
    # /home/username/.pyenv/shims/python3
    # [...]
    pyenv which python3
    # /home/username/.pyenv/versions/3.5.10/bin/python3
    ```
- certains shims peuvent exister pour pyenv mais pas pour le système :
    ```sh
    which python3.5
    # python3.5 not found
    pyenv shims
    # [...]
    # /home/username/.pyenv/shims/python3.5
    # [...]
    pyenv which python3.5
    # /home/username/.pyenv/versions/3.5.10/bin/python3.5
    ```
- dans le cas ci-dessus, la commande `python3.5` n'est disponible que si la version de python active est 3.5.10 ; donc si pyenv utilise le python système, il nous indique (fort logiquement) que `python3.5` n'existe pas
- cependant, pyenv (qui intercepte le call) sait que `python3.5` existe dans une AUTRE version de python que celle active, et nous l'indique gentiment :
    ```sh
    python3.5
    # pyenv: python3.5: command not found
    #
    # The `python3.5' command exists in these Python versions:
    #   3.5.10
    ```
- lorsque pyenv est installé, il n'a pas connaissance des commandes spécifiques à certains venvs (p.ex. une commande installée en tant que `entry_point` ou `console_scripts`)
    - du coup il ne peut pas disposer des shims adéquats, et faire sa tambouille magique en rendant la commande dispo quand on est dans le venv adéquat...
    - Pour corriger cela, on utilise :
    ```sh
    pyenv rehash
    ```
    - ce qui va créer les shims de toutes les commandes connues (et supprimer les shims qui avaient été ajoutés précédemments, mais qui n'existent plus dans aucune version de python ou venv)

## pyenv local

- `pyenv local` = permet de configurer pyenv pour toujours utiliser une version particulière de python si on est sous un répertoire donné :
    ```sh
    cd /tmp/mysuperproject

    # by default, version is the 'global' one :
    pyenv version
    # system (set by /home/username/.pyenv/version)

    # we can set a 'local' version :
    pyenv local 3.5.10
    pyenv version
    # 3.5.10 (set by /tmp/mysuperproject/.python-version)
    cat /tmp/mysuperproject/.python-version
    # 3.5.10

    # it is specific to the project folder, in another folder, the version is unchanged :
    cd /home/username/  # another folder
    pyenv version
    # system (set by /home/username/.pyenv/version)
    ```
- ça a l'air magique : dès qu'on est dans le répertoire (ou un de ses fils) ou bien qu'on le quitte, la version de python utilisée est mise à jour !
- comment ça fonctionne :
    - pyenv local se contente de créer un fichier `.python-version` décrivant la version souhaitée
    - derrière, une feature du shell (`precmd_functions`, cf. ci-dessous) permet d'exécuter des fonctions avant chaqe invite de commande, et pyenv ajoute une fonction d'activation de version
- détail sympa : `python versions` (et `python version`) indiquent _pourquoi_ la version actuelle a été choisie par pyenv :
    ```sh
    # version globale (valable partout sur le système) :
    pyenv version
    # system (set by /home/username/.pyenv/version)

    # version locale (propre au répertoire du projet) :
    pyenv version
    # 3.5.10 (set by /tmp/mysuperproject/.python-version)

    # version shell (valable uniquement dans le shell courant) :
    pyenv version
    # 3.10.1 (set by PYENV_VERSION environment variable)
    ```

## pyenv virtualenv

### utilisation TL;DR

```sh
# création du venv avec la version de python voulue :
pyenv virtualenv 3.5.10 mysuperenv

# configuration de myproject pour utiliser automatiquement mysuperenv :
cd /path/to/myproject/
pyenv local mysuperenv

# si besoin, au sein de myproject, on peut revenir temporairement sur le python système :
pyenv shell system
```

### comment ça marche

Ça n'est pas baseline avec pyenv, c'est un plugin. Ce que ça fait est décrit par [la doc sur github](https://github.com/pyenv/pyenv-virtualenv) :

> pyenv manages multiple versions of Python itself.
>
> virtualenv/venv manages virtual environments for a specific Python version.
>
> pyenv-virtualenv manages virtual environments for across varying versions of Python.

Utilisation :

- création du venv, en précisant la version de python (gérée par pyenv, of course) souhaitée :
    ```sh
    pyenv virtualenv 3.5.10 mysuperenv
    ```
- pyenv traite les virtualenvs un peu comme des versions particulières de python (c'est pas si idiot que ça, puisqu'il s'accompagne p.ex. de possibles nouveaux shims). Du coup, le virtualenv apparaît dans le résultat de `pyenv versions` :
    ```sh
    ll ~/.pyenv/versions
    # lrwxrwxrwx 1 username username   52 janv. 11 11:41 mysuperenv -> /home/username/.pyenv/versions/3.5.10/envs/mysuperenv
    pyenv versions
    # * system (set by /home/username/.pyenv/version)
    #   3.10.1
    #   3.5.10
    #   3.5.10/envs/mysuperenv
    #   mysuperenv
    ```
- du coup, on peut définir un virtualenv comme "version de python" locale d'un projet :
    ```sh
    cd /path/to/myproject/
    pyenv local mysuperenv
    pyenv version
    # mysuperenv (set by /path/to/myproject/.python-version)
    pyenv which python
    # /home/username/.pyenv/versions/mysuperenv/bin/python
    python -V
    # Python 3.5.10
    ```
- le truc cool : si le plugin pyenv-virtualenv est dans le `PATH` (`eval "$(pyenv virtualenv-init -)"`), avec `pyenv local` zsh reconnaît automatiquement qu'à l'intérieur du répertoire `myproject/`, je suis dans le venv `mysuperenv`
- du coup, de la même façon que je pouvais avoir une version de python automatique par projet, je peux avoir un virtualenv automatique par projet \o/

### qu'apporte pyenv activate ?

Le fait d'ajouter la ligne `eval "$(pyenv virtualenv-init -)"` au `.zshrc` permet d'activer automatiquement le venv lorsqu'on rentre dans le répertoire du projet...

Mais dans la mesure où `pyenv local` permet **déjà** d'activer une version de python, et que les venvs sont traités comme des versions de python, à quoi ça sert ?

D'après la doc, cette ligne du `.zshrc` permet d'appeler automatiquement `activate` au lieu de simplement définir la version de python...

La question demeure : à quoi ça sert ? Typiquement, quel intérêt de faire `pyenv activate` plutôt que `pyenv shell` ?







À quoi sert `pyenv activate mysuperenv` ?
- si on a la petite ligne dans zshrc, `pyenv activate` est lancé automatiquement lorsqu'on rentre dans le répertoire du projet
- (et si on n'a pas la ligne dans `.zshrc`, on peut appeler `pyenv activate` manuellement)
- on peut aussi appeler `pyenv activate` manuellement (et `deactivate` aussi) si on veut utiliser un venv ailleur que dans le répertoire du projet :
    ```sh
    cd /tmp/a_random_dir/
    pyenv version
    # system (set by /home/username/.pyenv/version)
    pyenv activate mysupervenv
    pyenv version
    # mysupervenv (set by PYENV_VERSION environment variable)
    pyenv deactivate
    pyenv version
    # system (set by /home/username/.pyenv/version)
    ```
- par rapport à `pyenv shell`, j'ai l'impression à le lecture [du code](https://github.com/pyenv/pyenv-virtualenv/blob/master/bin/pyenv-sh-activate) et avec quelques tests, que ça sert surtout à définir des envvars ; notamment `VIRTUAL_ENV` (utilisée p.ex. par zsh pour définir le PS1)
- j'ai vérifié que l'activation automatique (et le set de `VIRTUAL_ENV`) était bien apporté par la ligne dans `.zshrc` : si je l'enlève, le comportement apporté par `pyenv local` (activation automatique de la version de python lorsqu'on entre dans le répertoire) reste actif , mais PAS la définition automatique de `VIRTUAL_ENV`, il faut `pyenv activate` manuellement.

### comment deactivate ?

Question : comme le venv est activé automatiquement quand on rentre dans le répertoire du projet, comment le désactiver ?

Réponse : il faut en fait "activer" un python alternatif : `pyenv shell system` (sauf à supprimer la version locale, mais je pars du principe que c'est pas ce qu'on veut faire) ; du coup `pyenv deactivate` ou `source deactivate` ne servent à rien.

En fait, avec l'activation automatique du venv, le mécanisme est inversé par rapport à virtualenvwrapper/workon :

- avec `workon` : par défaut, j'utilise le python système, et je dois activer explicitement le venv
- avec `pyenv local` : par défaut, j'utilise le python du venv, et je dois faire une action explicite pour revenir sur le python système

----

Question bonus : quid si le venv a été activé manuellement (par exemple dans un répertoire random dans lequel on a fait `pyenv activate mysupervenv`) ?

Réponse : ici `pyenv deactivate` ou `source deactivate` resettent les variables d'environnement `PYENV_VERSION` et `VIRTUAL_ENV`, ce qui ramène l'état à sa situation globale.

### virtualenv, entry_points, shims et rehash

Le mécanisme utilisé par pyenv (shims) ne marche pas très bien quand on installe des binaires dans le virtualenv, par exemple via les `entry_points` :
- dans un venv classique, le fait d'activer le venv prepend le répertoire des binaires du venv au PATH, ce qui fait qu'à l'intérieur du venv, on peut utiliser le binaire ; mais avec pyenv, on n'est JAMAIS en situation où le répertoire du venv est dans le PATH : seuls le répertoire des shims de pyenv est dans le PATH !
- or, il n'y a pas de raison qu'il existe des shims pour ces binaires propres aux venvs...
- du coup, la conséquence, c'est que si `myproject` définit un `console_script` qui s'appelle p.ex. `do_the_work`, faire ce qui suit ne fonctionne PAS :
    ```sh
    cd /path/to/myproject
    pyenv virtualenv 3.10.1 mysupervenv
    pip install -e .
    do_the_work
    # ERROR : le shim n'existe pas, et le répertoire des binaires du venv n'est pas dans le PATH !
    which do_the_work
    # do_the_work not found
    ```
- (of course, la bonne solution n'est PAS d'utiliser le chemin absolu du venv : `~/.pyenv/versions/mysupervenv/bin/do_the_work`)
- la solution est plutôt de mettre à jour les shims de pyenv, pour lui ajouter un shim pour `do_the_work` :
    ```sh
    pyenv rehash
    do_the_work 
    # OK :-)
    which do_the_work
    # /home/username/.pyenv/shims/do_the_work
    ```
- en gros, `pyenv rehash` regarde dans tous les répertoires `bin` de tous les environnements qu'il connaît, et vérifie que ses shims sont bien à jour avec les exécutabiles qu'il trouve : il ajoute des nouveaux shims pour les exécutables qu'il ne connaît pas encore, et supprime les éventuels shims installés par le passé qui ne sont plus utilisés par aucun environnement pyenv.
- du coup, un shim donné n'est pas nécessairement disponible sur tous les envs ; pour connaître les environnements sur lequel le shim correspond à un vrai binaire : `pyenv whence`

**Important** : il y a donc une différence conceptuelle entre `source /path/to/mysupervenv/bin/activate` et `pyenv activate mysupervenv` :

- `source /path/to/mysupervenv/bin/activate` modifie dynamiquement le `PATH` pour prepend `/path/to/mysupervenv/bin/`, et permet donc de trouver les binaires de mysupervenv
- `pyenv activate mysupervenv` laisse le `PATH` inchangé (il avait déjà été modifié dans `.zshrc` pour prepend le chemin vers les shims), on se repose sur l'existence d'un shim "global" (à créer éventuellement avec `pyenv rehash`) pour rediriger sur les binaire de mysupervenv

## connexe = zsh precmd_functions

(c'est pas vraiment du pyenv, mais ça reste corrélé au sujet, et c'est intéressant)

Comment pyenv détecte-t-il magiquement la bonne version à utiliser quand on rentre dans un projet ? Plus compliqué : je constate que la variable `VIRTUAL_ENV` est mise à jour correctement _simplement en changeant de répertoire_, comment ça marche ?!

TL;DR : ça utilise une feature de zsh, les `precmd_functions`, qui lance des fonctions avant chaque invite de commande, cf. [la doc](https://zsh.sourceforge.io/Doc/Release/Functions.html) (voir aussi [cet autre lien](https://github.com/rothgar/mastering-zsh/blob/master/docs/config/hooks.md)):

> Hook Functions
>
> precmd
>
> Executed before each prompt. Note that precommand functions are not re-executed simply because the command line is redrawn, as happens, for example, when a notification about an exiting job is displayed.

En effet, la [doc de pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv/blob/29fb3c7cd1a1fce154407ea728e2e2aa63a36544/README.md) indique de `eval` des choses :

> Add `pyenv virtualenv-init` to your shell to enable auto-activation of virtualenv

Et quand je regarde le contenu de `pyenv virtualenv-init -`, je vois qu'il mets à jour `precmd_functions` :

```sh
[...]
typeset -g -a precmd_functions
if [[ -z $precmd_functions[(r)_pyenv_virtualenv_hook] ]]; then
  precmd_functions=(_pyenv_virtualenv_hook $precmd_functions);
fi
```

Comme ça a l'air spécifique à zsh, je ne sais pas trop comment bash s'en sort, mais j'imagine qu'il y a un équivalent...
