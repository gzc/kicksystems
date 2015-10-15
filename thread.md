# Thread

##intro
每个thread都有自己的thread context, 比process的context要小很多. Thread context包括TID(thread ID), stack, stack pointer, PC, general-purpose registers and condition code. 所有的thread共享宿主process的virtual address space.

Thread由kernel调度, 并且kernel知道有这么一个thread(通过PID). 因为一个process下的thread都跑在其virtual address space, 这些threads就共享了code, global data, heap, shared libraries and open files(file descriptors).

Thread的context switch要比process的context switch快很多. 跟process有父子关系不同, threads之间地位平等，可以互相kill,wait,也能互相读写shared data.

##Posix Threads
Posix threads (Pthreads) is a standard interface for manipulating threads from C programs. It was adopted in 1995 and is available on most Unix systems. Pthreads defines about 60 functions that allow programs to create, kill, and reap threads, to share data safely with peer threads, and to notify peers about changes in the system state.

sample code:

    void *thread(void *vargp);
    int main() {
        pthread_t tid;
        Pthread_create(&tid, NULL, thread, NULL); \\create a thread and run function thread
        Pthread_join(tid, NULL); \\wait for peer thread terminate
        exit(0);
    }
    void *thread(void *vargp) /* Thread routine */
    {
        printf("Hello, world!\n");
        return NULL;
    }

###Creating Threads
    #include <pthread.h>
    typedef void *(func)(void *);
    int pthread_create(pthread_t *tid, pthread_attr_t *attr, func *f, void *arg);
    
                                                Returns: 0 if OK, nonzero on error
<br />

    #include <pthread.h> 
    pthread_t pthread_self(void);
    
                                                Returns: thread ID of caller

###Terminating Threads
The thread terminates explicitly by calling the **pthread_exit** function. If the main thread calls pthread_exit, it waits for all other peer threads to terminate, and then terminates the main thread and the entire process with a return value of thread_return.

    #include <pthread.h>
    void pthread_exit(void *thread_return);

                            Returns: 0 if OK, nonzero on error
                            
如果thread调用**exit()**,那么会terminate该process和与其关联的所有threads.