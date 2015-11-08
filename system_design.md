# System Design

##1.Large String Process
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

Q : In the main server, the map is <string, vector<string>> pair, the intermediate result can be very huge, how to improve?

A : It is a compression problem.