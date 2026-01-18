**CONTEXTE** = janvier 2026, j'essaye PlantUML (attention, la doc est blindée de pubs...)


# Installation :

En tant que service web, via docker :

```
docker pull plantuml/plantuml-server:jetty
docker run -p 8080:8080 plantuml/plantuml-server:jetty

# http://localhost:8080/
```

# Notes à l'usage

- extension = `my_file.puml`
- gitlab peut rendre directement du puml inliné via , via  :
    ```
    \`\`\`plantuml
    ```
- la liste des couleurs utilisables est [ici](https://plantuml.com/fr/color)


# Exemple :

```plantuml
@startuml
top to bottom direction

frame "Titre de mon diagramme" {
    cloud "Service CLOUD" #LightGreen {
        component "webhook" as CLOUD_SERVICE
    }

    package "Contexte 1" as CONTEXT_1 #LightBlue {
        actor "Humain n°1" as HUMAN_1
        component "IHM lourde" as IHM
    }

    package "Contexte 2" #LightBlue {
        actor "Humain n°2" as HUMAN_2
        component "MobileApp" as MOBILEAPP
    }

    node "Services BACKEND" as SERVICES_BACKEND {
        component "Aggregator" as AGGREGATOR #DarkGrey
        component "Receiver" as RECEIVER #DarkGrey
        component "Service X" as SERVICE_X #DarkGrey
        component "Da Server" as DA_SERVER #DarkGrey

        package "Réseau interne\n(non exposé)" as PRIVATE_NETWORK #Ivory {
            component "Service Y" as SERVICE_Y
            component "Service Z" as SERVICE_Z
            database "Data" as DB
                note bottom of DB
                On peut positionner des notes
                end note
            database "AnotherData" as DB2
        }
    }

    ' ceci est un commentaire
    HUMAN_2 --> MOBILEAPP
    HUMAN_1 --> IHM
    CLOUD_SERVICE --> RECEIVER

    ' ceci est un autre commentaire
    IHM --> SERVICE_X : label XXX
    SERVICE_X --> MOBILEAPP : label YYY
    MOBILEAPP --> AGGREGATOR
    AGGREGATOR --> SERVICE_Y
    AGGREGATOR --> SERVICE_Z
    RECEIVER --> SERVICE_Y

    SERVICE_Z --> DA_SERVER : label AAA
    SERVICE_Y --> DB
    SERVICE_Z --> DB2 : label BBB

    ' pour "forcer" le positionnement, on peut insérer des liens cachés :
    DB2 -[hidden]-> DA_SERVER
    SERVICE_X -[hidden]-> AGGREGATOR
}
@enduml
```


