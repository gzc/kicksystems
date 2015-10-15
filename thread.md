# Thread

每个thread都有自己的thread context, 比process的context要小. Thread context包括TID(thread ID), stack, stack pointer, PC, general-purpose registers and condition code. 所有的thread共享宿主process的virtual address space.
