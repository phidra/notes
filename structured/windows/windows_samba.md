**Contexte** = début 2025, j'essaye de me setup une VM windows depuis un host linux.

([doc ubuntu](https://doc.ubuntu-fr.org/samba))

# Depuis l'host linux

```sh
sudo apt install samba

sudo mkdir /mnt/shared_with_vm
sudo chmod 777 /mnt/shared_with_vm

sudo vim /etc/samba/smb.conf :
# [PartagePourVM]
# path = /mnt/shared_with_vm
# available = yes
# valid users = nobody
# read only = no
# browsable = yes
# public = yes
# writable = yes
# create mask = 0777
# directory mask = 0777
# force user = nobody
# force group = nogroup

sudo systemctl restart smbd
```

Noter l'IP de l'host + le nom du partage :

```sh
# pour connaître mon IP locale, p.ex. 192.168.1.50 :
hostname --all-ip-addresses|cut -d" " -f 1

# pour connaître le nom du répertoire partagé, p.ex. PartagePourVM :
smbclient -L localhost --no-pass
```

# Depuis le guest windows

Dans l'explorateur de fichiers, clic-droit sur "Ce PC" > Connecter un lecteur réseau

Laisser le mapping par défaut vers `Z:`, et entrer le chemin suivant :

```
\\192.168.1.50\PartagePourVM
```

C'est tout : le répertoire samba est maintenant accessible sous `Z:`.
