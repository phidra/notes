En résumé :

- **gnome-keyring** est un service pour que les clients (des applications telles que chromium, ou protonmail-bridge) puissent enregistrer des mots de passe de façon secure.
- il est verrouillé initialement, et ce n'est qu'une fois délocké que les applications peuvent l'utiliser
    - le déverouillage du keyring est global pour l'user : une fois que j'ai délocké le keyring pour une application, les autres applications ne demandent plus le mot de passe
    - (ça explique la popup au démarrage de chromium : elle sert à délocker le keyring)
- si la connexion au démarrage du PC n'est pas automatique (i.e. autologin désactivé, cf. mes notes sur lightdm), alors un mot de passe est obligatoire pour se connecter au PC, et comme le mot de passe par défaut du keyring est le même, le keyring est délocké à la connexion
- **seahorse** est l'IHM permettant d'interagir avec le keyring ; elle est accessible en tant que "Mots de passe et clés"
    - la liste des mots de passe gérés est dans "Connexion"
- je ne le fais pas (cf. mes notes de workflow), mais si besoin, le keyring peut gérer les passphrases SSH

----

* [Concepts](#concepts)
* [Gestion des clés SSH](#gestion-des-clés-ssh)


# Concepts

J'annote [cette ressource](https://itsfoss.com/ubuntu-keyring/) qui explique très bien les concepts :

> This keyring keeps your ssh keys, GPG keys and keys from applications that use this feature, like Chromium browser.

^ le keyring permet aux applications de déléguer le stockage de mots de passe à gnome-keyring, ce qui permet de ne pas les stocker en clair.

> By default, the keyring is locked with a master password which is often the login password of the account. Every user on your system has its own keyring with (usually) the same password as that of the user account itself.

^ chaque user a son propre keyring, par défaut, le mot de passe du keyring est celui de la session = le mot de passe root.

> When you login to your system with your password, your keyring is unlocked automatically with your account’s password.

^ quand on utilise la connexion automatique, le keyring est délocké automatiquement.

> The problem comes when you switch to auto-login in Ubuntu. This means that you login to the system without entering the password. In such case, your keyring is not unlocked automatically.

^ mais quand on est en auto-login, le keyring n'est pas délocké explicitement :-(

> Imagine that on your Linux desktop, you are using auto-login.
> Anyone with access to your desktop can enter the system without password but you have no issues with that perhaps because you use it to browse internet only.
>
> But if you use a browser like Chromium or Google Chrome in Ubuntu, and use it to save your login-password for various websites, you have an issue on your hand.
> Anyone can use the browser and login to the websites for which you have saved password in your browser. That’s risky, isn’t it?
>
> This is why when you try to use Chrome, it will ask you to unlock the keyring repeatedly.
> This ensures that only the person who knows the keyring’s password (i.e. the account password) can use the saved password in browser for logging in to their respective websites.
> If you keep on cancelling the prompt for keyring unlock, it will eventually go away and let you use the browser. However, the saved password won’t be unlocked

^ tout ceci explique pourquoi chromium utilise le keyring : car il stocke des mots de passe.

En résumé, toute application qui stocke des mots de passe utilise le keyring (ou devrait le faire !) ; du coup si on ne délocke pas le keyring, on ne peut pas utiliser les features de chromium qui nécessite un password.

> You can use this GUI application to see what application use the keyring to manage/lock passwords.

^ la page recommande **seahorse** (`sudo apt install seahorse`), lançable par "Mots de passe et clés".

(et je confirme que si on lance seahorse sans avoir déverouillé le keyring, on n'a pas accès aux applis qui stockent des trucs ; on a le message "Le trousseau est verrouillé")

> You can also use this application to manually store passwords for website. For example, I created a new password-protected keyring called ‘Test’ and stored a password in this keyring manually.
>
> This is slightly better than keeping a list of passwords in a text file. At least in this case your passwords can be viewed only when you unlock the keyring with password.

On peut s'en servir pour stocker ses mots de passe manuels : c'est vraiment une alternative à KeePass.

Plus bas, leur résumé est très très bien :

> - Most Linux has this ‘keyring feature’ installed and activated by default
> - Each user on a system has its own keyring
> - The keyring is normally locked with the account’s password
> - Keyring is unlocked automatically when you login with your password
> - For auto-login, the keyring is not unlocked and hence you are asked to unlock it when you try to use an application that uses keyring
> - Not all browsers or application use the keyring feature
> - There is a GUI application installed to interact with keyring
> - You can use the keyring to manually store passwords in encrypted format
> - You can change the keyring password on your own
> - You can export (by unlocking the keyring first) and import it on some other computer to get your manually saved passwords

Par ailleurs :

> Suppose you changed your account password. Now when you login, your system tries to unlock the keyring automatically using the new login password. But the keyring still uses the old login password.
>
> In such a case, you can change the keyring password to the new login password so that the keyring gets unlocked automatically as soon as you login to your system.

^ même si on pourrait penser le contraire (car le fait de se logger au PC délocke le keyring), le mot de passe du keyring _n'est pas syrnchonisé avec le mot de passe de connexion_.

Si on modifie le mot de passe de connexion, il faut mettre à jour manuellement le password du keyring (seahorse permet de le faire).

Il est aussi possible de supprimer la protection par mots de passe du keyring, mais c'est une mauvaise idée car les mots de passe sont alors stockés en clair.

# Gestion des clés SSH

gnome-keyring permet d'enregistrer les passphrases des clés SSH : si le keyring est délocké, on peut utiliser la clé.

cf. mes notes de workflow : je choisis de ne pas le faire car ça ne fonctionne que pour les sessions graphiques, et pas si j'ouvre une session remote par ssh sur mon poste.

Pour supprimer du keyring une clé SSH qu'on aurait ajoutée à tort :

- il ne faut PAS supprimer la clé dans l'IHM gnome-keyring (j'ai testé : ça la supprime non seulement de gnome-keyring, mais également du disque !)
- il y a tout simplement un mot de passe stocké dans "Connexion" (qui est la liste des mots de passe gérés par gnome-keyring) qui concerne la clé SSH
- son titre est :
    ```
    Mot de passe de déverrouillage pour <adresse-email-de-la-clé>
    ```
- on peut voir dans "Détails" que c'est une clé SSH, il indique ssh-store + le chemin de la clé :
    ```
    ssh-store:/home/myself/.ssh/id_ed25519
    ```
