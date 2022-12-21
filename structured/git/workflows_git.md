* [Gitflow](#gitflow)
   * [Extraits](#extraits)
   * [Les keypoints](#les-keypoints)
* [Trunk-based development](#trunk-based-development)
   * [Extraits](#extraits-1)
   * [Les keypoints](#les-keypoints-1)
* [Feature flags](#feature-flags)
   * [What are feature flags](#what-are-feature-flags)
   * [Benefits of Feature Flags](#benefits-of-feature-flags)
   * [Feature flag use cases](#feature-flag-use-cases)
   * [How to implement feature flags](#how-to-implement-feature-flags)

**Contexte** : fin 2022, je m'intéresse aux différents workflow, et atlassian a quelques pages résumant le sujet.

# Gitflow

https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow

## Extraits

> Instead of a single main branch, this workflow uses two branches to record the history of the project.
>
> The main branch stores the official release history, and the develop branch serves as an integration branch for features.
>
> It's also convenient to tag all commits in the main branch with a version number.

^ La branche `main` contient les différentes releases : chaque commit dans la branche main est une release. La branche `develop` est la branche de travail au quotidien.

> Each new feature should reside in its own branch, which can be pushed to the central repository for backup/collaboration.
>
> But, instead of branching off of main, feature branches use develop as their parent branch.
>
> When a feature is complete, it gets merged back into develop.

^ C'est comme ce qu'on fait au boulot (ou plutôt, ce qu'on faisait avant de supprimer develop) = on tire les feature-branchs depuis develop, et on merge son travail sur develop une fois terminé (en effet, develop est la branche de travail quotidienne, main étant réservée aux releases)

> Once develop has acquired enough features for a release (or a predetermined release date is approaching), you fork a release branch off of develop.
>
> Creating this branch starts the next release cycle, so no new features can be added after this point—only bug fixes, documentation generation, and other release-oriented tasks should go in this branch.
>
> Once it's ready to ship, the release branch gets merged into main and tagged with a version number.
>
> In addition, it should be merged back into develop, which may have progressed since the release was initiated.

Livrer une release revient à merger develop sur main :
- pour cela, on tire une branche "release" depuis develop (cette branche de release servant à ce qu'elle ne reçoive pas les nouvelles features qui continuent d'être mergées sur develop)
- le cas échéant, on publie les bugfix sur cette branche de release
- tout ce qu'on fixe dans cette branche de release doit être rétro-mergé develop
- et quand la release est prête (on a corrigé tous les bugs et mis à jour toute la doc), la branche de release est mergée dans main (puis détruite)

> Using a dedicated branch to prepare releases makes it possible for one team to polish the current release while another team continues working on features for the next release.
>
> It also creates well-defined phases of development (e.g., it's easy to say, “This week we're preparing for version 4.0,” and to actually see it in the structure of the repository).


Intérêt = on peut freezer une release en terme de feature tout en continuant à corriger les bugs dessus.

Intérêt = on identifie bien où on en est dans le cycle : s'il existe une branche de release, c'est qu'on est en train de préparer une livraison.


> “hotfix” branches are used to quickly patch production releases
>
> Hotfix branches are a lot like release branches and feature branches except they're based on main instead of develop.
>
> This is the only branch that should fork directly off of main.
>
> As soon as the fix is complete, it should be merged into both main and develop (or the current release branch), and main should be tagged with an updated version number.
>
> You can think of maintenance branches as ad hoc release branches that work directly with main

Si on a besoin de corriger une release :
- on créée une branche de hotfix tirée depuis main
- on fait le hotfix sur cette branche
- puis on rétro-merge dans master (en créant un nouveau numéro de release) ET dans develop

De ce point de vue, une branche de hotfix est proche d'une branche de release, vu qu'elle donnera naissance à une nouvelle release (mais elle est basée sur main plutôt que sur develop).

## Les keypoints

> The overall flow of Gitflow is:
> - A develop branch is created from main
> - A release branch is created from develop
> - Feature branches are created from develop
> - When a feature is complete it is merged into the develop branch
> - When the release branch is done it is merged into develop and main
> - If an issue in main is detected a hotfix branch is created from main
> - Once the hotfix is complete it is merged to both develop and main
>
> Gitflow is a legacy Git workflow that was originally a disruptive and novel strategy for managing Git branches. Gitflow has fallen in popularity in favor of trunk-based workflows, which are now considered best practices for modern continuous software development and DevOps practices

^ last but not least (même si l'extrait est le début de la page) : gitflow est considéré comme deprécié au profit de trunk-based


# Trunk-based development

https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development

## Extraits

> Trunk-based development is a version control management practice where developers merge small, frequent updates to a core “trunk” or main branch

^ ici, on merge directement sur le tronc, et fréquemment

> It’s a common practice among DevOps teams and part of the DevOps lifecycle since it streamlines merging and integration phases.
>
> In fact, trunk-based development is a required practice of CI/CD.

^ en lien avec CI/CD = ça revient à faire des "releases" (= merge sur le tronc) très souvent, avec tests automatisés

> Developers can create short-lived branches with a few small commits compared to other long-lived feature branching strategies.

^ le focus est sur le fait de faire des petits commits unitaires mergés rapidement sur le tronc, plutôt que des feature-branchs au long cours

> Gitflow has more, longer-lived branches and larger commits than trunk-based development
>
> Under this model, developers create a feature branch and delay merging it to the main trunk branch until the feature is complete.
>
> These long-lived feature branches require more collaboration to merge as they have a higher risk of deviating from the trunk branch and introducing conflicting updates.

(NdM : je suppose que c'est pour ça que gitflow est déprécié en faveur de trunk-based : mieux vaut éviter de garder des feature-branchs au long court)

> Trunk-based development is far more simplified since it focuses on the main branch as the source of fixes and releases.
>
> In trunk-based development the main branch is assumed to always be stable, without issues, and ready to deploy.


^ Intéressant (et comment conjuguer les bugfix et les features en cours de développement — j'imagine décrit dans la suite du post, probablement par feature-switch — sera intéressant aussi)

> Trunk-based development is a required practice for continuous integration.
>
> If build and test processes are automated but developers work on isolated, lengthy feature branches that are infrequently integrated into a shared branch, continuous integration is not living up to its potential.

^ ça n'est intéressant d'avoir une CI (i.e. une validation automatique du tronc) que si on merge fréquemment dans le tronc

> When developers finish new work, they must merge the new code into the main branch.
>
> Yet they should not merge changes to the truck until they have verified that they can build successfully.
>
> During this phase, conflicts may arise if modifications have been made since the new work began.
>
> In particular, these conflicts are increasingly complex as development teams grow and the code base scales.
>
> This happens when developers create separate branches that deviate from the source branch and other developers are simultaneously merging overlapping code.

^ Rien de nouveau : c'est au moment de merger une branche au long cours (ou de la rebaser) que les soucis arrivent.

Plus le travail dans la branche est grand, plus on a de chance que ça se passe mal. NdM : à deux niveaux :

- quand on rebase sa branche sur master
- après qu'on a mergé sa branche sur master, quand un autre dev doit se rebaser sur le master intégrant ce gros travail

Ce qui est sous-entendu implicitement, c'est que merger fréquemment limite les risques de conflits.

> When new code is merged into the trunk, automated integration and code coverage tests run to validate the code quality.

^ rien de nouveau non plus : une CI bien ordonnée commence par des tests auto.

> The rapid, small commits of trunk-based development make code review a more efficient process.

^ autre avantage du trunk based = avec de plus petits commits, les MR sont plus faciles

> Teams should make frequent, daily merges to the main branch.
>
> Trunk-based development strives to keep the trunk branch “green”, meaning it's ready to deploy at any commit.

^ Ah, enfin l'article commence à expliquer concrètement de quoi on parle :-)

- point 1 = merges fréquents (quotidiens)
- point 2 = le tronc est tout le temps vert, prodable à tout moment

> Trunk-based development ensures teams release code quickly and consistently. The following is a list of exercises and practices that will help refine your team's cadence and develop an optimized release schedule.

^ en fait il y a toujours pas de process clairement défini (en dehors du process simpliste = le trunk doit être toujours livrable + tests automatiques au merge sur le trunk + merges fréquents sur le trunk)

## Les keypoints

- **Develop in small batches** : viser des petits merges.
- **Feature flags** :
    > This allows developers to forgo creating a separate repository feature branch and instead commit new feature code directly to the main branch within a feature flag path.
    - NDM : pour moi, c'est le point clé qui permet les merges fréquents sur develop (par contre, reste la question de "quand tu actives la features, tu te prends plein de murs pas testés avant ?")
- **Implement comprehensive automated testing**
    > Short running unit and integration tests are executed during development and upon code merge.
    >
    > Longer running, full stack, end-to-end tests are run in later pipeline phases against a full staging or production environment.
- **Perform asynchronous code reviews** : la description est pas très en phase avec le titre, mais l'idée est de faire les revues rapidement.
- **Have three or fewer active branches in the application's code repository** : se reformule en "nettoyer ses branches une fois qu'on les a mergées sur le trunk"
- **Merge branches to the trunk at least once a day** : ne pas attendre avant de merger ses branches ; tous les soirs, on peut considérer l'état du trunk comme "la prochaine release".
- **Reduced number of code freezes and integration phases** : essayer de limiter les situations où on freeze le code (e.g. pour des besoins d'intégration avec d'autres équipes)
- **Build fast and execute immediately** :
    > In order to maintain a quick release cadence, build and test execution times should be optimized.
    >
    > CI/CD build tools should use caching layers where appropriate to avoid expensive computations for static.

Conclusion : considéré comme le standard actuel, notamment car permet des releases fréquents.

# Feature flags

https://www.atlassian.com/continuous-delivery/principles/feature-flags

> If you plan to continuously integrate features into your application during development, you may want to consider feature flags

^ outil permettant de faire du trunk-based development

## What are feature flags

> Feature flags (also commonly known as feature toggles) is a software engineering technique that turns select functionality on and off during runtime, without deploying new code.
>
> During development, software engineers wrap desired code paths in a feature flag.

^ ça va sans dire : ça doit être pensé à l'avance, au moment du dev

## Benefits of Feature Flags

- feature flags enable code to be committed and deployed to production in a dormant state and then activated later.
    - c'est le coeur du truc
- Validate feature functionality
    - principe = pouvoir "déployer" dynamiquement des trucs en prod sans avoir à réellement déployer
    - on déploie la nouvelle feature en prod
    - on l'active
    - on teste
    - si tout va bien, on laisse la feature activée
    - si ça va pas, on disable la feature
- Minimize risk
    - l'exemple qui est donné consiste à désactiver dynamiquement des features gourmandes en perfs si jamais le monitoring montre que l'app est à la ramasse (NDM : je trouve ça bancal...)
- Modify system behavior without disruptive changes
    - l'idée est qu'au lieu que tous les devs fassent leur travail dans une branche, et d'attendre que tout le travail soit fait avant d'intégrer, chacun pousse son code sur master, de façon désactivée, sans avoir besoin d'attendre le travail des autres
    - quand tout le monde a fini de bosser, alors on peut activer la feature
    - NDM : c'est l'intérêt principal à mes yeux

## Feature flag use cases

**Product testing** = l'idée est de déployer une feature encore incomplète au plus tôt à un petit nombre d'utilisateurs finaux, pour voir si ça vaut le coup d'investir dans le développement de la feature :

- si la feature est bien reçue, on augmente le nombre d'utilisateurs impactés + on finit de la développer
- si la feature est mal reçue, on l'annule et on n'en parle plus

**Conducting experiments** = faire de l'A/B testing (NDM : c'est pas très différent du point précédent, qui n'est qu'un exemple d'A/B testing)

**Migrations** = il y a des cas où on doit déployer plusieurs composants de façon coordonnée (e.g. une nouvelle DB, qui nécessite du nouveau code pour être pris en charge) ; dans ce cas, on peut déployer un code compatible avec les deux versions de la DB en prod, et n'activer que l'ancien par feature-flag, puis activer le nouveau lorsque la DB est migrée.

**Canary launches** =
    > Canary launches in software development occur when a new feature or code change is deployed to a small subset of users to monitor its behavior before releasing it to the full set of users.
    >
    > If the new feature shows any indication of errors or failure, it is automatically rolled back.

NDM : ici aussi c'est un cas particulier d'A/B testing.

**System outage** = l'idée est d'avoir un moyen simple et rapide de couper une app, via les feature-flags, plutôt qu'en intervenant en prod (NDM : ici aussi je trouve ça bancal, voire douteux...)

**Continuous deployment** :
    >  Feature flags make it safer to deploy continuously by separating code changes from revealing features to users.
    >
    >  New code can automatically merge and deploy to production and then wait behind a feature flag.

(Feature branches vs. feature flags : ça n'a rien à voir, et les feature flags sont justement un moyen d'éviter d'avoir à créer des feature-branchs qui vont vivre longtemps, et seront coûteux à maintenir à jour la branche principale.)

## How to implement feature flags

> Feature flagging has some infrastructure dependencies that need to be addressed to function properly.
>
> (...) it becomes critical to have an authoritative data store and a management mechanism for the flags.
>
> Many third-party feature flag services provide this data store dependency.
>
> (...) However, if your team has third-party security concerns, it may be in your best interest to implement your security flag backend.
