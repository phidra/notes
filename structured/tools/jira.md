
Cheatsheet [jql](https://www.atlassian.com/software/jira/guides/jql/cheat-sheet) :

**Exemple** :

```
project = MyProject AND labels = externallyCreated AND statusCategory != Done ORDER BY updated DESC
```

# inclure / exclure des tickets

- ne garder que les tickets d'un projet donné :
    ```
    project = MYPROJECT
    ```
- exclure les tickets déjà résolus :
    ```
    statusCategory != Done
    ```
- ne garder que les tickets (pas encore commencés / en cours de traitement / terminés) :
    ```
    statusCategory = "To Do"
    statusCategory = "In Progress"
    statusCategory = Done
    ```
- ne garder que les tickets qui ont le label "POUET" (parmi d'autres labels possibles) :
    ```
    labels = "POUET"
    ```
- ne garder que les tickets qui ont à la fois le label "POUET1" et le label "POUET2" (parmi d'autres labels possibles ):
    ```
    labels = POUET1 AND labels = POUET2
    ```
- exclure les tickets qui ont le label "POUET" :
    ```
    (labels != POUET OR labels IS EMPTY)
    ```
- ne garder que les tickets créés il y a moins de 30j :
    ```
    created >= -30d
    ```
- contenant la chaîne "POUET" dans les champs textuels (commentaires INCLUS) :
    ```
    text ~ "POUET"
    ```
- contenant la chaîne "POUET" dans les champs textuels (commentaires EXCLUS) :
    ```
    textfields ~ "POUET"
    ```
- ne garder que les tickets qui me sont assignés / qui ne sont pas assignés :
    ```
    assignee = currentUser()
    assignee IS EMPTY
    ```

# trier les tickets retenus

```
ORDER BY updated DESC
```

Champs utiles :
- **updated** = par date de dernière mise à jour
- **created** = par date de création
- **priority** = par priorité


