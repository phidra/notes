* [Installation](#installation)
* [Configuration](#configuration)
   * [Fichiers de config](#fichiers-de-config)
   * [Execution](#execution)
   * [Scenarios](#scenarios)
   * [Exemple de fichier complet](#exemple-de-fichier-complet)

**C'est quoi ?** Un outil client pour requêter en boucle un service et mesurer son temps de réponse.

L'outil s'appelle **taurus** ([sa doc](https://gettaurus.org/docs/Index/)), mais il est désigné par la commande `bzt` (*BlaZeMeter Taurus*).


# Installation

Taurus est [installable via pip](https://gettaurus.org/docs/Installation/) ou pipx (auquel cas, c'est `bzt` qu'il faut pipx-installer et non `taurus` !).

Sinon, un Dockerfile simple permet de l'utiliser :

```
FROM blazemeter/taurus:1.16.34
COPY ./input_data.txt /input_data.txt
COPY ./my_bzt_config.yml /my_bzt_config.yml
ENTRYPOINT ["bzt", "/my_bzt_config.yml"]
```

# Configuration


## Fichiers de config

Le principe de la CLI est :

- d'avoir un ou plusieurs fichiers de config yaml (qui seront mergés en une seule config)
- d'overrider si nécessaire des clés avec l'option `-o` :

```sh
bzt \
    -o settings.env.my_param="my-value" \
    my_settings.yml \
    my_scenarios.yml \
    my_execution.yml \
    some_other_file.yml
```

On configure le comportement du client notamment avec les clés `execution` et `scenarios`.

## Execution

La clé `execution` ([doc](https://gettaurus.org/docs/ExecutionSettings/)) indique le comportement général du client. Par exemple, l'exécution suivante dure 20s en tout dont 10 secondes de montée en charge, envoie les requêtes une par une, en exécutant le scénario `my-localtest` :

```yaml
execution:
  - concurrency: 1
    ramp-up: 10s
    hold-for: 10s
    scenario: my-localtest
```

Comme on peut retarder des exécutions avec `delay`, on peut définir un scénario variable par morceaux.

## Scenarios

La clé `scenarios` indique grosso-modo les requêtes à passer au service.

Typiquement, le fichier de config indique un template de requête, et les variables sont lues dans un fichier de données :


```yaml
scenarios:
  my-localtest:
    data-sources:
      - path: /input_data.txt
        delimiter: ;
        loop: true
    requests:
      - url: "http://localhost:7777/endpoint?arg1=${arg1}&arg2=${arg2}&arg3=${arg3}"
        method: GET
        headers:
          Accept: application/json
        assertions:
          - contains: "success"
          - status_code: 200
```

Dans l'exemple ci-dessus, le fichier `/input_data.txt` pourrait contenir :

```
arg1;arg2;arg3
toto;tata;titi
yoyo;yaya;yiyi
lolo;lala;lili
```

Les requêtes qui seront passées seront :

```
http://localhost:7777/endpoint?arg1=toto&arg2=tata&arg3=titi
http://localhost:7777/endpoint?arg1=yoyo&arg2=yaya&arg3=yiyi
http://localhost:7777/endpoint?arg1=lolo&arg2=lala&arg3=lili
```

## Exemple de fichier complet


```yaml
---
execution:
  - concurrency: 1
    ramp-up: 10s
    hold-for: 10s
    scenario: my-localtest
settings:
  default-executor: gatling
  aggregator: consolidator
modules:
  gatling:
    version: 3.9.5
    java-opts: -Xms2g -Xmx5g
    properties:
      gatling.data.file.bufferSize: 512
scenarios:
  my-localtest:
    data-sources:
      - path: /input_data.txt
        delimiter: ;
        loop: true
    requests:
      - url: "http://localhost:7777/endpoint?arg1=${arg1}&arg2=${arg2}&arg3=${arg3}"
        method: GET
        headers:
          Accept: application/json
        assertions:
          - contains: "success"
          - status_code: 200
```
