**Contexte** : 2022 = mon boulot impose Ubuntu 20.04 et surtout gnome-shell auquel je ne suis pas habitué, d'où ces notes vrac.

* [Les concepts de gnome-shell](#les-concepts-de-gnome-shell)
   * [vue des activités](#vue-des-activités)
   * [dash](#dash)
   * [vue des applications](#vue-des-applications)
* [Vrac](#vrac)

# Les concepts de gnome-shell

Tuto = https://doc.ubuntu-fr.org/gnome-shell

## vue des activités

- tout ce qu'on tape au clavier va dans la recherche (on peut retrouver des fenêtres ou des fichiers)
- liste des fenêtres (je cite : "Ce mécanisme permet théoriquement de se passer d'une barre de tâches, devenue superflue, pour naviguer entre les fenêtres.")
- depuis la vue des activités, tab permet de switcher entre les fenêtres du workspace affiché, et PageUp/PageDown (ou bien la molette de la souris) permet de switcher entres les workspaces)

On dirait que c'est le bouton "Actitivés" qui me montre toutes les fenêtres ouvertes d'un même workspace et me permet de choisir dedans

- j'y accède soit en cliquant tout en haut à gauche, soit en appuyant sur la touche Windows
- note : les fenêtres montrées sont celles du workspace courant (et il me montre sur le côté droit des aperçus des autres workspaces)
- (sinon, Alt+Tab fonctionne bien)

## dash

Dash est la barre latérale où j'ai mes "applications favorites"

- je peux glisser/déposer pour les réordonner
- je peux faire clic droit pour supprimer un favori
- ce qui est un peu trompeur, c'est qu'il y a DEUX types d'icônes dans le dash :
    - les "vrais" favoris (qui jouent aussi le rôle de lanceur d'application) : même si aucune instance de ce programme n'est en cours d'exécution, l'icône est dans le dash, pour permettre de lancer l'application
    - les "applications non-favorites en cours d'exécution" : dès que je ferme le programme, l'icône disparaît, elle ne servait qu'à matérialiser le fait que le programme était en cours d'exécution
- quand le dash est constamment visible (comme sur mon PC du boulot) on parle plutôt de dock (j'en déduis que sur d'autres versions de gnome-shell, le dash n'apparaît que dans la vue des applications)
- en bas du dock, j'ai le bouton pour accéder à la vue des applications

## vue des applications

vue des applications = liste de toutes les applications possibles
- toutes les applications = triées par ordre alphabétiques
- fréquemment utilisées = celles que j'utilise beaucoup
- tout ce qu'on tape au clavier va dans la recherche (on peut retrouver des applications)

raccourcis claviers par défaut = il y a en fait plein de trucs utiles !

- (note : `Super` est la touche Windows, entre `Fn` et `Alt`)
- `Super` = ouvrir la vue des activités
- `Super+A` = ouvrir la vue des applications
- `Super+M` = ouvrir la vue des notifications
- `Super+L` = ouvrir la vue des notifications
- `Alt+²` = alterner entre les fenêtres d'une MÊME application ! (le `²` est la touche au dessus de `TAB`, et en dessous d'`Escape`)

# Vrac

```sh
gnome-shell --version
# GNOME Shell 3.36.9
```

On dirait que j'ai toujours nb workspaces + 1 : dès que je créé qqch sur le workspace vide, ça instancie un nouveau workspace (c'est pas idiot, je trouve)

Les icônes à gauche ont un point rouge par fenêtre ouverte les concernant (tous workspaces confondus). Au final, c'est assez dur de s'y retrouver...

[Gnome tweaks](https://doc.ubuntu-fr.org/gnome-tweaks) (Gnome ajustements en français) permet de configurer gnome shell :

Voir aussi [ce tuto](https://doc.ubuntu-fr.org/tutoriel/personnaliser_gnome) pour la personnalisation.

Bon, après avoir pris 1h pour regarder de plus près gnome-shell, je pense que je pourrais clairement m'y habituer, il y a pas mal de trucs très pratiques. Ce que j'aime moins = c'est plus difficile qu'avant de se faire un modèle conceptuel des fenêtres qui tourne et de comment en changer ; essentiellement car on n'a plus constamment sous les yeux la liste des fenêtres d'un espace de travail (mais aussi car il y a plusieurs "vues" différentes sur les mêmes fenêtres)

Avec Ubuntu 20.04, les core-dumps sont centralisés (et il faut les droits sudo pour les supprimer) :

```sh
sudo rm /var/lib/apport/coredump/xxx.yyy
```

Dans le profil du terminal :

- je peux régler la largeur du terminal par défaut -> c'est celle qui est utilisée quand j'utilise le raccourci pour démarrer un nouveau terminal
- il faut désactiver "Bip du terminal" (car sinon j'ai des bips réguliers)

Réglé le souci qui faisait que le switch de workspace ne marchait que sur l'écran principal :

Ce qui NE MARCHE PAS ([lien](https://www.suse.com/support/kb/doc/?id=000018910)), car cette conf est ignorée :

```sh
sudo apt install dconf-editor
dconf-editor
# /org/gnome/shell/overrides/workspaces-only-on-primary
# à mettre à False (et tout sauvegarder)

# vérification :
dconf read /org/gnome/shell/overrides/workspaces-only-on-primary
# false
```

Ce qui marche ([lien](https://askubuntu.com/questions/837813/switching-workspaces-on-primary-display-only/1183794#1183794)) :

```sh
sudo apt install gnome-tweak-tool
gnome-tweaks
# Espaces de travail > choisir "Les espaces de travail couvrent les écrans"
# (au passage, gnome-tweaks permet de revenir à un nombre fixe d'espaces de travail)
```

Mes raccourcis actuels (possiblement, ce sont mes raccourcis persos et pas ceux par défaut) :

- `Ctrl+Alt+E` pour ouvrir terminal (TODO : la taille des terminaux ouverts est pas folle, je préfèrerais les voir prendre toute la largeur de l'écran)
- `Ctrl+Alt+F` pour ouvrir browser internet
- `Ctrl+Alt+H` pour ouvrir un filebrowser
- `Ctrl+Alt+up/down` pour naviguer entre les bureaux virtuels (et `Shift` pour y déplacer la fenêtre courante)
