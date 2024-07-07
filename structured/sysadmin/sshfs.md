Pour monter le répertoire distant `/remote-path/to/remote-folder` de l'host `remote-host` sur le répertoire local `/path/to/local` :

(note : pour être sûr que le répertoire est monté avec le bon user en local, même si les uid/gid ne matchent pas, il faut mapper les uid/gid)

```sh
mkdir /path/to/local
sshfs \
    -o uid=$(id -u mylocalusername) \
    -o gid=$(id -g mylocalusername) \
    remote-user@remote-host:/remote-path/to/remote-folder/  /path/to/local/
```

Pour démonter :


```sh
fusermount -u /path/to/local/
```
