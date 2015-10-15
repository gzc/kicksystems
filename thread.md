# Thread

##intro
每个thread都有自己的thread context, 比process的context要小很多. Thread context包括TID(thread ID), stack, stack pointer, PC, general-purpose registers and condition code. 所有的thread共享宿主process的virtual address space.

Thread由kernel调度, 并且kernel知道有这么一个thread(通过PID). 因为一个process下的thread都跑在其virtual address space, 这些threads就共享了code, global data, heap, shared libraries and open files(file descriptors).

Thread的context switch要比process的context switch快很多. 跟process有父子关系不同, threads之间地位平等，可以互相kill,wait,也能互相读写shared data.

##Posix Threads
Posix threads (Pthreads) is a standard interface for manipulating threads from C programs. It was adopted in 1995 and is available on most Unix systems. Pthreads defines about 60 functions that allow programs to create, kill, and reap threads, to share data safely with peer threads, and to notify peers about changes in the system state.

