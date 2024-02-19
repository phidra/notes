**Contexte** = février 2024, je tombe sur [carapace-bin](https://github.com/rsteube/carapace-bin/) qui a l'air intéressant pour la complétion de commandes ([doc](https://rsteube.github.io/carapace-bin/carapace-bin.html)).

Installation :

- sur [la page des releases](https://github.com/rsteube/carapace-bin/releases), à la date où j'essaye le tool, la version la plus récente est la `0.30.1`, donc je télécharge [la tarball](https://github.com/rsteube/carapace-bin/releases/download/v0.30.1/carapace-bin_linux_amd64.tar.gz) de mon architecture
- la tarball contient essentiellement le binaire `carapace`, que je copie dans un chemin de mon `PATH` :
    ```sh
    cp /tmp/carapace ~/.local/bin
    ```
- je modifie mon `.zshrc` :
    ```sh
    # 2024-02 = carapace-bin = completion :
    # cf. https://github.com/rsteube/carapace-bin
    export CARAPACE_BRIDGES='zsh,fish,bash,inshellisense' # optional
    zstyle ':completion:*' format $'\n\e[2;37m==== Completing %d\e[m'
    source <(carapace _carapace)
    ```

J'essaye, puis finis par abandonner le tool :

- on dirait qu'il merdoie ma complétion avec nvim : j'ai un alias `nvim="nvim -p"`, qui semble confuser carapace-bin... Je contourne en remplaçant cet alias par une autocommand sur `VimEnter` qui active les tabs
- puis, je constate que la complétion de `git show` fait planter le tool...

Au final, ça fait un ratio "feature apportée vs. galère" trop important, j'abandonne l'outil pour le moment, je garderai peut-être un oeil dessus.

