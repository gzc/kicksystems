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

####1. 基于raft协议的可靠性消息队列Express

为了达到上面列出的第一个目标，我们选择了在消息队列系统中引入raft一致性协议进行消息同步，简单地说，我们的系统一次起N个实例，客户端可以连到任意一个实例上进行入队和出队操作，只要集群中有N/2+1个节点存活且确认拿到消息，即可认为消息处理成功；即使有少部分实例挂掉，系统依然可以对外提供服务；当挂掉的这些实例重启或者新加入几个实例之后，它们也可以自动同步到最新的状态，然后继续对外服务。

在数据库领域谈可用性、一致性之类的问题已经谈了很多年，大家都觉得raft已经是论文论证过，工业可用，而且比paxos简单一万倍的东西，应该是很好实现的，包括我自己也这么想。然而理想很丰满，现实很骨干，消息队列的应用场景跟数据库是完全不同的，引入raft一个最大的问题就是处理能力严重不足！

MQ这种东西，本来就是解决生产者和消费者速度不匹配的问题而诞生的，那么MQ系统一个最最基本的要求就是写入速度必须要快，哪怕出队速度慢点也无所谓，因为业务高峰期持续时间是有限的，高峰结束之后有的是时间让消费者慢慢消化，更别说简单粗暴多加几台消费者就好了。而一旦引入raft一致性协议，每个消息都要等半数以上的express实例确认之后才能返回成功，延时非常之高！要达到50ms左右（这个延时可以通过设置raft heartbeat来降低，但同时会提高系统负载，效果不明显）。

再加上，MQ系统在队列堵塞的时候，短时间内累积的消息数量非常之多，以峰值10亿条，每条消息200字节算，如果存在内存里，需要200GB的内存，对我们公司来说太奢者了，所以必须落地存储。而raft协议同步的基础是增量日志（WAL），这就导致每一个消息会带来两次磁盘写入，大家都知道磁盘I/O是非常慢的操作，更进一步降低了系统入队速度。

为了克服这个困难，我们一度改写了raft库对WAL的操作逻辑，将WAL同时作为我们MQ的落地存储模块，依靠记录log ID之类的方法，改写了raft生成snapshot时的操作流程，使得一个消息写入只会带来一次磁盘I/O。虽然提高了一点点性能，但是开发成本实在高得离谱。

再后来我们发现导致延时高更重要的原因是多个实例之间同步消息过程。举例来说，集群入队一个消息需要50ms的话，实例之间通过raft库相互同步和确认就需要花掉40ms以上，与这样的消耗相比，上面说到的两次磁盘写操作根本不值一提。正在这个时候，etcd发布了它们的首个正式版本，并在博客中大肆吹嘘了一番他们自己开发的raft库，由于goraft优化无望，我们开始尝试使用etcd的raft库重新进行开发。

这个过程按下不表，总之找到了一些捷径之后很快就实现了MQ的业务逻辑，进行测试之后性能也确实有较大改观，看来goraft真的是不行。考虑到上面分析的性能瓶颈所在的结论，和对etcd/raft进行改造带来的巨大工作量，我们改回了双写落地的方案，也并没有慢多少。

到此时，关于可靠性这个目标已经实现了。但是不管怎么说，raft所带来的延时是无法避免的，即使是etcd本身，处理能力也就1000qps而已。最终结果就是，express只能用于我们自己的业务线上，因为我们更看重可靠/可用性，性能要求不是特别高。