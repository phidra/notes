# "Clean" Code, Horrible Performance

- **url** = https://www.youtube.com/watch?v=tD5NrevFtbU
- **type** = vidéo
- **auteur** = [Casey MURATORI](https://caseymuratori.com/about), spécialisé dans la R&D des Game Engines
- **date de publication** = 2023-02-28
- **source** = chaîne YouTube de [MollyRocket](https://mollyrocket.com/), studio qui mélange trucs artistiques et R&D
- **tags** = language>C++ ; topic>clean-code ; topic>perfs ; level>intermediate

**TL;DR** = très intéressante prez, en gros en choisissant de ne pas respecter les pratiques de clean code, il gagne entre 15x et 24x d'accélération sur ses perfs.

Ça semble être un talk bonus à une série de cours : https://www.computerenhance.com/p/clean-code-horrible-performance

J'en pense plusieurs choses :

- les bons principes architecturaux sont à peser en face de leur coût en terme de perfs
- la plupart du temps, avoir du code testable et maintenable apporte plus de valeur qu'un code qui va vite (obviously ça dépend des métiers)
- seule une petite partie du programme (hotspots) est sujette à un mauvais tradeoff qui pencherait dans l'autre sens ; donc à la rigueur, c'est le seul endroit où ça pourrait valoir le coup de ne pas appliquer les bons principes
- (et en side-note : il applique son analyse aux conseils de clean code, que je suis loin de partager à 100% ; ce qui compte c'est surtout l'esprit de lisibilité qui va avec)

Mon avis :

- il y a une tension entre :
    1. isoler des composants afin de "raisonner localement"
    2. ne pas isoler les composants, pour autoriser les optimisations avec une vision globale
- il y a donc **un curseur à positionner** pour "choisir" le ratio entre les optimisations et des composants isolés
    - attention à ne choisir la piste "optimisation" que si on en a besoin !
- souvent, on tire beaucoup plus de bénéfices de l'approche "isoler des composants"
    - et c'est encore plus vrai avec une équipe junior qui mets des bugs partout
- éventuellement, privilégier la piste "optimisation" pour les hotspots, en connaissance de cause
- Casey muratori a un taf qui le conduit naturellement à avoir besoin de privilégier la piste optimisation, donc il fait le bon choix **pour lui**...
    - Ça ne veut pas dire que c'est le bon pour tout le monde !
    - Il **FAUT** se poser la question systématiquement (et très souvent, la réponse sera : mieux vaut payer le petit surcoût de perf, et gagner du code architecturé en composants isolés, plus testable et maintenable)

Il parle un peu plus de son point de vue ici, pendant une heure (notamment , il adopte un point de vue "tradeoff à choisir", qui me parle plus) : https://youtu.be/ffDXc6oup3Q?si=5E9flV5YulTUpZey

La quote _"premature optimization is the root of all evil"_ est mal interprétée, il FAUT penser à une archi performante dès le début (sinon, on arrivera à un point où on ne pourra pas forcément optimiser les hotspots). La quote s'appliquait à de l'optimisation manuelle en assembleur de points très spécifiques.

