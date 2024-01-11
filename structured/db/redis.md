Le client `redis-cli` est installé par le package `redis-tools` (qui est installé avec le package `redis-server`).

La lib [hiredis](https://github.com/redis/hiredis), qui fait partie des clients [recommandés par redis](https://redis.io/resources/clients/), s'installe indépendamment :

```sh
sudo apt install libhiredis-dev
```

(on peut la linker statiquement pour que le binaire final n'en dépende pas dynamiquement)
