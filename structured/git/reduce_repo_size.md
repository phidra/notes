[Cette doc gitlab](https://docs.gitlab.com/ee/user/project/repository/reducing_the_repo_size_using_git.html) a d'excellentes pistes pour analyser son historique et le réécrire pour supprimer les gros trucs.

Notamment :

- le fait d'export un bundle du projet
- le fait de cloner le bundle pour l'analyser avec filter-repo, puis le corriger en local
- la façon de pusher les modifications profondes sur le repo gitlab

Ça peut s'avérer utile car [il existe un bug gitlab](https://gitlab.com/gitlab-org/gitlab/-/issues/212763) qui fait que des commits survivent dans le repo, même s'ils sont orphelins : filter-repo permet de les supprimer effectivement.
