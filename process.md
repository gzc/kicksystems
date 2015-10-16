# Process

1. **Private Address Space**
2. **user mode & kernel mode**
3. **context switch**

context switch will make cache cold.

user mode and kernel mode can switch by setting EFLAGS bits.

当system call发生错误时,典型的会返回-1，并设置全局整数变量errno来表示出了什么错误.

###Obtaining Process IDs

    #include <sys/types.h> 
    #include <unistd.h>
    
    pid_t getpid(void); 
    pid_t getppid(void);

                        Returns: PID of either the caller or the parent
                        
###Creating and Terminating Processes
    #include <stdlib.h> 
    
    void exit(int status) 
                        This function does not return
<br />
    #include <sys/types.h> 
    #include <unistd.h>
    
    pid_t fork(void)
                        Return 0 to child, return child pid to parent
                        
新创建的child process几乎和parent process一模一样, child process得到与parent process虚拟地址空间相同的（但是独立的）一份拷贝，包括文本,数据,bss段,堆以及用户栈,还有文件描述符. 

注意, fork是COW(copy on write)的.

###Reaping Child Processes
A terminated process that has not yet been reaped is called a zombie.

If the parent process terminates without reaping its zombie children, the kernel arranges for the init process to reap them. The init process has a PID of 1 and is created by the kernel during system initialization. 

A process waits for its children to terminate or stop by calling the **waitpid** function.

