**Contexte** = début 2025, j'essaye de me setup une VM windows depuis un host linux.

* [Installation](#installation)
* [CLI vs. virt-manager](#cli-vs-virt-manager)
* [Configuration via virt-manager](#configuration-via-virt-manager)
* [Copier-coller entre host et guest](#copier-coller-entre-host-et-guest)
* [Raccourcis et utilisation](#raccourcis-et-utilisation)
* [Modifier l'escape key](#modifier-lescape-key)
* [Utiliser les touches Alt](#utiliser-les-touches-alt)
* [Utilisation du partage samba intégré à qemu](#utilisation-du-partage-samba-intégré-à-qemu)
* [Augmenter la taille du disque qemu](#augmenter-la-taille-du-disque-qemu)
* [Debugging](#debugging)


# Installation

```sh
sudo apt install qemu-system-x86
sudo apt install virt-manager
sudo adduser $USER libvirt
```

# CLI vs. virt-manager

qemu (utiliable via la commande `kvm`, [doc ubuntu](https://doc.ubuntu-fr.org/kvm)) peut être utilisé soit via une CLI, soit via une GUI frontale utilisant libvirt = **virt-manager** ([doc ubuntu](https://doc.ubuntu-fr.org/virt-manager)).

Une fois n'est pas coutume, je recommande d'utiliser la GUI, plus pratique d'utilisation.

Voici tout de même quelques commandes CLI :

```sh
# Créer un HDD de 40 Gio :
qemu-img create -f qcow2 win10.img 40G

# Installer windows sur ce HDD :
sudo kvm -m 4096 -hda win10.img -cdrom Win10_22H2_French_x64v1.iso -boot d -enable-kvm

# Lancer la VM — basics :
sudo kvm -m 4096 -hda /path/to/win10.img -enable-kvm -smp 8

# Lancer la VM — utilisation d'un partage samba + mapping du RightCtrl comme touche d'échappement :
mkdir /tmp/shared
sudo kvm -m 4096 -hda /path/to/win10.img -enable-kvm -smp 8 -display sdl,grab-mod=rctrl -nic user,id=nic0,smb=/tmp/shared
```

# Configuration via virt-manager

Préambule = virt-manager est une IHM utilisant libvirt pour gérer des VM, du coup c'est surtout libvirt qu'il faut configurer.

Ça passe par un fichier xml qui représente "la configuration de ma machine virtuelle du point de vue de libvirt" ; chez moi, le XML de ma VM est dans :

```
/etc/libvirt/qemu/nom_de_ma_VM.xml
```

La commande `virsh` permet d'ouvrir un éditeur sur le XML d'une VM :

```sh
sudo virsh edit nom_de_ma_VM
```

Sinon, la commande `virt-xml` permet d'éditer programmatiquement les lignes du XML d'une VM (il montre la modification qu'il s'apprête à faire, et on a la possibilité de valider ou d'annuler) :

```sh
sudo virt-xml nom_de_ma_VM --edit --confirm --qemu-commandline=pouet
```

Et de toutes façons, il reste possible d'éditer le fichier manuellement.


L'IHM `virt-manager` permet de consulter le XML d'une option.

# Copier-coller entre host et guest

Il faut installer les guest-additions sur la VM, le copier-coller fonctionne aussitôt (pas besoin de redémarrage de la VM ; pas d'action à prendre chez l'host).

Dans une VM windows, il faut exécuter l'installer `spice-guest-tools-latest.exe` à télécharger depuis cette page : https://www.spice-space.org/download.html

Dans une VM ubuntu :

```sh
sudo apt install spice-vdagent
```

# Raccourcis et utilisation

Raccourcis claviers = https://www.qemu.org/docs/master/system/keys.html

**Toggle fullscreen** = `<escape-key> + F` (avec RightCtrl, ça donne RightCtrl+F)

**Quitter le plein-écran depuis une VM virt-manager** : déplacer la souris en haut au milieu de l'écran, un dropdown apparaît pour quitter le plein écran (ou envoyer une combinaison de touches).

# Modifier l'escape key

En version française, la façon de modifier la touche d'échappement de l'host n'est pas intuitive :

Dans la GUI virt-manager (et non dans la machine virtuelle !), aller dans les préférences, onglet `Console`.

Il y a une ligne `Obtenir les clés` (qui est une **TRÈS** mauvaise traduction de _"Grab keys"_...), il faut cliquer sur `Modifier...`

Du coup, je peux utiliser le **RightCtrl** comme escape key.

# Utiliser les touches Alt

Quand j'utilise une VM qemu via virt-manager, `Alt-Tab` et `Alt-F4` fonctionnent comme je le souhaite = par défaut les commandes sont dirigées vers le guest.

Je peux les diriger vers l'host en appuyant d'abord sur l'escape key.

Autre truc cool : si je ferme par erreur la fenêtre de la VM, elle reste vivante en background, je peux la rouvrir depuis virt-manager.

# Utilisation du partage samba intégré à qemu

TL;DR = mieux vaut monter moi-même mon propre partage samba que d'utiliser celui de qemu.

qemu permet de créer automatiquement un partage samba entre l'host et le guest

Le principe est que si samba est installé sur l'host linux, alors QEMU peut monter automatiquement un disque samba directement accessible par windows, sur l'IP hardcodée à `\\10.0.2.4\qemu` : https://wiki.archlinux.org/title/QEMU#QEMU's_built-in_SMB_server


Avec la CLI, il faut tuner la commande de lancement de la VM pour la lancer avec l'option `-nic` et utiliser le partage samba, [ce post](https://www.ashbysoft.com/posts/libvirt-qemu-samba/) indique comment faire.

Avec virt-manager, il faut bricoler — notes vrac :

- il faut modifier le XML pour passer des arguments à la commande qemu (les arguments permettant théoriquement d'utiliser le partage samba de qemu).
- suppression de l'interface réseau et de me reposer sur celle que je créais via des arguments custom
- création et attribution des bons droits d'accès au répertoire partagé, sinon j'ai une erreur :
    ```
    Erreur lors du démarrage du domaine: internal error: QEMU unexpectedly closed the monitor (vm='win10bis'): 2025-03-25T09:18:24.137413Z qemu-system-x86_64:
    -nic user,id=nic0,smb=/tmp/shared: Error accessing shared directory '/tmp/shared': No such file or directory
    ```
- selon [cette page](https://unix.stackexchange.com/questions/188301/how-to-set-up-samba-sharing-with-libvirtd/194352#194352), il a fallu que je modifie la conf de AppArmor pour éviter l'erreur suivante :
    ```
    Erreur lors du démarrage du domaine: internal error: QEMU unexpectedly closed the monitor (vm='win10bis'): 2025-03-25T08:52:34.361486Z qemu-system-x86_64:
    -nic user,id=nic0,smb=/tmp/shared: Could not create samba server dir
    ```
- j'ai aussi eu des erreurs avec le bus PCI :
    ```
        Erreur lors du démarrage du domaine: internal error: QEMU unexpectedly closed the monitor (vm='win10bis'): 2025-03-25T09:30:25.174465Z qemu-system-x86_64:
        -device {"driver":"qxl-vga","id":"video0","max_outputs":1,"ram_size":67108864,"vram_size":67108864,"vram64_size_mb":0,"vgamem_mb":16,"bus":"pcie.0","addr":"0x1"}: PCI: slot 1 function 0 not available for qxl-vga, in use by e1000e,id=(null)
    ```
- au final, je finis par abandonner et monter moi-même un partage samba, plutôt que d'utiliser celui de qemu

# Augmenter la taille du disque qemu

Si le fichier image est ici :
```
/var/lib/libvirt/images/win10ter.qcow2
```

Alors les commandes suivantes augmentent la taille du disque :


```sh
# ajouter 60G à la taille existante :
sudo qemu-img resize /var/lib/libvirt/images/win10ter.qcow2 +60G


# ajouter ce qu'il faut pour atteindre une taille finale de 60G
sudo qemu-img resize /var/lib/libvirt/images/win10ter.qcow2 60G
```

À noter que le fichier image grandit dynamiquement avec l'occupation du disque : un fichier image qui "cible" 60G mais qui n'est rempli qu'au quart aura une taille effective de 15G.

Par contre la partition n'est pas agrandie automatiquement... il faut resizer (et c'est plus compliqué sous windows que que sous linux).

# Debugging

Si je me suis gouré dans une config, la VM refuse de démarrer et une popup indique pourquoi.

Pour débugger les erreurs de libvirt :


```sh
sudo grep libvirt /var/log/syslog
```

Je n'ai pas creusé, mais cette page indique comment débugger des problèmes de virtualisation : https://fedoraproject.org/wiki/How_to_debug_Virtualization_problems
