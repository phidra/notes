Infos diverses sur les clés SSH et leur passphrase.

Obviously, quand on parle de "clé", on parle en fait de "paire de clés" :

```sh
~/.ssh/mykey      # clé privée
~/.ssh/mykey.pub  # clé publique
```

* [vérifier qu'on a connaît bien la passphrase d'une clé](#vérifier-quon-a-connaît-bien-la-passphrase-dune-clé)
* [savoir quelle clé est connue de github/autre service ?](#savoir-quelle-clé-est-connue-de-githubautre-service-)
* [lister les clés connues de ssh-agent](#lister-les-clés-connues-de-ssh-agent)


# vérifier qu'on a connaît bien la passphrase d'une clé

La commande suivante prompte la passphrase, ce qui permet de vérifier qu'on a la bonne :

```
ssh-keygen -y -f ~/.ssh/id_ed25519
```

# savoir quelle clé est connue de github/autre service ?

- dans les settings github, on a des infos sur le hash de la clé qu'il connaît, p.ex. :
    ```
    SHA256:ymhgZdbwQnR4vxPmbuG2eOhdzDEe0IWnX1V0jZ3E3bU

    ```
- pour connaître le hash de ses clés, il y a une commande indiquant la même info pour une de ses clés :
    ```sh
    ssh-keygen -lf ~/.ssh/id_ed25519.pub
    # 256 SHA256:ymhgZdbwQnR4vxPmbuG2eOhdzDEe0IWnX1V0jZ3E3bU myemail@example.com (ED25519)
    ```
- c'est sans doute possible, mais je n'ai pas trouvé comment jouer avec `sha256sum` pour obtenir ce même résultat

# lister les clés connues de ssh-agent

```
ssh-add -l
ssh-add -L
```
