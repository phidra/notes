**TL;DR** : aws-cli est une CLI (duh !) permettant d'utiliser les services AWS. Notamment, elle permet de configurer le login.

# Installation

## Ce que j'ai suivi

Il y a deux versions, la V1 est deprecated, il faut utiliser la V2.

```sh
cd /tmp

# téléchargement + checksum :
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
nvim /tmp/key.txt  # ici, insérer la clé publique PGP
gpg --import /tmp/key.txt
curl -o awscliv2.sig https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip.sig
gpg --verify awscliv2.sig awscliv2.zip

# installation
sudo ./aws/install

# vérification :
aws --version
# aws-cli/2.15.21 Python/3.11.6 Linux/5.15.0-91-generic exe/x86_64.ubuntu.20 prompt/off
```

Doc de désinstallation = [lien](https://docs.aws.amazon.com/cli/latest/userguide/uninstall.html)


## À ne pas faire

J'avais également installé aws-cli via pipx, **à ne pas faire** notamment car c'est la V1 qui est installée ; j'ai dû la déinstaller :

```sh
aws --version
# aws-cli/1.29.57 Python/3.8.10 Linux/5.15.0-91-generic botocore/1.31.57

type aws
# aws is /home/myself/.local/bin/aws

ll /home/myself/.local/bin/aws
# lrwxrwxrwx 1 myself/home/myself/.local/bin/aws -> /home/myself/.local/pipx/venvs/awscli/bin/aws

pipx list
# venvs are in /home/myself/.local/pipx/venvs
# apps are exposed on your $PATH at /home/myself/.local/bin
#    package awscli 1.29.57, installed using Python 3.8.10
#     - aws
#     - aws.cmd
#     - aws_bash_completer
#     - aws_completer
#     - aws_zsh_completer.sh
#    ...

pipx uninstall awscli
# uninstalled awscli! ✨ 🌟 ✨
```


# Authentication

## AWS_PROFILE

L'envvar `AWS_PROFILE` sert à ne pas avoir à préciser le profile à chaque commande.

Ainsi, les deux commandes suivantes :

```sh
export AWS_PROFILE=my-dev-profile
aws sts get-caller-identity
```

Sont équivalentes à passer une unique commande :

```sh
aws sts get-caller-identity --profile=my-dev-profile
```

(NDM : mon prompt zsh semble entraîné à utiliser l'envvar `AWS_PROFILE` : il modifie le prompt lorsqu'elle est définie)

## avec SSO

Configurer le sso dans `~/.aws/config` :

```ini
[profile my-dev-profile]
sso_start_url = https://...
sso_region = eu-west-3
sso_account_id = XXXXXX
sso_role_name = YYYYYY
region = eu-west-3
output = json
```


Puis :

```
aws sso login --profile my-dev-profile
```

Ça m'affiche un code sur le terminal, et ça m'ouvre un browser pour que je confirme le code ; une fois fait, je suis considéré comme loggé.

Note : si j'utlise le SSO, je n'ai pas besoin de toucher moi-même au fichier `~/.aws/credentials`.

## get-caller-identity

Cette API permet de vérifier que je suis bien loggé.

Si je la lance avant de m'être loggé avec le SSO :

```sh
aws sts get-caller-identity --profile=my-dev-profile
# Error loading SSO Token: Token for https://mycompany.awsapps.com/start does not exist
```

Si je me logge avec SSO, le retour de la commande change pour m'indiquer mes infos de logging :

```sh
aws sts get-caller-identity --profile=my-dev-profile
# {
#     "UserId": "XXXXXX",
#     "Account": "YYYYYY",
#     "Arn": "ZZZZZZ"
# }
```

À noter que dans mon cas, l'erreur suivante indique juste que je n'ai pas passé le profil :

```sh
aws sts get-caller-identity
# Unable to locate credentials. You can configure credentials by running "aws configure".
```

Pour connaître les profils disponibles, cette commande est lançable sans aucune connexion :

```sh
aws configure list-profiles
# my-dev-profile
# my-staging-profile
# my-prod-profile
# default
```

## Company's startpage

Sur [cette page d'accueil](https://mycompany.awsapps.com/start#/), je trouve des infos sur les envvars ou les credentials à entrer manuellement pour permettre l'authentication sans le SSO.

Dans l'onglet `Access keys`.

# Utilisation

La tronche générale des commandes a l'air d'être :

```sh
aws SERVICE COMMANDE

# p.ex. :
aws sts get-caller-identity
```
