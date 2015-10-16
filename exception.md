# Exception Control Flow

##Exception
系统中为每种可能的异常都分配了一个唯一的非整数的异常号(exception number),其中一些号码是由处理器的设计者分配的，如divide by zero, page faults, memory access violations, break- points, and arithmetic overflows；另外一些是由os kernel的设计者分配的，如system calls and signals from external I/O devices.

当系统启动时，操作系统会分配和初始化一个jump table,我们称为exception table. table的kth entry(k is exception number)包含address of the handler for exception k. Exception table的地址存放在exception table base register.


