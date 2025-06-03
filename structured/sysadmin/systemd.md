**TL;DR** : pour gérer les services, utiliser la CLI de **systemd** = `systemctl` :

```sh
sudo systemctl ACTION SERVICE
```

* [HOWTO gérer ses services avec systemd](#howto-gérer-ses-services-avec-systemd)
* [Logs des services systemd](#logs-des-services-systemd)
* [Historique : pourquoi plusieurs commandes ? Quel lien avec sysV init ou upstart ?](#historique--pourquoi-plusieurs-commandes--quel-lien-avec-sysv-init-ou-upstart-)

# HOWTO gérer ses services avec systemd

- status d'un service :
    ```sh
    systemctl status redis-server  # sudo est inutile
    ```
- le service est-il actuellement en train de tourner :
    ```sh
    systemctl is-active redis-server  # sudo est inutile
    # attention, il indique "inactive" même pour les services inexistants
    # (il faut utiliser status pour savoir si un service existe ou non)
    ```
- le service est-il démarré automatiquement au boot du système :
    ```sh
    systemctl is-enabled redis-server  # sudo est inutile
    ```
- [démarrer/redémarrer/recharger/arrêter] un service  (n'a pas d'impact sur l'autostart au boot du système) :
    ```sh
    sudo systemctl [start/restart/reload/stop] redis-server
    ```
- marquer le service comme [à démarrer/à ne pas démarrer] au prochain boot du système (n'a pas d'impact sur le statut actuel) :
    ```sh
    sudo systemctl [enable/disable] redis-server
    ```
- lister les services connus :
    ```sh
    systemctl list-unit-files '*service' | sort
    ```

# Logs des services systemd

**TL;DR** = `journalctl` pour consulter les logs d'un service géré par systemd.

Pas mal de flags et options, cf. [source1](https://www.linuxtricks.fr/wiki/systemd-utiliser-journalctl-les-logs-de-systemd), [source2](https://www.loggly.com/ultimate-guide/using-journalctl/).

```sh
# limiter à un service en particulier :
sudo journalctl -u NetworkManager
sudo journalctl -u systemd-networkd

# limiter aux logs depuis le dernier reboot :
sudo journalctl -b

# lister les reboots :
sudo journalctl --list-boots
```


# Historique : pourquoi plusieurs commandes ? Quel lien avec sysV init ou upstart ?

Sur ubuntu, historiquement, il y a eu 3 façons de démarrer un service lorsque le système boot :

- le plus ancien = [init de systemV](https://fr.wikipedia.org/wiki/Init#%C2%AB_init_%C2%BB_de_Unix_System_V_(SysV_init))
    - script dans `/etc/init.d`
    - `/etc/rc#.d`
    - puis appel de `update-rc.d`
- l'intermédiaire = [upstart](https://fr.wikipedia.org/wiki/Upstart), je ne l'ai jamais utilisé directement
- l'actuel = [systemd](https://fr.wikipedia.org/wiki/Systemd)
    - systemd = system daemon
    - controversé, mais aujourd'hui utilisé partout (notamment : pour debian à partir de jessie, pour ubuntu à partir de 15.04)
