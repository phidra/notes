#  What is API Gateway?

- **url** = https://www.youtube.com/watch?v=6ULyxuHKxg8
- **type** = vidéo
- **auteur** = [ByteByteGo](https://www.youtube.com/@ByteByteGo) = chaîne youtube sur le system-design, dont les vidéos sont concises et claires
- **date de publication** = 2022-11-01
- **source** = [la chaîne youtube ByteByteGo](https://www.youtube.com/@ByteByteGo)
- **tags** = language>agnostic ; topic>api-manager ; topic>system-design ; level>beginner

TL;DR : API Gateway = couche intermédiaire entre un client et une collection de backends qui servent l'application : au lieu de requêter directement les backends, le client requête l'API Gateway.

J'ai eu envie de prendre ces notes surtout pour le listing clair des features que permet un API gateway :

- authentication + security policy enforcement (e.g. IP blacklist)
- load balancing + circuit breaking
- protocol translation
- service discovery
- monitoring + logging + analytics + billing
- caching
- rate limiting
- parameter validation
