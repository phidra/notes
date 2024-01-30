**Contexte** : janvier 2024 = toujours avec [ccrypto](https://vpn.ccrypto.org/), je transitionne d'OpenVPN vers wireguard.

La procédure est décrite sur [leur guide](https://vpn.ccrypto.org/page/install-linux) :

- installer wireguard :
    ```sh
    sudo apt install wireguard
    ```
- depuis [la page wireguard de mon compte ccrypto](https://vpn.ccrypto.org/account/wireguard), créer une clé wireguard
- une fois la clé créée, télécharger son fichier de conf depuis la même page (attention : bien choisir le serveur français)
- le placer au bon endroit :
    ```sh
    sudo cp temporaire/ccvpn-fr.conf /etc/wireguard
    ```
- vérifier mon IP avant activation :
    ```sh
    myip
    # en alternative, l'IP de sortie est sur le pied de page de n'importe quelle page ccrypto :
    # https://vpn.ccrypto.org/
    ```
- activer wireguard :
    ```sh
    sudo wg-quick up ccvpn-fr
    ```

Remarque : mes anciennes notes sur l'utilisation d'openvpn sont dans mes notes OTL archivées que je n'ai pas rendues publiques.
