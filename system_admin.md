# System Admin

##1. 如何实现ls
1. View the metadata of this file.
2. if it is a normal file, only print filename.
3. if it is a directory, open the directory(Note : directory is also a file, the content of directory is file and inode.), print all filenames in this directory.

##2. 关于Top
Top是一个超级资源监视器,提供了几乎可以说是所有的我们所需要的系统资源使用监测.

    top - 09:11:34 up 22:55,  5 users,  load average: 0.03, 0.04, 0.05
    Tasks: 170 total,   2 running, 168 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   1008644 total,   881924 used,   126720 free,   100628 buffers
    KiB Swap:  1046524 total,   113260 used,   933264 free.   160680 cached Mem

    PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                             
    1103 root      20   0  374836  38268  12980 S  0.7  3.8   2:06.44 Xorg                                                                                
    2174 ganzhen+  20   0 1231068 145548  27752 S  0.3 14.4   5:01.95 compiz                                                                              
    2401 ganzhen+  20   0  349000   5036   3872 S  0.3  0.5   0:00.26 zeitgeist-daemo                                                                     
    2709 ganzhen+  20   0  581584  27624  13720 S  0.3  2.7   1:14.56 gnome-terminal                                                                      
    1 root      20   0   33780   3728   2264 S  0.0  0.4   0:02.63 init                                                                                
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd                                                                            
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.96 ksoftirqd/0
    
Top 遍历 /proc 文件夹，以收集进程信息./proc本身是一个虚拟文件系统，意味着它并非存在于真实硬盘之中，而是由 Linux 内核凭空创建，保存在内存中的。在文件夹中，如/proc/2097（2097为 PID），Linux 内核将与之关联的信息打印到此文件中，而这里就是 Top 的消息来源.

注意: top不是实时的. /proc/stat文件中的信息是实时的，每0.01秒刷新一次.
而top默认每隔3秒从stat中取出当前的值记录下来.那么CPU利用率可以使用以下两个方法.先取两个采样点，然后计算其差值(不用去管具体的含义,了解大致就行)：

    cpu usage=(idle2-idle1)/(cpu2-cpu1)*100
    cpu usage=[(user_2 +sys_2+nice_2) - (user_1 + sys_1+nice_1)]/(total_2 - total_1)*100 
    
对于这种采点的计算方式，采点的间隔时间越长，结果越准确

所以，有了这个认知之后，使用适度的更新间隔是明智的。基于文件系统访问内存也是需要时间的.经验法则是，对于多数用户来说，1到3秒的间隔就足够了。如果你的 Linux系统相对比较空闲，随意使用短间隔，如果不是，最好为重要的任务保留你的 CPU 时间.一个减少Top工作的办法是只监测特定的几个 PID。这样，Top 无需遍历 /proc 下所有的子文件夹.