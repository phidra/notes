* [Utilisation](#utilisation)
* [utiliser le plugin vidéo glide64 au lieu de rice](#utiliser-le-plugin-vidéo-glide64-au-lieu-de-rice)
* [configuration des manettes pour Mupen64plus](#configuration-des-manettes-pour-mupen64plus)
   * [Nouvelle manette Logitech F310](#nouvelle-manette-logitech-f310)
   * [Ancienne manette Logitech Dual Action](#ancienne-manette-logitech-dual-action)
   * [Copie de l'état initial](#copie-de-létat-initial)


# Utilisation

Lancer une ROM :

```sh
mupen64plus /path/to/OcarinaOfTime.n64
```

Les sauvegardes sont dans :

```
/home/myself/.local/share/mupen64plus/save/OcarinaOfTime.st0
```

Et tant que j'y suis, les shortcuts :
- touches `1` à `9` = choisir le slot courant
- `F5` = save l'état mupen64 dans le slot courant
- `F7` = load l'état mupen64


# utiliser le plugin vidéo glide64 au lieu de rice

Sur mes deux PC, glide64 était moins buggé que rice. Pour l'utiliser, rien d'autre à faire que :

```sh
sudo vim ~/.config/mupen64plus/mupen64plus.cfg

# remplacer ceci :
VideoPlugin = "mupen64plus-video-rice.so"

# par cela :
VideoPlugin = "mupen64plus-video-glide64"
```

(rien d'autre à faire, il configure le bouzin tout seul)

# configuration des manettes pour Mupen64plus

On peut définir plusieurs commandes sur la même ligne (pour que chaque plusieurs boutons de la manette différents aient le même effet)

Note : zsnes a un mode pour enregistrer les boutons dynamiquement, c'est plus simple.

Les correspondances sont à définir dans :

```
/usr/share/games/mupen64plus/InputAutoCfg.ini
```

## Nouvelle manette Logitech F310

C'est la manette la plus récente (avec les gachettes analogiques)

Les correspondances :

```
axis(1+)     = joystick gauche vers le bas
axis(1-)     = joystick gauche vers le haut
	c'est probablement ceci qui définit quelle est le joystick qui contrôle le joypad 64 :
		X Axis = axis(0-,0+)
		Y Axis = axis(1-,1+)
	Et on dirait que le coin de référence est en haut à gauche de l'image :
		0- = vers la gauche
		0+ = vers la droite
		1- = vers le haut
		1+ = vers le bas
button(5)    = R (en bouton poussoir, pas en gachette analogique)
hat(0 Right) = croix directionnelle droite
hat(0 Left)  = croix directionnelle gauche
hat(0 Down)  = croix directionnelle bas
hat(0 Up)    = croix directionnelle haut
----------------------------------------
L'axis 2 a l'air d'être la gachette analogique de gauche
L'axis 3 a l'air d'être le joystick de droite horizontal.
L'axis 4 a l'air d'être le joystick de droite vertical.
L'axis 5 a l'air d'être la gachette analogique de droite
button(7) est le bouton start
button(6) est le bouton select
button(1) est le bouton B de la manette
button(3) est le bouton Y de la manette
button(0) est le bouton A de la manette
button(2) est le bouton X de la manette
button(4) est le L (en bouton poussoir, pas en gachette analogique)
button(5) est le R (en bouton poussoir, pas en gachette analogique)
```

Commandes souhaitées :

- le bouton Z est déclenché par l'une ou l'autre des gachettes analogiques
- les boutons L et R sont normaux
- sur la manette Logitech, A et X correspondent à A et B de la N64, ce qui est le plus intuitif) (et Y et B de la manette ne servent à rien)
- le joystick de droite sert à utiliser les 4 boutons C

Du coup, fort de tout ça, je peux configurer les commandes souhaitées pour la maentte Logitech F310 :

```
[Logitech Gamepad F310]
plugged = True
plugin = 2
mouse = False
AnalogDeadzone = 4096,4096
AnalogPeak = 32768,32768
DPad R = hat(0 Right)
DPad L = hat(0 Left)
DPad D = hat(0 Down)
DPad U = hat(0 Up)
Start = button(7)
Z Trig = axis(2+) axis(5+)
B Button = button(2)
A Button = button(0)
C Button R = axis(3+)
C Button L = axis(3-)
C Button D = axis(4+)
C Button U = axis(4-)
R Trig = button(5)
L Trig = button(4)
Mempak switch =
Rumblepak switch =
X Axis = axis(0-,0+)
Y Axis = axis(1-,1+)
```


**EDIT** = modification des boutons C : avec la config ci-dessus, ça arrive trop souvent de misclick et d'appuyer p.ex. sur C-gauche en même temps que C-haut. C'est assez casse-bonbons pour jouer les mélodies d'Ocarina of Time, du coup je change la config afin que le joystick de droite ne commande que C-haut et C-bas (C-gauche et C-droite étant commandés par les deux boutons inutilisés). La nouvelle config des boutons C est :

```
C Button R = button(1)
C Button L = button(3)
C Button D = axis(4+)
C Button U = axis(4-)
```

## Ancienne manette Logitech Dual Action

C'est la manette la plus ancienne (avec les gachettes tout ou rien).

Les correspondances sur l'ancienne manette Logitech Dual Action :

```
[Logitech Dual Action]
button(0) est le bouton "1" de la manette
button(1) est le bouton "2" de la manette
button(2) est le bouton "3" de la manette
button(3) est le bouton "4" de la manette
button(4) est le bouton "5" de la manette = le bouton L le plus haut
button(5) est le bouton "6" de la manette = le bouton R le plus haut
button(6) est le bouton "7" de la manette = le bouton L le plus bas (celui de la gachette analogique s'il en avait une)
button(7) est le bouton "8" de la manette = le bouton R le plus bas (celui de la gachette analogique s'il en avait une)
button(8) est le bouton "9" de la manette (= à l'emplacement habituel de select)
button(9) est le bouton "10" de la manette (= à l'emplacement habituel de start)
```

Du coup, fort de tout ça, je peux configurer les commandes souhaitées pour la manette Logitech Dual Action (EDIT : en simplifiant les sticks C pour OcarinaOfTime) :

```
[Logitech Dual Action]
plugged = True
plugin = 2
mouse = False
AnalogDeadzone = 4096,4096
AnalogPeak = 32768,32768
DPad R = hat(0 Right)
DPad L = hat(0 Left)
DPad D = hat(0 Down)
DPad U = hat(0 Up)
Start = button(9)
Z Trig = button(6) button(7)
B Button = button(0)
A Button = button(1)
# utiliser les axes pour C-gauche et C-droite n'est pas pratique pour OcarinaOfTime :
#C Button R = axis(2+)
#C Button L = axis(2-)
C Button R = button(2)
C Button L = button(3)
C Button D = axis(3+)
C Button U = axis(3-)
R Trig = button(5)
L Trig = button(4)
Mempak switch =
Rumblepak switch =
X Axis = axis(0-,0+)
Y Axis = axis(1-,1+)
```

## Copie de l'état initial

```
[Logitech Gamepad F310]
plugged = True
plugin = 2
mouse = False
AnalogDeadzone = 4096,4096
AnalogPeak = 32768,32768
DPad R = hat(0 Right)
DPad L = hat(0 Left)
DPad D = hat(0 Down)
DPad U = hat(0 Up)
Start = button(7)
Z Trig = button(5)
B Button = button(2)
A Button = button(0)
C Button R = axis(3+)
C Button L = axis(3-)
C Button D = axis(4+)
C Button U = axis(4-)
R Trig = axis(5-)
L Trig = axis(2-)
Mempak switch = button(1)
Rumblepak switch = button(3)
X Axis = axis(0-,0+)
Y Axis = axis(1-,1+)

(...)

[Logitech Dual Action]
plugged = True
plugin = 2
mouse = False
AnalogDeadzone = 4096,4096
AnalogPeak = 32768,32768
DPad R = axis(4+) hat(0 Right)
DPad L = axis(4-) hat(0 Left)
DPad D = axis(5+) hat(0 Down)
DPad U = axis(5-) hat(0 Up)
Start = button(9)
Z Trig = button(6)
B Button = button(0)
A Button = button(1)
C Button R = axis(2+)
C Button L = axis(2-)
C Button D = axis(3+)
C Button U = axis(3-)
R Trig = button(5)
L Trig = button(4)
Mempak switch = button(8)
Rumblepak switch = button(7)
X Axis = axis(0-,0+)
Y Axis = axis(1-,1+)
```
