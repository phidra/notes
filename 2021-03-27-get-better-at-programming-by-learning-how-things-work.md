# Get better at programming by learning how things work

- **url** = https://jvns.ca/blog/learn-how-things-work/
- **type** = post
- **auteur** = [Julia EVANS](https://jvns.ca/about/) aka [b0rk](https://twitter.com/b0rk), une bloggeuse dont j'apprécie beaucoup le travail
- **date de publication** = 2021-03-24
- **source** = le [blog de Julia EVANS](https://jvns.ca/)
- **tags** = language>agnostic ; essay ; topic>learning


**TL;DR** : elle présente son point de vue sur l'apprentissage, en expliquant sa façon d'apprendre. Je me retrouve beaucoup dans ce post, et notamment sur :
- le fait que comprendre comment les choses fonctionnent est primordial pour devenir un meilleur dev
- la nécessité pour les devs seniors de continuer à apprendre
- l'importance de se laisser guider par sa curiosité
- le fait de ne pas hésiter à expérimenter
- l'importance d'avoir une idée des sujets sur lesquels on a des lacunes

Le coeur de l'article = "devenir meilleur" peut aussi vouloir dire "mieux comprendre comment les choses fonctionnent".

Note annexe : en ce qui me concerne à titre personnel, la notion de "revenir aux bases" est étroitement corrélée à la notion de "toucher concrètement du doigt ce qu'un concept représente" :
- par exemple, pour comprendre le dependency-inversion principle, il ne me suffit pas de lire "sans DIP, B dépend de A, alors qu'avec DIP, B et A sont décorrélés et dépendent tous deux de InterefaceA"
- à la place, j'ai plutôt besoin de voir concrètement que si on ne respecte pas le DIP, la recompilation de A va nécessiter la recompilation de B

## Notes vrac

- ses quelques exemples illustrant ce qu'elle entend par "how things work" me parlent beaucoup (notamment ceux de system programming), extraits :
    + how the event loop works
    + how virtual memory works
    + how numbers are represented in binary
    + how numbers are represented in binary
- you can use something without understanding how it works (and that can be ok!)
    + c'est ok de ne pas tout maîtriser (de toutes façons, on ne peut pas)
- **your bugs will tell you when you need to improve your mental model**
    + elle a appris à reconnaître les signes de "il me manque des choses à comprendre"
    + selon elle, c'est l'une des caractéristiques des devs senior, de savoir reconnaître les situations où il nous manque une connaissance et qu'il faut qu'on l'acquière
- **even senior developers need to learn how their systems work**
    + même les devs seniors ne savent pas tout et continuent à apprenre (duh!)
    + c'est parfois pas facile de réaliser (et d'accepter) qu'il nous manque des connaissances profondes sur quelque chose qu'on utilise depuis 10 ans (et donc qu'on est "censés" maîtriser)
- **how I go from “I’m confused” to “ok, I get it!”**
    + elle propose un framework pour s'approprier des choses
    + prendre conscience d'un sujet pas/mal-maîtrisé (*hey, when I write await in my Javascript program, what is actually happening?*)
    + en déduire des questions factuelles (*when there’s an await and it’s waiting, how does it decide which part of my code runs next*)
    + chercher les réponses : écrire (une POC), lire un article, demander à quelqu'un
    + TESTER LA COMPRÉHENSION : en écrivant un programme qui met ça en oeuvre
    + elle suggère d'essayer de mettre en pratique sa nouvelle connaissance très tôt : écrire une nouvelle feature, ou résoudre un bug
- **just learning a few facts can help a lot**
    + t'es pas obligé de tout savoir : juste savoir les grandes lignes, et connaître les sujets que tu ne connais pas est suffisant
    + (son exemple : elle a les grandes lignes du fonctionnement des flottants, et elle sait qu'il est compliqué d'écrire des algos stables numériquement)
- **connect new facts to information you already know**
    + ne pas héssiter 1. à relier les pans de connaissance, et 2. à se laisser guider par ce qui nous intéresse nous-même
- suggestion 1 pour apprendre = poser des questions fermées (yes/no questions)
    + ça lui permet de bien exprimer son modèle mental (et donc de le vérifier)
    + ça permet d'éviter que le questionné explique quelque chose de déjà maîtrisé, ou de non-pertinent
    + selon elle, on est plus vulnérable à voir son modèle mental remis en cause, donc c'est plus difficile (NdM : je ne partage pas cette opinion : je n'ai aucune inertie à remettre mon modèle mental en question)
- suggestion 2 pour apprendre = expérimenter avec l'ordinateur :
    + en gros, il s'agit de mettre en place une poc pour vérifier la réponse à une question
    + NdM : ici aussi, je partage grandement son avis
- dernier conseil = connaître les sujets sur lequel on a des lacunes est très important
    + NdM : ici aussi je me retrouve : j'ai une liste de sujets intéressants à creuser
