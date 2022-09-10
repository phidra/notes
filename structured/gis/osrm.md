**C'est quoi ?** Un moteur de routing open-source utilisant les CH, pensé pour utiliser la donnée OSM.

# Vrac

Références :

- https://project-osrm.org/
- https://github.com/Project-OSRM/osrm-backend/wiki
- http://project-osrm.org/docs/v5.22.0/api/#general-options

osrm utilise déjà un mécanisme à base de shared memory pour séparer le démarrage du service et le chargement de la donnée : https://github.com/Project-OSRM/osrm-backend/wiki/Configuring-and-using-Shared-Memory#using-shared-memory

autre algorithme supporté par OSRM (en plus de CH) = multi-level dijkstra

- https://github.com/Project-OSRM/osrm-backend/issues/4797
- https://github.com/michaelwegner/CRP
- https://pubsonline.informs.org/doi/abs/10.1287/trsc.2014.0579

un exemple d'utilisation d'OSRM : https://github.com/Project-OSRM/osrm-backend/blob/master/example/example.cpp

C'est quoi les PhantomNodes :

> Because a route might start or anywhere on the map, a "phantom node" is now generated for each of the locations in rawRoute.rawViaNodeCoordinates. A phantom node is a point on the closest edge based node, where the route starts or ends.
