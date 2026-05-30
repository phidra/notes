AWS EKS = Elastic Kubernetes Service : c'est un cluster kubernetes managé.

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

