# La dette technique, est-ce une fatalité ?

- **url** = [vidéo youtube](https://www.youtube.com/watch?v=C2COBA4EFrM), [compte-rendu du talk par Octo](https://blog.octo.com/la-dette-technique-est-ce-une-fatalite-compte-rendu-du-talk-de-mickael-wegerich-a-la-duck-conf-2021/)
- **type** = vidéo + post
- **auteur** = Mickaël WEGERICH, consultant OCTO [sa page medium](https://mickalwegerich.medium.com/)
- **date de publication** = 2021-03-15
- **source** = [page youtube de la DuckConf](https://www.youtube.com/channel/UCrzp80KZG1K_icINE6D36OA) = la conf octo, orientée archi

**TL;DR** : intéressante revue de diverses sources de dette technique
- le talk se concentre sciemment sur la dette technique involontaire (ce que j'apprécie particulièrement)
- il liste 9 mauvaises pratiques liées à cette dette technique involontaire :
    - hype-driven development = on prend une techno à la mode, et on la mets chez nous
    - overengeneering = on fait du code en trop "car on sait jamais"
    - on dit kiss/yagni trop vite = on fait l'impasse sur des choix jugés (à tort) overkill
    - on écrit du code en ayant assez peu de connaissances en design de code
    - le métier découle des choix techniques au niveau du code (au lieu que ce soit le contraire)
    - on mélange le métier et le code technique (alors que les deux ont des cycles de vies différents)
    - on couple trop les tests au code
    - on écrit des tests juste pour écrire des tests
    - on sépare deux équipes alors qu'elles travaillent sur le même produit (cf. loi de Conway)
- pour chacune, il revoit (assez brièvement, le talk de fait que 30 minutes) des pistes permettant de les garder sous contrôle
- autres points intéressants en vrac :
    - il insiste sur la complexité accidentelle (idem, ce que j'apprécie particulièrement)
    - conseil = designer **dès le début** une application pour pouvoir la remplacer petit bout par petit bout
    - erreur à ne pas faire n°1 = continuer comme avant en supposant qu'on arrivera mieux à éviter la dette (il faut **ajouter** quelque chose à l'équipe pour changer de paradigme)
    - erreur à ne pas faire n°2 = supposer qu'on a une bonne vision de ses lacunes (effet Dunning-Kruger = on ne sait pas ce qu'on ne sait pas, et il faut en être conscient)

## Introduction

00:30 les entreprises refont tous les deux ans leur système.

01:00 PATTERN = rattrapé par la dette technique :
- les tickets mettent de plus en plus de temps à aller en production.
- de plus en plus de bugs
- "c'est pas possible, faut tout refaire"

02:00 talk découpé en 3 parties :
- 1. on est acteur de la déterioration du code
- 2. outils à disposition
- 3. demain, retour à ma réalité

## PARTIE 1 = on est acteurs de la déterioration du code

02:30 dette technique contractée de manière consciente (on prend un raccourci qu'on remboursera par la suite) ou inconsciente, par manque de connaissance ou de compétences.  C'est sur la deuxième que le talk va se focaliser.

03:00 il va lister 9 problématiques, réparties en 4 piliers à l'origine de cette dette technique inconsciente :
- choix des technos et architecture SI
- architecture et design du code
- stratégie de tests
- organisation

03:30 **pilier n°1 = choix des technos et architecture SI**
- bad = hype-driven development = on prend une techno à la mode, et on la mets chez nous (e.g. partir sur du microservice là où 1. on n'en a pas forcément besoin et 2. on n'a pas les compétences)
- bad = overengeneering = on fait du code "car on sait jamais" : on fait trop de code
- bad = on dit kiss/yagni trop vite = on faire une impasse sur des choix jugés (à tort) overkill

05:00 **pilier n°2 = architecture et design de code**
- bad = on écrit du code en ayant au final assez peu de connaissances en design de code -> ça amène au plat de spaghetti
- bad = le métier découle des choix techniques au niveau du code (au lieu que ce soit le contraire), p.ex. on choisit la base de données, et on laisse ce choix de BDD guider comment on résout la problématique métier (au lieu de faire l'inverse)
- bad = on mélange le métier et le code technique (alors que les deux ont des cycles de vies différents). E.g. si on veut changer de BDD/de format de stockage, il faut que les règles métiers soient découplées pour rester inchangées.

06:30 **pilier n°3 = stratégie de test**
- bad = on couple trop les tests au code (1 fichier de test = 1 fichier de code -> mauvais, car dès qu'on change l'implémentation, le test passe au rouge)
- bad = on écrit des tests pour écrire des tests -> les tests traversent plusieurs couches à responsabilités différentes

08:00 **pilier n°4 = organisation**
- bad = on sépare deux équipes alors qu'elles travaillent sur le même produit (cf. loi de Conway)

L'impact de ces (mauvais) choix nous rattrapent.

La construction d'un système informatique n'est pas un sprint mais un marathon. NdM : je mets cette phrase en rapport avec le fait que l'essentiel du cycle de vie/coût d'un logiciel c'est sa maintenance.

## PARTIE 2 = outils à disposition

### constitution d'un système informatique

09:30 : complexité :
- **complexité essentielle** = celle du métier qu'on cherche à résoudre (on a peu ou pas de marge de manoeuvre dessus)
- **complexité obligatoire** = celle qui est nécessaire pour résoudre le métier (on a peu ou pas de marge de manoeuvre dessus)
- **complexité accidentelle** = le reste -> tous nos mauvais choix techniques (et nos bugs) se retrouvent là-dedans

10:30 : écrire du code, c'est facile, mais contrôler l'entropie du code c'est plus compliqué.

11:00 loi de Conway (le design technique finit par refléter la structure humaine qui prend des décisions). Au lieu d'avoir deux équipes A et B qui développent un produit P, on a un mur de communication, et deux équipes A et B qui développent DEUX produits Pa et Pb.

11:50 c'est plus simple de modifier le code que d'aller parler à une autre équipe (NdM : ne s'applique pas dans mon cas, mais je conçois que ça puisse être le cas ailleurs)

### la technique

Listing de ce qu'on aimerait faire (e.g. améliorer la testabilité de l'application).

Mais au final, on souhaite un design de code permettant de faire ça -> c'est ce que permet l'archi hexagonale.

14:00 : archi hexagonale
- au milieu, le code à valeur ajoutée
- à gauche ce qui utilise ce code (composant react, API rest, producer rabbitmq)
- à droite, composants pour délivrer le service (e.g. composant pour envoyer des mails, composant BDD, etc.)

14:30 Les tests U et d'acceptation n'ont qu'une surface à tester = celle du centre (celle qui rapporte de l'argent).

15:00 Test d'intégration (qui intègrent avec un autre système) sont testés en isolation du reste. Ici, on teste les parties de gauche et de droite, EN ISOLATION du centre.

15:45 on veut tester par transitivité (en gros, sur la partie centrale, ne pas écrire du code pour tester l'ensemble, mais juste du code pour pour tester le module de haut niveau ; on considère que les modules de bas-niveau sont testés via le module de haut-niveau)

moins de test à maintenir, plus de flexibilité dans le code

(cette partie est assez peu concrète, en même temps le talk ne fait que 30 min)

test end-to-end -> on traverse toute l'archi hexagonale (toute la surface)

À ce stade, on a traité 4 des 9 problématiques :
- bad = on couple trop les tests au code (1 fichier de test = 1 fichier de code -> mauvais, car dès qu'on change l'implémentation, le test passe au rouge)
- bad = on sépare deux équipes alors qu'elles travaillent sur le même produit (cf. loi de Conway)
- bad = on mélange le métier et le code technique (alors que les deux ont des cycles de vies différents). E.g. si on veut changer de BDD/de format de stockage, il faut que les règles métiers soient découplées pour rester inchangées.
- bad = on écrit des tests pour écrire des tests -> les tests traversent plusieurs couches à responsabilités différentes

17:00 citation d'Alberto BRANDOLINI = ce qui se retrouve en prod, c'est la mauvaise compréhension (ou dans les bons cas : la bonne compréhension) des développeurs, et non celle des experts métiers (ce que j'en comprends : un expert métier aura beau être une pointure de son domaine et le maîtriser parfaitement, si le dev qui l'implémente n'a rien pigé, ça va se voir en prod)

NdM : ce qu'il adresse via cette citation et ces outils, c'est : _bad = le métier découle des choix techniques au niveau du code (au lieu que ce soit le contraire), p.ex. on choisit la base de données, et on laisse ce choix de BDD guider comment on résout la problématique métier (au lieu de faire l'inverse)_

Pour limiter cet impact, il propose deux outils (sans les détailler) :
- Event storming
- DDD = Domain Driven design

19:00 Ce qui reste à adresser parmi ses 9 points :
- bad = hype-driven development = on prend une techno à la mode, et on la mets chez nous (e.g. partir sur du microservice là où 1. on n'en a pas forcément besoin et 2. on n'a pas les compétences)
- bad = overengeneering = on fait du code "car on sait jamais" : on fait trop de code
- bad = on dit kiss/yagni trop vite = on fait l'impasse sur des choix jugés (à tort) overkill
- bad = on écrit du code en ayant au final assez peu de connaissances en design de code -> ça amène au plat de spaghetti

Ça viendra pas par magie : il faut se cultiver, lire, échanger + faire une veille constante (il liste plein de livres)

20:00 Conseil = dès qu'on conçoit une application, il faut se poser dès le début la question de "comment on va la remplacer" : **il faut DESIGNER l'application pour pouvoir la remplacer petit bout par petit bout**

21:00 side effect sympa : améliorer la qualité du système, ça va également automatiquement motiver les équipes de dev (NdM et le contraire : une mauvaise qualité démotive)

## PARTIE 3 = demain, retour à ma réalité

"on a pas le temps de faire tout ça, on est toujours sous l'eau"

22:00 Sa réponse : le temps, ça se prend.

Couper un cercle vicieux : il reste beaucoup d'US à finir en fin de sprint -> on génère de la dette à rembourser au sprint suivant, etc.

23:30 Ce cercle vicieux faire grossir petit à petit la complexité accidentelle, jusqu'à ce qu'il faille bloquer un temps conséquents (semaines, mois) pour désendetter le code.

25:00 plan d'action
- piège n°1 = réessayer ce qui n'a pas marché : son message c'est qu'il faut obligatoirement "AJOUTER" qqch à l'équipe : formation, mindset, nouvelle personne, etc. Juste se dire "je comprends mieux le métier" ne sera pas suffisant !
- piège n°2 = effet Dunning-Kruger = on ne sait pas ce qu'on ne sait pas (il faut en être conscient).

Il FAUT faire évoluer notre manière de penser et de faire.

26:30 vers où aller :
- skills techniques (craftsmanship, design, archi, ...)
- soft skills : communication, écoute, transmettre, savoir dire non, continuer à apprendre

27:00 son message = c'est plus rentable d'investir là-dedans en continu plutôt que de refaire tous les deux ans

conclusion = être rattrapé par la DT n'est pas une fatalité, mais la combattre demande un effort quotidien.
