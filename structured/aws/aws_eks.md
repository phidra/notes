- [Utilisation](#utilisation)
- [Installation de kubectl](#installation-de-kubectl)
- [Configuration de kubectl vers un cluster AWS EKS](#configuration-de-kubectl-vers-un-cluster-aws-eks)


# Utilisation

Lister les pods et leurs infos :

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

Pour toutes les commandes suivantes, on a besoin de l'id du pod. Le plus simple est de faire une envvar :

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
