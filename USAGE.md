# Usage

Cette page décrit mes choix pour utiliser le présent repo.

**Note** : ces instructions ont été rédigées le 28 février 2021, et correspondent à une première itération, à l'état _ACTUELLEMENT_ retenu : il est probable qu'à l'usage, je revienne modifier ce workflow.

**Edit du 18 janvier 2022** : je commence à ajouter à ce repo un équivalent markdown de mes notes OTL privées, dans un sous-répertoire `structured`

## nom du repo github

Nom retenu = `notes`

La grande question, c'est le conflit possible entre :
- les présentes notes déstructurées sur des ressources que je consulte
- des notes OTL publiques que j'aimerais faire (EDIT janvier 2022 : que je commence à intégrer dans ce repo, quoi qu'au format markdown)
- mes notes OTL privées
- mes notes bibliographiques (sur des papiers scientifiques), actuellement dans : https://github.com/phidra/biblio

J'ai tout de même retenu `notes`, malgré les potentiels conflits :
- plus tard le repo fera office de monorepo de notes, à l'intérieur duquel je distinguer les notes structurées et les notes au fil de l'eau (voire les notes bibliographiques) ; pour le moment, seules les notes déstructurées sont dans ce repo (EDIT janvier 2022 : j'ai commencé, le repo contient maintenant également des notes structurées)
- je pourrai toujours [renommer le repo](https://docs.github.com/en/github/administering-a-repository/renaming-a-repository) plus tard

Il y a un lien avec le besoin du lecteur RSS (feedly, miniflux), puisqu'actuellement feedly centralise systématiquement ce que je consulte, et joue le rôle de "mémoire" :
- la plupart des articles sont "simplement" lus, et marqués comme lus
- certains articles sont estampillés `TopArticle`
- quelques articles donnent lieu à des actions, et l'action la plus fréquente est justement de prendre des notes déstructurées (ou parfois structurées) sur le sujet (c'est en ce sens qu'il y a un lien entre le présent repo et le sujet du lecteur RSS). Pour le moment, je ne vais toutefois pas prendre de notes via le lecteur RSS.
- plus rarement, il s'agit de quelque chose à essayer concrètement (e.g. une commande linux)
- parfois encore, il s'agit de pérenniser la ressource dans un réservoir de trucs à faire plus tard : outil à explorer, ou projet qui a l'air chouette à mener

**EDIT janvier 2022** : du coup, les règles qui suivent ne s'appliquent qu'aux notes déstructurées.

## nom de chaque fichier de notes

Le nom du fichier est la concaténation :

1. la date de consultation de la ressource en préfixe (ainsi, les notes sont classées par l'ordre dans lequel j'ai lu/visionné les ressources)
2. un nom de fichier slugifié, rappelant le contenu (si applicable, le titre de la note slugifié)

Exemple ([lien](https://github.com/phidra/notes/blob/c16c543a0de1b27592a12390dd824d1e7a699312/2021-02-24-cppcon-back-to-basics-the-abstract-machine.md)):

```
2021-02-24-cppcon-back-to-basics-the-abstract-machine.md
```
Notamment, les fichiers ne sont PAS classés par date de PUBLICATION de la ressource, ni par la date de RÉDACTION des notes : c'est bien la date de CONSULTATION qui sert de clé de classement.

## emplacement des notes

Actuellement, toutes les notes sont à la racine du repo : il n'y a pas de sous-répertoire

## copie locale des articles

Pour le moment, je ne sauvegarde pas de copie locale des articles (histoire de simplifier la prise de notes), mais c'est une question importante, que j'adresserai à plus tard.

(mon opinion à froid : plutôt que de faire quelque chose moi-même, utiliser les outils de sauvegarde locale du lecteur RSS, genre pocket, miniflux et consorts)

## historique git

Autant que possible, essayer de publier les notes en un seul commit.

Ça pourra être utile de faire plusieurs commits (par exemple, pour pusher l'état et consulter les notes sur le viewer github), et dans ce cas, pusher ces différents commits dans une branche qui sera squashée, puis mergée sur main en fast-forward, pour conserver un historique propre.

## convention = titre

Le premier contenu du fichier est un titre de niveau 1, qui contient le titre exact de la ressource. C'est le seul titre de niveau 1 du fichier.

Exemple :

```
# Back to Basics: The Abstract Machine - Bob Steagall - CppCon 2020
```

## convention = méta-infos

Le second contenu du fichier est une liste à puces contenant les méta-infos de la note.

Exemple :

```
- **url** = https://www.youtube.com/watch?v=ZAji7PkXaKY
- **type** = vidéo
- **auteur** = [Bob STEAGALL](https://github.com/BobSteagall), membre du comité C++
- **date de publication** = 2020-09-22
- **source** = [la chaîne youtube de la CppCon](https://www.youtube.com/channel/UCMlGfpWw-RUdWX_JbLCukXg)
```

Les méta-infos obligatoires sont ceux donnés en exemple ci-dessus :
- `url`
- `type`
- `auteur`
- `date de publication`
- `source`
- `tags`

Précisions :
- il doit y avoir soit une clé `auteur`, soit une clé `auteurs`, jamais les deux, et jamais aucun
- il n'y a volontairement PAS de clé `titre`, pour éviter de dupliquer l'information, qui est déjà dans le titre de niveau 1 du fichier
- il n'y a volontairement PAS de clé `date de lecture`, celle-ci étant dans le titre du fichier
- les `tags` sont séparés par des points-virgules (éventuellement entourés d'espace) : ` ; `

## mention "NdM"

À l'intérieur d'une note, une mention `NdM` (signifiant "Note de Moi") identifie ce qui correspond à mon avis à moi, plutôt qu'une note ou un avis extrait de l'article lui-même, et représentant le point de vue de l'auteur.

Exemple 1 :

> Subsidizing the cost of early console sales. (NdM : incite à l'achat de la console car elle ne coûte pas cher)

- `Subsidizing the cost of early console sales.` est extrait de l'article, et représente l'avis de l'auteur.
- `NdM : incite à l'achat de la console car elle ne coûte pas cher` n'est **PAS** extrait de l'article, et représente ma propre analyse sur le sujet.

Exemple 2 :

> Au chapitre des exemples de second-system effect : python3 (NdM : mais je suis pas tout à fait d'accord avec lui : un changement non-rétrocompatible n'est pas la même chose qu'une réécriture de zéro)

- `au chapitre des exemples de second-system effect : python3` est la note sur l'article (qui représente l'avis de l'auteur)
- `NdM : mais je suis pas tout à fait d'accord avec lui : un changement non-rétrocompatible n'est pas la même chose qu'une réécriture de zéro` n'est **PAS** dans l'article, c'est mon propre avis sur le sujet.

## scripting et base de données

En quelque sorte, du point de vue du scripting, un fichier de note au format markdown va jouer le rôle de "base de données", mais consultable :
- il stocke le contenu de la note, et les métadonnées associées (titre et tags)
- si je m'y prends bien, les métadonnées sont facilement parsables à partir du markdown, d'où le côté "BDD"
- mais l'avantage par rapport à une base de donnée, c'est que le markdown est directement consultable (alors que si je stockais la note dans une BDD, il faudrait une étape de rendu pour pouvoir consulter)

Comme je suis plus intéressé par la prise de note et consultation facile que par le scripting, cet usage un peu bizarre (parser le markdown pour extraire les infos) est un bon compromis à mes yeux.

## Les sujets pending

### résumé de la note

Rendre obligatoire la présence d'un TL;DR en début d'article ?
- +++ à la rédaction de la note, ça me force à prendre du recul
- +++ à la lecture de la note, ça a BEAUCOUP de valeur pour me la remettre en tête sans tout relire
- --- ça augmente l'inertie à la prise de notes

Rien que pour l'inertie, il vaut sans doute mieux pas rendre le TL;DR obligatoire, même si je vais essayer d'en mettre le plus possible.

Éventuellement, forcer la présence d'un TL;DR, en acceptant de ne pas l'avoir rempli (i.e. en acceptant les TL;DR vides ?). Ainsi, j'ai plus de chances d'avoir envie de le remplir, sans que ce soit obligatoire.

### autres tags intéressants ?

Liste non-exhaustive :
- tag tl;dr stockant une description résumant la note ?
- keywords ? (+ gagne en utilité avec une vérification automatique des keywords, par exemple pour corriger les typos)

### TOC

Arf, vu le manque de support de github markdown sur le sujet, il va falloir que j'utilise un outil pour générer automatiquement les TOC... `(>_<')`

### tooling = annotation des vidéos youtube

Lors des notes sur des vidéos youtube, j'ai tendance à préfixer chaque note par la position approximative dans la vidéo. La ligne de note commence alors par quelque chose comme `XX:YY(:ZZ)`.

Je gagnerais à scripter la conversion de ces timestamp en un lien vers le bon endroit de la vidéo.

### tooling = faciliter la conversion depuis des notes OTL

À terme, je pourrai envisager un script qui convertit automatiquement des notes OTL en markdown (pour faciliter la prise de notes) :
- définit le nom du fichier à partir de la date et du titre
- convertit le body OTL en markdown

### tooling = scripting pour vérifier le format des notes

Plus généralement, je peux scripter la vérification du format des notes, par exemple vérifier que tous les tags obligatoires sont bien présents.

### tooling = staleness des permalinks

Partout où je pourrai utiliser des permalinks vers ces notes markdown, je gagnerais à vérifier/mettre à jour automatiquement ces permalinks :
- vérifier que la ligne pointée vers le permalink n'a pas changé dans une version plus récente du repo
- si elle n'a pas changé, remplacer le permalink par un permalink encore plus à jour
- si elle a changé et que la ligne a juste été déplacée, retrouver la ligne équivalente
- si elle a changé et que la ligne n'a pas juste été déplacée, bloquer pour forcer l'utilisateur à retrouver la bonne ligne ?
- (éventuellement, accepter de pointer sur une ancienne ligne depuis disparue ?)
