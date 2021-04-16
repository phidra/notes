# De bronze à légendaire, comment réussir vos IA de bot

- **url** = https://www.youtube.com/watch?v=8kBQMQyLHME
- **type** = vidéo
- **auteur** = [Grégory RIBÉRON](https://github.com/Manwe56/competitive-programming), dev java/C++ à Murex, qui fait beaucoup de competitive programming, en a gagné certaines.
- **date de publication** = 2017-04-17
- **source** = [la chaîne youtube de Devoxx](https://www.youtube.com/channel/UCsVPQfo5RZErDL41LoWvk0A)
- **tags** = language>none ; topic>competitive-programming

**TL;DR** : vidéo présentant les principes du competitive programming, et quelques notions générales de la programmation de bots

# Notes vrac

## principe général

- code contre les autres
- envoyer son code en un seul fichier
- communication via io
- pas d'accès à des 3rd party software
- chaque tour en 100ms
- logs pour débugger

## conseils généraux

- commencer simple, IA basique
- 5 composants dans un bot :
    - parsing
    - high level AI (décision plutôt stratégique)
    - fonction d'évaluation
    - AI algorithm
    - game rules
- tests unitaires sur le moteur du jeu et sur la fonction d'évaluation-> rentable !
- possiblement tests unitaires aussi sur AI algorithm (ou alors réutiliser du code existant, déjà testé)
- pour le parsing et l'AI high-level, utiliser plutôt des logs
- system tests sur les features clés (je comprends pas bien son system test, l'idée est de mettre un état du jeu dans une string pour y accéder ?)
- ne pas hésiter à tester le moteur du jeu, pour bien comprendre comment fonctionne le serveur sur lequel s'exécute le jeu (p.ex. il avait pas les mêmes coordonnées que le serveur après une collision)
- regarder son bot jouer
    - bugs
    - voler les bonnes idées :-)
- gestion du temps sur 10 jours
    - premières heures : bootstrapper
    - puis : code, code, code
    - dernières heures : regarder le bot jouer
- regarder les post-mortems des top-players + googler (stratégies)
- gestion du temps :
    - prioriser les features : ROI = s'intéresser d'abord à une feature qui rapporte gros !
    - timeboxer 

## types de jeux et techniques

Les caractéristiques des jeux : branching-factor + actions-per-turn

### game tree

- tree traversal :
    - fixed depth : minimax + maxntree
    - promising first : monte-carlo + tree search
    - attribuer à l'adversaire sa meilleure réponse possible
    - élagage = pruning : alpha-béta / ...
- monte-carlo : attention à l'OOM
- équilibre profondeur vs. fonction d'évaluation

### resource management

- algos de diffusion
- point intéressant = pré-calculer des données, puis s'en servir de différentes façons en fonction des décisions à prendre

### simulation

- bad random / good random
- algos génétiques

Note : en terme d'AI, le machine-learning peu efficace car difficile à entraîner ! En revanche, stratégies génétiques semblent efficaces.

## Vrac

Les gagnants ont des profils très variés (AI, student, music teacher !)

Best language ? C++ très souvent devant car optimisé.

Le [github de Grégory RIBÉRON](https://github.com/manwe56) est bien fourni, et notamment, sur [son repo dédié au competitive programming](https://github.com/Manwe56/competitive-programming), il y a ses algos, et un builder pour combiner les classes.

Les top players partagent leur stratégie et font vivre la communauté, ce qui est top.
