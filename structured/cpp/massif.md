* [TL;DR](#tldr)
* [Points d'attention](#points-dattention)
   * [type de mémoire mesuré](#type-de-mémoire-mesuré)
   * [snapshot normaux + snapshots détaillés + peak snapshot](#snapshot-normaux--snapshots-détaillés--peak-snapshot)
   * [caveat = utilisation avec pyenv](#caveat--utilisation-avec-pyenv)
* [Notes à l'usage](#notes-à-lusage)

# TL;DR

massif = heap profiler

https://valgrind.org/docs/manual/ms-manual.html

```sh
# installation :
sudo apt install valgrind
sudo apt install massif-visualizer

# profiling :
valgrind --tool=massif ./my-program --param1=pouet arg1 arg2

# visualisation :
massif-visualizer massif.out.97971
```

# Points d'attention

## type de mémoire mesuré

**TL;DR** : par défaut, tous les types d'allocations mémoire ne sont pas monitorés (notamment ceux avec `mmap`), donc attention à prendre les résultats avec des pincettes ; si besoin, utiliser `--pages-as-heap=yes` pour mesurer l'ensemble de la mémoire (mais perdre la ventilation).

> It is worth emphasising that by default Massif measures only heap memory, i.e. memory allocated with malloc, calloc, realloc, memalign, new, new[], and a few other, similar functions. (And it can optionally measure stack memory, of course.) This means it does not directly measure memory allocated with lower-level system calls such as mmap, mremap, and brk.
>
> Heap allocation functions such as malloc are built on top of these system calls. For example, when needed, an allocator will typically call mmap to allocate a large chunk of memory, and then hand over pieces of that memory chunk to the client program in response to calls to malloc et al. Massif directly measures only these higher-level malloc et al calls, not the lower-level system calls.
>
> Furthermore, a client program may use these lower-level system calls directly to allocate memory. By default, Massif does not measure these. Nor does it measure the size of code, data and BSS segments. Therefore, the numbers reported by Massif may be significantly smaller than those reported by tools such as top that measure a program's total size in memory.
>
> However, if you wish to measure all the memory used by your program, you can use the --pages-as-heap=yes. When this option is enabled, Massif's normal heap block profiling is replaced by lower-level page profiling. Every page allocated via mmap and similar system calls is treated as a distinct block. This means that code, data and BSS segments are all measured, as they are just memory pages. Even the stack is measured, since it is ultimately allocated (and extended when necessary) via mmap; for this reason --stacks=yes is not allowed in conjunction with --pages-as-heap=yes.

## snapshot normaux + snapshots détaillés + peak snapshot

**TL;DR** : massif fonctionne en prenant des snapshots réguliers, mais tous les snapshots n'enregistrent pas la ventilation de la RAM : seuls les snapshots détaillés le font. Un snapshot détaillé particulier est enregistré au moment où le max de consommation est atteint. Les options utiles :

- `--detailed-freq` : par défaut, un snapshot sur 10 est détaillé
- `--max-snapshots` : par défaut, `100`
- `--depth` : par défaut, `30`

> Massif starts by taking snapshots for every heap allocation/deallocation, but as a program runs for longer, it takes snapshots less frequently. It also discards older snapshots as the program goes on; when it reaches the maximum number of snapshots (100 by default, although changeable with the --max-snapshots option) half of them are deleted. This means that a reasonable number of snapshots are always maintained.
>
> Most snapshots are normal, and only basic information is recorded for them.
>
> Some snapshots are detailed. Information about where allocations happened are recorded for these snapshots [...] By default, every 10th snapshot is detailed, although this can be changed via the --detailed-freq option.
>
> Finally, there is at most one peak snapshot. The peak snapshot is a detailed snapshot, and records the point where memory consumption was greatest.

La détection du peak peut être imprécise :

> Massif's determination of when the peak occurred can be wrong, for two reasons.
>
> Peak snapshots are only ever taken after a deallocation happens. This avoids lots of unnecessary peak snapshot recordings (imagine what happens if your program allocates a lot of heap blocks in succession, hitting a new peak every time). But it means that if your program never deallocates any blocks, no peak will be recorded. It also means that if your program does deallocate blocks but later allocates to a higher peak without subsequently deallocating, the reported peak will be too low.
>
> Even with this behaviour, recording the peak accurately is slow. So by default Massif records a peak whose size is within 1% of the size of the true peak. This inaccuracy in the peak measurement can be changed with the --peak-inaccuracy option.

## caveat = utilisation avec pyenv

Attention si utilisation pour profiler un programme `prog` lancé avec pyenv :

```sh
which superprog
# /home/moi/.pyenv/shims/superprog

pyenv which superprog
# /home/moi/.pyenv/versions/superenv/bin/superprog

# NON :
valgrind --tool=massif superprog

# OUI :
valgrind --tool=massif /home/moi/.pyenv/versions/superenv/bin/superprog
```

# Notes à l'usage

- massif ne dumpe les infos qu'en sortie de programme ; ce serait plus pratique de pouvoir trigger un dump, mais ça n'a pas l'air faisable directement (ça a l'air possible via gdbserver : [lien](https://valgrind.org/docs/manual/manual-core-adv.html#manual-core-adv.gdbserver-commandhandling), mais j'ai pas creusé)
- attention : tous les snapshots ne sont pas des snapshots détaillés
- avec massif-visualizer, la ventilation est limitée aux consommations les plus importantes ; les petites sont masquées (ce qui explique le gap entre la conso totale et les consos ventilées
- mieux vaut passer massif-visualizer en anglais (c'est pas inutitif, mais le changement de lanque est dans le menu `Aide`)

