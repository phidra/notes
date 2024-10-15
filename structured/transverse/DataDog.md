# Observer l'exécution d'un job de gitlab CI sur DataDog

- retrouver l'id de son pod dans les logs du pipeline
    ```
    Waiting for pod gitlab-runner/MY_SUPER_POD to be running, status is Pending
    Running on MY_SUPER_POD via ...
    ```
- sous DataDog, dans le menu à gauche cliquer sur `Infrastructure > Container Images`
- dans `Select Resources` à gauche, cliquer sur `Containers`
- dans le menu `Filter by`, entrer le nom du pod `MY_SUPER_POD`
- en haut à droite, choisir une plage temporelle où le container était actif

