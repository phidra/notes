# /usr/bin/time

Attention à passer le chemin complet (sans quoi, on utilise la shell builtin "time", qui est différente) :

```sh
/usr/bin/time -p ls
```


D'après le man permet de mesurer pas mal de choses liées aux perfs, qu'on peut afficher avec format. Morceaux choisis :

- E      Elapsed real (wall clock) time used by the process, in [hours:]minutes:seconds.
- e      Elapsed real (wall clock) time used by the process, in seconds.
- -----
- F      Number of major, or I/O-requiring, page faults that occurred while the process was running.
- R      Number of minor, or recoverable, page faults.
- -----
- I      Number of file system inputs by the process.
- O      Number of file system outputs by the process.
- -----
- P      Percentage of the CPU that this job got.  This is just user + system times divided by the total running time.  It also prints a percentage sign.
- S      Total number of CPU-seconds used by the system on behalf of the process (in kernel mode), in seconds.
- U      Total number of CPU-seconds that the process used directly (in user mode), in seconds.
- -----
- c      Number of times the process was context-switched involuntarily (because the time slice expired).
- w      Number of times that the program was context-switched voluntarily, for instance while waiting for an I/O operation to complete.


Exemples :

```sh
alias tim='/usr/bin/time -f "\n%C\ntime = %E\nmajor page faults = %F\nFS inputs = %I\nspent in userland = %U\nspent in kernel = %S\ninvolontary switch=%c\nvolontary switch=%w"'

tim find /etc/ -name \*.conf 1> /dev/null
# time = 0:00.06
# major page faults = 2
# FS inputs = 5072
# spent in userland = 0.01
# spent in kernel = 0.01
# involontary switch=453
# volontary switch=455


tim ls -lh ~ 1> /dev/null
# time = 0:00.00
# major page faults = 0
# FS inputs = 0
# spent in userland = 0.00
# spent in kernel = 0.00
# involontary switch=0
# volontary switch=1


tim sleep 2.3
# time = 0:02.30
# major page faults = 0
# FS inputs = 0
# spent in userland = 0.00
# spent in kernel = 0.00
# involontary switch=0
# volontary switch=1
```
