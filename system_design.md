# System Design

##1. 如何直接跳出深层递归而不是一层一层跳出？

1. throw exception，try catch
2. 把递归函数放到一个独立线程执行，在主线程做 condition wait，递归结束时notify下，然后直接退出线程。
3. C语言的setjmp + longjmp

##2. 如何实现ls
1. View the metadata of this file.
2. if it is a normal file, only print filename.
3. if it is a directory, open the directory(Note : directory is also a file, the content of directory is file and inode.), print all filenames in this directory.

##3. 关于Top
Top是一个超级资源管理器,提供了几乎可以说是所有的我们所需要的系统资源使用监测.

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
    
Top 遍历 /proc 文件夹，以收集进程信息。/proc本身是一个虚拟文件系统，意味着它并非存在于真实硬盘之中，而是由 Linux 内核凭空创建，保存在内存中的。在文件夹中，如/proc/2097（2097为 PID），Linux 内核将与之关联的信息打印到此文件中，而这里就是 Top 的消息来源。

注意: top不是实时的. stat文件中的信息是实时的，每0.01秒刷新一次。
而top默认每隔3秒从stat中取出当前的值记录下来。那么CPU利用率可以使用以下两个方法。先取两个采样点，然后计算其差值(不用去管具体的含义,了解大致就行)：

    cpu usage=(idle2-idle1)/(cpu2-cpu1)*100
    cpu usage=[(user_2 +sys_2+nice_2) - (user_1 + sys_1+nice_1)]/(total_2 - total_1)*100 
    
对于这种采点的计算方式，采点的间隔时间越长，结果越准确

##4. Large String Process
[Work BreakII](https://leetcode.com/problems/word-break-ii/)

Q : Assuming length of string reach 1 trillion, how to design the system.

A : If I have n machines, I could divide the string into n parts and do it and merge the result.

Q : How to improve?

A : Set up a main server to cache the intermediate result, for example the word "whereiswhere", if machine 1 has processed the word "where", machine 1 can cache the result in main server, when machine 2 encounter another "where" he can ask main server for results.

Q : It seems that each local machine will frequently communicate with the main server, how to improve the bottleneck?

A : In the main server, we have a map. At one time only support one machine to write. Like this :
    
    Lock()
    //write
    Unlock()
    
We can make 26 small maps according to the first character of the word. If the word starts with 'b', we can do like this :

    Lock(b)
    //wrirte
    Unlock(b)
    
Apply this methodology, we can speed up 26 times in theory.

Q : In the main server, the map is 

    <string, vector<string> > 
pair, the intermediate result can be very huge, how to improve?

A : It is a compression problem. First, we can apply Huffman encoding, CLRS tells me Huffman usually can compress 25% to 80%. Second, we can apply run-length encoding, for example "wwwww" can be "5Tw" and "aaaaaa" can be "6Ta".

Q : It would be very expensive if the string we query is very long.

A : Making query in local machine will be much cheaper than making query to main server. So we can make a copy in locak machine and update every hour, but it may be very expensive.

Q : Yes it is very expensive, I mean passing a string is very expensive, how to deal with it?

A : Pass by hash value.