**MultiTenant** = le fait d'avoir une même instance utilisée par plusieurs tenants, mais de façon isolée entre eux (= ils n'ont pas connaissance les uns des autres)

Parallèle selon moi avec serveur mutualisé vs. serveur dédié

Avantages :

- centralisation (e.g. pour mettre à jour son offre, on n'a qu'un endroit à mettre à jour = l'instance centralisée)
- facilité de déploiement pour un nouveau client (ne pas avoir N isntances à gérer)
- moins cher de déployer une seule instance et partager les ressources

Inconvénients :

- sécurité : clients moins isolés que si chacun a son instance
- partage des ressources : si un client bouffe toute la ressource, les autres en auront moins

**MultiExploitant** = architecture où plusieurs exploitants utilisent une même plateforme/infrastructure/service, tout en maintenant une certaine forme d'isolation entre leurs opérations.

Multi-exploitant = plusieurs "personnes" se partagent une même infra physique, ce qui inclut des contraintes opérationnelles/logistiques (NDM : e.g. un câble réseau sous l'océan ?)

Différence avec multi-tenant = on parle plutôt de multi-exploitant d'une infrastructure, i.e. plusieurs personnes se partagent une infrastructure (là où multi-tenant, et plusieurs personnes se partagent une instance d'une application, avec leurs données/utilisateurs propres).
