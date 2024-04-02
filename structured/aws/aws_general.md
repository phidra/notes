AWS, c'est une collection de plein plein (>200) de services différents.

Quelques uns des services pertinents :

- **S3** (= Simple Storage Service) = file storage
- **EC2** = équivalent de ce que j'avais chez gandi = un serveur privé virtualisé (= IAAS)
- **Elastic Beanstalk** = wrapper autour d'autres services AWS pour avoir du PAAS (e.g. pour avoir un serveur python ou php)
- **Elastic Load Balancing** = load balancing
- **RDS** (= Relational Database Service) = plein de bases de données relationnelles
- **dynamoDB** = base de données NoSQL
- **OpenSearch** = équivalent AWS "maison" de elastic search
- **Aurora** = équivalent AWS "maison" de MySQL / PostgreSQL
- **DocumentDB** = équivalent AWS de MongoDB
- **ElasticCache** = équivalent AWS de redis et memcached
- **EKS** = Elastic Kubernetes Service = cluster k8s managé
- **ECS** = Elastic Container Service = hébergement et orchestration de conteneurs
- **Lambda** = fonctions serverless
- **Amazon Elastic Container Registry** = registry docker
- **SNS** = Simple Notification Service = envoi de notifs (SMS, mails, etc.)
- **SQS** = Simple Queue Service = message broker
- **MSK** = kafka managé
- **MQ** = RabbitMQ (ActiveMQ) managé
- **CloudFront** = CDN
- **IAM** = Identity and Access Management = gestion des identités et accès

Il y a une notion de "service managé", où l'utilisateur du servic;e (moi) ne gère ni le déploiement ni la maintenance et les mises à jour de sécurité, c'est AWS qui fait le taf.

Les différents services AWS sont très bien intégrés entre eux.

