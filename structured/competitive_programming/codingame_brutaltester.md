**Contexte** : janvier 2024, après ma participation au [CodinGame Fall Challenge 2023](https://www.codingame.com/multiplayer/bot-programming/seabed-security) (le challenge avec les drônes et les fishes), j'essaye d'utiliser [cg-brutaltester](https://github.com/dreignier/cg-brutaltester) pour pouvoir faire jouer mes AI en local contre elles-mêmes.

* [TL;DR](#tldr)
* [Installation de java](#installation-de-java)
   * [Installation d'une version récente de java](#installation-dune-version-récente-de-java)
   * [Installation d'une version ancienne java 8](#installation-dune-version-ancienne-java-8)
* [Historique du debugging](#historique-du-debugging)
   * [Analyse des modifs à apporter au referee pour l'utiliser avec brutaltester](#analyse-des-modifs-à-apporter-au-referee-pour-lutiliser-avec-brutaltester)
   * [Reflection et version de java](#reflection-et-version-de-java)
   * [Manifest manquant](#manifest-manquant)


# TL;DR

**Préambule** : à date de rédaction, les referee codingame ne tournent que sur une version ancienne de java = java 8.

- on travaille dans un répertoire dédié :
    ```sh
    mkdir ~/temporaire/CODINGAME
    cd ~/temporaire/CODINGAME
    ```
- compiler le brutaltester :
    ```sh
    git clone https://github.com/dreignier/cg-brutaltester.git
    cd cg-brutaltester
    mvn package
    # copie de l'exécutable :
    cp target/cg-brutaltester-1.0.0-SNAPSHOT.jar /tmp/cg-brutaltester.jar
    # on vérifie qu'il est bien exécutable :
    java -jar /tmp/cg-brutaltester.jar -h
    ```
- modifier et compiler le referee :
    ```sh
    git clone https://github.com/CodinGame/FallChallenge2023-SeabedSecurity.git
    cd FallChallenge2023-SeabedSecurity
    # FAIRE LES MODIFICATIONS POUR RENDRE LE REFEREE COMPATIBLE AVEC BRUTALTESTER
    mvn package
    cp target/fall-2023-fish-1.0-SNAPSHOT.jar /tmp/referee.jar
    ```
- compilation de mon AI :
    ```sh
    cd /path/to/codingame-fall-challenge-2023
    cargo build
    cp target/debug/codingame /tmp/myai
    ```
- lancement du brutaltester :
    ```sh
    mkdir /tmp/logs/
    java -jar /tmp/cg-brutaltester.jar -r "java -jar /tmp/referee.jar" -p1 "/tmp/myai" -p2 "/tmp/myai" -t 2 -n 2 -l "/tmp/logs/"
    ```

NOTE : même lorsqu'un challenge est terminé, on peut encore y participer en retrouvant le puzzle [dans l'onglet MULTI > COMBATS DE BOTS](https://www.codingame.com/multiplayer/bot-programming).

C'est cool car je peux retrouver les IA que j'avais soumises pour le challenge, ce qui me permet de tester brutaltester :

- [fall 2023 = drônes et fishes](https://www.codingame.com/multiplayer/bot-programming/seabed-security)
- [spring 2022 = araignées et chevaliers](https://www.codingame.com/multiplayer/bot-programming/spring-challenge-2022)
- [spring 2021 = les arbres sur la grille hexagonale](https://www.codingame.com/multiplayer/bot-programming/spring-challenge-2021)

(et pour une raison que je ne m'explique pas vraiment, mon classement s'est amélioré tout seul, p.ex. pour fall 2023, je suis maintenant 17e de la gold league, soit 87e au total, alors que j'étais 126e au total lors de la cloture)

# Installation de java

Note : je fais tout ceci sur le PC fixe en ubuntu 22.04 (docs ubuntu : [java](https://doc.ubuntu-fr.org/java), [openjdk](https://doc.ubuntu-fr.org/openjdk), [maven](https://doc.ubuntu-fr.org/maven)).

## Installation d'une version récente de java

```sh
sudo apt install default-jre  # (mais en fait déjà installé)
sudo apt install maven
java --version
# openjdk 11.0.21 2023-10-17
# OpenJDK Runtime Environment (build 11.0.21+9-post-Ubuntu-0ubuntu122.04)
# OpenJDK 64-Bit Server VM (build 11.0.21+9-post-Ubuntu-0ubuntu122.04, mixed mode, sharing)
```

^ NDM : la version de java installée par apt est java 11

## Installation d'une version ancienne java 8

Pourquoi ? Car les referee codingcame ne sont pas compatibles avec java >= 9.

- Je vérifie qu'avant d'installer une version plus ancienne, java 11 est la seule alternative disponible sur mon poste en Ubuntu 22.04 :
    ```sh
    sudo update-alternatives --config java
    # Il n'existe qu'une « alternative » dans le groupe de liens java (qui fournit /usr/bin/java) : /usr/lib/jvm/java-11-openjdk-amd64/bin/java
    ```
- J'installe une version ancienne avec le PPA :
    ```sh
    sudo add-apt-repository ppa:openjdk-r/ppa
    sudo apt update
    sudo apt install openjdk-8-jdk
    ```
- Je configure les alternatives pour utiliser java 8 :
    ```sh
    # AVANT :
    java -version
    # openjdk version "11.0.21" 2023-10-17
    # ^ ça n'a pas changé

    # CHANGEMENT :
    sudo update-alternatives --config java
    # Il existe 2 choix pour l'alternative java (qui fournit /usr/bin/java).
    #   Sélection   Chemin                                          Priorité  État
    # ------------------------------------------------------------
    # * 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      mode automatique
    #   1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      mode manuel
    #   2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      mode manuel
    #   ----------------------------------------
    # Appuyez sur <Entrée> pour conserver la valeur par défaut[*] ou choisissez le numéro sélectionné :2
    # update-alternatives: utilisation de « /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java » pour fournir « /usr/bin/java » (java) en mode manuel

    # APRES :
    java -version
    # openjdk version "1.8.0_392"
    ```

# Historique du debugging

## Analyse des modifs à apporter au referee pour l'utiliser avec brutaltester

Pour savoir quelles modifications apporter au referee, j'ai regardé les modifications déjà apportées pour un challenge précédent = [celui du spring 2022](https://github.com/CodinGame/SpringChallenge2022/compare/main...johnpage-agixis:SpringChallenge2022:main) :

```sh
git clone https://github.com/johnpage-agixis/SpringChallenge2022.git
git diff e89e2ed8f24128856643e1ad785103a120534c18.. --name-status
# M   .gitignore
# M   README.md
# M   config/statement_en.html.tpl
# M   config/statement_fr.html.tpl
# M   pom.xml
# A   src/main/java/com/codingame/gameengine/runner/CommandLineInterface.java
# A   src/main/resources/view/graphics/Deserializer.js
# A   src/main/resources/view/graphics/MessageBoxes.js
# A   src/main/resources/view/graphics/ViewModule.js
# A   src/main/resources/view/graphics/assetConstants.js
# M   src/main/resources/view/package-lock.json
# A   starterAIs/starter.php
```

Les deux seules modifs que j'ai eu besoin de répercuter pour faire un referee pour Fall 2023 sont :

- ajout de la CLI qui contient le main :
    ```
    src/main/java/com/codingame/gameengine/runner/CommandLineInterface.java
    ```
- modification du `pom.xml` pour ajouter la dépendane à la CLI + faire en sorte que le jar soit exécutable et démarre la CLI

Par ailleurs, il y a eu quelques autres modifs :

- ajout de `src/main/resources/view/graphics` à la gestion de conf + tous les fichiers déjà faits
- dans du HTML, remplacement de trucs servis depuis un serveur codingame par des trucs servis par le serveur local :
    ```
    --- <img src="https://static.codingame.com/servlet/fileservlet?id=20669992024930" />
    +++ <img src="/servlet/fileservlet?id=20669992024930" />
    ```
- modification du packagelock pour le serveur local

## Reflection et version de java

C'est le point qui m'a pris le plus de temps avant que je ne comprenne qu'il faille utiliser une version de java antérieure à la 9.

Quand je lançais le brutaltester, j'avais l'erreur suivante :

```
java -jar /tmp/cg-brutaltester.jar -r "java -jar /tmp/referee.jar" -p1 "/tmp/myai" -p2 "/tmp/myai" -t 2 -n 2 -l "/tmp/logs/"

10:16:54,901 INFO  [com.magusgeek.brutaltester.Main] Referee command line: java -jar /tmp/referee.jar
10:16:54,902 INFO  [com.magusgeek.brutaltester.Main] Player 1 command line: /tmp/myai
10:16:54,902 INFO  [com.magusgeek.brutaltester.Main] Player 2 command line: /tmp/myai
10:16:54,902 INFO  [com.magusgeek.brutaltester.Main] Number of games to play: 2
10:16:54,902 INFO  [com.magusgeek.brutaltester.Main] Number of threads to spawn: 2
10:16:57,577 ERROR [com.magusgeek.brutaltester.GameThread] Problem with referee output in game2. Output content:WARNING: sun.reflect.Reflection.getCallerClass is not supported. This will impact performance.
10:16:57,578 ERROR [com.magusgeek.brutaltester.GameThread] Error during game 2
10:16:57,578 ERROR [com.magusgeek.brutaltester.GameThread] WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.inject.internal.cglib.core.$ReflectUtils$2 (file:/tmp/referee.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of com.google.inject.internal.cglib.core.$ReflectUtils$2
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
10:16:57,578 ERROR [com.magusgeek.brutaltester.GameThread] If you want to replay and see this game, use the following command line:
10:16:57,578 ERROR [com.magusgeek.brutaltester.GameThread] java -jar /tmp/referee.jar -p1 /tmp/myai -p2 /tmp/myai -l /tmp/logs/game2.json -s -d
seed=1706519815594

[...]

10:16:57,868 INFO  [com.magusgeek.brutaltester.Main] *** End of games ***
+----------+----------+----------+
| Results  | Player 1 | Player 2 |
+----------+----------+----------+
| Player 1 |          | 50,00%   |
+----------+----------+----------+
| Player 2 | 50,00%   |          |
+----------+----------+----------+
```

À d'autres moments, j'avais aussi une erreur sur le logger, mais qui n'était qu'une distraction : la vraie erreur est bien l'erreur de reflection :

```
22:04:19,433 ERROR [com.magusgeek.brutaltester.GameThread] Problem with referee output in game2. Output content:WARNING: sun.reflect.Reflection.getCallerClass is not supported. This will impact performance.
22:04:19,434 ERROR [com.magusgeek.brutaltester.GameThread] Error during game 2
22:04:19,434 ERROR [com.magusgeek.brutaltester.GameThread] ERROR StatusLogger No log4j2 configuration file found. Using default configuration: logging only errors to the console. Set system property 'log4j2.debug' to show Log4j2 internal initialization logging.
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by com.google.inject.internal.cglib.core.$ReflectUtils$2 (file:/tmp/referee.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of com.google.inject.internal.cglib.core.$ReflectUtils$2
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

REPRISE ICI : continuer à formatter les notes.
REPRISE ICI : continuer à formatter les notes.
REPRISE ICI : continuer à formatter les notes.



Je me renseigne et tombe sur divers posts : par exemple [celui-ci](https://forum.codingame.com/t/fall-challenge-2023-bugs-and-questions/202669/19) qui montre que le tool est exploitable pour le fall challenge 2023, ou [celui-là](https://www.codingame.com/forum/t/cg-brutaltester-because-you-love-to-be-brutal/2716/38).



J'essaye des trucs sans succès, notamment `--illegal-access=permit` et `--add-opens java.base/java.lang=ALL-UNNAMED` :

```
java  -jar /tmp/cg-brutaltester.jar -r "java -jar /tmp/referee.jar" -p1 "/tmp/myai" -p2 "/tmp/myai" -t 2 -n 2 -l "/tmp/logs/"
```

In fine, [ce lien](https://www.baeldung.com/java-illegal-reflective-access) m'aide à comprendre qu'il s'agit de l'utilisation d'une API qui a été dépréciée en java 9.

La solution est donc d'installer et utiliser java 8 sur mon poste, ce que j'ai fait.


## Manifest manquant

Une autre erreur à laquelle j'ai été confrontée :

```
java -jar /tmp/cg-brutaltester.jar -r "java -jar /tmp/referee.jar" -p1 "/tmp/myai" -p2 "/tmp/myai" -t 2 -n 2 -l "/tmp/logs/"

10:09:28,318 INFO  [com.magusgeek.brutaltester.Main] Referee command line: java -jar /tmp/referee.jar
10:09:28,319 INFO  [com.magusgeek.brutaltester.Main] Player 1 command line: /tmp/myai
10:09:28,319 INFO  [com.magusgeek.brutaltester.Main] Player 2 command line: /tmp/myai
10:09:28,319 INFO  [com.magusgeek.brutaltester.Main] Number of games to play: 2
10:09:28,320 INFO  [com.magusgeek.brutaltester.Main] Number of threads to spawn: 2
10:09:28,404 ERROR [com.magusgeek.brutaltester.GameThread] Error during game 1
10:09:28,404 ERROR [com.magusgeek.brutaltester.GameThread] Error during game 2
10:09:28,405 ERROR [com.magusgeek.brutaltester.GameThread] aucun attribut manifest principal dans /tmp/referee.jar
10:09:28,405 ERROR [com.magusgeek.brutaltester.GameThread] aucun attribut manifest principal dans /tmp/referee.jar
10:09:28,405 ERROR [com.magusgeek.brutaltester.GameThread] If you want to replay and see this game, use the following command line:
10:09:28,405 ERROR [com.magusgeek.brutaltester.GameThread] java -jar /tmp/referee.jar -p1 /tmp/myai -p2 /tmp/myai -l /tmp/logs/game2.json -s
10:09:28,406 ERROR [com.magusgeek.brutaltester.GameThread] If you want to replay and see this game, use the following command line:
10:09:28,406 ERROR [com.magusgeek.brutaltester.GameThread] java -jar /tmp/referee.jar -p1 /tmp/myai -p2 /tmp/myai -l /tmp/logs/game1.json -s
10:09:28,406 INFO  [com.magusgeek.brutaltester.GameThread] End of game 2	 0,00% 0,00%
10:09:28,406 INFO  [com.magusgeek.brutaltester.GameThread] End of game 1	 0,00% 0,00%
10:09:28,406 INFO  [com.magusgeek.brutaltester.Main] *** End of games ***
+----------+----------+----------+
| Results  | Player 1 | Player 2 |
+----------+----------+----------+
| Player 1 |          | 0,00%    |
+----------+----------+----------+
| Player 2 | 0,00%    |          |
+----------+----------+----------+
```

L'erreur est :

```
aucun attribut manifest principal dans /tmp/referee.jar
```

L'explication [ici](https://stackoverflow.com/a/9689877) est qu'il faut que je modifie `pom.xml` non seulement pour inclure la dépendance à la CLI (que j'utilise pour rendre le referee compatible avec brutaltester), mais également pour configurer maven afin de rendre le jar exécutable, et que l'exécution du jar pointe vers le main de la CLI.
