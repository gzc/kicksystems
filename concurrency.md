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

##### Concurrent Programming with Thread

##Synchronizing Threads with Semaphores
Threads的模型中, 会出现race. 采用P,V操作来解决.

下面看一些经典问题.

###Readers/Writers
1. An object is shared among several threads
2. Some threads only read the object, others only write it
3. We can allow multiple readers but only one writer

Use three variables
1. int readcount – number of threads reading object
2. Semaphore mutex – control access to readcount
3. Semaphore w_or_r – exclusive writing or reading
 
    
    // number of readers
    int readcount = 0;
    // mutual exclusion to readcount
    Semaphore mutex = 1;
    // exclusive writer or reader
    Semaphore w_or_r = 1;

    writer {
        wait(w_or_r); // lock out readers
        Write;
        signal(w_or_r); // up for grabs
    }
    
    reader {
        wait(mutex); // lock readcount
        readcount += 1; // one more reader
        if (readcount == 1)
            wait(w_or_r); // synch w/ writers
        signal(mutex); // unlock readcount
        Read;
        wait(mutex); // lock readcount
        readcount -= 1; // one less reader
        if (readcount == 0)
            signal(w_or_r); // up for grabs
        signal(mutex); // unlock readcount}
    }
    
###Bounded Buffer
    Semaphore mutex = 1; // mutual exclusion to shared set of buffers
    Semaphore empty = N; // count of empty buffers (all empty to     start)
    Semaphore full = 0; // count of full buffers (none full to start)
    
    producer {
        while (1) {
            Produce new resource;
            wait(empty); // wait for empty buffer
            wait(mutex); // lock buffer list
            Add resource to an empty buffer;
            signal(mutex); // unlock buffer list
            signal(full); // note a full buffer
        }
    }
    
    consumer {
        while (1) {
            wait(full); // wait for a full buffer
            wait(mutex); // lock buffer list
            Remove resource from a full buffer;
            signal(mutex); // unlock buffer list
            signal(empty); // note an empty buffer
            Consume resource;
        }
    }
  
###Related Problems
####Thread Safety
1. Functions that do not protect shared variables.
2. Functions that keep state across multiple invocations.(rand函数是threadd unsafe的)
3. Functions that return a pointer to a static variable.
4. Functions that call thread-unsafe functions.

####Reentrancy
可重入函数，是thread safe的，因为这种函数不会引入共享数据

####Race

####Deadlock&Livelock
**DEADLOCK** Deadlock is a condition in which a task waits indefinitely for conditions that can never be satisfied - task claims exclusive control over shared resources - task holds resources while waiting for other resources to be released - tasks cannot be forced to relinguish resources - a circular waiting condition exists

**LIVELOCK** Livelock conditions can arise when two or more tasks depend on and use the some resource causing a circular dependency condition where those tasks continue running forever, thus blocking all lower priority level tasks from running (these lower priority tasks experience a condition called starvation)