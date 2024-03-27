Notes vrac sur AWS S3, notamment à la lecture de [la doc](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html).

- [Utilisation](#utilisation)
  * [Console web](#console-web)
  * [CLI](#cli)
- [Introduction du userguide](#introduction-du-userguide)
   * [Features](#features)
      + [Storage classes](#storage-classes)
      + [Storage Management features](#storage-management-features)
      + [Access management and security features](#access-management-and-security-features)
      + [Data Processing features](#data-processing-features)
      + [Storage logging and monitoring features](#storage-logging-and-monitoring-features)
      + [Analytics and insights features](#analytics-and-insights-features)
   * [How Amazon S3 works](#how-amazon-s3-works)
      + [Bon résumé des concepts](#bon-résumé-des-concepts)
      + [Buckets](#buckets)
      + [Objects](#objects)
      + [Keys](#keys)
      + [S3 Versioning](#s3-versioning)
      + [Version ID](#version-id)
      + [Bucket policy](#bucket-policy)
      + [S3 Access Points](#s3-access-points)
      + [Access control lists (ACLs)](#access-control-lists-acls)
      + [Regions](#regions)
      + [Amazon S3 data consistency model](#amazon-s3-data-consistency-model)
      + [Related services](#related-services)
- [Liste des fonctions de l'API](#liste-des-fonctions-de-lapi)
- [Liste des régions S3](#liste-des-régions-s3)

# Utilisation

## Console web

Via la console web, je peux consulter les buckets et leurs fichiers.

Pour chaque fichier, je peux voir ses propriétés et attributs (et j'ai un bouton pour le télécharger)

## CLI

```sh
# préalable :
aws sso ...
export AWS_PROFILE=...

# lister les buckets :
aws s3 ls

# lister les fichier d'un bucket :
aws s3 ls my-staging-bucket

# télécharger un fichier d'un bucket :
aws s3 cp s3://my-staging-bucket/myfile.txt /tmp/youpi
```

# Introduction du userguide

https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

Les atouts mis en avant :

- availability
- security
- performance

Note : S3 est utilisé pour d'énormes volumes, donc même pour mes gros usages, je ne serais jamais limité (sauf par le coût).

## Features

### Storage classes

Il y a plusieurs types de stockages à différents prix, en fonction des patterns d'accès :

- e.g. stockage pour les fichiers archivés très peu souvent consultés
- e.g. S3 Express One Zone a une très faible latence : single-digit-millisecond
- il y a même une classe de stockage "intelligente" qui s'adapte dynamiquement aux accès effectués

### Storage Management features

- lifecycle (e.g. faire expirer des objets après une période de temps)
- object lock = empêcher que des objets soient supprimés
- replication = cloner le fichier et ses métadonnées dans un autre bucket / une autre région
- batch operations = faire d'un seul coup plusieurs opérations sur une multitude de fichiers

### Access management and security features

- Block public access = empêcher l'accès public (c'est le cas par défaut)
- IAM = Identity and Access Management = contrôle de qui a accès à quoi
- etc.

### Data Processing features

- lambda
- event notifications

### Storage logging and monitoring features

.

### Analytics and insights features

.

## How Amazon S3 works

### Bon résumé des concepts

- Amazon S3 is an object storage service that stores data as objects within buckets.
- An object is a file and any metadata that describes the file.
- A bucket is a container for objects.
- To store your data in Amazon S3, you first create a bucket and specify a bucket name and AWS Region.
- Then, you upload your data to that bucket as objects in Amazon S3.
- Each object has a key (or key name), which is the unique identifier for the object within the bucket.

> you can use S3 Versioning to keep multiple versions of an object in the same bucket,

^ possibilité de gérer plusieurs versions d'une même donnée.


### Buckets

- A bucket is a container for objects stored in Amazon S3. You can store any number of objects in a bucket and can have up to 100 buckets in your account.
- Every object is contained in a bucket.
- For example, if the object named `photos/puppy.jpg` is stored in the `DOC-EXAMPLE-BUCKET` bucket in the `US West (Oregon)` Region, then it is addressable by using the URL `https://DOC-EXAMPLE-BUCKET.s3.us-west-2.amazonaws.com/photos/puppy.jpg`.

### Objects

- The metadata is a set of name-value pairs that describe the object.
- These pairs include some default metadata, such as the date last modified, and standard HTTP metadata, such as Content-Type.

^ la date de dernière modification est déjà incluse par défaut dans les metadata, inutile de la gérer moi-même.

### Keys

- An object key (or key name) is the unique identifier for an object within a bucket. Every object in a bucket has exactly one key.
- The combination of a bucket, object key, and optionally, version ID (if S3 Versioning is enabled for the bucket) uniquely identify each object.
- So you can think of Amazon S3 as a basic data map between "bucket + key + version" and the object itself.

^ une bonne vision du stockage S3 (et qui explique en partie qu'il n'existe pas d'opération de renommage)

NDM : l'exemple donné semble indiquer qu'on peut hiérarchiser les fichiers au sein d'un bucket.

### S3 Versioning

> You can use S3 Versioning to keep multiple variants of an object in the same bucket.

### Version ID

> When you enable S3 Versioning in a bucket, Amazon S3 generates a unique version ID for each object added to the bucket.

^ l'attribution d'un id est automatique ; les objets déjà existants au moment où on active le versioning ont l'id `null`.

### Bucket policy

- A bucket policy is a resource-based AWS Identity and Access Management (IAM) policy that you can use to grant access permissions to your bucket and the objects in it.
- Only the bucket owner can associate a policy with a bucket.
- The permissions attached to the bucket apply to all of the objects in the bucket that are owned by the bucket owner.

^ l'owner d'un bucket peut définir des permissions

### S3 Access Points

- Amazon S3 Access Points are named network endpoints with dedicated access policies that describe how data can be accessed using that endpoint.
- Access Points are attached to buckets that you can use to perform S3 object operations, such as GetObject and PutObject.

^ apparemment, là d'où on peut accéder à un bucket S3 est un sujet pertinent (mais je ne creuse pas).

### Access control lists (ACLs)

> You can use ACLs to grant read and write permissions to authorized users for individual buckets and objects.

^ les ACL sont également un moyen de contrôler l'accès au bucket.

### Regions

- You can choose the geographical AWS Region where Amazon S3 stores the buckets that you create.
- Objects stored in an AWS Region never leave the Region unless you explicitly transfer or replicate them to another Region.

^ un bucket est associé à une unique région

### Amazon S3 data consistency model

Il y a pas mal d'explications sur la concurrence, à laquelle je ne m'intéresse pas trop pour le moment, car ce n'est pas mon contexte actuel.

### Related services

> After you load your data into Amazon S3, you can use it with other AWS services.

^ sans surprise, S3 est intégré avec les autres services AWS.

# Liste des fonctions de l'API

https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations_Amazon_Simple_Storage_Service.html

Par exemple [la doc de GetObject](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObject.html)

# Liste des régions S3

https://docs.aws.amazon.com/general/latest/gr/s3.html

