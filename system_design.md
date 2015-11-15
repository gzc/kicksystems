# System Design

##1. Large String Process
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

##2. Message Queue
消息队列系统是大型分布式系统中常见的组成部分之一.

做个搬运工 [reference](http://blog.buaa.us/talk-about-mq/)

Goal :
1. 可靠性.开发一个可靠性非常高的消息队列，无需为单点问题担忧.
2. 多条消费者队列.一个消息入队之后，消费者只能从队列里取出来一次，在某些场景下，一个消息可能需要被多个消费者系统消费，比如微博图片传上来之后，需要被压缩系统拿来进行压缩，同时还要被分类系统拿去进行分类，还会被审核系统拿去审查，等等. 目前的做法是，一个消费者先从队列里拿到消息，处理完之后再重新入队，这样其他系统就可以继续拿出来消费。弊端是将一个并行的任务生生做成了串行，而且更严重的是，一旦某个消费者系统拿了消息之后发生故障，未能将消息塞回来，那么后续的消费者就无法处理这个消息了. 因此我们的一个目标是支持多个消费者队列，它们之间互不影响，各自消费各自队列里的消息.
3. 消息确认和重新入队.一个消息被某个消费者拿走之后，还没来得及处理，这个消费者系统就挂了，那么这个消息就丢失了，任何其他消费者都无法对它进行处理. 必须支持消息确认机制，未确认的消息一段时间之后将会自动重新进入队列中，无需使用者操心.