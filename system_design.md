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

为了达到上面列出的第一个目标，我们选择了在消息队列系统中引入raft一致性协议进行消息同步，简单地说，我们的系统一次起N个实例，客户端可以连到任意一个实例上进行入队和出队操作，只要集群中有N/2+1个节点存活且确认拿到消息，即可认为消息处理成功；即使有少部分实例挂掉，系统依然可以对外提供服务；当挂掉的这些实例重启或者新加入几个实例之后，它们也可以自动同步到最新的状态，然后继续对外服务.

在数据库领域谈可用性、一致性之类的问题已经谈了很多年，大家都觉得raft已经是论文论证过，工业可用，而且比paxos简单一万倍的东西，应该是很好实现的，包括我自己也这么想。然而理想很丰满，现实很骨干，消息队列的应用场景跟数据库是完全不同的，引入raft一个最大的问题就是处理能力严重不足！

MQ这种东西，本来就是解决生产者和消费者速度不匹配的问题而诞生的，那么MQ系统一个最最基本的要求就是写入速度必须要快，哪怕出队速度慢点也无所谓，因为业务高峰期持续时间是有限的，高峰结束之后有的是时间让消费者慢慢消化，更别说简单粗暴多加几台消费者就好了. 而一旦引入raft一致性协议，每个消息都要等半数以上的express实例确认之后才能返回成功，延时非常之高！要达到50ms左右（这个延时可以通过设置raft heartbeat来降低，但同时会提高系统负载，效果不明显）.

再加上，MQ系统在队列堵塞的时候，短时间内累积的消息数量非常之多，以峰值10亿条，每条消息200字节算，如果存在内存里，需要200GB的内存，显得太奢者了，所以必须落地存储.而raft协议同步的基础是增量日志（WAL），这就导致每一个消息会带来两次磁盘写入，大家都知道磁盘I/O是非常慢的操作，更进一步降低了系统入队速度.

为了克服这个困难，我们一度改写了raft库对WAL的操作逻辑，将WAL同时作为我们MQ的落地存储模块，依靠记录log ID之类的方法，改写了raft生成snapshot时的操作流程，使得一个消息写入只会带来一次磁盘I/O。虽然提高了一点点性能，但是开发成本实在高得离谱.

再后来我们发现导致延时高更重要的原因是多个实例之间同步消息过程。举例来说，集群入队一个消息需要50ms的话，实例之间通过raft库相互同步和确认就需要花掉40ms以上，与这样的消耗相比，上面说到的两次磁盘写操作根本不值一提。正在这个时候，etcd发布了它们的首个正式版本，并在博客中大肆吹嘘了一番他们自己开发的raft库，由于goraft优化无望，我们开始尝试使用etcd的raft库重新进行开发。

这个过程按下不表，总之找到了一些捷径之后很快就实现了MQ的业务逻辑，进行测试之后性能也确实有较大改观，看来goraft真的是不行。考虑到上面分析的性能瓶颈所在的结论，和对etcd/raft进行改造带来的巨大工作量，我们改回了双写落地的方案，也并没有慢多少。

到此时，关于可靠性这个目标已经实现了。但是不管怎么说，raft所带来的延时是无法避免的，即使是etcd本身，处理能力也就1000qps而已。最终结果就是，express只能用于我们自己的业务线上，因为我们更看重可靠/可用性，性能要求不是特别高。

####2. Topic（话题）和Line（消费线路）

为了支持多条消费者队列，在express中，存在topic和line这两个概念。一个topic可以拥有多条line，所有的消息入队的目标是topic，只需要入队一次；而消息出队只能从line里拿，每个line都有自己的名字，代表这条消费线路的具体用途，以图片上传举例：

