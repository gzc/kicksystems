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