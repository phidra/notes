- [Utilisation avec la console web](#utilisation-avec-la-console-web)
- [Utilisation avec la CLI simplifiée aws s3](#utilisation-avec-la-cli-simplifi-e-aws-s3)
  * [Lister les buckets](#lister-les-buckets)
  * [Lister les fichier d'un bucket](#lister-les-fichier-d-un-bucket)
  * [Télécharger un fichier d'un bucket](#t-l-charger-un-fichier-d-un-bucket)
- [Utilisation avec la CLI complète aws s3api](#utilisation-avec-la-cli-compl-te-aws-s3api)
  * [Lister les tags d'un fichier](#lister-les-tags-d-un-fichier)
- [Liste des fonctions de l'API](#liste-des-fonctions-de-lapi)
- [Liste des régions S3](#liste-des-régions-s3)

# Utilisation avec la console web

Via la console web, je peux consulter les buckets et leurs fichiers.

Pour chaque fichier, je peux voir ses propriétés et attributs (et j'ai un bouton pour le télécharger)

# Utilisation avec la CLI simplifiée aws s3

La commande `aws s3` est une façade qui expose des fonctions de haut-niveau, plus simples mais moins puissantes.

Préalable :

```sh
aws sso login --profile my-dev-profile
export AWS_PROFILE=my-dev-profile
```

## Lister les buckets

```sh
aws s3 ls
```

## Lister les fichier d'un bucket
```sh
aws s3 ls my-staging-bucket
```

## Télécharger un fichier d'un bucket

```sh
aws s3 cp s3://my-staging-bucket/myfile.txt /tmp/youpi
```

# Utilisation avec la CLI complète aws s3api

La commande `aws s3api` expose toute la richesse de l'API.

## Lister les tags d'un fichier

```sh
aws s3api get-object-tagging --bucket my-staging-bucket --key test.txt
# {
#     "TagSet": [
#         {
#             "Key": "test-key",
#             "Value": "test-value"
#         }
#     ]
# }
```

# Liste des fonctions de l'API

https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html

Par exemple [la doc de GetObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html)

# Liste des régions S3

https://docs.aws.amazon.com/general/latest/gr/s3.html
