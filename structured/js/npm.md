* [Concepts](#concepts)
* [Commandes utiles](#commandes-utiles)
   * [Usages courants](#usages-courants)
   * [Autres usages](#autres-usages)
   * [npm run build et npm run dev](#npm-run-build-et-npm-run-dev)
      * [npm run build](#npm-run-build)
      * [npm run dev](#npm-run-dev)
* [Installation](#installation)
   * [Via un PPA](#via-un-ppa)
   * [analyser la version installée — npm](#analyser-la-version-installée--npm)
   * [analyser la version installée — node](#analyser-la-version-installée--node)
   * [nvm](#nvm)
   * [n](#n)

# Concepts

**C'est quoi ?**

- `node.js` est un moteur d'exécution javascript ; il joue le rôle d'interpréteur javascript indépendant du navigateur (ce qui permet de coder des apps backend en js).
- `npm` est le gestionnaire de paquets pour `node.js` (équivalent de pip pour python) ; il permet de gérer les dépendances d'un projet, son installation nécessite node.js.

**Installation locale / globale** : tout comme pip, npm peut installer des paquets globalements ou localement :

- On peut gérer les paquets localement (dans le répertoire du projet) ou globalement (dans le répertoire système de la machine).
- En général, on n'installe jamais rien en GLOBAL. La seule exception éventuelle (et encore), c'est si on veut installer un outil à utiliser partout (e.g. `typescript` pour avoir le compilateur `tsc`)
- NOTE : par défaut, c'est l'installation LOCALE qui est privilégiée (comme dans pip).

**Fichier package.json**
- Regroupe les infos d'un projet js, et notamment les infos sur les dépendances nécessaires.
- C'est un peu l'équivalent pour npm de requirements.txt pour pip.
- Parmi les clés intéressantes, on a :
    -`dependencies` = dépendances du projet pour un environnement de production (ex: `jquery`)
    -`devDependencies` = dépendances du projet pour un environnement de développement (ex: `typescript`)

**package.json vs. package-lock.json**
- `package.json` est écrit par le développeur, il correspond à la spec de ce qu'il souhaite installer (e.g. _une version de moment.js plus récente que la 2.4_)
- `package-lock.json` est auto-généré par npm, il correspond à la description exhaustive des packages installés, y compris indirectement (e.g. _la version 2.5.8 de moment.js, ainsi que la version 1.2.3 de tartenpion.js dont moment.js a besoin_)

**Scripts**
- npm permet de configurer des "scripts" (tâches automatisées) dans le `package.json`
- ça permet de fournir des commandes simples (e.g. `npm run dev`) pour exécuter des tâches possiblement complexes
- ça permet également de lancer ces tâches en utilisant l'environnement du projet (e.g. d'utiliser le compilateur `tsc`, qui n'est pas installé globalement, mais localement)

**Emplacement des paquets**
- LOCAL  = les paquets sont installés dans le répertoire `node_modules` du projet (créé si nécessaire)
- GLOBAL = les paquets sont installés dans le répertoire d'installation de node (par défaut `/usr/local`)
- Connaître / modifier le répertoire d'installation global :
    ```
    npm config get prefix
    npm config set prefix /home/myself/.npm-global/
    ```

# Commandes utiles

## Usages courants

- `npm install MONPAQUET` = installer `MONPAQUET` **localement** pour le projet courant :
    - ajoute `MONPAQUET` à `dependencies` dans `package.json`
    - télécharge le paquet et ses dépendances dans `node_modules`
    - remplit `package-lock.json` avec les versions exactes 1. du paquet installé et 2. de ses dépendances (récursives)
- `npm install --save-dev  MONPAQUET` = idem, mais `MONPAQUET` est plutôt ajouté à `devDependencies`
- `npm install -g MONPAQUET` = installer `MONPAQUET` globalement :
    - à le même effet en pratique que `pipx install `MONPAQUET`
    - à n'utiliser que pour des outils indépendants des projets (e.g. `http-server`)
    - **attention** : quand on installe des trucs globalement, mieux vaut le faire SANS `sudo` (grâce à la config `set prefix` vers un chemin local à l'user, cf. ci-dessous)
- `npm uninstall MONPAQUET`
- `npm help KEYWORD` = e.g. `npm help install`
- `npm run SCRIPT` = lancer le script définit dans `package.json`, p.ex. :
    - `npm run dev`
    - `npm run build`
- `npx COMMAND` alias `npm exec COMMAND` = exécute un tool installé localement via npm :
    - quand on installe un tool js localement (e.g. le compilateur `tsc` installé avec `npm install --save-dev typescript`), le tool `tsc` existe dans le répertoire `node_modules` du projet, mais il n'est pas dans mon path → je ne peux donc pas l'utiliser !
    - npm sait l'utiliser en tant que script, donc si j'ajoute la ligne suivante dans mon `package.json`, je peux lancer `npm run compile`, et npm utilisera bien le `tsc` de `node_modules` :
        ```
        "compile": "tsc --strict"
        ```
    - `npx` permet de faire l'équivalent, mais sans avoir à définir un script dans `pacakge.json` : `npx tsc --strict`
    - à noter que si le tool n'existe pas dans `node_modules` (et/ou `package.json`), npx propose de le télécharger pour l'utiliser directement :
        ```
        npx tsc
        Need to install the following packages:
          tsc@2.0.4
        Ok to proceed? (y) n
        ```

## Autres usages

- `npm init` = bootstrapper un projet, et créer un `package.json` (à partir des réponses entrées au prompt)
- `npm init vue@latest` = bootstrapper un projet en utilisant le package `create-vue`, cf. `npm help init` :
    > initializer in this case is an npm package named create-<initializer>, which will be installed by npm help npm-exec, and then have its main bin executed -- presumably creating or updating package.json and running any other initialization-related operations.
- `npm update MONPAQUET`
- `npm ls (--depth=1)` = lister les paquets gérés dans le projet actuel (en se limitant à la profondeur 1)
- `npm ls --global` = lister les paquets gérés globalement
- `npm ls MONPAQUET` = indique si `MONPAQUET` est présent ou pas dans le projet
- `npm prune` = supprimer les paquets externes au projet (qui n'apparaissent pas dans package.json)
- `npm outdated` = liste les projets à mettre à jour


## npm run build et npm run dev

Techniquement, ce ne sont pas des commandes npm mais des scripts. Je décris les usages pour `vue.js`, mais dans le cas général, il faut aller voir la configuration dans `package.json`.

### npm run build

**Ça fait quoi ?** Ça crée le répertoire `dist/`, contenant l'app (i.e. les fichiers HTML+CSS+JS) prête à être servie par un serveur web en production.

Ce répertoire est **buildé** par le build-tool (possiblement à partir de code dans des langages différents de HTML+CSS+JS, e.g. typescript ou SASS).

Il contient notamment toutes les librairies estampillées comme dépendances de prod dans `package.json` (probablement bundlées en un seul js).

Exemple :

```
dist
├── assets
│   ├── index.28e3a8fa.js
│   ├── index.f3fa6de6.css
│   └── logo.da9b9095.svg
├── favicon.ico
└── index.html
```

Derrière, on peut p.ex. tester son app avec :

```
python3 -m http.server --directory=dist/ 8787
```

### npm run dev

**Ça fait quoi ?** Ça lance un serveur de dev pour tester mes modifs localement.

Pourquoi ? Parce que appeler `npm run build` suivi du fait de lancer un serveur sur `dist/` à chaque modif est contraignant... À la place, le serveur de dev :

- gère les dépendances décrites dans `package.json`
- gère l'autoreload lors des modifs

# Installation

Pour ce que j'utilise actuellement comme méthode d'installation cf. [mes notes de workflow](../workflow/dev-workflow.md).

## Via un PPA

[Cette page digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04-fr) recommande l'utilisation de [ce PPA](https://github.com/nodesource/distributions#deb), c'est ce que j'ai suivi :

```sh
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install nodejs
sudo apt autoremove
```

Derrière, pour configurer npm afin d'installer des paquets GLOBAUX (e.g. un outil qui sert globalement tel que `http-server`) sans nécessiter de droits root, cf. https://stackoverflow.com/a/59227497 ; dire à npm d'installer les packages globaux dans un répertoire local :

```
REGULARUSER> npm config set prefix '~/.local/'
```

Si nécessaire :

```sh
mkdir -p ~/.local/bin
echo 'export PATH=~/.local/bin/:$PATH' >> ~/.zshrc
```

Une fois ceci fait, je peux installer "globalement" un package en tant que regular-user :

```
REGULARUSER> npm install -g http-server
```

Pour mettre à jour npm :

```
npm install -g npm@latest
```

Pour mettre à jour node via le PPA :

```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```

## analyser la version installée — npm

```sh
npm -v
# 9.2.0

type npm
# npm is /home/myself/.local/bin/npm

readlink -e ~/.local/bin/npm
# /home/myself/.local/lib/node_modules/npm/bin/npm-cli.js

npm ls -g npm
# /home/myself/.local/lib
# └── npm@9.2.0
```

^ Mon npm a été installé... avec npm ! Ça soulève une question : avec quel npm ai-je installé npm ?

```sh
locate npm ...
# ...
# /usr/bin/npm

/usr/bin/npm -v
# 8.15.0
```

^ On dirait que j'ai un npm system-wide à la version `8.15.0`, c'est sans doute avec lui que j'ai installé globalement le package `npm` dans `~/.local/bin/npm`.

```sh
dpkg -l npm
# rc  npm            6.14.4+ds-1ubuntu2 all          package manager for Node.js

apt-cache policy npm
# npm:
#  Installé : (aucun)
#  Candidat : 8.5.1~ds-1
# Table de version :
#     8.5.1~ds-1 500
#        500 http://fr.archive.ubuntu.com/ubuntu jammy/universe amd64 Packages
#        500 http://fr.archive.ubuntu.com/ubuntu jammy/universe i386 Packages
#     6.14.4+ds-1ubuntu2 -1
#        100 /var/lib/dpkg/status
```

^ La version de npm packagée avec ubuntu est `8.5.1`, et elle n'est pas installée, ce n'est donc pas elle qui a été utilisée.

Je soupçonne que l'installation de nodejs avec le PPA a installé npm à cette version `8.15.0` (spoiler : c'est le cas), pour le vérifier, j'analyse le package. Sur [la page du PPA](https://www.ubuntuupdates.org/package/node_16.x/jammy/main/base/nodejs), on dirait que je ne peux télécharger que la dernière version (`16.19.0`), mais en éditant l'URL de download, j'arrive quand même à télécharger ma version :

```sh
mkdir /tmp/analyse_nodejs_16.17.1
cd /tmp/analyse_nodejs_16.17.1
wget https://deb.nodesource.com/node_16.x/pool/main/n/nodejs/nodejs_16.17.1-deb-1nodesource1_amd64.deb
mkdir deb_extracted
dpkg-deb -xv nodejs_16.17.1-deb-1nodesource1_amd64.deb deb_extracted
grep version ./deb_extracted/usr/lib/node_modules/npm/package.json
# "version": "8.15.0",
```

^ En résumé, c'est bien l'installation de nodejs par le PPA qui a installé system-wide une version de npm, et j'ai utilisé ce npm system-wide pour installer un npm récent (et géré par npm lui-même) dans `~/.local/bin/npm`.

## analyser la version installée — node

```sh
node -v
# v16.17.1

type node
# node is /usr/bin/node

dpkg -S /usr/bin/node
# nodejs: /usr/bin/node

dpkg -l nodejs
# ii  nodejs         16.17.1-deb-1nodesource1 amd64        Node.js event-based server-side javascript engine

apt-cache policy nodejs
# nodejs:
#   Installé : 16.17.1-deb-1nodesource1
#   Candidat : 16.19.0-deb-1nodesource1
#  Table de version :
#      16.19.0-deb-1nodesource1 500
#         500 https://deb.nodesource.com/node_16.x focal/main amd64 Packages
#  *** 16.17.1-deb-1nodesource1 100
#         100 /var/lib/dpkg/status
#      12.22.9~dfsg-1ubuntu3 500
#         500 http://fr.archive.ubuntu.com/ubuntu jammy/universe amd64 Packages

# NDM : ci-dessus, 500 est plus prioritaire que 100
```

^ La version installée chez moi est `16.17.1`, elle provient du PPA `node_16.x`, la [page du PPA](https://www.ubuntuupdates.org/ppa/node_16.x?dist=jammy) indique les versions disponibles :

```
2022-12-19 21:09:13 UTC 	nodejs 	16.19.0-deb-1nodesource1
2022-11-04 19:10:55 UTC 	nodejs 	16.18.1-deb-1nodesource1
2022-10-18 21:09:20 UTC 	nodejs 	16.18.0-deb-1nodesource1
2022-09-23 18:08:14 UTC 	nodejs 	16.17.1-deb-1nodesource1
2022-08-16 22:08:10 UTC 	nodejs 	16.17.0-deb-1nodesource1
```

Par ailleurs, [la page du paquet Ubuntu](https://launchpad.net/ubuntu/jammy/+source/nodejs) confirme que la version packagée par Ubuntu est :

```
nodejs 12.22.9~dfsg-1ubuntu3 (universe)
```

Enfin, un petit coup d'oeil à `/var/lib/dpkg/status` :

```sh
grep -A 25 "Package: nodejs" /var/lib/dpkg/status

# Package: nodejs
# Status: install ok installed
# [...]
# Maintainer: Ivan Iguaran <ivan@nodesource.com>
# Version: 16.17.1-deb-1nodesource1
# [...]
```


## nvm

[nvm = Node Version Manager](https://github.com/nvm-sh/nvm) est un outil pour installer et gérer facilement plusieurs versions concurrentes de nodejs.

**Si possible, éviter de l'utiliser** : après l'avoir un temps utilisé, j'ai fini par le dégager, car le profiling du lancement de zsh montrait que nvm ralentissait l'ouverture d'un terminal de façon démentielle.

Je conserve tout de même mes notes d'installation/utilisation :

- installation de nvm pour utiliser node.js / npm sur ubuntu 20.04 :
    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
    ```
- ça me modifie mon .zshrc pour ajouter les lignes suivantes :
    ```
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
    ```
- derrière, je peux accéder à nvm :
    ```sh
    nvm --version
    # 0.39.1
    ```
- lister les versions disponibles :
    ```sh
    nvm list-remote
    # [...]
    # v18.2.0
    ```
- installer nodejs+npm :
    ```sh
    nvm install v18.2.0
    # Downloading and installing node v18.2.0...
    # Downloading https://nodejs.org/dist/v18.2.0/node-v18.2.0-linux-x64.tar.xz...
    # ####################################################################################################################################################################################################################################### 100,0%
    # Computing checksum with sha256sum
    # Checksums matched!
    # Now using node v18.2.0 (npm v8.9.0)
    # Creating default alias: default -> v18.2.0

    npm --version
    # 8.9.0

    node --version
    # v18.2.0
    ```

## n

Un autre outil que j'ai utilisé = [n](https://www.npmjs.com/package/n).

D'un côté il nécessite déjà npm, mais de l'autre il permet d'avoir simplement un npm récent sur un debian based :

```sh
apt install npm                         # installe une vieille version de npm
npm cache clean -f && npm install -g n  # installe n
n stable                                # installe une version récente de npm
```
