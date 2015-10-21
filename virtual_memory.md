# Virtual Memory

**MMU** : Performing the translation of virtual memory addresses to physical addresses. It is usually implemented as part of the central processing unit (CPU).

**TLB** : A translation lookaside buffer (TLB) is a memory cache that stores recent translations of virtual memory to physical addresses for faster retrieval.

**Page Fault** : A page fault (sometimes called #PF or PF) is a type of interrupt, called trap, raised by computer hardware when a running program accesses a memory page that is mapped into the virtual address space, but not actually loaded into main memory.

**Multi-Level Page Tables : ** Imaging by yourself.

**Core i7 ** : Although 64 bits, it actually utilize only 48 bits. 4-level page tables.(9+9+9+9+12).

**CR3 ** : Used when virtual addressing is enabled, hence when the PG bit is set in CR0. CR3 enables the processor to translate linear addresses into physical addresses by locating the page directory and page tables for the current task. Typically, the upper 20 bits of CR3 become the page directory base register (PDBR), which stores the physical address of the first page directory entry.

**COW ** : Copy-on-write (sometimes referred to as "COW"), sometimes referred to as implicit sharing, is an optimization strategy used in computer programming. Like <code>fork()</code>.