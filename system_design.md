# System Design

##1. 如何直接跳出深层递归而不是一层一层跳出？

1. throw exception，try catch
2. 把递归函数放到一个独立线程执行，在主线程做 condition wait，递归结束时notify下，然后直接退出线程。
3. C语言的setjmp + longjmp

##2. 如何实现ls

##3. Large String Process
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