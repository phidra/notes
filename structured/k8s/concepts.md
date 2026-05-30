kubernetes gère son cluster avec de nombreux fichiers yaml déclaratifs

* [Deployments](#deployments)
* [Service](#service)
* [ConfigMap](#configmap)
* [Secrets](#secrets)
* [Ingress](#ingress)
* [Namespace](#namespace)


# Deployments

`deployment.yaml` = décrit l'application à exécuter :

- nombre de pods (si conf statique, sinon, la conf dynamique est dans un fichier hpa.yaml = Horizontal Auto Scaler)
- image docker à utiliser
- ressources à attribuer (CPU/RAM)
- variables d'environnement à passer
- ports des container à exposer
- updates/rollbacks

**NOTE** : pour les services stateful (e.g. une BDD), les pods ne sont plus interchangeables, on utilise d'autres objets que `Deployment` pour gérer la persistence, comme `StatefulSet` ou `PersistentVolume`.

- avec un Deployment stateless, les pods sont interchangeables : si un pod tombe, un nouveau pod apparaît
- avec un StatefulSet stateful, si le pod `postgres-1` tombe, k8s essaie de recréer spécifiquement `postgres-1`, et non un pod anonyme équivalent

# Service

`service.yaml` = définit un point d’accès réseau stable vers un ensemble de pods, par d'autres applications dans le même cluster

- en effet, sinon, les IP des pods changeraient continuellement au gré des démarrages de pods, qui sont éphémères
- il expose un non DNS stable, un port stable, et il route le trafic vers les bons pods

# ConfigMap

`configmap.yaml` = stocke la configuration (non-sensible) séparément de l’image applicative :

- variables de config
- URLs des dépendances
- niveaux de logs
- flags de fonctionnalités

# Secrets

`secret.yaml` = stocke les données sensible

- mot de passe PostgreSQL
- token d’appel d’une API partenaire
- certificat TLS

(habituellement combiné avec un vrai SecretManager type vault)

# Ingress

`ingress.yaml` = expose des services HTTP/HTTPS vers l’extérieur du cluster.

Définit des règles de routage du genre :

> "api.example.com" doit être dirigé vers le service "my-api"

couplé à un ingress controller (type nginx ou traefik)

# Namespace

`namespace.yaml` = définit une séparation logique dans un même cluster, p.ex. pour avoir dans un même cluster :

- dev
- int
- prod

