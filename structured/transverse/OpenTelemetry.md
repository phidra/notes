* [Télémétrie avec OpenTelemetry](#télémétrie-avec-opentelemetry)
  * [Observabilité et OpenTelemetry](#observabilité-et-opentelemetry)
  * [Instrumentation vs. export vs. collecte vs. visualisation](#instrumentation-vs-export-vs-collecte-vs-visualisation)
  * [Telemetry data : traces vs. spans vs. metrics](#telemetry-data---traces-vs-spans-vs-metrics)
  * [opentelemetry-cpp](#opentelemetry-cpp)
    + [Concepts](#concepts)
    + [Important : API vs. SDK](#important---api-vs-sdk)

# Télémétrie avec OpenTelemetry

**Préambule** : pour avoir une utilisation concrète d'OpenTelemetry, voir mes POCs sur l'utilisation d'opentelemetry-cpp ; l'une d'elle a un docker-compose qui envoie des telemetry data en grpc à un backend jaeger permettant de les visualiser via une IHM web.

## Observabilité et OpenTelemetry

OpenTelemetry [est un standard de fait](https://signoz.io/blog/opentelemetry-alternatives/), il y a du lourd derrière (e.g. maintenu par la [CNCF](https://fr.wikipedia.org/wiki/Cloud_Native_Computing_Foundation)), et il y a des tools dans la plupart des langages mainstream.

ChatGPT définit l'observabilité comme "la capacité à comprendre l'état interne d'un système en examinant ses sorties".

https://opentelemetry.io/docs/languages/cpp/ :

> OpenTelemetry, also known as OTel for short, is a vendor-neutral open source Observability framework for instrumenting, generating, collecting, and exporting telemetry data such as traces, metrics, logs.

^ `OTel` = OpenTelemetry (et au passage, `OTLP` = OpenTelemetry Protocol)

https://opentelemetry.io/docs/concepts/observability-primer/

> An application is properly instrumented when developers don’t need to add more instrumentation to troubleshoot an issue, because they have all of the information they need.

https://opentelemetry.io/docs/concepts/semantic-conventions/

^ le "standard" définit notamment des conventions de nommages, afin que tout le monde parle de la même chose


https://opentelemetry.io/docs/concepts/instrumentation/manual/
^ en fonction des outils et du langage, on peut instrumenter de façon automatique pour les langages qui le permettent (e.g. python qui peut monkeypatcher, je suppose) ou de façon manuelle sinon, comme ce qu'on fait en C++.

## Instrumentation vs. export vs. collecte vs. visualisation

Dans l'esprit, l'application envoie des traces — p.ex. en gRPC — à un backend (tel que [jaeger](https://www.jaegertracing.io/) ou [datadog](https://www.datadoghq.com/)), et le backend permet d'observer son application.

En pratique, on peut aussi exporter sur stdout pour le debug, cf. mes POCs.

NDM : je distingue 4 étages :

- création des telemetry data = instrumentation du code : ajouter du code pour définir des spans
- export des telemetry data = le fait que les spans soient exportées vers un collecteur
- collecte des telemetry data = un composant reçoit et agrège les traces exportées
- visualisation des telemetry data = un outil permet de visualiser les données, et de les exploiter pour en inférer de la connaissance

## Telemetry data : traces vs. spans vs. metrics

https://opentelemetry.io/docs/concepts/observability-primer/

> A span represents a unit of work or operation. It tracks specific operations that a request makes, painting a picture of what happened during the time in which that operation was executed.

^ un span représente... l'étendue de temps (le concept porte bien son nom) d'une opération, pendant lequel il peut se passer des trucs

> A distributed trace, more commonly known as a trace, records the paths taken by requests (made by an application or end-user) as they propagate through multi-service architectures, like microservice and serverless applications.
> (...)
> A trace is made of one or more spans. The first span represents the root span. Each root span represents a request from start to finish.

^ on en vient à avoir des genre de flamegraphs avec le root span à la racine, et des spans plus petits pour ce qui se passe


https://opentelemetry.io/docs/what-is-opentelemetry/

^ 3 types de telemetry data :

- **Metrics** = état de la machine : RAM, CPU, req/s, etc.
- **Traces** = enchaînement d'actions, séquence d'évènements (je reçois telle requête, j'appelle telle fonction, je passe par tel code)
- **Logs** = un poil plus flou, c'est un simple "record de quelque chose" :
    - un log est simplement "une information timestampée"
    - une trace est un log particulier, associé à un évènement utilisateur + à des infos permettant de la corréler à d'autres logs/traces.

----

Quelle différence entre trace et span ? Une trace peut regrouper plusieurs spans ; c'est plus facile de comprendre quand on réfléchit sur une stack à plusieurs composants :

- un composant `C` chapeau...
- qui requête un provider `P` externe
- et une base de données `BDD`

Avec cette stack :

- **trace** = les évènements liés à la requête utilisateur, qui concernent tous les composants
- **span** = les évènements liés au traitement d'une requête par L'UN des composants de la stack

Par exemple, si un utilisateur fait une requête pour voir la liste des produits disponibles sur un site de e-commerce :

- on aura une trace pour "requête de la liste des produits"
- cette trace contiendra 3 spans :
    - la span dans le composant chapeau `C`
    - la span pour l'appel à la `BDD` (pour récupérer la liste des produits)
    - la span pour le provider externe `P`

## opentelemetry-cpp

C'est la lib officielle pour faire de l'OpenTelemetry en C++.

Cf. mes POCs pour l'utiliser, et notamment la builder (éventuellement de façon conditionnelle, en utilisant toujours l'API, mais en n'utilisant le SDK que si un flag de compilation est présent).

### Concepts

https://opentelemetry-cpp.readthedocs.io/en/latest/sdk/GettingStarted.html

- SpanProcessor (concrètement, un `SimpleSpanProcessor`) : c'est ce qui choisit si on envoie les spans une par une, ou bien en batch
- SpanExporter (concrètement, un `OStreamSpanExporter` pour exporter sur STDOUT) : c'est ce qui décide où exporter les spans
    > An exporter is responsible for sending the telemetry data to a particular backend.
- TracerProvider (concrètement, un... `TracerProvider`) :
    > TracerProvider instance holds the SDK configurations ( Span Processors, Samplers, Resource).
    >
    > There is single global TracerProvider instance for an application, and it is created at the start of application.
- Tracer (issu de `GetTracer("basic_tracer")`)
- Span begin/end
- Event dans une span

### Important : API vs. SDK

opentelemetry-cpp permet d'utiliser [deux modules différents mais complémentaires](https://opentelemetry-cpp.readthedocs.io/en/latest/api/Overview.html), **l'API** et **le SDK** :

- **l'API** est une lib légère (header-only) qui permet d'instrumenter son code et de créer des telemetry data ; c'est par exemple elle qui permet de créer des spans :
    ```cpp
    auto my_span = tracer->StartSpan(span_name);
    // do something
    my_span->End();
    ```
- **le SDK** = est une lib qui permet d'exporter ses telemetry data ; assez lourde (et avec des dépendances configurables au build, selon de où on veut exporter ses traces)
- on peut tout à fait n'utiliser que l'API et pas le SDK ; tout appel à l'API OpenTelemetry sera alors un noop vide
- utiliser le SDK pour initialiser la télémétrie a pour effet de remplacer les noop vides de l'API par du code qui exportera réellement les telemetry data

Du coup, on peut instrumenter tout son code tout le temps, et cantonner le choix d'initialiser ou non la télémétrie (avec un flag au compile-time) dans un unique fichier cpp ; c'est ce que j'ai fait dans mes POCs.

