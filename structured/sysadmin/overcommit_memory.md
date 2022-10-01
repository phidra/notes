**TL;DR** : c'est un setting du kernel Linux modifiable via `/proc/sys/vm/overcommit_memory` autorisant ou non à allouer plus de mémoire virtuelle que de mémoire physique.

# Principe

C'est lié à la différence entre mémoire virtuelle et mémoire physique : on peut très bien ne disposer que d'1 Gio de mémoire physique, et allouer 99 Gio de mémoire virtuelle (j'avais même fait une POC là-dessus ; dans ce cas, c'est juste que 98Gio de cette mémoire ne sera jamais backée par de la mémoire physique, ce qui n'est pas forcément embêtant, p.ex. si elle n'est jamais utilisée).

Au moment d'allouer de la mémoire virtuelle, le kernel peut checker s'il aura assez de mémoire physique pour backer la mémoire virtuelle qu'on essaye d'allouer ; `overcommit_memory` contrôle ce check.

https://serverfault.com/a/606193

> The simple answer is that setting overcommit to 1, will set the stage so that when a program calls something like malloc() to allocate a chunk of memory (man 3 malloc), it will always succeed regardless if the system knows it will not have all the memory that is being asked for.
>
> The underlying concept to understand is the idea of virtual memory. Programs see a virtual address space that may, or may not, be mapped to actual physical memory. By disabling overcommit checking, you tell the OS to just assume that there is always enough physical memory to backup the virtual space.


# Doc et usage

La doc est dans `man 5 proc` :

- `man 5` = c'est une page de man d'un format de fichier
- `/proc` = _The proc filesystem is a pseudo-filesystem which provides an interface to kernel data structures_
- `man 5 proc` = le man explique ce qu'on trouve sous `/proc`

Du coup, `man 5 proc` explique `/proc/sys/vm/overcommit_memory` :
- `0` = une heuristique décide si le kernel checke
- `1` = le kernel ne checke jamais (et autorise systématiquement à allouer une mémoire virtuelle dépassant la capacité de la mémoire physique)
- `2` = le kernel checke systématiquement

Par exemple :

```sh
sudo su -
echo 1 > /proc/sys/vm/overcommit_memory
```

## Cas d'usage où on peut vouloir overcommit_memory=1

redis a sa DB in-memory, et sauvegarde sa DB sur disque en tâche de fond.

pour faire ça, redis forke le process, ce qui a pour effet de doubler la mémoire virtuelle nécessaire, mais pas nécessairement la mémoire physique :

- avec le mécanisme de copy-on-write, les pages ne sont dupliquées que si le fork (ou le process original) écrit dessus
- si personne n'écrit sur la DB le temps que le fork finisse de pérenniser la DB sur disque, le copy-on-write fait que le fork n'aura pas nécessité de mémoire physique supplémentaire, yay \o/
- à l'inverse, dans le cas extrême où quelqu'un aura écrit sur TOUTES les pages de RAM utilisées par la DB, le copy-on-write n'aura servi à rien
- et dans le cas (que j'imagine le plus probable) :
    - la majorité des pages n'aura pas été modifiée (donc le COW fera qu'on aura pérennisé ces pages sur disque en background sans nécessiter de RAM supplémentaire)
    - une petite partie des pages aura été modifiée, donc copiées en RAM : le process original aura modifié ces pages, et le fork aura pérennisé la copie originale

https://redis.io/docs/getting-started/faq/#background-saving-fails-with-a-fork-error-on-linux

> The Redis background saving schema relies on the copy-on-write semantic of the fork system call in modern operating systems: Redis forks (creates a child process) that is an exact copy of the parent. The child process dumps the DB on disk and finally exits. In theory the child should use as much memory as the parent being a copy, but actually thanks to the copy-on-write semantic implemented by most modern operating systems the parent and child process will share the common memory pages. A page will be duplicated only when it changes in the child or in the parent. Since in theory all the pages may change while the child process is saving, Linux can't tell in advance how much memory the child will take, so if the overcommit_memory setting is set to zero the fork will fail unless there is as much free RAM as required to really duplicate all the parent memory pages. If you have a Redis dataset of 3 GB and just 2 GB of free memory it will fail.
>
> Setting overcommit_memory to 1 tells Linux to relax and perform the fork in a more optimistic allocation fashion, and this is indeed what you want for Redis.
