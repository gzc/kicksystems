# Thread

每个thread都有自己的thread context, 比process的context要小很多. Thread context包括TID(thread ID), stack, stack pointer, PC, general-purpose registers and condition code. 所有的thread共享宿主process的virtual address space.

Thread由kernel调度, 并且kernel知道有这么一个thread(通过PID). 因为一个process下的thread都跑在其virtual address space, 这些threads就共享了code, global data, heap, shared libraries and open files(file descriptors).