所有消息入队到名叫wb_img_upload的topic中
压图模块从名叫img_to_compress的line中拿属于自己的消息并进行压缩
同时，分类模块从名叫img_to_analysis的line中拿属于自己的消息并进行分类
其他各种模块比如审核、统计等等，也从各自的line中拿消息，它们之间互不影响
这种设计类似于nsq的topic和channel，但是nsq中为了使多个channel之间相互独立，采用复制消息的方案，也就是说，入队的消息，有几个channel就得多复制几次。而在express中，line里面其实只是存了消息的游标，实际消息还是存在topic中的，入队也只需要写入一次即可。

####3. 消息确认

消息确认是针对line来说的，每条line在创建的时候就需要设定一个recycle（回收）时间，从line里拿走一个消息时，会同时得到该消息的ID，当你处理完之后，需要明确地告诉express以确认这条消息已经消费完毕，否则，超过recycle时间之后，该消息会重新加入到这条line的消息队列中，可以被其他消费者再次消费. 由于重新入队这个机制的存在，会打乱消息入队的顺序，甚至导致消息重复消费，从而引出时序性和幂等性的问题，这个我们在下一节进行探讨.

#### 时序性和幂等性

消息队列系统的时序性也是一个常见的问题了，在某些场景下，我们确实需要非常严格的按时序排列的消息，比如微博的feed，当然是需要先入队的消息先展示，后入队的消息后展示。那么如果MQ本身是严格时序的，当然是最好了（其实对MQ来说也是最简单的了，比如mcq就是），但问题是，你可以保证MQ按时间顺序出队，却无法保证消费者按时间顺序消费完毕啊。注意我说的是消费完毕，毕竟每个消费者遇到的情况都是不同的，总有干的快的也总有干的慢的。解决办法也很简单，在消息体内包含消息生成的时间即可，所有消息在需要展示的时候才按生成时间再排序，即可严格时序输出。

那么，实际上来说就是，真正保证时序性的，并不是消息队列，而是消费者！

除非有一种情况，消费者只有一个，所有出队的消息都由它来展示，那在消费者这一层面就不可能打乱消息顺序，只需要MQ按时序出队即可。以我目前贫瘠的知识量来看，这种场景是小作坊产品才会遇到的，根本不可能出现在需要解决流量高峰问题的互联网产品中；再者，如果只需要一个消费者就能顺序处理的话，你还有必要引入一个消息队列吗？当然，由于视野所限，这一段内容不一定正确，总之核心思想就是这种只有一个消费者的场景太罕见了，直接忽略掉算了。

接上面的结论继续说，由于严格时序性需要靠消费者这一层来保证，那么对于MQ来说，只需要在一定程度上保证时序性即可。为了能够实现消息确认和重入来保证消息不丢失，是不可能同时保证严格时序性的。事实上，redis作者Antirez在介绍他的disque时，也基本上是这么说的.

接下来说幂等性，MQ里的幂等性指的是一个消息如果被消费者多次消费，对于消费者来说效果应该是一样的。比如，用户上传了一张图片，这个图片需要被压图模块压缩成缩略图，压图服务器A取到了消息并压图完毕，但是在跟express确认消息的时候由于网络故障没成功，过了一段时间之后express让这条消息重新入队，压图服务器B又拿到了这个消息并压图完毕，这个时候，对于整个压图模块来说，是没有影响的，因为B压好的图，文件名和路径跟A之前压好的是完全一样的，只是把那个图覆盖了而已，用户依然可以看到，这就叫幂等消费者.

在某些消费者逻辑中，消息不是幂等的，重复处理同一个消息会造成数据错乱。这种情况下就不适合依靠MQ的消息重入机制来保证消息不丢了，而因该消费者自己制定策略. 那么在express中创建line的时候，只需要设置recycle为空即可.

这儿再插入一块关于丢消息的内容. 在我们的系统中，消息一旦被确认入队，就不可能丢失，因为入队时已经落地存储完成，只可能多次出队，这样的设计是为了最大程度保证消息不丢.而有些MQ并不能保证不丢消息，比如NSQ，有一部分消息是在内存里的，如果这时候NSQ宕机，消息就没了。这是速度和安全取舍的问题，并无优劣之分.

another userful [article](http://www.cnblogs.com/netfocus/p/3183110.html)
