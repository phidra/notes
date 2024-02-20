**TL;DR** : pour gérer les services, utiliser :

```sh
sudo systemctl ACTION SERVICE
```

# HOWTO

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

# Pourquoi plusieurs commandes ?

Sur ubuntu, historiquement, il y a eu 3 façons de démarrer un service lorsque le système boot :
- le plus ancien = [init de systemv](https://fr.wikipedia.org/wiki/Init#%C2%AB_init_%C2%BB_de_Unix_System_V_(SysV_init))
    - script dans `/etc/init.d`
    - `/etc/rc#.d`
    - puis appel de `update-rc.d`
- l'intermédiaire = [upstart](https://fr.wikipedia.org/wiki/Upstart), je ne l'ai jamais utilisé directement
- l'actuel = [systemd](https://fr.wikipedia.org/wiki/Systemd)
    - systemd = system daemon
    - controversé, mais aujourd'hui utilisé partout (notamment : pour debian à partir de jessie, pour ubuntu à partir de 15.04)
