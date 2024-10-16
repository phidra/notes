Notes vrac sur les concepts AWS S3, à la lecture de [la doc](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html).

* [Différence entre tags et metadata](#différence-entre-tags-et-metadata)
   * [Tags](#tags)
   * [Metadata](#metadata)
   * [Éléments supplémentaires](#éléments-supplémentaires)
* [Introduction du userguide](#introduction-du-userguide)
   * [Features](#features)
      * [Storage classes](#storage-classes)
      * [Storage Management features](#storage-management-features)
      * [Access management and security features](#access-management-and-security-features)
      * [Data Processing features](#data-processing-features)
      * [Storage logging and monitoring features](#storage-logging-and-monitoring-features)
      * [Analytics and insights features](#analytics-and-insights-features)
   * [How Amazon S3 works](#how-amazon-s3-works)
      * [Bon résumé des concepts](#bon-résumé-des-concepts)
      * [Buckets](#buckets)
      * [Objects](#objects)
      * [Keys](#keys)
      * [S3 Versioning](#s3-versioning)
      * [Version ID](#version-id)
      * [Bucket policy](#bucket-policy)
      * [S3 Access Points](#s3-access-points)
      * [Access control lists (ACLs)](#access-control-lists-acls)
      * [Regions](#regions)
      * [Amazon S3 data consistency model](#amazon-s3-data-consistency-model)
      * [Related services](#related-services)


# Différence entre tags et metadata

Les **tags** et les **metadata** permettent tous deux d'associer une paire `key=value` à un objet S3. Quelle différence entre les deux ?

**TL;DR** : les tags permettent de catégoriser les objets les uns par rapport aux autres ; les metadata ont plus vocation à ajouter une info à un objet individuel, sans notion de relation aux autres objets.


## Tags

Limités à 10 par objet S3.

On peut les utiliser pour rechercher et filtrer par API S3, ils sont donc utilisés pour catégoriser et filtrer les données (e.g. il semble possible de récupérer "tous les fichiers ayant ce tag")

## Metadata

Ce sont des infos supplémentaires ajoutés avec le fichier.

Ils ne sont pas utilisables pour filtrer/catégoriser dans les API S3 (e.g. impossible de récupérer "tous les fichiers ayant cette metadata")

Par contre, il est possible, étant donné un fichier en particulier, de récupérer ses metadata

On peut donc les utiliser pour le terme littéral "metadata" = des données sur l'objet S3.

## Éléments supplémentaires

[Cette page stackoverflow](https://stackoverflow.com/questions/42126348/difference-between-object-tags-and-object-metadata) est ancienne (2017) mais donne des éléments supplémentaires :

Les metadata "vont avec" l'objet → quand on modifie les metadata, on créée en quelque sorte un nouvel objet avec ses nouvelles metadata (l'objet est versionné différemment car l'objet S3 a muté) ; les metadata sont d'ailleurs incluses dans les headers de la réponse HTTP quand on GET un object S3.

À l'inverse, les tags sont des objets à part → quand on modifie les tags, on modifie un objet différent (ce qu'on mute n'est pas l'objet S3, c'est un objet "tag" associé à l'objet S3).


Ils n'ont pas les mêmes limites :

- The total of all metadata keys and values for each object is limited to 2KB.
- The limits on tags are different. Each object can have up to 10 tags, each tag key is limited to 128 characters (not bytes), and each tag value is limited to 256 characters (not bytes)

> Neither tags nor metadata can be used for "scanning" objects. It is not possible to ask the S3 service for a list of objects with specific tags or with specific metadata.

^ ce n'est pas ce qu'a dit ChatGPT, donc soit il s'est planté (ce serait pas la première fois...) soit les choses ont changé depuis la date de la réponse.

Les tags autorisent deux usages que ne permettent pas les metadata :

- définir des autorisations qui dépendent des tags (e.g. tel user ne peut pas accéder à tel objet, SAUF s'il a tel tag défini à telle valeur)
- permettr de customiser des politiques de suppression automatique de données (c'est justement ce qu'on va faire)


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
