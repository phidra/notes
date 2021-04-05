# What problems do people solve with strace?

- **url** = https://jvns.ca/blog/2021/04/03/what-problems-do-people-solve-with-strace/
- **type** = post
- **auteur** = [Julia EVANS](https://jvns.ca/about/) aka [b0rk](https://twitter.com/b0rk), une bloggeuse dont j'apprécie beaucoup le travail
- **date de publication** = 2021-04-03
- **source** = le [blog de Julia EVANS](https://jvns.ca/)

**TL;DR** : une liste de chouettes problèmes concrets que `strace` peut nous aider à résoudre :

- problem 1: where’s the config file?
    * par exemple, regarder où `mypy` va lire son fichier de config ? À tester !
- problem 2: what other files does this program depend on?
    * généralisation du cas précédent
    * notamment, voir les shared-libs dont un programme dépend
- problem 3: why is this program hanging?
    * *you just need to run strace -p PID and look at what system call is currently running*
    * un cas d'usage intéressant : with strace `df -h` you can find the stuck mount and unmount it
- problem 4: is this program stuck?
    * you just want to know if it’s stuck or of it’s still making progress.
    * just strace it and see if it’s making new system calls!
- problem 5: why is this program slow?
    * profiling "à gros grains" (bien moins précis, mais peut-être plus simple ou mieux maîtrisé qu'un vrai profiler)
    * Optimizing app startup times… running `strace` can be an eye-opening experience, in terms of the amount of unnecessary file system interaction going on
    * Why is this program so slow to start? `strace` shows it opening/reading the same config file thousands of times.
- problem 6: hidden permissions errors
    * cas d'usage = le programme échoue sans trop préciser pourquoi. En fait, une permission error l'empêche de continuer, mais est "mangée (i.e. non-loggée) par le programme.
- problem 7: what command line arguments are being used?
    * Sometimes a script is running another program, and you want to know what command line flags it’s passing!
    * Wow, très intéressant. On peut voir les appels à tous les sous-programmes (sans avoir à ajouter des prints).
- problem 8: why is this network connection failing?
    * Basically the goal here is just to find which domain / IP address the network connection is being made to.
    * alternative à `tcpdump` (peut-être plus simple, ou simplement si on ne peut pas utiliser `tcpdump`)
- problem 9: why does this program succeed when run one way and fail when run in another way?
    * dans ce cas, comparer les résultats de `strace` dans les deux cas peut aider à comprendre ce qui change
    * en première approche, elle recommande plutôt de jeter un oeil aux envvars des deux environnements d'exécution
