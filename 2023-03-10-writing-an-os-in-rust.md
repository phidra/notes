# Writing an OS in Rust

- **url** = https://os.phil-opp.com/
- **type** = blogpost serie
- **auteur** = [Philipp OPPERMANN](https://github.com/phil-opp), je ne trouve pas son auto-présentation, mais il semble être un dev rust
- **date de publication** = 2018-02-10
- **source** = j'annote tout le contenu [du blog](https://github.com/phil-opp), donc la source et l'URL sont pour une fois identiques
- **tags** = language>agnostic ; language>rust ; topic>os ; level>advanced


**TL;DR** : une EXCELLENTE série d'articles, à la base pour coder son propre OS en rust ; en pratique, il explique de façon très pédagogique beaucoup de concepts bas-niveau qui m'intéressent :

- ce qui fait qu'un exécutable peut tourner sur une machine et pas sur une autre
- le boot d'un OS
- les interrupts
- la gestion de la mémoire (segmentation et paging)
- etc.

J'annote l'ensemble des contenus comme une seule ressource, même si tout est splitté en plusieurs posts.

* [Writing an OS in Rust](#writing-an-os-in-rust)
   * [Bare bones](#bare-bones)
      * [Préambule : Bare machine](#préambule--bare-machine)
      * [Préambule : Standalone program](#préambule--standalone-program)
      * [A freestanding rust binary](#a-freestanding-rust-binary)
         * [target triplet](#target-triplet)
      * [A minimal rust kernel](#a-minimal-rust-kernel)
      * [VGA Text Mode](#vga-text-mode)
      * [Testing](#testing)
   * [Interrupts](#interrupts)
      * [CPU Exceptions](#cpu-exceptions)
      * [Double Faults](#double-faults)
      * [Hardware Interrupts](#hardware-interrupts)
   * [Memory management](#memory-management)
      * [Introduction to paging](#introduction-to-paging)
      * [Paging Implementation](#paging-implementation)
      * [Heap Allocation](#heap-allocation)
      * [Allocator Designs](#allocator-designs)
   * [Multitasking](#multitasking)
      * [Async/Await](#asyncawait)

## Bare bones

### Préambule : Bare machine

https://en.wikipedia.org/wiki/Bare_machine

Je sous-commence par une explication externe au blog sur les standalone programs et le bare-metal, à partir d'un lien qu'il donne vers la page wikipedia de bare-machine qui résume :

> In computer science, bare machine (or bare metal) refers to a computer executing instructions directly on logic
> hardware without an intervening operating system. \
> [...] prior to the development of operating systems, sequential instructions were executed on the computer hardware
> directly using machine language without any system software layer. This approach is termed the "bare machine"
> precursor to modern operating systems. \
> Today it is mostly applicable to embedded systems and firmware generally with time-critical latency requirements, while conventional programs are run by a runtime system overlaid on an operating system.

^ Du coup, en première approche, ma compréhension est :

- exécutable lancé sur bare-machine = séquence d'instructions en language machine directement exécutables par le processeur
- exécutable lancé via OS = séquence d'instructions en language machine qui utilise un runtime = l'OS

Cette citation est intéressante aussi :

> Advantages : For a given application, in most of the cases, a bare-metal implementation will run faster, using less memory and so being more power efficient. This is because operating systems, as any program, need some execution time and memory space to run, and these are no longer needed on bare-metal. For instance, any hardware feature that includes inputs and outputs are directly accessible on bare-metal, whereas the same feature using an OS must route the call to a subroutine, consuming running time and memory.

> Disadvantages : For a given application, bare-metal programming requires more effort to work properly and is more complex because the services provided by the operating system and used by the application have to be re-implemented regarding the needs. These services can be:
>
> - System boot (mandatory)
> - Memory management: Storing location of the code and the data regarding the hardware resources and peripherals (mandatory)
> - Interruptions handling (if any)
> - Task scheduling, if the application can perform more than one task
> - Peripherals management (if any)
> - Error management, if wanted or needed
> - Debugging a bare-metal program is difficult [...]

^ ce que fournit le runtime = l'OS, c'est p.ex. la gestion du boot, la gestion de la mémoire, les threads, la gestion des fichiers, le réseau, etc.

Et en résumé, le bare-metal programming consiste à écrire un programme capable de tourner sans utiliser ces features fournies par l'OS.

NdM : c'est moins évident qu'il n'y paraît, car la libc (sur laquelle s'appuient toutes les libs standards des languages de programmation) utilise les fonctionnalités de l'OS ; du coup pour écrire un freestanding program, il ne faut pas utiliser la lib standard.

> Bare machine programming remains common practice in embedded systems, where microcontrollers or microprocessors often boot directly into monolithic, single-purpose software, without loading a separate operating system. Such embedded software can vary in structure, but the simplest form may consist of an infinite main loop, calling subroutines responsible for checking for inputs, performing actions, and writing outputs.

### Préambule : Standalone program

https://en.wikipedia.org/wiki/Standalone_program

> A standalone program, also known as a freestanding program, is a computer program that does not load any external
> module, library function or program and that is designed to boot with the bootstrap procedure of the target processor
> – it runs on bare metal.

Les programme destinés à tourner sur bare-metal sont appelés **standalone programs** / **freestanding program** / **bare-metal program**.

> Standalone programs are usually written in assembly language for a specific hardware.

### A freestanding rust binary

https://os.phil-opp.com/freestanding-rust-binary/

Overall post intéressant pour comprendre comment un freestanding programe tourne sur bare-metal.

> The first step in creating our own operating system kernel is to create a Rust executable that does not link the standard library. This makes it possible to run Rust code on the bare metal without an underlying operating system.

Ceci est maintenant clair suite à mes deux préambules : ce qui différencie un exe lancé depuis l'OS, et un exe lancé depuis le bare-metal = l'exe bare-metal ne fait pas appel aux fonctionnalités de l'OS, et réimplémente lui-même celles dont il a besoin, comme la gestion de la mémoire.

NdM : au final, vu comme ça, un OS est un freestanding program particulier, destiné à fournir un runtime pour exécuter d'autres programmes (un peu comme l'interpréteur cpython est un programme particulier, destiné à fournir un runtime pour exécuter d'autres programmes en python).

> To write an operating system kernel, we need code that does not depend on any operating system features. This means that we can’t use threads, files, heap memory, the network, random numbers, standard output, or any other features requiring OS abstractions or specific hardware.

^ quelques exemples des features qui sont fournies par le runtime de l'OS, et dont on ne dispose pas quand on développe un freestanding program.

> This means that we can’t use most of the Rust standard library, but there are a lot of Rust features that we can use. For example, we can use iterators, closures, pattern matching, option and result, string formatting, and of course the ownership system. These features make it possible to write a kernel in a very expressive, high level way without worrying about undefined behavior or memory safety.

Ce point est intéressant, en voici ma compréhension :

- si on veut développer un freestanding program, la lib standard rust (ou du moins, une grande partie) ne sera pas utilisable car elle utilise les fonctionnalités de l'OS
- mais un programme écrit en rust sera compilé en du code machine tout de même, donc si on n'utilise pas la lib standard rust, ce code machine sera freestanding
- du coup on peut tout à fait utiliser rust pour écrire un freestanding program, en profitant de beaucoup des features de rust _autres_ que sa lib standard (tel que son typage fort, ou le borrow checker)
- (et je suppose que pour disposer tout de même des fonctionnalités basiques telles qu'un print sur stdout, il faudra les réimplémenter nous-même, probablement en inlinant de l'assembleur)

> By default, all Rust crates link the standard library, which depends on the operating system for features such as threads, files, or networking. It also depends on the C standard library libc, which closely interacts with OS services.

^ On parle de la stdlib rust, mais le problème est le même avec la stdlib C.

> So we can no longer print things. This makes sense, since println writes to standard output, which is a special file descriptor provided by the operating system.

Ça confirme ce que je pensais plus haut. Et de façon logique mais intéressante, `stdout` est en fait une feature fournie par l'OS !

La suite du post indique quelques choses liées à la compilation rust en elle-même (plutôt qu'au bare-metal programming) : nécessité d'avoir un handler pour les panic, impossibilités d'unwind facilement les stacks sans utiliser `libunwind` qui dépend de l'OS, nécessité pour le linker rust de trouver un symbole `_start`, etc.

Au sujet du runtime :

> One might think that the main function is the first function called when you run a program. However, most languages have a runtime system, which is responsible for things such as garbage collection (e.g. in Java) or software threads (e.g. goroutines in Go). This runtime needs to be called before main, since it needs to initialize itself.
>
> In a typical Rust binary that links the standard library, execution starts in a C runtime library called crt0 (“C runtime zero”), which sets up the environment for a C application. This includes creating a stack and placing the arguments in the right registers. The C runtime then invokes the entry point of the Rust runtime, which is marked by the start language item. Rust only has a very minimal runtime, which takes care of some small things such as setting up stack overflow guards or printing a backtrace on panic. The runtime then finally calls the main function.

#### target triplet

Puis, il y a un paragraphe très intéressant sur les target-triplets = et ce qui fait qu'un programme est ou non compatible avec un ordinateur. Comme c'est aussi utile pour la cross-compilation (pour compiler sur ma machine un programme qui s'exécutera sur une autre machine), c'est bien expliqué dans [la doc clang](https://clang.llvm.org/docs/CrossCompilation.html#target-triple)

> The triple has the general format `<arch><sub>-<vendor>-<sys>-<abi>`, where:
>
> `arch` = x86_64, i386, arm, thumb, mips, etc. \
> `sub` = for ex. on ARM: v5, v6m, v7a, v7m, etc. \
> `vendor` = pc, apple, nvidia, ibm, etc. \
> `sys` = none, linux, win32, darwin, cuda, etc. \
> `abi` = eabi, gnu, android, macho, elf, etc.
>
> The sub-architecture options are available for their own architectures, of course, so `x86v7a` doesn’t make sense.
>
> The vendor needs to be specified only if there’s a relevant change, for instance between PC and Apple. Most of the time it can be omitted (and Unknown) will be assumed, which sets the defaults for the specified architecture.
>
> The system name is generally the OS (linux, darwin), but could be special like the bare-metal `none`.
>
> Finally, the ABI option is something that will pick default CPU/FPU, define the specific behaviour of your code (PCS, extensions), and also choose the correct library calls, etc.

NdM : en première approche, on aurait pu penser que seul l'architecture était nécessaire = `arch+sub` (afin d'émettre des instructions binaires adaptées au CPU), mais en fait d'autres facteurs semblent nécessaires :

- `abi` = l'ABI est importante, p.ex. pour que le programme émis utilise les mêmes calling-conventions que les librairies qu'il va utiliser
- `sys` = ce point est moins clair, mais j'en déduis que le code machine émis dépend de l'OS et plus précisément du kernel (p.ex. que le code machine qui permet d'utiliser un syscall va changer selon l'OS considéré ?)
- `vendor` = ce point est encore moins clair, j'en déduis que pour un même CPU ARM, on pourrait avoir une différence de code machine selon que le processeur est dans un PC ou un MAC, mais ça me paraît très bizarre...

[Cette autre page](https://wiki.osdev.org/Target_Triplet) donne des explications complémentaires :

> Notice how the vendor field is mostly irrelevant and is usually 'pc' for 32-bit x86 systems or 'unknown' or 'none' for other systems.

Elle précise également que le triplet est `{CPU-family}{vendor}{OS}`, donc je précise un peu les infos de clang :

- la combinaison de `arch` et `sub` sert à définir le CPU
- la combinaison de `sys` et `abi` sert à définir l'OS

NdM : l'OS peut être `none` pour indiquer qu'on veut générer du code pour un bare-metal program.

> Target triplets are intended to be systematic unambiguous platform names [...]. They lets build systems understand exactly which system the code will run on and allows enabling platform-specific features automatically.

La **platform** désigne ce sur quoi tournera un programme, donc elle est identifiée de façon unique par le `target-triple`.

### A minimal rust kernel

https://os.phil-opp.com/minimal-rust-kernel/

> In this post, we create a minimal 64-bit Rust kernel for the x86 architecture. We build upon the freestanding Rust binary from the previous post to create a bootable disk image that prints something to the screen.

Le premier post avait permis de compiler un freestanding exe pour x86_64, qui ne faisait rien du tout. Avec ce post, notre exe va booter et afficher quelque chose.

> When you turn on a computer, it begins executing firmware code that is stored in motherboard ROM. This code performs a power-on self-test, detects available RAM, and pre-initializes the CPU and hardware. Afterwards, it looks for a bootable disk and starts booting the operating system kernel.

Notre programme ne sera donc pas le premier truc qui tournera : le firmware hardcodé passera d'abord ; il y a deux standards = BIOS et UEFI.

> Almost all x86 systems have support for BIOS booting, including newer UEFI-based machines that use an emulated BIOS.

Je comprends cette phrase comme "tous les CPU x86 peuvent exécuter un BIOS". Pourquoi c'est important ? La suite du post le dira, mais je suppose que c'est parce qu'on a des mesures à prendre pour permettre au BIOS de charger notre freestanding program ?

> But this wide compatibility is at the same time the biggest disadvantage of BIOS booting, because it means that the CPU is put into a 16-bit compatibility mode called real mode before booting so that archaic bootloaders from the 1980s would still work.
>
> But let’s start from the beginning:
>
> When you turn on a computer, it loads the BIOS from some special flash memory located on the motherboard. The BIOS runs self-test and initialization routines of the hardware, then it looks for bootable disks. If it finds one, control is transferred to its bootloader, which is a 512-byte portion of executable code stored at the disk’s beginning. Most bootloaders are larger than 512 bytes, so bootloaders are commonly split into a small first stage, which fits into 512 bytes, and a second stage, which is subsequently loaded by the first stage.
>
> The bootloader has to determine the location of the kernel image on the disk and load it into memory. It also needs to switch the CPU from the 16-bit real mode first to the 32-bit protected mode, and then to the 64-bit long mode, where 64-bit registers and the complete main memory are available. Its third job is to query certain information (such as a memory map) from the BIOS and pass it to the OS kernel.

^ description du travail d'un bootloader

> Writing a bootloader is a bit cumbersome as it requires assembly language and a lot of non insightful steps like “write this magic value to this processor register”. Therefore, we don’t cover bootloader creation in this post

^ Ce ne sera pas abordé.

> To avoid that every operating system implements its own bootloader, which is only compatible with a single OS, the Free Software Foundation created an open bootloader standard called Multiboot in 1995. The standard defines an interface between the bootloader and the operating system, so that any Multiboot-compliant bootloader can load any Multiboot-compliant operating system. The reference implementation is GNU GRUB, which is the most popular bootloader for Linux systems.

Grub est un bootloader qui n'est pas maqué avec linux : il est capable de loader plusieurs OSes.

> We will discuss the exact layout of the VGA buffer in the next post, where we write a first small driver for it. For printing “Hello World!”, we just need to know that the buffer is located at address 0xb8000 and that each character cell consists of an ASCII byte and a color byte.

^ Intéressant : cette adresse n'est donc sans doute pas mappée à de la ram. Où puis je trouver l'info : dans la doc x86_64 ?

[Cette page](https://en.m.wikipedia.org/wiki/VGA_text_mode) indique elle aussi :

> The VGA text buffer is located at physical memory address 0xB8000.



> The bootimage tool performs the following steps behind the scenes:
>
> - It compiles our kernel to an ELF file.
> - It compiles the bootloader dependency as a standalone executable.
> - It links the bytes of the kernel ELF file to the bootloader.
>
> When booted, the bootloader reads and parses the appended ELF file. It then maps the program segments to virtual addresses in the page tables, zeroes the .bss section, and sets up a stack. Finally, it reads the entry point address (our `_start` function) and jumps to it.

La suite du post est centrée sur comment faire tout ça en rust, c'est très intéressant, mais hors-scope de ce que je souhaite annoter = les principes bas-niveau des OS.

### VGA Text Mode

https://os.phil-opp.com/vga-text-mode/

Ce post est surtout tourné vers de l'implémentation rust (donc hors-scope de ce que je souhaite annoter = les principes bas-niveau des OS).

> The VGA text buffer is accessible via memory-mapped I/O to the address 0xb8000. This means that reads and writes to that address don’t access the RAM but directly access the text buffer on the VGA hardware. This means we can read and write it through normal memory operations to that address.

^ Ceci confirme ce que j'ai dit plus haut

### Testing

https://os.phil-opp.com/testing/

Je skimme car hors-scope de ce que je souhaite annoter = les principes bas-niveau des OS : ce post concerne le testing d'un freestanding exécutable en rust.

## Interrupts

### CPU Exceptions

https://os.phil-opp.com/cpu-exceptions/

> An exception signals that something is wrong with the current instruction. For example, the CPU issues an exception if the current instruction tries to divide by 0. When an exception occurs, the CPU interrupts its current work and immediately calls a specific exception handler function, depending on the exception type.

Il y a une petite liste de quelques exceptions de x86, et [un lien](https://wiki.osdev.org/Exceptions) qui contient une liste plus exhaustive.

> Exceptions are quite similar to function calls: The CPU jumps to the first instruction of the called function and executes it. Afterwards, the CPU jumps to the return address and continues the execution of the parent function.
>
> However, there is a major difference between exceptions and function calls: A function call is invoked voluntarily by a compiler-inserted call instruction, while an exception might occur at any instruction

Le post montre comment gérer des handlers d'interrupt en rust. Ici aussi, l'intérêt pour moi est de mieux comprendre des trucs bas niveau. Par exemple :

> Calling conventions specify the details of a function call. For example, they specify where function parameters are placed (e.g. in registers or on the stack) and how results are returned. On x86_64 Linux, the following rules apply for C functions (specified in the System V ABI):
>
> - the first six integer arguments are passed in registers rdi, rsi, rdx, rcx, r8, r9
> - additional arguments are passed on the stack
> - results are returned in rax and rdx
>
> Note that Rust does not follow the C ABI (in fact, there isn’t even a Rust ABI yet), so these rules apply only to functions declared as extern "C" fn.

^ quelques exemples de ce que l'ABI C spécifie, MAIS rust ne la suit pas !

> The calling convention divides the registers into two parts: preserved and scratch registers.
>
> The values of preserved registers must remain unchanged across function calls. So a called function (the “callee”) is only allowed to overwrite these registers if it restores their original values before returning. Therefore, these registers are called “callee-saved”. A common pattern is to save these registers to the stack at the function’s beginning and restore them just before returning.
>
> In contrast, a called function is allowed to overwrite scratch registers without restrictions. If the caller wants to preserve the value of a scratch register across a function call, it needs to backup and restore it before the function call (e.g., by pushing it to the stack). So the scratch registers are caller-saved.
>
> On x86_64, the C calling convention specifies the following preserved and scratch registers:

^ en fonction de la calling-convention, certains registres doivent être sauvegardés par le caller, et d'autres par le callee.

> In contrast to function calls, exceptions can occur on any instruction. In most cases, we don’t even know at compile time if the generated code will cause an exception. For example, the compiler can’t know if an instruction causes a stack overflow or a page fault.
>
> Since we don’t know when an exception occurs, we can’t backup any registers before. This means we can’t use a calling convention that relies on caller-saved registers for exception handlers. Instead, we need a calling convention that preserves all registers. The x86-interrupt calling convention is such a calling convention, so it guarantees that all register values are restored to their original values on function return.

^ pour la gestion des exceptions, c'est un peu particulier.

Il y a tout un paragraphe sur la gestion des interruptions par le CPU (et même des liens intéressants pour creuser le sujet).


### Double Faults

https://os.phil-opp.com/double-fault-exceptions/

> This post explores the double fault exception in detail, which occurs when the CPU fails to invoke an exception handler. By handling this exception, we avoid fatal triple faults that cause a system reset.

Des trucs intéressants dans ce post, mais qui rentrent un peu trop dans les détails pour mes attentes du moment : je n'annote rien.

### Hardware Interrupts

https://os.phil-opp.com/hardware-interrupts/

> Interrupts provide a way to notify the CPU from attached hardware devices. So instead of letting the kernel periodically check the keyboard for new characters (a process called polling), the keyboard can notify the kernel of each keypress. This is much more efficient because the kernel only needs to act when something happened. It also allows faster reaction times since the kernel can react immediately and not only at the next poll.

Résumé :

> This post explained how to enable and handle external interrupts. We learned about the 8259 PIC and its primary/secondary layout, the remapping of the interrupt numbers, and the “end of interrupt” signal. We implemented handlers for the hardware timer and the keyboard and learned about the hlt instruction, which halts the CPU until the next interrupt.

Puis :

> Now we are able to interact with our kernel and have some fundamental building blocks for creating a small shell or simple games.

Ici aussi, des trucs intéressants dans ce post, mais qui rentrent un peu trop dans les détails pour mes attentes du moment : je n'annote rien.

## Memory management

### Introduction to paging

https://os.phil-opp.com/paging-introduction/

> This post introduces paging, a very common memory management scheme that we will also use for our operating system. It explains why memory isolation is needed, how segmentation works, what virtual memory is, and how paging solves memory fragmentation issues. It also explores the layout of multilevel page tables on the x86_64 architecture.

Ah, on rentre dans un sujet qui m'a beaucoup intéressé il y a quelques mois !

> One main task of an operating system is to isolate programs from each other. [...]
>
> To achieve this goal, operating systems utilize hardware functionality to ensure that memory areas of one process are not accessible by other processes

^ le hardware joue un rôle : ça n'est pas qu'une question d'implémentation software.

> On x86, the hardware supports two different approaches to memory protection: segmentation and paging.

^ il n'y a pas que le paging dans la vie ; la **segmentation** est en quelque sorte le précurseur du paging.

> The situation back then was that CPUs only used 16-bit addresses, which limited the amount of addressable memory to 64 KiB. To make more than these 64 KiB accessible, additional segment registers were introduced, each containing an offset address. The CPU automatically added this offset on each memory access, so that up to 1 MiB of memory was accessible.
>
> The segment register is chosen automatically by the CPU depending on the kind of memory access: For fetching instructions, the code segment CS is used, and for stack operations (push/pop), the stack segment SS is used.

^ avec la segmentation, la plage adressable est shiftée d'un offset contenu dans un registre : l'adresse finale est l'adresse demandée + l'offset.

> The idea behind virtual memory is to abstract away the memory addresses from the underlying physical storage device. Instead of directly accessing the storage device, a translation step is performed first. For segmentation, the translation step is to add the offset address of the active segment. Imagine a program accessing memory address 0x1234000 in a segment with an offset of 0x1111000: The address that is really accessed is 0x2345000.
>
> To differentiate the two address types, addresses before the translation are called virtual, and addresses after the translation are called physical. One important difference between these two kinds of addresses is that physical addresses are unique and always refer to the same distinct memory location. Virtual addresses, on the other hand, depend on the translation function. It is entirely possible that two different virtual addresses refer to the same physical address. Also, identical virtual addresses can refer to different physical addresses when they use different translation functions.

^ On peut voir la segmentation comme une forme de mémoire virtuelle, où l'adresse physique utilisée est décorrélée de l'adresse virtuelle vue par le programme, et où la traduction de l'un à l'autre se fait avec un MMU basique qui se contente d'offseter l'adresse.

Du coup, si chaque programme utilise des offsets différents, deux instances différents d'un même programme ont chacune leur propres adresses physiques qui ne se marchent pas sur les pieds, alors même que le programme "voit" les mêmes adresses virtuelles.

Dans ce cas, pourquoi passer au paging plutôt que de rester avec la segmentation ? Parce qu'en groupant toute la plage mémoire d'un programme en un seul bloc (conséquence du fait que la traduction est simpliste = un simple offset), on manque de granularité, ce qui gâche de la mémoire : c'est la **fragmentation**.

L'évolution naturelle est de découper la plage mémoire en plusieurs morceaux (les _memory pages_, qui se traduisent en _memory frames_), pour pouvoir orgnasier la mémoire physique comme on le souhaite, indépendamment de l'organisation de la mémoire virtuelle : on en vient naturellement au **paging** (et la page a des illustrations très claires du sujet).

À noter qu'il reste toujours une forme de fragmentation = on a la granularité de la page, et si un programme a besoin d'une mémoire égale à "une page + un octet", alors il n'aura pas d'autre choix que de consommer deux pages (et la deuxième page sera presqu'intégralement gâchée). Ce problème est nettement moins grave que la fragmentation apportée par la segmentation.

> We saw that each of the potentially millions of pages is individually mapped to a frame. This mapping information needs to be stored somewhere. Segmentation uses an individual segment selector register for each active memory region, which is not possible for paging since there are way more pages than registers. Instead, paging uses a table structure called page table to store the mapping information.

^ la **page table** est le mapping entre une _memory page_ et sa _memory frame_ correspondante.

> We see that each program instance has its own page table. A pointer to the currently active table is stored in a special CPU register. On x86, this register is called CR3. It is the job of the operating system to load this register with the pointer to the correct page table before running each program instance.
>
> On each memory access, the CPU reads the table pointer from the register and looks up the mapped frame for the accessed page in the table. This is entirely done in hardware and completely invisible to the running program. To speed up the translation process, many CPU architectures have a special cache that remembers the results of the last translations.

^ la gestion de la mémoire est donc un travail conjoint du hardware (qui propose naturellement de quoi abstraire la mémoire physique à ses programmes) et de l'OS qui gère la mémoire et la propose à ses programmes. Un exemple d'un tel travail conjoint est donné plus bas :

> The accessed and dirty flags are automatically set by the CPU when a read or write to the page occurs. This information can be leveraged by the operating system, e.g., to decide which pages to swap out or whether the page contents have been modified since the last save to disk.

Il y a une intéressante explication de la _multilevel page table_ (avec un exemple concret), mais qui rentre un poil trop dans les détails pour que je l'annote :

> The x86_64 architecture uses a 4-level page table and a page size of 4 KiB.

Et au sujet du TLB :

> A 4-level page table makes the translation of virtual addresses expensive because each translation requires four memory accesses. To improve performance, the x86_64 architecture caches the last few translations in the so-called translation lookaside buffer (TLB). 

Ah tiens, d'ailleurs, la gestion du TLB doit être faite "manuellement" par l'OS / le programme :

> Unlike the other CPU caches, the TLB is not fully transparent and does not update or remove translations when the contents of page tables change. This means that the kernel must manually update the TLB whenever it modifies a page table. To do this, there is a special CPU instruction called invlpg (“invalidate page”) that removes the translation for the specified page from the TLB, so that it is loaded again from the page table on the next access. The TLB can also be flushed completely by reloading the CR3 register, which simulates an address space switch.

Ce qui ouvre la possibilité d'erreurs humaines et donc de bugs :

> It is important to remember to flush the TLB on each page table modification because otherwise, the CPU might keep using the old translation, which can lead to non-deterministic bugs that are very hard to debug.

(...)

> One thing that we did not mention yet: Our kernel already runs on paging. The bootloader that we added in the “A minimal Rust Kernel” post has already set up a 4-level paging hierarchy that maps every page of our kernel to a physical frame. The bootloader does this because paging is mandatory in 64-bit mode on x86_64.
>
> This means that every memory address that we used in our kernel was a virtual address. Accessing the VGA buffer at address 0xb8000 only worked because the bootloader identity mapped that memory page, which means that it mapped the virtual page 0xb8000 to the physical frame 0xb8000.

^ ceci est intéressant : en x86_64, pas le choix : on est forcés d'utiliser du paging (le paging n'est donc pas une fonctionnalité de l'OS, mais du CPU ; même si l'OS fonctionne main dans la main avec le CPU)

> Paging makes our kernel already relatively safe, since every memory access that is out of bounds causes a page fault exception instead of writing to random physical memory.

^ un autre intérêt du paging (en plus de permettre à chaque process d'avoir son propre espace mémoire, sans introduire de fragmentation) = sécurité des accès mémoire out-of-bound.

La suite du post montre des exemples concrets de page-fault (en lecture et/ou en écriture), et l'accès aux registres contenant les détails des page-faults, et mentionne une limitation que je ne détaille pas (et qui sera addressée dans la page suivante).

### Paging Implementation

https://os.phil-opp.com/paging-implementation/

Le résumé du post :

> This post shows how to implement paging support in our kernel. It first explores different techniques to make the physical page table frames accessible to the kernel and discusses their respective advantages and drawbacks. It then implements an address translation function and a function to create a new mapping.
>
> The previous post gave an introduction to the concept of paging. It motivated paging by comparing it with segmentation, explained how paging and page tables work, and then introduced the 4-level page table design of x86_64. We found out that the bootloader already set up a page table hierarchy for our kernel, which means that our kernel already runs on virtual addresses. This improves safety since illegal memory accesses cause page fault exceptions instead of modifying arbitrary physical memory.
>
> The post ended with the problem that we can’t access the page tables from our kernel because they are stored in physical memory and our kernel already runs on virtual addresses. This post explores different approaches to making the page table frames accessible to our kernel.

En gros, c'est pas évident d'accéder à la page-table, car ce mapping vers des addresses physiques nous est abstrait.

> The problem for us is that we can’t directly access physical addresses from our kernel since our kernel also runs on top of virtual addresses.

^ Voilà.

> This way, the physical addresses of page tables are also valid virtual addresses so that we can easily access the page tables of all levels starting from the CR3 register.

Je me contente de skimmer, mais si je comprends bien, l'idée est de trouver un moyen d'accéder à la RAM qui stocke la page-table, c'est ce que le post explore, et ça nécessite aussi un travail sur le bootloader.

Du coup, je mets le résumé :

> In this post we learned about different techniques to access the physical frames of page tables, including identity mapping, mapping of the complete physical memory, temporary mapping, and recursive page tables. We chose to map the complete physical memory since it’s simple, portable, and powerful.
>
> We can’t map the physical memory from our kernel without page table access, so we need support from the bootloader. [...]

### Heap Allocation

https://os.phil-opp.com/heap-allocation/

> This post adds support for heap allocation to our kernel. First, it gives an introduction to dynamic memory and shows how the borrow checker prevents common allocation errors. It then implements the basic allocation interface of Rust, creates a heap memory region, and sets up an allocator crate

L'allocation dynamique (= la gestion d'un heap) est une feature de l'OS : rien n'oblige à splitter la mémoire virtuelle gérée par le processeur en deux zones (stack et heap).

> Static variables are stored at a fixed memory location separate from the stack. This memory location is assigned at compile time by the linker and encoded in the executable. Statics live for the complete runtime of the program

^ définition des static variable

> Local and static variables (...) both have their limitations: \
> Local variables only live until the end of the surrounding function or block. (...) \
> Static variables always live for the complete runtime of the program, so there is no way to reclaim and reuse their memory when they’re no longer needed.  (...) \
> Another limitation of local and static variables is that they have a fixed size. So they can’t store a collection that dynamically grows when more elements are added. \
> To circumvent these drawbacks, programming languages often support a third memory region for storing variables called the heap.

Plus généralement que ces quelques extraits, l'intro du post présente l'allocation dynamique, ses bénéfices et inconvénients.

> Rust takes a different approach to the problem: It uses a concept called ownership that is able to check the correctness of dynamic memory operations at compile time.

^ c'est bien l'ownership le concept important de rust

> Instead of letting the programmer manually call allocate and deallocate, the Rust standard library provides abstraction types that call these functions implicitly. The most important type is Box, which is an abstraction for a heap-allocated value.

^ `Box` semble être un mix des smart-pointers avec le concept d'ownership.

> Taking a reference to a value is called borrowing the value since it’s similar to a borrow in real life: You have temporary access to an object but need to return it sometime, and you must not destroy it. By checking that all borrows end before an object is destroyed, the Rust compiler can guarantee that no use-after-free situation can occur.

^ une autre explication en quelques mots expliquant que les références ne peuvent jamais être dangling en rust.

> As a basic rule, dynamic memory is required for variables that have a dynamic lifetime or a variable size.

^ raisons de nécessiter de l'allocation dynamique

> The most important type with a dynamic lifetime is `Rc`, which counts the references to its wrapped value and deallocates it after all references have gone out of scope.

^ `Rc` est un équivalent des `shared_ptr`, qui ne sait quand il doit desallouer que dynamiquement, en fonction de qui a encore des références sur la ressource.

> Examples for types with a variable size are Vec, String, and other collection types that dynamically grow when more elements are added.

^ les tableaux dynamiques nécessitent également une allocation dynamique, vu qu'ils ne connaissent la taille qui leur est nécessaire qu'au runtime.

Il faut définir nos propres fonctions allocate / desallocate , et indiquer à rust de les utiliser (via l'atteibute `#[global_allocator]`)

Probablement que l'allocator par défaut utilise celui de la libc.

> Before we can create a proper allocator, we first need to create a heap memory region from which the allocator can allocate memory. To do this, we need to define a virtual memory range for the heap region and then map this region to physical frames.

^ quelque part, ce n'est pas l'allocation dynamique en elle même qui consomme de la mémoire virtuelle : c'est le bouzin sous jacent (l'implémentation de l'allocateur, en quelque sorte le "moteur d'allocation") qui le fait, et l'allocation dynamique se contente de renvoyer un morceau de cette RAM consommée (et derrière, la RAM physique consommée dépend de ce qui backe la RAM virtuelle).

Du coup, sa première fonction mappe un gros chunk de RAM à des adresses virtuelles, et cette plage d'adresses (100 kio dans son exemple) constitue le heap. Au moment où il initialise le heap de la sorte, la RAM est consommée alors qu'aucune allocation dynamique n'a encore eu lieu :

> We now have a mapped heap memory region that is ready to be used.

Dans la suite du post, il n'implemente pas encore un allocator :

> Since implementing an allocator is somewhat complex, we start by using an external allocator crate. We will learn how to implement our own allocator in the next post.

### Allocator Designs

https://os.phil-opp.com/allocator-designs/

> The responsibility of an allocator is to manage the available heap memory. It needs to return unused memory on alloc calls and keep track of memory freed by dealloc so that it can be reused again. Most importantly, it must never hand out memory that is already in use somewhere else because this would cause undefined behavior.
>
> Apart from correctness, there are many secondary design goals. For example, the allocator should effectively utilize the available memory and keep fragmentation low. Furthermore, it should work well for concurrent applications and scale to any number of processors. For maximal performance, it could even optimize the memory layout with respect to the CPU caches to improve cache locality and avoid false sharing.

^ les objectifs d'un allocator

Derrière, il décrit plusieurs stratégies d'allocation, toutes liées au problème de savoir comment gérer la connaissance de quels morceaux du heap sont disponibles, et quels morceaux du heap sont occupés.

**Bump allocator** = on alloue les chunks les uns à la suite des autres, et on ne désalloue jamais, sauf si tous les chunks ont été libérés. Pour cela, un compteur garde trace du nombre de chunks actifs (et ce n'est que quand ce compteur tombe à zéro qu'on rend la RAM).

**Linked-list allocator** : on maintient une linked-list de chaque chunk libre (qui n'est donc pas vraiment libre et rendu à l'os : mais qui contient plutôt un node de la liste, chaque node a un pointeur vers le chunk suivant). La structure est une **free-list** = une linked liste des zones du heap qui sont free. Les allocators qui utilisent une free-list s'appellent des **pool allocators**. L'implémentation est bien détaillée, mais je la skimme. Le souci de ça, c'est qu'il faut traverser la liste pour trouver le premier emplacement libre suffisamment grand pour accueillir l'allocation demandée, ce qui peut être très long.

**Fixed Size Block Allocator** : seuls des blocs de taille fixes (16, 64, et 512, par exemple) sont allouables. Au lieu d'avoir une unique free-list, on a une free-list par taille de bloc : chaque liste ne contient que des blocs d'une même taille. Chaque allocation est arrondie à la taille de bloc supérieure, ce qui gâche de la mémoire (jusqu'à la moitié de la RAM !), mais on y gagne de meilleures performances que les pool allocators, car on se contente d'utiliser le head de la liste approprié pour répondre à une demande d'allocation, et on n'a donc pas à traverser la liste.

Une stratégie supplémentaires consiste à mixer les stratégies , e.g. utiliser le Fixed Size Block Allocator en général, mais utiliser un Pool Allocator pour les rares cas où on doit allouer des gros blocs.

Il continue son bestiaire des allocators :

**Slab Allocator** :

> The idea behind a slab allocator is to use block sizes that directly correspond to selected types in the kernel. This way, allocations of those types fit a block size exactly and no memory is wasted.

**Buddy Allocator**

> Instead of using a linked list to manage freed blocks, the buddy allocator design uses a binary tree data structure together with power-of-2 block sizes. When a new block of a certain size is required, it splits a larger sized block into two halves, thereby creating two child nodes in the tree. Whenever a block is freed again, its neighbor block in the tree is analyzed. If the neighbor is also free, the two blocks are joined back together to form a block of twice the size.
>
> The advantage of this merge process is that external fragmentation is reduced so that small freed blocks can be reused for a large allocation. It also does not use a fallback allocator, so the performance is more predictable. The biggest drawback is that only power-of-2 block sizes are possible, which might result in a large amount of wasted memory due to internal fragmentation.

Son summary est tellement bien que le cite tel quel :

> This post gave an overview of different allocator designs. We learned how to implement a basic bump allocator, which hands out memory linearly by increasing a single next pointer. While bump allocation is very fast, it can only reuse memory after all allocations have been freed. For this reason, it is rarely used as a global allocator.
>
> Next, we created a linked list allocator that uses the freed memory blocks itself to create a linked list, the so-called free list. This list makes it possible to store an arbitrary number of freed blocks of different sizes. While no memory waste occurs, the approach suffers from poor performance because an allocation request might require a complete traversal of the list. Our implementation also suffers from external fragmentation because it does not merge adjacent freed blocks back together.
>
> To fix the performance problems of the linked list approach, we created a fixed-size block allocator that predefines a fixed set of block sizes. For each block size, a separate free list exists so that allocations and deallocations only need to insert/pop at the front of the list and are thus very fast. Since each allocation is rounded up to the next larger block size, some memory is wasted due to internal fragmentation.
>
> There are many more allocator designs with different tradeoffs. Slab allocation works well to optimize the allocation of common fixed-size structures, but is not applicable in all situations. Buddy allocation uses a binary tree to merge freed blocks back together, but wastes a large amount of memory because it only supports power-of-2 block sizes. It’s also important to remember that each kernel implementation has a unique workload, so there is no “best” allocator design that fits all cases.

## Multitasking

### Async/Await

https://os.phil-opp.com/async-await/

> In this post, we explore cooperative multitasking and the async/await feature of Rust. We take a detailed look at how async/await works in Rust, including the design of the Future trait, the state machine transformation, and pinning

^ vue la qualité des posts précédents, je vais au moins skimmer, mais pas sûr que ça soit très lié aux concepts que j'essayais de creuser.

Chaque thread a sa propre callstack, de sorte que seuls les contenus des registres doivent être sauvegardés/restaurés lors des context-switch.

> The main advantage of preemptive multitasking is that the operating system can fully control the allowed execution time of a task. This way, it can guarantee that each task gets a fair share of the CPU time, without the need to trust the tasks to cooperate (...)
>
> Instead of forcibly pausing running tasks at arbitrary points in time, cooperative multitasking lets each task run until it voluntarily gives up control of the CPU. This allows tasks to pause themselves at convenient points in time, for example, when they need to wait for an I/O operation anyway. (...)
>
> The drawback of cooperative multitasking is that an uncooperative task can potentially run for an unlimited amount of time. Thus, a malicious or buggy task can prevent other tasks from running and slow down or even block the whole system

^ preemptive vs cooperative multitasking

Il donne des explication sur les `future` et `async/await` dans rust.

> Now that we understand how cooperative multitasking based on futures and async/await works in Rust, it’s time to add support for it to our kernel.

^ le reste du post explique l'implémentation de ça dans le toy kernel qu'il développe.

