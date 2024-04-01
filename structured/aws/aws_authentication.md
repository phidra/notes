- [Authentication](#authentication)
  * [AWS_PROFILE](#aws-profile)
  * [avec SSO](#avec-sso)
  * [get-caller-identity](#get-caller-identity)
  * [Depuis un programme](#depuis-un-programme)
  * [Company's startpage](#company-s-startpage)


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

## Depuis un programme

Par exemple depuis le SDK C++, il y a des moyens de laisser le SDK se débrouiller tout seul pour trouver les infos lui permettant de se logger : [explications](https://sdk.amazonaws.com/cpp/api/LATEST/root/html/md_docs_2_credentials___providers.html)

Par exemple, [l'un des constructeurs du client S3](https://sdk.amazonaws.com/cpp/api/LATEST/aws-cpp-sdk-s3/html/class_aws_1_1_s3_1_1_s3_client.html#ab9a39dc4fa2eb24d252e7ffacd1521ea) utilise [cette chaîne par défaut](https://sdk.amazonaws.com/cpp/api/LATEST/aws-cpp-sdk-core/html/class_aws_1_1_auth_1_1_default_a_w_s_credentials_provider_chain.html).

Pour qu'un programme utilisant cette chaîne puisse être authentifié, une façon est de définir les credentials dans des envvars. Comment savoir quelle valeur attribuer aux envvars lorsqu'on utilise le SSO ?

En fait, lorsqu'on est loggé avec le SSO, on peut récupérer des infos de connexion temporaires à placer dans des envvars, avec `aws configure` :

```sh
aws configure export-credentials --profile my-dev-profile --format env

# exemple de réponse :
# export AWS_ACCESS_KEY_ID=XXX
# export AWS_SECRET_ACCESS_KEY=YYY
# export AWS_SESSION_TOKEN=ZZZ
# export AWS_CREDENTIAL_EXPIRATION=XXX
```

Définir ces envvars permet à un programme de se connecter en utilisant (indirectement) la connexion du SSO.

## Company's startpage

Il se peut que mon entreprise `mycompany` définisse une page d'accueil personnalisée : https://mycompany.awsapps.com/start#/

J'y trouverais alors des infos sur les envvars ou les credentials à entrer manuellement pour permettre l'authentication sans le SSO ; dans l'onglet `Access keys`.
