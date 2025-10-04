Source = [C'est quoi Traefik ?](https://wiki.picasoft.net/doku.php?id=technique:tech_team:traefik)

**[Traefik](https://traefik.io/traefik) est un reverse proxy** = un frontal qui reçoit les requêtes , et les redistribue au serveur.

Notamment, si le serveur héberge plusieurs services, qui varient selon l'URL : `service1.example.com` et `service2.example.com` sont servis par le même serveur.

Il semble plutôt adapté au workflow où les services en backend tournent dans des containers docker.

Avantages :

- permet de router les requêtes vers un container approprié en backend
- permet de "rajouter" du https sur un service qui ne le fait pas (ou vu autrement : permet de centraliser la gestion du https)
- plus généralement, le reverse proxy peut jouer le rôle de middleware de sécurité
- NDM : il faut bien un point d'entrée unique pour les requêtes sur cet host !
- NDM : en tant que point d'entrée unique, il pourra load-balancer

Typiquement, ngninx peut être utilisé en reverse proxy, mais ça nécessite de le configurer statiquement (i.e. avoir une règle statique associant chaque URL au container qui soit la servir).

À l'inverse, traefik gère ça dynamiquement en analysant les labels docker.

Derrière, il route les requêtes vers les containers docker en http et non https (donc attention : traefik lui-même doit être bien sécurisé, car ce qui "franchit" traefik sera moins sécurisé)

