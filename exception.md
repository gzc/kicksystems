# Exception Control Flow

##Exception
系统中为每种可能的异常都分配了一个唯一的非整数的异常号(exception number),其中一些号码是由处理器的设计者分配的，如divide by zero, page faults, memory access violations, break- points, and arithmetic overflows；另外一些是由os kernel的设计者分配的，如system calls and signals from external I/O devices.

当系统启动时，操作系统会分配和初始化一个jump table,我们称为exception table. table的kth entry(k is exception number)包含address of the handler for exception k. Exception table的地址存放在exception table base register.

An exception is akin to a procedure call, but with some important differences :
1. 如果从用户程序转移到内核, 那么用户程序的状态会被压到**内核栈**中.
2. 异常处理程序运行在**kernel mode**

##不同的异常

###interrupt
中断是异步发生的,是来自处理外部的I/O设备信号的结果.

###trap & system call
trap是有意的异常(active),是执行一条指令的结果, 主动触发. trap最重要的用途是system call.

read, fork, execve, exit都需要system call, 这些函数是os提供的服务, 必须运行在kernel mode.

###fault
一个经典的fault是page fault exception. 因此fault可能返回到当前指令.

###abort
abort是不可恢复的致命错误造成的结果，通常是一些硬件错误.


##Linux/IA32系统中的异常
| 异常号   |      描述      |  类别 |
|----------|:-------------:|------:|
| 0 |  Divide error | fault |
| 13 |   General protection fault   |   fault |
| 14 | Page fault |  fault |
| 18 | Machine check |  abort |
| 32-127 | OS-defined exceptions |  Interrupt or trap |
| 128 (0x80) | System call |  trap |
| 129–255 | OS-defined exceptions | Interrupt or trap |


##Linux/IA32系统中的system call
linux提供上百种系统调用. 每个系统调用都有一个唯一的整数号,对应于一个到内核中跳转表的偏移量.

| Number   |    Name      |  Description | Number   |    Name      |  Description |
|:-------:|:--------:|: ------:|----------|:-------------:|------:|
| 1 |  exit | Terminate process | 27 |  alarm | Set signal delivery alarm clock |
| 2 |  fork | Create new process | 29 |  pause | Suspend process until signal arrives |
| 3 |  read | Read file | 37 |  kill | Send signal to another process |
| 4 |  write | Write file | 48 |  signal | Install signal handler |

System calls are provided on IA32 systems via a trapping instruction called int n, where n can be the index of any of the 256 entries in the IA32 exception table. Historically, system calls are provided through exception 128 (0x80).

system call的参数是通过register而不是stack进行传递的. %eax包含系统调用号, %ebx, %ecx, %edx, %esi, %edi, %ebp包含最多六个任意的参数. stack pointer %esp不能使用, 因为进入kernel mode, kernel会overwrite掉.

    .section .data
    string:
    .ascii "hello, world\n"
    string_end:
    .equ len, string_end - string
    .section .text
    .globl main
    main:
    \\First, call write(1, "hello, world\n", 13)
    movl $4, %eax           \\System call number 4
    movl $1, %ebx           \\stdout has descriptor 1
    movl $string, %ecx      \\Hello world string
    movl $len, %edx         \\String length
    int $0x80               \\System call code
    
    \\Next, call exit(0)
    movl $1, %eax           \\System call number 0
    movl $0, %ebx           \\Argument is 0
    int $0x80               \\System call code