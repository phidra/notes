**Contexte** : septembre 2022, j'essaye de pouvoir jouer à Heroes of the Storm directement depuis Linux en utilisant wine.

# wine vs. winecfg vs. winetricks

TL;DR :

- `wine` (ou `wine64`) est ce qui permet de lancer des programmes windows
- `winecfg` est une GUI permettant de configurer wine facilement
- `winetricks` permet d'installer facilement des dépendances (e.g. des bibliothèques) dans wine.

La [doc générale de wine](https://www.winehq.org/documentation) ; à l'occasion, lire [la doc utilisateur](https://wiki.winehq.org/Wine_User's_Guide) ou [la FAQ](https://wiki.winehq.org/FAQ) de wine sera sans doute instructif !

[Doc de winecfg](https://wiki.winehq.org/Winecfg).

[Doc de winetricks](https://wiki.winehq.org/Winetricks), [page ubuntu sur winetricks](https://doc.ubuntu-fr.org/winetricks)


Note : sans `winecfg`, on configurerait wine directement avec l'éditeur de registre : `wine64 regedit`

# Préfixe wine

J'ai l'impression que wine fonctionne par **préfixe** qui sont en quelque sorte les environnements virtuels de `wine`.

Pour utiliser un préfixe, il suffit de passer l'envvar sur la ligne de commande :

```sh
WINEPREFIX=~/.local/share/wineprefixes/32 winecfg
```

(et pour `winetricks`, la première fenêtre permet de choisir le préfixe à utiliser)


Chaque préfixe est une installation indépendante, du coup les **drive_c** sont indépendants :

- le préfixe par défaut est `~/.wine`, et son **drive_c** est donc `~/.wine/drive_c`
- la page que j'ai suivie pour installer HOTS m'a demandé d'installer un préfixe (en 32bits) appelé `32`, je le trouve dans : `~/.local/share/wineprefixes/32`, son **drive_c** est donc `~/.local/share/wineprefixes/32/drive_c`

Du coup, je peux rechercher quelque chose dans le disque dur (p.ex. le lanceur de HOTS) :

```sh
find "$HOME/.wine/drive_c/Program Files (x86)" -name \*.exe
# [...]
# /home/myself/.wine/drive_c/Program Files (x86)/Heroes of the Storm/Heroes of the Storm.exe
# /home/myself/.wine/drive_c/Program Files (x86)/Windows Media Player/wmplayer.exe
# /home/myself/.wine/drive_c/Program Files (x86)/Internet Explorer/iexplore.exe
# /home/myself/.wine/drive_c/Program Files (x86)/Windows NT/Accessories/wordpad.exe
# [...]
```

Et derrière, je peux lancer directement le programme :

```sh
wine64 ~/.wine/drive_c/Program\ Files\ \(x86\)/Heroes\ of\ the\ Storm/Heroes\ of\ the\ Storm.exe
```
