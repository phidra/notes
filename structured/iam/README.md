IAM = Identity and Access Management

Il faut distinguer :

- **Identity = authentication** = dire que je suis tel user (exemple de tool = keycloak)
- **Access Management = authorization** = sachant que je suis tel user, définir ce à quoi j'ai accès (exemple de tool = Oauth, Zanzibar, OpenFGA)

Beaucoup d'outils font les deux (AWS IAM, Azure AD, ...) ; l'acronyme IAM recouvre les DEUX volets.


## Access Management


Acronymes :

- `RBAC` = **Role-Based access control** = on associe des permissions à un rôle, puis les users peuvent avoir tels ou tels rôles, qui définissent ce qu'ils peuvent faire.
    - avantage = très répandu, simple
    - inconvénient = pas granulaire, peut nécessiter de créer des tétrachiées de rôles pour gérer les différents besoins métier.
    - encore très utilisé, mais actuellement considéré comme rendu obsolète par ABAC/ReBAC
- `ABAC` = **Attribute-based access control** = donner ou non l'accès est évalué à partir d'attributs :
    - un moteur de Policies (le PDP = policy décision point) évalue les attributs d'une requête d'un user pour savoir s'il donne l'accès ou non.
    - les attributs sont de 4 types :
        - sujet (des attributs sur qui veut accéder à la ressource)
        - action (que veut faire le sujet ?)
        - object (la ressource cible)
        - contexte (e.g. l'heure de la requête, p.ex. pour interdire les accès hors des heures de bureau)
    - Permet un contrôle plus fin des permissions.
    - Aussi connu sous le nom de `PBAC` (policy-based) ou `CBAC` (Claims-Based).
- `ReBAC` = **Relation-Based Access Control** = un user aura le droit de faire telle opération s'il existe une relation liant l'user aux opérations
    - alternative à ABAC plus lourde mais plus granulaire
    - concrètement, les ressources et les users sont les nodes d'un graphe, et les relations sont les edges
    - donc savoir si un user à le droit d'accéder à une ressource revient à un parcours de graphe pour savoir s'il existe un chemin entre les nodes
- `ACL` = **Access-Control List** = une liste associée à une ressource (chaque ressource a sa liste en métadonnée) indiquant qui a le droit de faire quoi, sous la forme de paires user+droits
    - Ma compréhension des choses, c'est que c'est une alternative au rôle based, et que ReBAC est dans la famille des ACL (mais au lieu d'associer l'ACL à la ressource, elle est positionnée dans un graphe, et sera reliée à la ressource, elle aussi dans le graphe)

Ressources vrac :

- https://en.wikipedia.org/wiki/Role-based_access_control
- https://en.wikipedia.org/wiki/Attribute-based_access_control
- https://en.wikipedia.org/wiki/Relationship-based_access_control
- https://en.wikipedia.org/wiki/Access-control_list
- https://en.wikipedia.org/wiki/Google_Zanzibar
- [talk de présentation de ReBAC](https://youtu.be/aJn0v9OR4K8?si=LEyUdAf50kuvkMNZ), très bien fait, qui montre une démo de OpenFGA ; vu le 29 juillet 2025
