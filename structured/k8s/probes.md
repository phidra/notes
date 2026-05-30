# k8s liveness readiness startup probes

TL;DR = les sondes permettent de répondre à une question différente :

- `liveness` = le container doit-il être killé ?
- `readiness` = le container doit-il être sorti du load-balancer ?

La sonde optionnelle `startup` permet d'inhiber les deux autres jusqu'à ce que le container ait fini de démarrer : k8s attendra que la sonde de `startup` ait répondu au moins une fois **OK** avant de commencer à utiliser les sondes `liveness` et `readiness`.

----

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

En gros, :

- la **liveness probe** sert à restart un container : si elle faile, l'application est considérée comme morte, k8s tue le pod et le restarte (on peut souhaiter ce comportement même si le process tourne encore ; p.ex. si notre app est bloquée dans un deadlock)
- la **readiness probe** sert à autoriser le routage de traffic vers le container, si elle faile, l'application est sortie du flux du load-balancer :
    > The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

Ces deux probes peuvent éventuellement être servies par le même endpoint HTTP :

> A common pattern for liveness probes is to use the same low-cost HTTP endpoint as for readiness probes, but with a higher failureThreshold. This ensures that the pod is observed as not-ready for some period of time before it is hard killed.

^ la sonde `readiness` est moins forte que `liveness`, de sorte qu'on essaye d'abord de sortir l'application du flux et de la laisser retrouver un état stable, avant de finalement la killer.

La **startup probe** est optionnelle, elle sert dans les cas où l'application met du temps à démarrer (e.g. elle a beaucoup de données à charger). Dans ce cas, la liveness probe ou la readiness probe peuvent ne pas être en mesure de répondre suffisamment vite avant que le container ne se fasse killer.

Dit autrement : le fonctionnement par liveness/readiness peut n'être prêt à faire son boulot que de longues minutes après le démarrage de l'application. La startup probe est là pour ça : pour désactiver le fonctionnement de liveness/readiness jusqu'à ce que le startup soit fini :

> The kubelet uses startup probes to know when a container application has started. If such a probe is configured, liveness and readiness probes do not start until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running.

### Quel intérêt de la sonde de startup ?


Prenons l'exemple d'une app qui doit charger des données en RAM pendant cinq minutes au démarrage. Pourquoi l'app ne choisirait pas de répondre `liveness=true + readiness=false` pendant toute la durée de son chargement ?

Il y a deux raisons pour laquelle ça ne marchera pas forcément : d'abord, toutes les apps n'ont pas le loisir de pouvoir répondre aux sondes _en même temps_ qu'elles effectuent le chargement nécessaire au démarrage ; dans ce cas, les sondes ne répondront pas tant que les données ne seront pas chargées, elles seront donc considérées comme en échec pendant les 5 premières minutes : possiblement, c'est suffisant pour que k8s considère l'application comme morte, et la kille.

k8s permet au devops de configurer un temps d'attente initial avant de commencer à prober `liveness+readiness`, ce qui nous amène à la deuxième raison : si le temps de chargement est variable, p.ex. entre 30 secondes et 5 minutes, comment le devops doit-il configurer le délai initial ?

- s'il configure à 30 secondes, l'app sera parfois killée avant même d'avoir fini de démarrer
- s'il configure à 5 minutes, l'app sera parfois laissée vivante alors même qu'elle a démarré depuis 30 secondes, et est en deadlock sur les 3 dernières minutes

Conclusion = pas facile de prendre en compte le temps de démarrage de façon externe... Ce qu'on voudrait, c'est configurer le délai initial à "tout pile le temps nécessaire au démarrage", i.e. commencer les liveness probes uniquement après le démarrage. C'est exactement à ça que sert la startup probe : tant qu'elle n'a pas répondu **OK** au moins une fois, les liveness+readiness probes ne démarrent pas.


