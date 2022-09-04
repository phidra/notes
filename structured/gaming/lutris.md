**C'est quoi ?** Wrapper autour de plein de trucs (wine, steam, dosbox, etc.) permettant de n'avoir qu'un seul point d'entrée pour lancer tous ses jeux depuis Linux.

Plein de types de jeux sont supportés :

- jeux atari
- jeux snes
- jeux gamecube via l'émulateur Dolphin
- jeux dosbox
- jeux steam
- jeux GOG
- jeux Blizzard


# Installation

Via ppa (cf. https://lutris.net/downloads )

```sh
sudo add-apt-repository ppa:lutris-team/lutris
sudo apt update
sudo apt install lutris

lutris --version
# lutris-0.5.10.1
```

Installation de wine directement dans les repos (cf. https://doc.ubuntu-fr.org/wine ) :

```sh
sudo apt install wine-stable

wine --version
# wine-6.0.3 (Ubuntu 6.0.3~repack-1)
```

Une fois installé, la section "Hardware Information" des Lutris Settings a des infos intéressantes :

```
[System]
OS:              Ubuntu 22.04 Jammy Jellyfish
Arch:            x86_64
Kernel:          5.15.0-47-generic
Desktop:         XFCE
Display Server:  x11

[CPU]
Vendor:          GenuineIntel
Model:           Intel(R) Core(TM) i3-4340 CPU @ 3.60GHz
Physical cores:  2
Logical cores:   4

[Memory]
RAM:             7.5 GB
Swap:            2.0 GB

[Graphics]
Vendor:          Intel
OpenGL Renderer: Mesa Intel(R) HD Graphics 4600 (HSW GT2)
OpenGL Version:  3.1 Mesa 22.0.5
OpenGL Core:     4.6 (Core Profile) Mesa 22.0.5
OpenGL ES:       OpenGL ES 3.2 Mesa 22.0.5
Vulkan:          Supported
```

Notamment, on dirait bien que Vulkan est supporté (au moins partiellement).

Toujours d'après les settings, les jeux sont installés dans :

```
/home/myself/Games
```


# Premiers essais = jeux Steam

## Baba is You

- je lance steam
- depuis steam, je setup un profile (ce que je n'avais pas fait)
- je passe la liste de mes jeux en _Public_ (elle était _Friends Only_)
- je relance Lutris, et quand je clique sur steam, je vois bien tous mes jeux \o/ (ceux qui ne sont disponibles mais pas installés sont grisés)
- quand je lance une première fois Baba is You via Lutris en cliquant sur Play, ça lance steam, puis ça semble rester bloqué...
- je relance une deuxième fois, ça lance Baba is you (via steam)
- j'essaye de quitter Steam d'abord, puis de lancer Baba is you une troisième fois via Lutris, j'ai un message d'erreur :
    ```
    Warning - Steam was unable to sync your Baba is You saves with the Steam Cloud
    If you play now, you may not have previous game progress and you may permanently lose it.
    ```
   - (je clique sur Cancel plutôt que Play anyway)
- en redémarrant le jeu une quatrième fois, ce coup-ci c'est bon : Baba is you se lance, et je retrouve mes sauvegardes \o/

NOTE : apparemment, pour que le jeu soit considéré comme arrêté par Lutris, il faut non seulement quitter le jeu, mais également le client steam (que Lutris a lancé)

## Borderlands pre-sequel

Ça fonctionne correctement \o/

NOTE : "Processing Vulkan Shaders" au démarrage prend plein de temps (mais c'était déjà le cas avec Steam).

La [page Lutris d'Heroes of the Storm](https://lutris.net/games/heroes-of-the-storm/) a une note à mettre en relation :

> The installer provides a pre-generated DXVK state cache to provide stutter-free experience. As a result, it will cause a temporary performance hit while the shaders are being compiled. The FPS will rise back up after it has finished.

Donc en gros, les shaders semblent livrés avec HOTS et je n'aurais pas cette phase de précalcul au démarrage (mais parfois plus tard ?)

# Deuxième essai = HOTS

Le site Lutris a une page dédiée = https://lutris.net/games/heroes-of-the-storm/ :

```
Wine Standard version Rating: ✅ PLAYABLE last published 2 months, 3 weeks ago
**CONFIGURE AND INSTALL THE DEPENDENCIES FIRST**
Instructions for Vulkan support: https://github.com/lutris/lutris/wiki/Installing-drivers
Instructions for proper functionality of Battle.Net (+ common problems) https://github.com/lutris/lutris/wiki/Game:-Blizzard-App
**IMPORTANT NOTES**
- The installer provides a pre-generated DXVK state cache to provide stutter-free experience. As a result, it will cause a temporary performance hit while the shaders are being compiled. The FPS will rise back up after it has finished.

Heroes of the Storm is a multiplayer online battle arena video game developed and published by Blizzard Entertainment for Microsoft Windows and macOS that was released on June 2, 2015.
To play use instructions given in 'battlenet' installer, and copy all content of Heroes of the Storm/Support64/* to drive_c/windows/system32 folder.
Also, using winetricks, install d3dcompiler_47 and d3d11_43, tested in Fedora Core 31 with nvidia GTX 1050
```


Comme d'après la propre page d'info Hardware de Lutri, le support Vulkan a déjà l'air actif (pour mon GPU Intel intégré à mon processeur), je commence par m'intéresser à la [page d'aide Lutris sur bnet](https://github.com/lutris/docs/blob/master/Battle.Net.md) :

> Wine dependencies are required for Overwatch to work. Please follow the instructions on Wine Dependencies page to get them.  https://github.com/lutris/docs/blob/master/WineDependencies.md

Ça c'est déjà ok : j'ai installé wine depuis le package-manager d'Ubuntu.

> Battle.net requires up-to-date, Vulkan capable graphics drivers. For instructions on how to install them, refer to How to: Installing Drivers.  https://github.com/lutris/lutris/wiki/Installing-drivers

Ça aussi c'est déjà ok, d'après le Hardware infos des Lutris settings.

Du coup, je clique sur "Install" sur la [page web de HOTS](https://lutris.net/games/heroes-of-the-storm/), ce qui lance l'installeur avec Lutris.

- il télécharge divers trucs (notamment... un truc "Overwatch" ?)
- il mets à jour Wine
- ça commence à marcher : je vois des popups bnet apparaître \o/
- et il semble commencer à mettre bnet à jour \o/

Bon... la mise à jour semble freezée à mi-parcours... je laisse tourner un peu pour voir si ça se débloque, et après un temps d'attente important (je dirais 5 à 10 minutes environ), j'ai une popup avec ce message :

```
Battle.net Installation
We're having trouble launching the Battle.net Update Agent.
Please wait one minute and try again.
If it happens again, try restarting your computer or reinstalling Battle.net
More help: BLZBNTBTS000000005C  (qui pointe sur nydus.battle.net/App/enUS/setup/error/BLZBNTBTS0000005C)
Please click the link above for more information or contact Customier Support if the problem persists.
```

Du coup, je cancel l'installation Lutris (sans supprimer les game files).

Je retente une deuxième fois l'installation (pour le moment sans rebooter, et directement depuis Lutris plutôt que depuis le site web), ça plante encore plus tôt, à l'étape _Installing Arial fonts_ (mais je soupçonne après coup que ce soit parce que je n'avais pas supprimé les Game files) :

```
Started initial process 24127 from /home/myself/.local/share/lutris/runtime/winetricks/winetricks --unattended arial
Start monitoring process.
Executing mkdir -p /home/myself/Games
------------------------------------------------------
warning: You are using a 64-bit WINEPREFIX. Note that many verbs only install 32-bit versions of packages. If you encounter problems, please retest in a clean 32-bit WINEPREFIX before reporting a bug.
------------------------------------------------------
------------------------------------------------------
WINEPREFIX INFO:
Drive C: total 28
drwxrwxr-x  7 myself myself 4096 sept.  3 09:43 .
drwxrwxr-x  5 myself myself 4096 sept.  3 10:02 ..
drwxrwxr-x  4 myself myself 4096 sept.  3 09:44 ProgramData
drwxrwxr-x  6 myself myself 4096 sept.  3 09:43 Program Files
drwxrwxr-x  6 myself myself 4096 sept.  3 09:43 Program Files (x86)
drwxrwxr-x  4 myself myself 4096 sept.  3 09:43 users
drwxrwxr-x 21 myself myself 4096 sept.  3 09:44 windows
Registry info:
/home/myself/Games/heroes-of-the-storm/system.reg:#arch=win64
/home/myself/Games/heroes-of-the-storm/user.reg:#arch=win64
/home/myself/Games/heroes-of-the-storm/userdef.reg:#arch=win64
------------------------------------------------------
------------------------------------------------------
warning: /home/myself/.local/share/lutris/runners/wine/lutris-7.2-2-x86_64/bin/wine cmd.exe /c echo '%AppData%' returned empty string, error message "" 
------------------------------------------------------
Monitored process exited.
Initial process has exited (return code: 256)
All processes have quit
Exit with return code 256
```

Ce coup-ci, au moment de Cancel, je supprime les Game files (ce que je n'avais pas fait le coup d'avant).

Je retente une troisème fois, depuis le site de nouveau.

- Ça passe l'étape qui a planté le coup d'avant (Installing Arial fonts) (EDIT : sans doute parce que j'ai supprimé les game files)
- Je reviens à Updating Battle.net Update Agent.
- Ça semble bloqué de nouveau, mais je vais attendre au moins la fin.
- Et boum, je confirme que j'ai la même popup.

Une recherche sur google m'autocomplète `lutris updating battle.net update agent` ; [ce post](https://www.reddit.com/r/wine_gaming/comments/bl0cmk/help_battlenet_update_agent/) me suggère d'installer les drivers 32 bits : `Do you have 32-bit support installed for WINE and dxvk?` +  `FIXED THE ISSUE! For some reason, I did not have all the relevant 32-bit libraries, so I ran the following command: sudo apt install libnvidia-gl-418:i386`


Donc avant d'acheter une CG, j'ai encore un ou deux trucs à essayer : rebooter + m'assurer que j'ai installé les drivers mesa 32 bits.

Première tentative = rebooter ne corrige pas, l'installeur reste bloqué au même endroit, pour finir sur la même erreur 05C.

Deuxième tentative = installer les drivers 32bits pour ma carte graphique intégrée à mon processeur :

- la [page Lutris sur l'installation des drivers](https://github.com/lutris/docs/blob/master/InstallingDrivers.md) donne une commande pour installer tous les drivers :
    ```
    sudo add-apt-repository ppa:kisak/kisak-mesa && sudo dpkg --add-architecture i386 && sudo apt update && sudo apt upgrade && sudo apt install libgl1-mesa-dri:i386 mesa-vulkan-drivers mesa-vulkan-drivers:i386
    ```
- mais bon, ça sent mauvais car ils indiquent aussi :
    > Note for Intel integrated graphics users: Only Skylake and newer Intel CPUs (processors) offer full Vulkan support. Broadwell, Haswell and Ivy Bridge only offer partial support, which will very likely not work with a lot of games properly. Sandy Bridge and older lack any Vulkan support whatsoever.
- Or, mon CPU est Haswell :
    - [ce lien](https://www.intel.fr/content/www/fr/fr/products/sku/77771/intel-core-i34340-processor-4m-cache-3-60-ghz/specifications.html) indique _Nom du code = Produit anciennement Haswell_
    - [cette page](https://unix.stackexchange.com/questions/230634/how-to-find-out-intel-architecture-family-from-command-line) donne des commandes :
        ```sh
        gcc -march=native -Q --help=target|grep march
        # -march=                     		haswell
        cat /sys/devices/cpu/caps/pmu_name
        # haswell
        ```
- je choisis de ne pas tenter la commande suggérée :
    ```sh
    sudo add-apt-repository ppa:kisak/kisak-mesa && sudo dpkg --add-architecture i386 && sudo apt update && sudo apt upgrade && sudo apt install libgl1-mesa-dri:i386 mesa-vulkan-drivers mesa-vulkan-drivers:i386
    ```
- en effet, les 3 paquets qu'elle installe sont **déjà** installés sur mon poste :
    ```sh
    dpkg -l |grep "mesa-"
    ii  libgl1-mesa-dri:amd64                     22.0.5-0ubuntu0.1                       amd64        free implementation of the OpenGL API -- DRI modules
    ii  libgl1-mesa-dri:i386                      22.0.5-0ubuntu0.1                       i386         free implementation of the OpenGL API -- DRI modules
    ii  libgl1-mesa-glx:amd64                     22.0.5-0ubuntu0.1                       amd64        transitional dummy package
    ii  mesa-utils                                8.4.0-1ubuntu1                          amd64        Miscellaneous Mesa utilities -- symlinks
    ii  mesa-utils-bin:amd64                      8.4.0-1ubuntu1                          amd64        Miscellaneous Mesa utilities -- native applications
    ii  mesa-va-drivers:amd64                     22.0.5-0ubuntu0.1                       amd64        Mesa VA-API video acceleration drivers
    ii  mesa-va-drivers:i386                      22.0.5-0ubuntu0.1                       i386         Mesa VA-API video acceleration drivers
    ii  mesa-vdpau-drivers:amd64                  22.0.5-0ubuntu0.1                       amd64        Mesa VDPAU video acceleration drivers
    ii  mesa-vulkan-drivers:amd64                 22.0.5-0ubuntu0.1                       amd64        Mesa Vulkan graphics drivers
    ii  mesa-vulkan-drivers:i386                  22.0.5-0ubuntu0.1                       i386         Mesa Vulkan graphics drivers
    ```
- de plus, l'installation du PPA indique que le support pour Ubuntu 22.04 = Jammy (mon poste actuel) est limité :
    ```sh
    sudo add-apt-repository ppa:kisak/kisak-mesa && sudo dpkg --add-architecture i386 && sudo apt update && sudo apt upgrade && sudo apt install libgl1-mesa-dri:i386 mesa-vulkan-drivers mesa-vulkan-drivers:i386
    [sudo] Mot de passe de myself : 
    PPA publishes dbgsym, you may need to include 'main/debug' component
    Dépôt : « deb https://ppa.launchpadcontent.net/kisak/kisak-mesa/ubuntu/ jammy main »
    Description :
    The goal of this PPA is to provide the latest point release of Mesa plus select non-invasive early backports. Deviations from upstream packages are listed on the package details page.
    
    --- Support status ---
    
    Bionic (18.04) - Supported (Please use the HWE X and kernel or equivalent)
    Focal (20.04) - Supported
    Impish (21.10) - End of Life, removal after 2022-08-26
    Jammy (22.04) - Preliminary support (Not tested locally)
    
    Note: Please report any issues to mesa. ARM builds are not tested locally.
    
    --- Is this PPA stable? ---
    
    Short answer: Mostly.
    
    Long answer: Compared to bleeding edge mesa PPAs, there is a much lower chance of complications, but as Ubuntu LTS releases age, the odds of unexpected interactions between older kernels / compilers / X increases with each major release. Anomalies are inevitable the further the host system gets from the configuration active mesa developers use on a daily basis. If you value stability over support, https://launchpad.net/~kisak/+archive/ubuntu/turtle/ is available as an alternative.
    
    In the event that there is a major issue with new mesa on an older Ubuntu LTS and the mesa devs are not interested in triaging the issue, then the Ubuntu LTS release will be dropped from this PPA and the last release pushed to kisak-mesa stable will be frozen.
    
    --- Package status ---
    
    llvm - Following latest point release supported by mesa
    mesa - Following latest point release
    libclc - Obsolete, now provided by llvm packaging
    libdrm - Updating as needed for mesa
    
    --- Removal ---
    
    It's strongly recommended to remove this PPA before upgrading to a newer Ubuntu release or using another mesa PPA.
    
    sudo apt install ppa-purge
    sudo ppa-purge ppa:kisak/kisak-mesa
    
    Note: Using ppa-purge with Ubuntu derivatives needs to include -d <based_on_name> to work safely. For example, Linux Mint 20 is based on Ubuntu Focal, so that would make it:
    
    sudo ppa-purge -d focal ppa:kisak/kisak-mesa
    
    --- Donations ---
    
    I can't accept donations and any random donation site account is a scam. If you have some extra money burning a hole in your pocket, please consider sending it to a charity of your choice (for the poors, animals, whatever else you may think it might need it), then send Oibaf a note that I stole the suggestion from his PPA.
    Plus d'informations : https://launchpad.net/~kisak/+archive/ubuntu/kisak-mesa
    Ajout du dépôt.
    Appuyez sur [ENTRÉE] pour continuer ou Ctrl-c pour annuler
    ```

- Du coup, j'annule la commande...

Au final, Lutris même si Lutris semble pas mal marcher, je pense que l'installation de bnet échoue à cause de mon GPU Intel intégré à mon processeur, trop ancien...
