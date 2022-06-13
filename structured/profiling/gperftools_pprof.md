* [gperftools / pprof c'est quoi](#gperftools--pprof-cest-quoi)
* [gperftools : comment générer le profil d'un programme](#gperftools--comment-générer-le-profil-dun-programme)
* [pprof : comment visualiser le profil généré](#pprof--comment-visualiser-le-profil-généré)

# gperftools / pprof c'est quoi

`gperftools` = outils de profiling de google, dont un outil pour profiler le CPU :

- https://gperftools.github.io/gperftools/cpuprofile.html
- https://github.com/gperftools/gperftools/wiki

`pprof` = outil de visualisation de profil de google, avec une UI web bien foutue :

- https://github.com/google/pprof
- super top : on peut switcher entre graphe, flamegraph, records vs. temps, zoomer sur des endroits, regarder le source, l'assembleur...

# gperftools : comment générer le profil d'un programme

Installation :

```sh
sudo apt install libgoogle-perftools-dev
```


Configurer pour compiler en mode release mais avec les infos de debug :

```sh
set(CMAKE_BUILD_TYPE RelWithDebInfo)
```


Possibilité 1 = en linkant avec libprofiler.so :

- note : pour faire marcher, il faut ajouter ces deux options DANS CET ORDRE :
    ```
    -Wl,--no-as-needed -lprofiler
    ```
- exemple avec cmake :
    ```
    target_link_options(${PP_BIN} PUBLIC "-Wl,--no-as-needed" PUBLIC "-lprofiler")
    ```
- lancement :
    ```
    rm -rf out profile.proto ; CPUPROFILE=./profile.proto CPUPROFILE_FREQUENCY=100 ./mysuperbinary
    ```

Possibilité 2 = avec `LD_PRELOAD` :

```sh
rm -rf out profile.proto ; LD_PRELOAD=/usr/lib/libprofiler.so CPUPROFILE=./profile.proto ./mysuperbinary
```

# pprof : comment visualiser le profil généré

```sh
sudo apt install golang-go

# dans le .zshrc :
export GOPATH="/home/pdrabczuk/.local/go"
export PATH="${PATH}:${GOPATH}/bin"

go get -u github.com/google/pprof
pprof -http="localhost:8888" ./graph-preprocess profile.proto
```
