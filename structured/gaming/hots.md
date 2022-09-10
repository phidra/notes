**Contexte** : septembre 2022, j'essaye de pouvoir jouer à Heroes of the Storm directement depuis Linux.

* [Installation et lancement](#installation-et-lancement)
* [Notes à l'usage](#notes-à-lusage)
   * [erreur au démarrage](#erreur-au-démarrage)
   * [améliorer les perfs](#améliorer-les-perfs)
   * [Mise à jour systématique](#mise-à-jour-systématique)
   * [Hors-plage au lancement de HOTS](#hors-plage-au-lancement-de-hots)

# Installation et lancement

D'où je viens :

- j'ai essayé de lancer le jeu via lutris, mais sans succès : l'installateur de battle.net bloque en cours de route
- j'ai acheté une carte graphique récente (car le GPU intégré à mon processeur Intel semblait trop vieux pour être correctement utilisé par wine/vulkan)
- mais j'ai le même problème -> j'essaye donc de jouer "manuellement" (i.e. sans passer par lutris) via wine


Pour l'essentiel, j'ai suivi cette page : https://linuxconfig.org/how-to-install-battle-net-on-ubuntu-22-04-linux-desktop :

```sh
sudo apt install wine64 winbind winetricks
```

C'était pas archi-clair, mais j'ai l'impression d'avoir installé/configuré (e.g. installation d'une font, ou encore d'`ie8̀` et `vcrun2015`) des trucs manuellement pour le préfixe 32 bits (et ce qui est pas clair, c'est que pourtant, je lance le jeu avec le préfixe par défaut, 64 bits ! Du coup, comme les **drive_c** sont indépendants, je ne comprends pas à quoi ça a servi d'installer des trucs dans le préfixe 32 bits ?!)

Derrière, j'ai commencé par installer battle.net :

```sh
wine64 ~/.wine/drive_c/Program\ Files\ \(x86\)/Battle.net/Battle.net.exe
```

Puis, via bnet, j'installe Heroes of the Storm, puis je quitte et je le lance directement :

```sh
wine64 ~/.wine/drive_c/Program\ Files\ \(x86\)/Heroes\ of\ the\ Storm/Heroes\ of\ the\ Storm.exe
```

# Notes à l'usage

## erreur au démarrage

J'ai une erreur au démarrage (mais ça n'a pas l'air de poser souci : je peux jouer \o/) :

```
https://eu.battle.net/support/fr/article/26566
L’application Battle.net nécessite l’activation du service d’ouverture de session secondaire.
Code d'erreur: BLZBNTBTS00000025


Battle.net nécessite l'activation du service d'ouverture de session secondaire de Windows.
Veuillez cliquer sur le code d'erreur ci-dessous pour obtenir des instructions sur la procédure d'activation.
BLZBNTBTS00000025

https://eu.battle.net/support/fr/article/26566
```

## améliorer les perfs

À l'usage, ça marche pas trop mal, mais le jeu a tendance a saccader un peu dans les gros teamfights... Du coup, quelques ressources pour améliorer les perfs :

- Plusieurs astuces à essayer ici : https://wiki.winehq.org/Performance
- Cette page a des astuces : https://appdb.winehq.org/objectManager.php?sClass=version&iId=32232
- Notamment, ils pointent vers ceci pour DXVK : https://linuxconfig.org/improve-your-wine-gaming-on-linux-with-dxvk
- Autre ressource : https://askubuntu.com/questions/313670/why-are-my-games-so-slow-through-wine

## Mise à jour systématique

Quand je démarre bnet, HOTS est systématiquement considéré comme "en train de se mettre à jour". Il faut quelques dizaines de secondes pour qu'il se rende compte qu'il n'y a pas de màj, et permettre de lancer le jeu.

(je ne me souviens plus si c'était le cas aussi sous windows)

## Hors-plage au lancement de HOTS

À noter aussi : parfois, quand je lance le jeu, bnet se lance correctement, mais quand je lance HOTS, l'écran s'éteint et affiche `Hors-plage`.

Le jeu semble lancé quand-même (j'entends le son, et je peux même quitter proprement le jeu avec Alt+F4).

Seul workaround trouvé jusqu'ici = rebooter le PC (rebooter bnet ou wine ne suffit pas).
