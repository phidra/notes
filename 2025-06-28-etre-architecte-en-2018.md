#  Être architecte logiciel en 2018

- **url** = https://www.youtube.com/watch?v=1igv2rHGKfo
- **type** = vidéo
- **auteur** = [Arnaud LOYER](https://x.com/aloyer) archi chez RATP Smart Systems et [Cyrille MARTRAIRE](https://ddd.academy/cyrille-martraire/), dev expérimenté chez [Arolla](https://www.arolla.fr/)
- **date de publication** = 2018-05-02
- **source** = [Devoxx 2018](https://www.youtube.com/@DevoxxFRvideos)
- **tags** = language>agnostic ; topic>architecture ; level>intermediate

**TL;DR** = prez (longue mais vraiment chouette) sur l'architecture logicielle en général

* [Être architecte logiciel en 2018](#être-architecte-logiciel-en-2018)
   * [Introduction](#introduction)
   * [Découpage](#découpage)
   * [Problématiques du découpage](#problématiques-du-découpage)
      * [Quels contrats d'interface ?](#quels-contrats-dinterface-)
      * [Où placer les boundaries](#où-placer-les-boundaries)
      * [Qui est l'autorité faisant foi pour la donnée ?](#qui-est-lautorité-faisant-foi-pour-la-donnée-)
   * [Suite de la prez](#suite-de-la-prez)
   * [Rôle d'architecte](#rôle-darchitecte)


## Introduction

Plusieurs définition d'architectes, qui s'appliquent à différents niveaux de zoom, du plus zoomé au moins zoomé :

- **applicatif** = architecturer UNE appli (e.g. en archi hexagonale)
- **système** = architecturer plusieurs applis (e.g. utiliser une MQ + un redis en cache)
- **entreprise** = définir les règles d'architecture applicables à toute l'entreprise (ici, on fait des powerpoints et des règles générales)

À chaque fois, on est de moins en moins concret : un archi d'entreprise ne code plus vraiment...

Attention à bien définir le problème avant de chercher des solutions !

Approche pas terrible = "pour résoudre un problème avec X, on va mettre plus de X..."

00:20:30 "blague" sur 3 niveaux de seniorité :

- Junior : I should write a framework for this
- Experienced developer : I wonder if there is a framework for this
- Senior developer : we don't even really need this...

Une "blague" intéressante = il y a trois nombres importants en informatique :

- **zéro** = le cas ne se produit jamais
- **un** = le cas se produit exactement une fois (donc on peut se permettre de ne PAS  gérer la multiplicité)
- **BEAUCOUP** (= e cas se produit plus d'une fois, il FAUT gérer la multiplicité, et dans ce cas, peu importe si c'est 2, 3 ou 200 : c'est "multiple"


**Quality Attributes** = dénomination consacré de tout un tas de notions qui viennent avec nos applications :

- scalability
- resilience
- statefull vs. stateless
- cache invalidation
- replication/consistency
- ...

00:27:00 Google a une règle : quand on doit réécrire un système pour scaler plus, on se force à ne scaler que d'un ordre de grandeur à la fois.

## Découpage

Importance de découper !

Mais plein de mauvaises façons de le faire :

- par couches techniques (BDD / web / business)
- par langage
- par entités
- etc.

**Une seule bonne façon de découper = par domaine fonctionnel , ce qui revient à créer des bounded contexts.**

Pourquoi ? Car une fois découpé en domaine fonctionnel, le couplage est naturellement plus rare et limité

(alors qu'à l'inverse, si p.ex. on découpe par BDD d'un côté, business logic de l'autre, les couches sont très couplées entre elles : à CHAQUE requête client, la couche business logic doit communiquer avec la couche BDD... Si ces couches sont sur des serveurs différents, ça implique plein de traversées du réseau)

Il y a un prix à payer pour découper en couches fonctionnelles : chaque entité doit être partiellement DUPLIQUÉE dans les différents bounded context.

Par exemple, un même USER sera :

- un VISITOR pour le domaine fonctionnel `catalog`
- un PROSPECT pour le domaine fonctionnel `panier`
- un DESTINATAIRE pour le domaine fonctionnel `shipping`

Trois fois la même notion, mais "dupliquée" ; avec des guillemets, car ce ne sont PAS LES MÊMES CHAMPS qui sont dupliqués : p.ex. le shopping ne s'intéresse qu'à l'adresse utilisateur, et pas à ses préférences, ou ses moyens de paiement.

## Problématiques du découpage

Quand on découpe en bounded context, on a trois problématiques :

- où placer les boundaries ?
- quels contrats d'interface ?
- qui est l'autorité faisant foi pour la donnée ?


### Quels contrats d'interface ?

En gros, le contrat d'interface, c'est très très important, et une fois défini, on ne peut plus revenir en arrière.

> "À chaque fois qu'on pète un contrat on offre à tous nos clients de changer de fournisseur"

01:06:00 = le contrat va plus loin que les formats de données échangées : les bugs, les timeouts, ça fait partie du contrat.

Loi d'hyrum = tout le comportement observable devient le contrat

### Où placer les boundaries

Abordé plus haut : par domaine fonctionnel.

Pour faciliter avec la duplication d'objets entre mes bounded contexts, utiliser un cache (si besoin, on peut le pré-charger)

Ça se stratégise : p.ex. dans le bounded context shopping, on n'a pas besoin du très volumineux catalogue de Books, on peut donc ne mettre en cache que ce qui est consulté (et même vider le cache après 30 min)

Ce qui circule entre les bounded contexts, ce sont des DTO

01:16:30 trade off à choisir entre :

- **Autonomy** (shopping copie une partie des données du catalog : il est un peu en retard, mais est autonome : il peut continuer à fonctionner même si le catalog est down ; il est aussi plus rapide)
- **Authority** (à chaque fois qu'il a besoin de données, shopping fait une request au catalog : plus lent, mais la donnée sera forcément à jour)

Si on fait pencher le curseur vers l'Autonomy (ce qu'ils recommandent ici), il ne faut pas que shipping MUTE les données de shopping : il n'a le droit QUE de les consulter en read-only.

### Qui est l'autorité faisant foi pour la donnée ?

01:17:30 KNOW YOUR GOLDEN SOURCE : qui détient l'autorité sur une donnée ? Dans leur exemple, c'est le catalog qui a la golden source.

01:21:00 étapes pour travailler avec sa donnée :

1. connaître sa golden source
2. établir un flux de synchro entre la golden source et les reader
3. détecter les désynchros
4. les réparer

## Suite de la prez

Ils recommandent ces bouquins :

- [Building Microservices](https://www.amazon.fr/Building-Microservices-Sam-Newman/dp/1491950358)
- [Building Evolutionary Architectures: Support Constant Change](https://www.amazon.fr/Building-Evolutionary-Architectures-Support-Constant/dp/1491986360)
- Le livre d'Evans sur le DDD
- [Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions](https://www.amazon.fr/Enterprise-Integration-Patterns-Designing-Deploying/dp/0321200683)
- et également : [A Pattern Language: Towns, Buildings, Construction](https://www.amazon.com/Pattern-Language-Buildings-Construction-Environmental/dp/0195019199), livre sur la construction de bâtiments ; pour le lien entre architecture logicielle et construction de bâtiments

01:31:00 les écrans (UI) peuvent interagir avec PLUSIEURS domaines fonctionnels.

01:38:40 les problématiques de base pour bien découper son application, et les règles associées :

- où placer les boundaries = **Split par fonctionnalités**
- quels contrats d'interface = **Contracts are forever**
- data gouvernance = **Know your golden source**

Exemple concret : quand la golden source est modifiée, elle publie un message sur un MQ pour avertir les readers (qui peuvent p.ex. mettre à jour leur cache)

01:50:50 parallèle entre archi logicielle et de bâtiments :

- building blocks = apps/services/subsystems
- streets = les moyens de communication : bus / MQ / REST
- city planning = rules

01:52:00 différents niveaux de parapluie, pour illustrer le caractère "multi-niveaux" de l'architecture :

- service level
- system level
- enterprise level

01:53:50 archi hexagonale = pas nécessairement à appliquer partout. P.ex. si on n'a pas de logique à protéger à l'intérieur de notre domaine.

"Hexagonal light" = si je comprends bien, une moins forte séparation entre l'hexagone et le monde extérieur (si on n'a pas de logique métier très complexe, je suppose)

Autre cas où archi hexagonale = overkill : le CRUD = l'app est un "cahier" où on écrit dedans, et on lit dedans (le CRUD n'a pas bonne presse, mais a sa place quand même parfois).

Active record (les objets métiers ont leur save et load intégrés), encore plus rare et encore moins bonne presse que le CRUD.

Tous ces patterns sont des patterns LOCAUX (au niveau de parapluie "app", ou au niveau d'architecte bâtiment "maison", par opposition à "bâtiment" et derrière à "city-planning")

01:56:00 "l'architecture c'est que des curseurs" --> NDM = j'aime cette vision orientée "trade-off"

01:57:45 ne pas se tromper de niveau et appliquer un style d'architecture locale (e.g. hexagonal) à toute l'entreprise.

Autre curseur à positionner : **build or buy**

02:01:45 bubble context = mettre un tout petit peu d'innovation dans un legacy (pour ne pas avoir à tout refaire)

02:02:00 exposer un legacy pourri derrière une façade propre, sous la forme d'un macro service (c'est quelque chose que j'ai vu pratiqué avec succès, je suis convaincu que c'est une bonne pratique d'aborder le legacy).

02:15:30 de façon un peu contre-intuitive, qu'un micro service soit très beau à l'intérieur et très bien codé, **on s'en fout**. Ce qui compte c'est les ÉCHANGES entre les services !

02:16:45 le monolithe modulaire est une approche "micro services, mais sans le réseau "

Avec un monolithe modulaire, c'est plus facile de changer les boundaries que si on a des services qui passent par le réseau.

Mêmes patterns archi à appliquer à différents niveaux : app, système, enterprise.

## Rôle d'architecte

02:35:00 = le rôle d'architecte consiste à faire le lien entre les executives et l'IT

L'architecture, c'est 50% de tech, 50% de comm

- Il y a un volet comm
- il faut défendre et faire passer ce qu'on propose
- si on ne fait pas ça, on atteint un plafond de verre dans sa carrière


Différencier 3 trucs :

- rôle d'architecte
- activité d'architecting (plus important que le rôle !)
- architecture = asset (les architectes finiront par partir, mais l'archi qu'ils ont mise en place, elle elle reste)


02:39:30 à la fin de la prez, il y a environ 10 minutes sur les défauts des architectes, assez intéressants :

- astronaute = la théorie, QUE la théorie
- tour d'ivoire = déconnecté de la réalité du terrain
- zero impact = pour diverses raisons, ce qu'il propose n'est jamais adopté
- obsolète = coincé dans le passé (e.g. encore raisonner en scalabilité verticale)
- héros = ils sauvent tout, mais ce qu'ils laissent derrière eux est inutilisable pour les équipes
- celui qui veut impressioner (et qui finit par over-engineerer)
- single-standard zealot = celui qui veut tout unifier à tout prix, et n'utiliser qu'une unique techno partout
- illusions = "cette fois-ci, on fera mieux qu'avant"
- deferred difficulty = je fais un framework qui rend tout facile, après "y'aura plus qu'à l'utiliser"
- (...)
- ops = on doit avancer avec la prod et les ops (sinon ils peuvent tuer ce qu'on fait)

