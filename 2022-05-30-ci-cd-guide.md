# Guide CI/CD TeamCity

- **url** = https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/
- **type** = white book
- **auteur** = [jetbrains](https://www.jetbrains.com) = éditeur de logiciels qui développe des outils (entre autres CLion et PyCharm, mais également le langage Kotlin) pour les développeurs
- **date de publication** = ?
- **source** = [doc teamcity = serveur CI/CD par jetbrains](https://www.jetbrains.com/fr-fr/teamcity/)
- **tags** = language>agnostic ; topic>CI/CD ; level>beginner


**TL;DR** : livre blanc sur CI/CD et les bonnes pratiques associées. Dans ces notes déstructurées, je reprends le découpage du livre blanc.

* [Guide CI/CD TeamCity](#guide-cicd-teamcity)
   * [Qu'est-ce que l'intégration continue (CI) ?](#quest-ce-que-lintégration-continue-ci-)
   * [Comprendre la livraison continue](#comprendre-la-livraison-continue)
   * [Qu'est-ce que le déploiement continu (CD) ?](#quest-ce-que-le-déploiement-continu-cd-)
   * [Qu'est-ce que le CI/CD en DevOps ?](#quest-ce-que-le-cicd-en-devops-)
   * [Intégration vs. livraison vs. déploiement continus](#intégration-vs-livraison-vs-déploiement-continus)
   * [Quels sont les avantages du CI/CD ?](#quels-sont-les-avantages-du-cicd-)
   * [Comprendre les pipelines de CI/CD](#comprendre-les-pipelines-de-cicd)
   * [Construire un pipeline dans le Cloud](#construire-un-pipeline-dans-le-cloud)
   * [Démarrer avec l'automatisation des builds](#démarrer-avec-lautomatisation-des-builds)
   * [Tests automatisés pour CI/CD](#tests-automatisés-pour-cicd)
   * [Bonnes pratiques CI/CD](#bonnes-pratiques-cicd)
   * [Guide pour les outils de CI/CD](#guide-pour-les-outils-de-cicd)
   * [Qu'est-ce qu'un serveur de CI ?](#quest-ce-quun-serveur-de-ci-)
   * [CI/CD et méthode agile](#cicd-et-méthode-agile)
   * [DevSecOps et son rôle dans le CD](#devsecops-et-son-rôle-dans-le-cd)
   * [Mesure et suivi des performances de CI/CD](#mesure-et-suivi-des-performances-de-cicd)
   * [Concepts de CI/CD](#concepts-de-cicd)

## Qu'est-ce que l'intégration continue (CI) ?

[Qu'est-ce que l'intégration continue (CI) ?](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/continuous-integration/)

**Intégration Continue = Continous Integration = CI** = vérifier automatiquement à chaque commit d'un dev de l'équipe que tout fonctionne toujours.

Sans CI, on se rend compte trop tard de soucis liés à un travail commité il y a longtemps.

Avec CI, on détecte les problèmes très tôt.

Ingrédients = VCS + build automatisé + tests automatisés + infrastructure pour supporter tout ça (e.g. runner gitlab).

## Comprendre la livraison continue

[Comprendre la livraison continue](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/continuous-delivery/)


**Livraison Continue = Continuous Delivery = CD** = mise en prod automatisée : on n'a qu'un seul bouton à appuyer pour mettre en prod (mais la mise en prod n'est pas systématique et requiert toujours une action humaine).

Intérêt :

> Une fois que la publication est fiable et reproductible, il devient facile d'en faire plus souvent et vous pouvez commencer à livrer de petites améliorations sur une base hebdomadaire, quotidienne ou toutes les heures.


## Qu'est-ce que le déploiement continu (CD) ?

[Qu'est-ce que le déploiement continu (CD) ?](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/continuous-deployment/)

**Déploiement Continu = Continuous Deployment = CD** = tout commit valide sera automatiquement déployé en prod, sans intervention humaine.

> Si une modification du code passe avec succès toutes les étapes précédentes du pipeline, cette modification est automatiquement déployée en production sans aucune intervention manuelle.
>
> La [mise en prod] peut devenir un non-événement ayant lieu plusieurs fois par jour. 

Intéressant :

> Le déploiement Canary limite le déploiement du code mis à jour à un petit pourcentage d'utilisateurs, qui deviennent sans le savoir des testeurs en production.

Écueil à éviter = ne pas avoir de synchro avec les PO et le market : ils ont alors l'impression que le déploiement en prod échappe à leur contrôle.

NdM = l'acronyme `CD` est ambigü, vu qu'il peut s'appliquer au déploiement continu ou à la livraison continue, mais en vrai, les deux pratiques ne diffèrent que par la façon dont la mise en prod est déclenchée.

## Qu'est-ce que le CI/CD en DevOps ?

[Qu'est-ce que le CI/CD en DevOps ?](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/devops-ci-cd/)

Les 3 notions mentionnées plus haut sont des pratiques devops.

Cette page explique le contexte : l'agile et ce que ça apporte + le devops et le problème auquel ça répond :

> Même si les développeurs travaillaient plus efficacement au sein de leur propre équipe, une fois que le build était remis aux responsables opérationnels pour qu'ils le déploient en production, il était fréquent que le processus se retrouve bloqué. Des dépendances manquantes, des problèmes de configuration de l'environnement et des bugs qui ne pouvaient pas être reproduits sur les machines locales des développeurs créaient des allers-retours incessants entre les équipes et des désaccords pour déterminer qui était responsable de la résolution du problème.
>
> Combinant « développement » et « opérations », le terme « DevOps » souligne la nécessité de l'intégration des activités des deux équipes pour livrer efficacement des logiciels fonctionnels.

Équipes opérationnelles = celles responsables de la gestion de l'infrastructure et du déploiement des logiciels.

Keypoint devops = automatisation.

Keypoint devops = attraper les erreurs au plus tôt pour 1. être encore dans le contexte (pour faciliter la résolution) et 2. ne pas avoir rajouté plein de code par dessus le code fautif.

Un pipeline de CI/CD mets en pratique les idéaux devops.

## Intégration vs. livraison vs. déploiement continus

[Intégration vs. livraison vs. déploiement continus](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/continuous-integration-vs-delivery-vs-deployment/)

Créer d'un seul coup from scratch un pipeline CI/CD avec les trois étapes ci-dessus n'est pas facile : mieux vaut y aller étape par étape :

CI :

- Utiliser le contrôle de code source
- Valider tôt, valider souvent
- Compiler votre solution à chaque commit
- Automatiser les tests
- **Écouter les retours**
- Développer une culture DevOps

Livraison continue :

- Build unique
- Automatiser chaque déploiement
- **Séparer les préoccupations environnementales**
- Stocker la configuration dans le contrôle de code source
- Nettoyer vos environnements
- Maintenir le pipeline

Déploiement continu :

- **Avoir confiance en vos tests**
- Choisir ce qu'il faut publier
- Contrôler le déploiement
- Surveiller la production
- Rationaliser votre pipeline

## Quels sont les avantages du CI/CD ?

[Quels sont les avantages du CI/CD ?](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/benefits-of-ci-cd/)

Mise sur le marché plus rapide

> L'objectif principal d'un pipeline CI/CD est de délivrer rapidement et fréquemment des logiciels fonctionnels aux utilisateurs.

- Réduction du risque (à entendre comme "risque marketing" : en mettant en prod très tôt, on peut pivoter au plus tôt si une feature ne trouve pas son marché, plutôt que de s'enferrer après avoir investi des années en préparation)
- Temps de révision plus court (à entendre comme "revue de code plus courtes", car comme on commit/livre souvent, on travaille sur de plus petits incréments)
- Meilleure qualité de code (car les tests étant automatisés, ils peuvent être exécutés sur chaque build)
- Chemin simplifié vers la production (entendre : en travaillant régulièrement sur le process de mise en prod, on le maîtrise et l'améliore, vs. on fait une grosse mise en prod douloureuse tous les 18 mois)
- Correction plus rapide des bugs (comme les livraisons sont incrémentales, on détecte facilement les bugs ; comme les livraisons sont rapides, on peut livrer un correctif plus rapidement)
- Infrastructure efficace (l'infrastructure as code qui accompagne naturellement un pipeline de CI/CD est bénéfique en soi)
- Progrès mesurables (les pipelines de CI/CD s'accompagnent de mesures statistiques qui aident au progrés petit à petit ; e.g. couverture de code, temps de build, etc.)
- Boucles de feedback plus courtes (retours d'information plus rapide : trucs cassés, performances en prod, acceptation par les utilisateurs ; permet l'A/B testing)
- Collaboration et communication ( _Supprimer les silos entre le développement et les opérations est le début d'un cercle vertueux._ )
- Créativité maximale = les devs peuvent se concentrer sur les tâches créatives, vu qu'ils sont libérés des tâches lourdes et répétitives.

## Comprendre les pipelines de CI/CD

[Comprendre les pipelines de CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/ci-cd-pipeline/)

Pipeline de CI/CD = ensemble d'étapes mettant en oeuvre la CI/CD :

- commit sur master
- build + tests unitaires + reporting des résultats
- déploiement sur environnement intermédiaire → tests automatisés de niveau supérieur (voire, tests manuels)
- déploiement en production déclenché manuellement (livraison continue) ou automatiquement (déploiement continu)

Feature flag pour ne pas mettre en prod des features encore en cours de dev.

Serveur de build dédié pour éviter des problèmes de dépendances (ça marche sur mon PC)

Environnements de tests automtisés aussi proches que possible de la prod -> VM ou conteneurs aident à la reproductibilité des builds et de l'exécution.

## Construire un pipeline dans le Cloud

[Construire un pipeline dans le Cloud](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/cloud-ci-cd-pipeline/)

Pour mettre en place les tests automatisés et le pipeline de CI/CD, il faut _non seulement des compétences et des outils DevOps, mais aussi un ensemble d'infrastructures pour fournir la capacité de calcul nécessaire au serveur CI, aux agents de build, aux environnements de test et aux banques de données._

On-premises = la question du dimensionnement est pas facile (d'un côté il faut tenir les occasionnelles fortes charges, de l'autre si on surdimensionne, on a des ressources sous-utilisées la plupart du temps).

Cloud = bonne solution dans ce cas, car on adapte les ressources à la charge.

IAAS = la ressource fournie est une VM (ou des conteneurs), dont le dimensionnement est dynamique.

Infrastructure-as-code = les ressources physiques sont vues comme du bétail (cattle) interchangeable, et géré automatiquement par du code (par opposition au bare-metal, où les serveurs sont des pets, avec une durée de vie longue et quelqu'un pour les bichonner). Les environnements peuvent être créés automatiquement.

Conteneurs = alternatives aux VM encore plus unitaire, fournissant plus d'isolation.

Application divisée en plusieurs conteneurs (sur la même machine, ou sur des machines d'un même réseau). Orchestrateur de conteneurs tels que Kubernetes pour faciliter le déploiement et la gestion à grande échelle.

Conteneurs simplifient le pipeline de CI/CD (l'artefact de build est une image de conteneur).

CAAS = Containers as a Service.

Externalisation du matériel = l'équipe ne se concentre plus sur la gestion du matériel, mais plutôt sur le pipeline.

Inconvénient de la CI dans le cloud :

- Connaissances et compétences = équipe à faire monter en compétence.
- Architecture du système = une grosse appli monolithique s'y prêtera moins qu'une appli déjà découpée en microservices.
- Coût = penser à libérer les ressources qu'on n'utilise plus !
- Sécurité = expertise à avoir pour comprendre les risques

Approches hybrides : infrastructure-as-code, conteneurs+orchestration ont leurs origines dans le cloud, mais ils peuvent être utilisées on-premises aussi ; autre possibilité = seule une partie du pipeline de CI/CD est exécutée dans le cloud.

Quand passer dans le cloud ? Quand l'infrastructure disponible est un facteur limitant en termes de rapidité et de débit :

> En déplaçant votre pipeline dans le Cloud, vous n'avez plus à faire un choix entre le provisionnement d'un nombre suffisant de serveurs pour faire face aux pics de demande et le coût d'achat et de gestion de machines qui ne sont pas utilisées en permanence.

## Démarrer avec l'automatisation des builds

[Démarrer avec l'automatisation des builds](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/automated-builds/)

C'est la première étape pour mettre en place un pipeline de CI/CD.

>  Dans ce contexte, le terme « build » ne se limite pas à la compilation et la liaison du code source pour créer un exécutable. Le processus de build comprend une série de vérifications ainsi que le rassemblement de tous les éléments nécessaires à l'exécution de votre programme. Par conséquent, même avec un langage interprété, une étape de build est nécessaire.

Artefacts de build = les fichiers produits par le build (ils continuent leur chemin dans le pipeline de CI/CD pour aller vers les étapes de test, puis de déploiement, voire jusqu'en prod).

C'est important d'automatiser pour la reproductibilité du build, et pour simplifier l'exécution du pipeline (sinon, il faudrait déclencher chaque build manuellement).

- Déclenchement d'un build = à chaque commit sur master.
- Exécution d'un build = sur un serveur de build dédié.
- Exécution des tests = tests U + linting + static checkers à faire en même temps que chaque build.
- Publication des artefacts du build = publier l'artefact (e.g. un installeur ou un binaire) dans un référentiel d'artefacts.
- Métriques et retour d'information = consulter les résultats et des stats (e.g. couverture de code)

## Tests automatisés pour CI/CD

[Tests automatisés pour CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/automated-testing/)

C'est la seconde étape pour mettre en place un pipeline de CI/CD.

Les tests automatisés ne rendent pas les testeurs obsolètes : déjà ils participent au développement des tests automatisés, et de plus ils restent nécessaires pour certains tests manuels.

Pyramide de tests = permet de hiérarchiser les tests dans un pipeline de CI/CD : beaucoup de petits tests rapides à écrire et à exécuter (tests unitaires), moins de tests complexes et lent (tests d'intégration), et encore moins de tests très complexes et très lents (tests end-to-end) ; avec en parallèle des tests de perfs.

Au plus l'environnement est proche de la prod, au mieux c'est ; de plus, l'environnement doit être répétable -> automatiser la création d'un env de test est un bon investissement.

Utiliser les retours d'information = il est important de prendre en compte les retours sur les tests qui échouent, e.g. board.

Le CI/CD est-il la fin des tests manuels ? TL;DR = non :

> Au lieu de passer du temps sur des tâches répétitives, les testeurs peuvent se concentrer sur la définition de cas de test, la rédaction de tests automatisés et l'application de leur créativité et de leur ingéniosité à des tests exploratoires

test exploratoire = recherche manuelle de ce qui n'est pas encore testé automatiquement.

Amélioration continue pour l'automatisation des tests = les tests automatiques doivent être maintenus ! _la construction d'une suite de tests n'est pas quelque chose que l'on fait une fois pour toutes sans jamais y revenir._

Attention aussi :

>  il faut éviter le piège qui consiste à penser que la couverture des tests est un objectif en soi. Le véritable objectif est de livrer régulièrement des logiciels fonctionnels à vos utilisateurs.

## Bonnes pratiques CI/CD

[Bonnes pratiques CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/ci-cd-best-practices/)

Il faut voir la construction d'un pipeline de CI/CD comme une tâche évolutive :

> La construction d'un pipeline de CI/CD ne devrait pas être un exercice passif. Tout comme le logiciel en cours de développement, il est payant d'adopter une approche itérative de vos pratiques de CI/CD

- Valider tôt, valider souvent = essayer de pusher sur master (et ainsi trigger le pipeline et ses vérifications) au moins une fois par jour. La culture d'équipe aide à ça.
- Maintenir les builds dans le vert = _Si un build échoue pour une raison quelconque, la priorité de l'équipe doit être de le faire refonctionner._
- Ne faire qu'un build = _le même artefact de build doit être promu à travers chaque étape du pipeline de CI/CD et publié en production_ (pour ça, il faut que le build ne dépende pas de l'environnement : _Les variables, les paramètres d'authentification, les fichiers de configuration ou les scripts doivent être appelés par le script de déploiement plutôt que d'être incorporés dans le build lui-même_)
- Rationaliser vos tests = attention à ne pas faire de tests longs car trop exhaustifs. Commencer par les tests rapides, puis seulement investir dans des tests plus complexes.
- Nettoyer vos environnements = idéalement, chaque exécution d'un run de test repart d'un environnement clean.
- En faire le seul moyen de déployer en production = ne pas shunter les tests (même exceptionnellement) pour mettre en prod.
- Surveiller et mesurer votre pipeline = analyser les mesures pour identifier les problèmes ou les domaines à améliorer.
- En faire un travail d'équipe = décloisonner, faire participer tout le monde, promouvoir une culture de confiance.

## Guide pour les outils de CI/CD

[Guide pour les outils de CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/ci-cd-tools/)

Cette page regroupe les critères de choix d'un outil de CI/CD :

- Prise en charge de la pile technologique = l'outil de CI/CD gère les langages, frameworks, etc. utilisés par l'équipe.
- Des interfaces pour tous = IHM pour les développeurs, mais également PO ou équipe d'assistance.
- Personnalisable = chaque équipe a ses spécificités, l'outil doit pouvoir s'adapter.
- Retours d’expérience et commentaires = pouvoir être notifié de l'état du pipeline.
- Options d'infrastructure = choix du cloud à utiliser ? On-premises possible ?
- Sécurité = e.g. l'outil garde une trace des modifications.
- Performances = e.g. l'outil doit permettre de tirer parti de ressources parallélisées dans le cloud.
- Métriques = fournir des stats permettant le monitoring et l'amélioration du pipeline.
- Assistance = assistance payante dispo ?
- Coût (NdM : je note que c'est leur dernier critère)

Bonus important :

> DevOps est souvent décrit comme nécessitant trois éléments clés pour sa mise en pratique : la culture, les processus et les outils.

## Qu'est-ce qu'un serveur de CI ?

[Qu'est-ce qu'un serveur de CI ?](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/ci-cd-tools/servers/)

Intégration avec le VCS : surveillance des commits pour trigger des builds/tests (possiblement, avant acceptation du commit).

> Afin d'éviter les conflits de ressources et les problèmes de performance, il est recommandé de garder votre serveur CI distinct des agents de build sur lesquels vous exécutez les builds et les tests

On peut avoir plusieurs agents de builds (qui permettent d'exécuter des tâches en parallèle).

> Un serveur de qui s'intègre à une infrastructure hébergée dans le cloud, comme AWS, vous permet de bénéficier de ressources flexibles et évolutives pour exécuter vos builds et vos tests.

Questions à se poser pour définir ce qu'est un échec du pipeline exactement (e.g. le linting doit-il faire échouer le pipeline ? et une diminution de la couverture de tests ?)

La plupart des serveurs de CI s'occupe aussi du CD.

Éventuellement, utiliser un artifact repository.

On devrait pouvoir suivre les avancées et être notifiés des résultats + historiser les métriques.

## CI/CD et méthode agile

[CI/CD et méthode agile](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/agile-continuous-integration/)

> En tant que précurseur de DevOps, et donc de l'intégration, la livraison et le déploiement continus (CI/CD), la méthode agile est étroitement liée à ces approches.

Rappel des principes agiles = _l'objectif est de délivrer un logiciel fonctionnel et que l'acceptation du changement et la promotion de la collaboration entre les individus est un moyen plus efficace d'y parvenir que de suivre scrupuleusement un plan répondant à un ensemble contractuel d'exigences_

> si vous définissez des exigences immuables dès le départ et que vous suivez rigoureusement un plan pour les satisfaire, vous perdez la flexibilité de pouvoir adapter ce que vous développez à mesure que vous en apprenez davantage et que le contexte et les besoins de vos utilisateurs évoluent. L'approche agile consiste à fixer un objectif final et à progressivement définir les détails de la manière de l'atteindre.

Agile se marie bien avec CI/CD, qui accélère le feedback en permettant de mettre en prod rapidement. Autre exemple : travailler par petits incréments, ce que promeut le pipeline CI/CD fonctionne aussi bien avec des sprints courts de l'agile.

## DevSecOps et son rôle dans le CD

[DevSecOps et son rôle dans le CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/what-is-devsecops/)

> la sécurité est souvent considérée par les équipes de développement comme une charge plutôt qu'un atout

...

> L'approche DevSecOps consiste à appliquer aux pratiques de sécurité les mêmes principes de collaboration, de partage des responsabilités et d'automatisation de tout ce qui peut l'être

Avant, avec le modèle waterfall, tout était très long, mais il y avait une étape d'audit de sécurité.

DevSecOps = intégrer la sécurité aux trois piliers devops = culture, processus, outils + appliquer le shift-left à la sécurité.

NdM : je découvre l'expression [Shift-left](https://en.wikipedia.org/wiki/Shift-left_testing) = décaler vers la gauche, pour échouer au plus tôt (plutôt que tardivement).

Concrètement shift-left la sécurité, ça consiste à :

- Nommer des ambassadeurs de la sécurité (pour partager la culture de la sécurité)
- Faire de la sécurité une contrainte de conception (intégrée aux stories, ou à la stratégie de tests)
- Ajouter des tests de sécurité automatisés au pipeline
- Obtenir l'aide d'un expert : utiliser les outils spécialement conçus pour ça (e.g. analyseurs statiques, ou fuzzers).
- Vérifier vos dépendances (il existe des outils pour ça SCA)
- Faire appel à l'équipe rouge (une équipe interne joue le rôle de l'ennemi)
- Traiter toutes les vulnérabilités comme des bugs (i.e. elles ne sont pas distinctes des bugs ordinaires, et la responsabilité de les corriger revient à toute l'équipe)
- Continuer à surveiller la production

## Mesure et suivi des performances de CI/CD

[Mesure et suivi des performances de CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/devops-ci-cd-metrics/)

Principe = recueillir du feedback pour améliorer continuement les différentes étapes du pipeline lui-même.

Les quatres métriques du DevOps Research and Assessment (DORA), permettant de mesurer la performance du pipeline :

- Lead time = délai d'exécution = En théorie : temps écoulé entre le moment où une fonctionnalité est évoquée pour la première fois et celui où elle est mise à la disposition des utilisateurs ; en pratique : temps écoulé entre la validation du code et le déploiement.
- Deployment frequency = Fréquence de déploiement (en vrai, c'est un proxy pour mesurer plutôt la taille de ce qui est poussé en prod)
- Change failure rate = Taux d'échec des modifications : nombre de fois où une MEP entraine un échec (bug, downtime, rollback, etc.)
- Mean time to recovery = MTTR = Temps moyen de récupération (et une métrique connexe = MTTD : temps entre le déploiement d'une modification et la détection par votre système de surveillance d'un problème introduit par cette modification)

Autres métriques opérationnelles en plus de ces quatre là :

- Couverture du code
- Durée du build
- Taux de réussite du test
- Temps de correction des tests (temps qui s'écoule entre le moment où un build signale l'échec d'un test et celui où le même test réussit lors d'un build ultérieur)
- Déploiements échoués : l'objectif idéal n'est PAS forcément que ça tende vers zéro, puisque dans ce cas, on peut être paralysés en ne voulant livrer que quand on est certains à 100% que tout va bien.
- Nombre de défauts (= nombre de tickets dans le backlog)
- Taille du déploiement

La page conclut sur un warning : il faut contextualiser toute métrique :

> l'objectif n'est pas les chiffres eux-mêmes, mais la rapidité et la fiabilité de votre pipeline, afin que vous puissiez continuer à offrir de la valeur aux utilisateurs

## Concepts de CI/CD

[Concepts de CI/CD](https://www.jetbrains.com/fr-fr/teamcity/ci-cd-guide/concepts/)

Cette page est une liste de concepts avec définition, je prends plutôt l'équivalent en anglais : https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/

- [Canary Release](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/canary-release/) = livrer une feature en prod à un petit nombre d'utilisateurs qui vont servir de testeurs.
- [Code Coverage](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/code-coverage/) = proportion de code couverte par les tests automatiques
- [Configuration Management](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/configuration-management/) = versionner la configuration des serveurs des différents environnements (e.g. en prod, on a 16 coeurs, mais le serveur de test n'en a que 2)
- [Continuous Delivery Maturity Model](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/continuous-delivery-maturity-model/) = modélise l'adoption graduelle des pratiques de CI/CD, depuis un degré zéro (le code n'est même pas versionné) à un degré maximal (plusieurs MEP par jour en déploiement continu)
- [Deployment Automation](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/deployment-automation/) = automatisation du déploiement : une seule commande pour déployer une mise à jour sur tous les environnements, y compris la prod, nécessaire pour le CD.
- [Feature Flags](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/feature-flags/) = activer/désactiver certaines features sans modifier le code (dans le cas le plus simple, avec un fichier de conf, mais il existe des tools plus évolués)
- [Flaky Tests](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/flaky-tests/) = des tests qui parfois passent, parfois non, alors que le code n'a pas été modifié ; dangereux car diminuent la confiance en les tests automatiques.
- [Release Orchestration](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/release-orchestration/) = l'orchestration de plusieurs tâches (possiblement réalisées par plusieurs agents indépendants) pour livrer les mises à jour en prod.
- [Static Code Analysis](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/static-code-analysis/) = checks automatiques sur le code-source.
- [Trunk Based Development](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/trunk-based-development/) = stratégie de gestion des branches VCS, où tout est commité sur master (ce qui trigge le pipeline) plutôt que sur des branches. À coupler si besoin avec les feature flags.
- [Value Stream Mapping](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/value-stream-mapping/) = technique d'analyse des process pour identifier les endroits sous-optimaux où on perd du temps (e.g. unwanted features, task switching, waiting, etc.)
- [Version Control](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/version-control/) = VCS = garder une trace de tout changement dans la codebase.
- [Artifact Repository](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/artifact-repository/) = un repo pour stocker les artefacts de build, dans lequel piocher (via une API) pour effectuer les déploiements.
- [Branching strategies for CI/CD](https://www.jetbrains.com/teamcity/ci-cd-guide/concepts/branching-strategy/) = quelles branches utiliser, et quand les merger.
