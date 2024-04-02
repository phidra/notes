**TL;DR** : aws-cli est une CLI (duh !) permettant d'utiliser les services AWS. Notamment, elle permet de configurer le login.

- [Utilisation](#utilisation)
  * [Découpage par service](#d-coupage-par-service)
  * [Exemple avec S3](#exemple-avec-s3)
  * [Aide à la construction de la commande](#aide---la-construction-de-la-commande)
- [Installation](#installation)
  * [Ce que j'ai suivi](#ce-que-j-ai-suivi)
  * [À ne pas faire](#--ne-pas-faire)

# Utilisation

## Découpage par service

La tronche générale des commandes a l'air d'être :

```sh
aws SERVICE COMMANDE

# p.ex. :
aws sts get-caller-identity
```

## Exemple avec S3

Exemple S3 avec utilisation du SSO et de `AWS_PROFILE` :

```sh
# préalable :
aws sso ...
export AWS_PROFILE=...

# télécharger un fichier d'un bucket :
aws s3 cp s3://my-super-bucket/myfile.txt /tmp/youpi
```

## Aide à la construction de la commande

Les commandes disposent toutes d'une option `--cli-auto-prompt` qui permet de construire interactivement la commande :

- en listant les paramètres existants
- en indiquant ceux obligatoires et facultatifs
- en donnant une petite explication de chaque paramètre
- en autocomplétant quand c'est possible (e.g. pour les profils)

# Installation

## Ce que j'ai suivi

**Attention** : il y a deux versions, la V1 est deprecated, il faut utiliser la V2.

La doc d'installsation est très bien faite : https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```sh
cd /tmp

# téléchargement + checksum, pour x86_64 (il y a l'équivalent pour ARM) :
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
nvim /tmp/key.txt  # ici, insérer la clé publique PGP copiée depuis la doc d'installation
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
