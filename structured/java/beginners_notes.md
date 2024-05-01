**Contexte** : 2024, je n'ai jamais fait de java mais j'ai besoin de hacker des outils java pour un challenge CodinGame, sans avoir de temps à consacrer à un apprentissage correct ; je fais quelques explorations générales, aidé de ChatGPT.

* [Structure de répertoires](#structure-de-répertoires)
* [Exécuter un programme](#exécuter-un-programme)
* [Gestion des dépendances](#gestion-des-dépendances)


# Structure de répertoires

_Peux-tu me décrire la structure de répertoires classiques d'un projet java ? Notamment, le répertoire src semble contenir beaucoup de sous-répertoires java, com, etc._

> La structure de répertoires d'un projet Java suit généralement une convention standard pour faciliter l'organisation et la gestion du code. Voici une description de base de cette structure :
> 1. **src** : Ce répertoire contient tous les fichiers source Java (.java) du projet. Il est souvent divisé en deux sous-répertoires principaux :
>    - **main** : Contient le code source de l'application.
>    - **test** : Contient le code source des tests unitaires.
>    Chacun de ces sous-répertoires peut contenir un répertoire **java** qui contient le code source organisé en packages.
> 2. **com, org, net, etc.** : Ces répertoires sont généralement le début des packages Java. Par exemple, si vous avez un package appelé "com.example.myapp", vous auriez un répertoire "com" dans "src/main/java", qui contiendrait un sous-répertoire "example", qui contiendrait à son tour un sous-répertoire "myapp".
> 3. **resources** : Ce répertoire est souvent présent sous "src/main" et "src/test". Il contient des fichiers non-Java tels que les fichiers de configuration, les fichiers de propriétés, les fichiers XML, les images, etc.
> 4. **target** : Ce répertoire est généré lorsque vous compilez votre projet. Il contient tous les fichiers de classe compilés (.class) et le fichier JAR ou WAR final.
> 5. **lib** : Ce répertoire contient toutes les bibliothèques tierces nécessaires au projet.
> 6. **doc** : Ce répertoire contient la documentation du projet, comme les fichiers Javadoc.
> 7. **.git** : Si vous utilisez Git pour la gestion de version, ce répertoire contient tous les métadonnées de Git.
> 8. **.idea** : Si vous utilisez IntelliJ IDEA comme IDE, ce répertoire contient les métadonnées spécifiques à IntelliJ.
> Notez que cette structure peut varier en fonction des besoins spécifiques du projet et des outils utilisés pour le développement, comme les systèmes de build (Maven, Gradle, etc.) et les IDEs (Eclipse, IntelliJ, etc.).

_Peux-tu m'en dire plus sur com, org, net, etc ? À quoi correspondent ces noms, dans quel cas placer mes fichiers sources dans quels répertoires ?_

> Les termes "com", "org", "net", etc. sont généralement utilisés comme préfixes de package en Java pour éviter les conflits de noms entre les packages. Ils sont basés sur le nom de domaine inversé de l'organisation qui a créé le package, ce qui garantit l'unicité du nom du package.
> Voici comment cela fonctionne :
> - "com" est généralement utilisé par les organisations commerciales. Par exemple, si une entreprise appelée "MyCompany" avec le domaine "mycompany.com" créait un package, elle pourrait le nommer "com.mycompany.mypackage".
> - "org" est souvent utilisé par les organisations non commerciales. Par exemple, Apache, une fondation à but non lucratif, utilise "org.apache" comme préfixe de package.
> - "net" est souvent utilisé par les fournisseurs de services réseau.
> Ces préfixes ne sont pas obligatoires, mais ils sont fortement recommandés pour éviter les conflits de noms. Vous pouvez choisir le préfixe qui correspond le mieux à votre organisation ou à votre projet.
> En ce qui concerne l'endroit où placer vos fichiers sources, chaque sous-répertoire sous "com", "org", "net", etc. correspond à un sous-package. Par exemple, si vous avez un fichier source qui fait partie du package "com.mycompany.mypackage", vous le placeriez dans le répertoire "src/main/java/com/mycompany/mypackage".

**TL;DR** : `com`, `org`, etc. est un "nom de domaine inversé", pour éviter les conflits de noms.

# Exécuter un programme

_Lorsque je crée un fichier contenant un main, par exemple celui-ci :_

```
public class HelloWorld {
    public static void main(String[] args) {
        // Prints "Hello, World" in the terminal window.
        System.out.println("Hello, World");
    }
}
```
_Comment exécuter ce programme ?_

> Pour exécuter ce programme, vous devez suivre les étapes suivantes :
> 1. **Compilation** : Tout d'abord, vous devez compiler le fichier source Java (.java) en bytecode Java (.class) en utilisant le compilateur `javac`. Ouvrez un terminal, naviguez jusqu'au répertoire contenant votre fichier `HelloWorld.java` et exécutez la commande suivante :
>    ```
>    javac HelloWorld.java
>    ```
>    Cela générera un fichier `HelloWorld.class` dans le même répertoire.
> 2. **Exécution** : Ensuite, vous pouvez exécuter le programme en utilisant la commande `java`. Notez que vous devez utiliser le nom de la classe sans l'extension `.class`. Voici comment vous pouvez le faire :
>    ```
>    java HelloWorld
>    ```
>    Cela devrait afficher "Hello, World" dans le terminal.
> Notez que ces instructions supposent que vous avez installé le JDK (Java Development Kit) sur votre système et que vous l'avez correctement configuré dans votre variable d'environnement PATH. Si ce n'est pas le cas, vous devrez le faire avant de pouvoir compiler et exécuter des programmes Java.

_Et que sont les fichiers jar ?_

> Un fichier JAR (Java Archive) est un type de fichier utilisé pour regrouper et distribuer un ensemble de fichiers Java. Il est utilisé pour stocker les fichiers de classe compilés (.class), les fichiers associés (images, sons, etc.) et les fichiers de manifeste qui contiennent des métadonnées sur le JAR.
> Les fichiers JAR sont essentiellement des fichiers zip qui ont été renommés avec l'extension .jar. Ils sont utilisés pour plusieurs raisons :
> 1. **Distribution** : Les fichiers JAR facilitent la distribution de logiciels Java. Au lieu de distribuer de nombreux fichiers .class, vous pouvez distribuer un seul fichier .jar.
> 2. **Réutilisation** : Les bibliothèques Java sont souvent distribuées sous forme de fichiers JAR pour que d'autres développeurs puissent les inclure et les utiliser dans leurs propres projets.
> 3. **Exécution** : Si un fichier JAR contient une classe avec une méthode `main`, il peut être exécuté directement à partir de la ligne de commande avec la commande `java -jar`.
> Pour créer un fichier JAR, vous pouvez utiliser l'outil `jar` qui est inclus avec le JDK. Par exemple, la commande suivante crée un fichier JAR appelé `myapp.jar` à partir de tous les fichiers .class dans le répertoire courant :
> ```
> jar cf myapp.jar *.class
> ```
> Pour exécuter un fichier JAR qui contient une méthode `main`, vous pouvez utiliser la commande `java -jar`. Par exemple :
> ```
> java -jar myapp.jar
> ```

# Gestion des dépendances

_Et qu'est le fichier pom.xml ?_

> Le fichier `pom.xml` est un fichier de configuration pour Apache Maven, un outil de gestion de projet et de compréhension de projet utilisé dans les projets Java. "POM" signifie "Project Object Model".
> Le fichier `pom.xml` contient des informations sur le projet et des détails de configuration utilisés par Maven pour construire le projet. Il inclut les éléments suivants :
> 1. **Informations sur le projet** : Cela comprend le nom du projet, sa version, son URL, ses développeurs, ses licences, etc.
> 2. **Dépendances** : Il s'agit d'une liste des autres projets dont votre projet dépend. Maven téléchargera automatiquement ces dépendances lors de la construction de votre projet.
> 3. **Plugins** : Ce sont des outils qui fournissent des tâches de construction, comme la compilation du code Java et la création de fichiers JAR.
> 4. **Goals** : Ce sont des tâches spécifiques que Maven peut exécuter, comme la compilation du code source et l'exécution des tests unitaires.
> 5. **Build profiles** : Ce sont des configurations de construction alternatives qui peuvent être utilisées pour construire le projet dans différents environnements.
> 6. **Repositories** : Ce sont des emplacements où Maven peut trouver les dépendances de votre projet.
> En résumé, le fichier `pom.xml` est essentiellement une recette que Maven utilise pour construire votre projet. Il décrit ce qui est nécessaire pour construire votre projet et comment le construire.
