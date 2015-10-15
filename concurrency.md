# Concurrency

Application-level concurrency is useful in other ways as well:

1. Accessing slow I/O devices : When an application is waiting for data to arrive from a slow I/O device such as a disk, the kernel keeps the CPU busy by running other processes.
2. Interacting with humans : For example, mouse click.
3. Reducing latency by deferring work.
4. Servicing multiple network clients.
5. Computing in parallel on multi-core machines.

Modern operating systems provide three basic approaches for building concurrent programs:  

1. Processes. IPC for communication. For example, shared memory, pipe, file, Message queue, signal, socket.
2. I/O multiplexing.
3. Threads.

##### Concurrent Programming with Processes


Pro : It is impossible for one process to accidentally overwrite the virtual memory of another process, which eliminates a lot of confusing failures—an obvious advantage.

Con : Separate address spaces make it more difficult for pro- cesses to share state information. To share information, they must use explicit IPC (interprocess communications) mechanisms. (See Aside.) Another disadvantage of process-based designs is that they tend to be slower because the overhead for process control and IPC is high.


##### Concurrent Programming with I/O Multiplexing

## Concurrent Programming with Thread
