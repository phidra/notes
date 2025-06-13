* [Installation](#installation)
* [Partager un répertoire en lecture seule en HTTP](#partager-un-répertoire-en-lecture-seule-en-http)
* [Partager un répertoire en lecture/écriture avec webdav](#partager-un-répertoire-en-lectureécriture-avec-webdav)

# Installation

Sur ubuntu 24.04, ceci m'a suffi :

```
sudo apt install rclone
```

# Partager un répertoire en lecture seule en HTTP

```
# depuis le répertoire à servir :
rclone serve http . --addr :8080
```

Côté client, ouvrir un browser sur http://ip-du-serveur:8080/

# Partager un répertoire en lecture/écriture avec webdav

```
# depuis le répertoire à servir :
rclone serve webdav . --addr :8080

# si besoin de limiter en lecture-seule :
rclone serve webdav . --addr :8080 --read-only
```

Côté client, e.g. dans le navigateur de fichiers windows : **Connecter un lecteur réseau** et entrer http://ip-du-serveur:8080/

(si besoin, le partage webdav permet aussi d'ouvrir un browser sur http://ip-du-serveur:8080/ )

