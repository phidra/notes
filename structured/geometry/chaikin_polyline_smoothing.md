# Algorithme de chaikin

Sert à smoother une polyline.

**TL;DR** = étant donnée une polyline, on fonctionne par itération : à chaque itération, on "découpe" un sommet (= le point de jonction entre deux segments de la polyline) pour l'aplanir.

Le principe est bien décrit ici : http://www.idav.ucdavis.edu/education/CAGDNotes/Chaikins-Algorithm/Chaikins-Algorithm.html

Il y a un exemple interactif ici : https://observablehq.com/@pamacha/chaikins-algorithm

Il y a une implémentation dans [postgis](https://postgis.net/docs/ST_ChaikinSmoothing.html) (mais pas dans spatialite, on dirait).
