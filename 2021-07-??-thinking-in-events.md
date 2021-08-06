# Thinking in Events: From Databases to Distributed Collaboration Software (ACM DEBS 2021)

- **url** = https://www.youtube.com/watch?v=72W_VvFRqc0
- **type** = vidéo
- **auteur** = [Martin KLEPPMANN](https://martin.kleppmann.com/), dev assez généraliste qui fait plein de trucs intéressants
- **date de publication** = 2021-07-01
- **source** = [la chaîne youtube de Martin KLEPPMAN](https://www.youtube.com/channel/UClB4KPy5LkJj1t3SgYVtMOQ)
- **tags** = language>agnostic ; topic>event-based systems ; level>intermediate

**TL;DR** : vidéo "guided-tour" des event-based systems

# Notes vrac

- 0:30 catégoriser (taxonomy) les event-based systems. Event = notification, persistent record, or both
    1. Notif = callback (e.g. button, non blocking IO, fp, dataflow programming). Ephemeral, in memory (not on disk)
    2. Persistent record (e.g. timeseries). Ils ne triggent rien : ils sont juste DISPONIBLES pour querying et analyse.
    3. 07:00 Both = "stream processing". E.g. message brokers, database stream queries.  Typically, not long-time storage.
- 0:00 distinction= windowing : Grouper les évents (e.g. par jour de la semaine, ou bien sliding Windows)  ou non
- 4:00 twitter clone exemple.
    * Objectif = afficher les tweets des comptes qu'un user follow
    * Implémenter une Query SQL est compliqué.
    * A contrario, avec Les évents que sont envoyer un tweet, et follow/unfollow, c'est assez simple.
- 9:00 database réplication exemple
    * Objectif : quand on écrit sur le master, ça réplique sur les Slaves
    * Réplication log : ses items peuvent être vus comme des events.
- 2:00 Primary backup réplication.
- 4:00 state machine réplication (=event sourcing) : le log n'est plus entre le master et les replicas, mais entre le user et le système
- 6:00 exemple de cas de state machine réplication (mais je ne comprends pas bien l'avantage comparatif, 29:30 dit : pour aider à comprendre comment le système est arrivé dans cet état)
- 9:50 = pros/cons des deux systèmes
- 4:00 ~ : totally ordered vs partially ordered events
    * (En ordonnant par timestamp, il y a moyen de retrouver des events totally ordered à partir d'events partially ordered)
- 40:00 Time warp
- 3:40 matrice synthétique : todo = donner les critères, et les techniques
- 4:50 crdt = conflict free replicated data types
- 0:00 récap du talk (todo : à annoter)



