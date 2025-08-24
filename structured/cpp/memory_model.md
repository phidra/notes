[Cette page](https://doc.rust-lang.org/nomicon/atomics.html) du rustonomicon est une bonne explication du memory model du C++.

Notamment, le code qu'il y a entre **acquire** et **release** (qui fonctionnent ensemble, au passage) fonctionne comme une _section critique_ = l'acquisition d'un genre de "lock", garantissant que ce qui est dans la section critique sera vu dans le même ordre par les deux threads, celui qui écrit et celui qui lit.

Autre info intéressante : il faut tenir compte du hardware !

Notamment, x86 est strongly-orderered par défaut, là où arm est weakly-orderered : sur x86, on n'a quasiment jamais d'intérêt à utiliser relaxed au lieu de acquire/released ; sur arm, si !

