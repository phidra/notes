https://proton.me/fr/blog/what-is-two-factor-authentication-2fa

- les mots de passe sont insuffisants (fuites de données, phishing)
- avec 2FA, sécurité même si mot de passe révélé
- les facteurs d'authentification sont :
    - facteur de possession = quelque chose qu'on possède (e.g. smartphone pour SMS)
    - facteur de connaissance = quelque chose qu'on connaît (e.g. mot de passe)
    - facteur de caractéristiques biométriques = quelque chose qu'on est (e.g. empreinte digitale ou iris)
- 2FA = mélanger les deux premiers : si un malfaiteur acquiert l'un des deux facteurs, il lui manquera l'autre pour se connecter
- autres facteurs moins fréquents :
    - facteur de localisation = l'endroit d'où on se connecte
    - facteur de temps = l'heure à laquelle on essaye de se connecter
- 2FA par SMS (= on reçoit un OTP = One-Time Password sur son téléphone) déconseillé :
    - il faut donner son numéro de téléphone à l'entreprise
    - le SMS se pirate facilement
    - si son smartphone est déjà le moyen de réinitialiser son mot de passe, un accès compromis au téléphone résout les deux facteurs d'un coup
- 2FA avec application d'authentification (= une application génère un TOTP = Time-based One-Time Password) :
    - toutes les 30 à 60 secondes, l'application génère un token
- MFA = niveau au dessus (M pour multi, et multi > 2)
- perte du mot de passe :
    - si on perd l'appareil 2FA, on perd la possibilité de se connecter !
    - pour éviter ça, les sites proposent des mots de passe de sauvegarde 2FA, permettant de shunter le 2FA
    - l'idée est d'imprimer ces codes d'accès et de les conserver en fallback

> Bien que certaines méthodes d’A2F, comme le SMS, soient moins sécurisées que d’autres, toute méthode d’A2F est beaucoup plus sûre que d’utiliser uniquement un mot de passe.
>
> Quand vous configurez l’A2F pour vos comptes, n’oubliez pas de télécharger ou d’imprimer les codes de sauvegarde s’ils sont disponibles.
>
> De nombreuses plateformes, comme Proton, vous permettent aussi de configurer l’A2F avec une application d’authentification et une clé de sécurité, pour que vous ayez toujours une solution de secours.

**Q and A :**

- c'est demandé à chaque fois ?
    - a priori oui : on ne peut donc se connecter que si on a l'appareil 2FA
- quid si je perds mon smartphone ?
    - à l'activation du 2FA, les sites proposent généralement des codes de secours permettant de se connecter (une seule fois) et de désactiver le 2FA, le temps de retrouver un setup correct
- quid pour thunderbird ?
    - on dirait qu'il est possible pour thunderbird de continuer à bosser même avec le 2FA (mais faudra que je teste)
    - cf. https://forum.manjaro.org/t/thunderbird-and-gmail-with-two-factor-authentication/89789/2
