# Processes or Threads ?

- **url** = https://lucisqr.substack.com/p/processes-or-threads
- **type** = post
- **auteur** = [Henrique BUCHER](https://substack.com/profile/84566051-henrique-bucher) = dev orienté high-frequency trading, qui bosse chez [Citadel](https://www.citadel.com/)
- **date de publication** = 2022-10-04
- **source** = ses publications sur [substack](https://substack.com/) = plate-forme pour _independent writers_

- **tags** = language>C++ ; topic>IPC ; topic>multiprocessing ; topic>multithreading ; level>intermediate

**TL;DR** : un article qui défend l'utilisation de plusieurs process plutôt que plusieurs threads.

La première partie est intéressante car elle liste plein de raisons de préférer les process aux threads, y compris des raisons auxquelles je n'avais pas pensé, qui tournent autour du fait qu'il est plus facile de manipuler des process indivudels plutôt que des threads individuels (e.g. on peut shutdown des process un par un, mais pas des threads) :

1. Support staff cannot restart a thread if something go wrong. It is straightforward to restart a process.
2. Support staff can tweak processes as making its priority higher (chrt) or lower (nice), change its affinity (taskset). Some of this can be done with threads as well but - I can count in my left hand how many admins know how to do it.
3. Developers can individually attach a problematic process to a debugger without affecting the rest of the running application
4. Developers can have different binaries compiled with distinct compiler arguments (debug vs release, instrumented vs not) and mix and match them in realtime to fit their needs
5. Support staff can start as many processes as necessary at any time and shut them down cleanly, giving fine control over the utilization of resources - without need to explicitly code and release it in C/C++
6. Processes and threads on Linux are basically the same thing, they are even created by the same system call (clone). Processes consume a bit more resources but unless in extreme cases, it is not even justifiable to think that as a con.
7. Programing single-thread processes is way simpler than dealing with all the complexity that arises with shared resources at a micro scale. Even Joe the intern can handle it.
8. Single threaded processes can more easily control the resources it uses like memory cache. With the new era of L3-cache administration coming up, this is a must have.
9. If one process crashes, the damage is limited. If one thread crashes, the entire application goes down.
10. If one process gets hacked by a buffer overflow, the invasion is typically contained while if a thread is compromised, the entire application is too.
11. Threads incentivize the use of patterns and idioms that lead to too fine grained locking contention which is a major performance killer.
12. It is very easy for a junior (or malicious senior) developer to bypass facilities to deal with shared resources across threads and access memory directly, possibly resulting in all sorts of hard-to-debug problems as race conditions, thread starving and segfaults.
13. Debugging is much harder or even impossible on a heavily multi-threaded application while it is piece of cake on a multiple single-process setup. (This is somewhat repeated but here for emphasys).
14. If you use a file-backed shared memory container and your process crashes, the operating system will write the changes written to memory into the disk after the process finishes for you.

La suite de l'article est une illustration d'IPC à base de shared-memory (et pour moi, il se tire une balle dans le pied pour son argumentation, car il laisse à penser que c'est super-simple, alors que le code n'est pas trivial, même si c'est vrai que le multithreading ne l'est pas non plus).

Ça reste assez intéressant,  p.ex. il montre comment initialiser une instance dans une shared-mem en utilisant `placement new`.
