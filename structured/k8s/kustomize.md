**C'est quoi ?** Un moyen d'adapter la configuration, sans duplication ni templating.

**BESOIN** = adapter la configuration, typiquement aux environnements :

- en prod = on veut 8 replicas, et un niveau de log à "INFO"
- en dev  = on veut 1 replicas, et un niveau de log à "DEBUG"


**SANS KUSTOMIZE** :

- PISTE 1 = dupliquer les fichiers yaml de configuration
    - pas fou, car duplication
- PISTE 2 = templater les fichiers yaml de configuration
    - mieux, mais complexe à gérer (moteur de template + valeurs à stocker)

**AVEC KUSTOMIZE**, pas besoin de duplication ou de templating :

- on définit une config "de base", qui contient toutes les valeurs
- on définit des "patchs" à appliquer sur certains environnement

Exemple :

Dans la version de base, on a un unique replica pour l'application, de petite taille :

```
NB_REPLICAS = 1
RAM = 1Gio
```

Le fichier "de base" est utilisable à 100% (il ne contient pas de placeholder pour les templates p.ex.) ; on ajoute un fichier de patch ("overlay") qui vient écraser ces valeurs :

```
NB_REPLICAS = 8
RAM = 4Gio
```

