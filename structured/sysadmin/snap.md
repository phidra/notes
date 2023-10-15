Notes diverses et un peu vrac sur les [snap](https://doc.ubuntu-fr.org/snap) utilisés par ubuntu.

# Fonctionnement

Les snaps sont dans `/snap/`.

Lister les snaps installés :

```
snap list
```

Des explications générales sont dans `/snap/README`

Pour avoir des détails sur un snap en particulier :

```
snap info chromium
```

# Snaps inaccessibles au démarrage

Ça m'est arrivé sans que je comprenne trop pourquoi (peu après un `sudo apt autoremove` qui avait supprimé des vieux kernels).

Manifestation du problème = mon raccourci pour chromium ne répond plus ; `snap list` tourne indéfiniment sans rien renvoyer.

Analyse : `systemctl status snapd` montre que snapd est **loaded** mais **inactive (dead)**

Workaround :

```
sudo systemctl restart snapd
```

# tmpfs accessible à chromium

Contexte : septembre 2023, j'essaye d'avoir un tmpfs utilisable avec le snap chromium ; je n'arrive pas à faire ce que je veux, mais je logge quand même mes tentatives.

## Pourquoi vouloir un tmpfs ?

`/tmp` fait ce que je veux, notamment, il se vide à chaque reboot pour éviter de grossir inutilement. (`~/temporaire` complète bien si je veux des trucs pas très pérennes, mais tout de même à conserver entre chaque reboot)

Problème : à cause des permissions snap, chromium n'a pas accès à `/tmp`

À noter que c'est un problème commun aux snaps ; [la page du wiki ubuntu](https://doc.ubuntu-fr.org/snap#contournement_des_repertoires) sur les snaps donne des billes sur le sujet.

## Préambule = deux chromium sur ma machine

Je me rends compte que j'ai DEUX chromium sur mon poste :

- `/usr/bin/chromium-browser` (qui est lancé quand je tape `chromium-browser` en ligne de commande)
    ```
    Version 112.0.5615.49 (Build officiel) Built on Ubuntu , running on Ubuntu 20.04 (64 bits)
    ```
    - au démarrage, il affiche un message qui montre que ce n'est pas mon navigateur : "Chromium n'est pas votre navigateur par défaut"
- `/snap/bin/chromium` = le chromium lancé quand je clique sur le raccourci dash (qui se trouve être un snap) :
    ```
    Version 116.0.5845.187 (Build officiel) snap (64 bits)
    ```

D'après [la doc ubuntu de chromium-browser](https://doc.ubuntu-fr.org/chromium-browser), la version disponible dans les dépôts officiels installe le paquet snap de Chromium. Un PPA officiel permet d'installer un paquet deb, voir ci-dessous.

J'avais apparemment installé le deuxième (non-snap) moi-même, donc, à partir du PPA car je vérifie que j'ai bien le PPA :

```
/etc/apt/sources.list.d/phd-ubuntu-chromium-browser-focal.list
    deb http://ppa.launchpad.net/phd/chromium-browser/ubuntu focal main
    # deb-src http://ppa.launchpad.net/phd/chromium-browser/ubuntu focal main
```

J'ai bien les deux versions affichées dans la liste des applications ubuntu : on dirait que j'avais juste l'habitude de lancer celui de snap plutôt que celui du PPA...

Question : comment savoir quel chromium est lancé par l'icône dash ?
- pour mon cas particulier, j'ai pu la déduire (en regardant la version de l'application lancée) mais comment savoir dans le cas général ?
- apparemment, c'est ce fichier qui la définit :
    ```
    /usr/share/applications/chromium-browser.desktop
    ```
- mais son contenu se contente de lancer chromium-browser, du coup je ne comprends pas pourquoi/comment c'est le snap qu'il lance...

## Mes essais

### essai 1 = monter un tmpfs dans un emplacement accessible au chromium de snap

Commandes :

```
mkdir ~/tmpfs/
chmod 777 ~/tmpfs
sudo vim /etc/fstab
# tmpfs /path/to/tmpfs tmpfs defaults,size=1g 0 0
```

(j'ai aussi vu le premier "tmpfs" remplacé par "none" dans la ligne fstab, ça ne change rien chez moi)

Résultats = pas concluant :
- mon objectif = comme chromium a accès à `~/temporaire`, je crée un répertoire `~/tmpfs` qui sera accessible à chromium
- j'essaye diverses variantes de ça, mais au reboot, snap refuse de démarrer
- après investigations, il semblerait qu'apparmor ait un bug avec snapd qui empêche de monter des tmpfs : [lien](https://www.reddit.com/r/archlinux/comments/140g2ox/issues_with_apparmor_snapd/)

### essai 2 = essai de binder le tmpfs existant dans un endroit accessible à chromium

J'avais pris l'idée de [ce post](https://askubuntu.com/questions/1263843/how-to-allow-snap-applications-to-access-tmp-folder) :

```
sudo mount --bind /tmp/ /home/myself/snap/chromium/current/tmpalias/
```

(et normalement, je peux créer ce bind dans fstab)

Ça ne marche pas car les droits ne sont pas bons...

### essai 3 = utiliser un répertoire normal + le vider avant d'éteindre le PC

Je m'appuie [là-dessus](https://opensource.com/life/16/11/running-commands-shutdown-linux)

Commandes :

```sh
sudo vim /usr/lib/systemd/system-shutdown/clear-folder.shutdown
cd /usr/lib/systemd/system-shutdown/
sudo chmod --reference=fwupd.shutdown clear-folder.shutdown

# -rwxr-xr-x 1 root root  67 sept. 21 21:57 clear-folder.shutdown
# -rwxr-xr-x 1 root root 252 mai   11 04:21 fwupd.shutdown
```

Ça n'a pas l'air de marcher, et de toutes façons, je ne suis pas super serein avec la suppression manuelle de fichiers (ça peut backfire sévère, c'est du vécu).

Du coup j'abandonne cette piste

### essai 4 = passer par un removable-media accessible aux snaps

cf. https://doc.ubuntu-fr.org/snap#extension_du_confinement

Sur mon poste pro, je ne peux pas faire cette modif-moi-même, car il faut le mot de passe administrateur de mon poste (plus fort que le mot de passe sudo).
