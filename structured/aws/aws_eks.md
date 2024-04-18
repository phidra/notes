AWS EKS = Elastic Kubernetes Service : c'est un cluster kubernetes managé.

(je mets aussi ici mes notes k8s, même si en réalité ça n'est pas spécifique à AWS : ça 'applique à n'importe quel cluster k8s)

* [Utilisation](#utilisation)
   * [Lister les pods et leurs infos](#lister-les-pods-et-leurs-infos)
   * [Avoir des infos sur un pod en particulier](#avoir-des-infos-sur-un-pod-en-particulier)
* [Installation de kubectl](#installation-de-kubectl)
* [Configuration de kubectl vers un cluster AWS EKS](#configuration-de-kubectl-vers-un-cluster-aws-eks)
* [OpenLens](#openlens)
   * [Lens vs OpenLens](#lens-vs-openlens)
   * [Installation :](#installation-)
   * [Utilisation](#utilisation-1)


# Utilisation

## Lister les pods et leurs infos

```sh
kubectl get namespaces
# NAME                                     STATUS   AGE
# myapp-staging                            Active   48d

kubectl get pods --namespace myapp-staging
# NAME                               READY   STATUS    RESTARTS   AGE
# myapp-app-7d776d5c6d-ks8dl         1/1     Running   0          72m
# myapp-canary-app-7bbd86586-mqqgh   1/1     Running   0          72m
#
# NDM : les 72 minutes correspondent au temps écoulé depuis le démarrage du pod.

kubectl get deployments --namespace myapp-staging
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# myapp-app          1/1     1            1           48d
# myapp-canary-app   1/1     1            1           48d
#
# NDM : les 48 jours correspondent au temps écoulé depuis la création de l'infra.

kubectl get services --namespace myapp-staging
# NAME               TYPE           CLUSTER-IP       EXTERNAL-IP                     PORT(S)   AGE
# myapp              ExternalName   <none>           myapp-this-that.staging.local   80/TCP    48d
# myapp-app          ClusterIP      123.45.678.901   <none>                          80/TCP    48d
# myapp-canary-app   ClusterIP      123.45.678.902   <none>                          80/TCP    48d
#
# NDM : c'est comme ceci qu'on peut connaître l'IP des pods.
```

## Avoir des infos sur un pod en particulier

Pour toutes les commandes suivantes, on a besoin de passer l'id d'un pod. Le plus simple est d'en faire une envvar :

```sh
export POD=myapp-app-7d776d5c6d-ks8dl
```

Faire un tail -f des logs d'un pod :

```sh
kubectl logs "$POD" --namespace myapp-staging --follow
```

Lancer un bash dans son container :

```sh
kubectl exec -ti "$POD" --namespace myapp-staging bash
# NOTE : on est root sur le container
```

Connaître l'image docker qui tourne sur un pod :

```sh
kubectl get pod -n myapp-staging "$POD" -o=json | jq '.status.containerStatuses'
```

Récupérer... **BEAUCOUP** d'infos sur un pod :

```sh
kubectl describe pod "$POD" --namespace myapp-staging
# image docker exécutée + ports
# variables d'environnement auxquelles le pod a accès
# requests et limits pour CPU + RAM
# IP
# config des sondes de liveness/readiness/startup
# date de démarrage / restart
```


# Installation de kubectl

```sh
sudo snap install --classic kubectl
# kubectl 1.28.7 par Canonical✓ installé

kubectl version --client
# Client Version: v1.28.7
# Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
```

# Configuration de kubectl vers un cluster AWS EKS

Préambule = pour pouvoir utiliser la CLI AWS :

```sh
aws sso login --profile my-dev-profile
aws sts get-caller-identity --profile=my-dev-profile
# ^ pour vérifier que ça fonctionne
```

Récupérer la config du cluster :

```sh
aws eks update-kubeconfig --name staging-ifro-eks-cluster --kubeconfig ~/.kube/my-eks-staging-cluster.kubeconfig --region eu-west-3 --alias my-eks-staging-cluster --profile my-dev-profile
# Added new context my-eks-staging-cluster to /home/myself/.kube/my-eks-staging-cluster.kubeconfig

# ça créée le fichier suivant :
# ~/.kube/my-eks-staging-cluster.kubeconfig
```

On peut vérifier la config en essayant de requêter le cluster EKS :

```sh
kubectl --kubeconfig=/home/myself/.kube/my-eks-staging-cluster.kubeconfig get namespaces

# passer le fichier n'est pas pratique (d'autant qu'il faut expand le ~ soi-même)...
# ceci est donc plus simple :
export KUBECONFIG=~/.kube/my-eks-staging-cluster.kubeconfig
kubectl get namespaces

# on peut aussi vérifier que le retour de cette commande a bien des infos pertinentes :
kubectl config view
```

# OpenLens

**C'est quoi ?** Une IHM pour administrer un cluster k8s.

## Lens vs OpenLens

OpenLens est passé en sources fermées pour donner naissance Lens = la version commerciale :

- OpenLens = OPEN-SOURCE : https://github.com/MuhammedKalkan/OpenLens
- Lens = CLOSED-SOURCES : https://k8slens.dev/

D'après mon collègue, on peut sans souci utiliser la dernière version à sources ouvertes.

## Installation :

Télécharger le deb depuis [la page des releases](https://github.com/MuhammedKalkan/OpenLens/releases) :

- `OpenLens-6.5.2-366.amd64.deb`
- `OpenLens-6.5.2-366.amd64.deb.sha256`

Installer le deb :

```sh
sudo apt install ./OpenLens-6.5.2-366.amd64.deb
```

## Utilisation

Il trouve tout seul les clusters à partir des fichiers de config sous `~/.kube`.

On peut "éditer" une ressource (e.g. un job) pour visualiser le yaml de sa configuration.
