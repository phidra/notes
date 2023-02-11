# gitlab-ci

Notes de haut-niveau sur les concepts gitlab-ci.

* [gitlab-ci](#gitlab-ci)
   * [fichier .gitlab-ci.yml](#fichier-gitlab-ciyml)
   * [concepts](#concepts)
   * [stages](#stages)
   * [paramétrer le pipeline](#paramétrer-le-pipeline)
   * [persistence](#persistence)
   * [GUI](#gui)
   * [HOWTO mutualiser des trucs entre jobs](#howto-mutualiser-des-trucs-entre-jobs)

[Cette doc](https://docs.gitlab.com/ee/ci/quick_start/) permet de comprendre les concepts.

## fichier `.gitlab-ci.yml`

Tout est paramétré par le fichier `.gitlab-ci.yml` à la racine du projet, e.g. :

```yaml
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"

test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20

deploy-prod:
  stage: deploy
  script:
    - echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
  environment: production
```

## concepts

**TL:DR** = chaque entrée dans le yaml définit un **job** ; un job exécute le contenu de son `script` ; l'ensemble des jobs constitue le **pipeline**.

En première approche, chaque entrée racine dans le yaml définit un job, avec son nom.

Exception : certaines entrée racine ont une signification spéciale (et ne définissent pas un job) ; il y a une pléthode de keywords, comme p.ex. `default` ([lien](https://docs.gitlab.com/ee/ci/yaml/#default)).

`script` correspond aux commandes constituant le job.

L'ensemble des jobs définit le pipeline (du coup, un fichier `.gitlab-ci.yml` définit un pipeline).

## stages

Les stages sont des moyens de grouper certains jobs d'un pipeline entre eux :

- les jobs d'un même stage peuvent être lancés en parallèle
- on ne passe au stage suivant que si le stage précédent est complètement fini, et en succès (on peut changer ce modèle avec `needs`)
- on peut customiser les stages souhaités, mais par défaut, [il y a 3 stages](https://docs.gitlab.com/ee/ci/yaml/index.html#stages) (en laissant de côté les deux stages spéciaux `.pre` et `.post`) :
    - build
    - test
    - deploy

## paramétrer le pipeline

On peut utiliser certaines [variables prédéfinies](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html), comme p.ex. `CI_PROJECT_DIR = The full path the repository is cloned to, and where the job runs from.`.

L'exécution du pipeline elle-même peut être paramétrée (e.g. pour faire des trucs différents en prod, ou bien selon l'évènement qui a déclenché le pipeline), cf. le keyword `rules`, ou éventuellement `only + except`, même s'ils sont dépréciés en faveur de `rules`.

## persistence

On peut persister des trucs d'un job à l'autre :

- `cache` pour partager des entrées, i.e. des trucs dont le job dépend
- `artifacts` pour persister des sorties, i.e. un résultat du job

## GUI

La GUI gitlab donne accès aux pipelines, et aux jobs qui le constituent et leurs logs.

Il y a un éditeur GUI pour ne pas taper le yaml à la main (dont l'intérêt par rapport à l'édition manuel de `.gitlab-ci.yml` semble être d'aider à la visualisation/validation du pipeline).

## HOWTO mutualiser des trucs entre jobs

L'entrée racine `default` est particulière : ce n'est pas un job, mais plutôt un fourre-tout qui s'applique à tous les jobs, et dans lequel on peut donc mutualiser ce qui sera commun, e.g. `before_script`.

Un moyen de réutiliser une même commande dans plusieurs jobs = [les anchors yaml](https://docs.gitlab.com/ee/ci/yaml/yaml_optimization.html#yaml-anchors-for-scripts)
