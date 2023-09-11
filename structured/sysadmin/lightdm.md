**Lightdm** est le login-manager de xubuntu (ubuntu en utilise un différent = [GDM](https://doc.ubuntu-fr.org/gdm)).

[Cette page](https://doc.ubuntu-fr.org/lightdm) est très très complète avec beaucoup d'infos sur la configuration de lightdm, et plus généralement sur les gestionnaires de connexion.

Je m'y intéresse pour désactiver l'autologin afin que gnome-keyring soit délocké lorsque je me connecte au PC.

**autologin** = le fait de connecter automatiquement l'utilisateur 'myself' au démarrage du PC, sans avoir à entrer de mot de passe.

```
lightdm --show-config
```

^ donne la config light-dm, et sa provenance

```sh
sudo cat /etc/lightdm/lightdm.conf

# [Seat:*]
# autologin-guest=false
# autologin-user=myself
# autologin-user-timeout=0
```

Ma compréhension est : on affiche la liste des utilisateurs, et si aucun ne s'est connecté au bout de `autologin-user-timeout` secondes, alors on logge automatiquement `autologin-user`.

Il y a une option (`greeter-hide-users`) pour masquer la liste des utilisateurs et devoir plutôt taper au clavier le nom d'utilisateur par sécurité, mais je ne l'active pas car pas pratique sur le PC fixe.
