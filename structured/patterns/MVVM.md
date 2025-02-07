Source = https://learn.microsoft.com/fr-fr/dotnet/architecture/maui/mvvm

En résumé :

- **Model** = logique métier
- **ViewModel** = logique de présentation
- **View** = interface utilisateur

----

Contexte = C# / Windows

En gros :

- Model = le modèle des données (+ le code métier qui va avec, je suppose)
- View = ce que voit l'utilisateur
- ViewModel = la logique de présentation des données (e.g. "si tel modèle passe à forbidden, alors tel élément de l'UI doit afficher une croix rouge")

Le lien entre View et ViewModel utilise des technos ad-hoc (e.g. XAML → un clic sur un bouton exécute telle fonction du ViewModel)

Dépendance :

- le Model ne connaît personne
- le ViewModel connaît le Model mais ne connaît pas la View
- la View connaît le ViewModel (mais ne connaît pas le Model : la View *passe par* le ViewModel pour accéder au Model)

Le ViewModel est en quelque sorte l'interface par laquelle la View accède au Model

- en ce sens, le ViewModel et un adapter du Model utilisé par la View
- e.g. le ViewModel peut combiner deux attributs différents du modèle en un seul qu'il expose à la View

Les données et fonctions du ViewModel représentent ce que peut faire l'UI (et qui sont rendues visibles à l'utilisateur par la View)

Le ViewModel notifie la vue des changements du modèle via des notifications (NDM : probablement d'une façon découplée type observer)
