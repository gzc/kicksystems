# 100 Questions in OS

##1. 如何直接跳出深层递归而不是一层一层跳出？

1. throw exception，try catch
2. 把递归函数放到一个独立线程执行，在主线程做 condition wait，递归结束时notify下，然后直接退出线程。
3. C语言的setjmp + longjmp, setjump保存context, longjmp恢复context, try catch其实做的就是这个事情.


##2. Can the OS be interrupted? 
Sure.

##3. What creates the first process?
init is the first process, hard encoded in OS. Try run **pstree** in your linux machine.

##4. What state do you think a process is in most of the time? Running, Ready or Waitting?
Ready.

##5. How many processes can a system support?
In Linux it is 65536
    
    cat /proc/sys/kernel/pid_max
    
##6. What does it mean for exec to return?
Does not return if OK, returns −1 on error

##7. fork() can return an error. Why might this happen?
When running out of memory or exceed processes a system can support.

##8. What happens if you run “exec csh” in your shell? What happens if you run “exec ls” in your shell? Try it.