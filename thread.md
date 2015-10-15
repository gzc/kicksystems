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
                            
如果thread调用**exit()**,那么会terminate该process和与其关联的所有threads. 也可以调用**pthread_ cancel** terminal 当前进程.

    #include <pthread.h>
    int pthread_cancel(pthread_t tid);

                            Returns: 0 if OK, nonzero on error
                            
###Reaping Terminated Threads
Threads wait for other threads to terminate by calling the **pthread_join** function.
    #include <pthread.h>
    int pthread_join(pthread_t tid, void **thread_return);

                            Returns: 0 if OK, nonzero on error
                            
调用该函数的时候会堵塞，直到tid那个thread返回，然后回收已终止thread的所有存储器资源. **和wait()函数不同, pthread_join只能等待一个指定tid的thread, 而不能等待所有的.**

###Detaching Threads
一个没有被detach的thread可以被杀死. 而detached的thread不能够被其他thread kill, 它的资源在终止时由系统自动释放.

    #include <pthread.h>
    int pthread_detach(pthread_t tid);
            
                        Returns: 0 if OK, nonzero on error
                        
detach thread是一个很好的操作. 因为在web服务器中，每次有一个请求过来，我们不可能去调用**pthread_join**来显示回收资源. 最好的办法就是detach thread, 让系统自动回收资源.

