---

layout: post

title:  "阅读muduo和brpc开源库后记"

categories: network

tags:  network IO-Multiplexing epoll muduo brpc

---

近期阅读了muduo网络库和brpc远程调用库，从中学到了非常多的知识，专门来记录一下。

[muduo](https://github.com/chenshuo/muduo)是一个基于epoll（LT）的IO复用reactor模式的网络库，代码风格是基于std::bind+std::function对象的回调风格，很C++11。可以利用它轻松构建一个高性能的服务器程序，主要思想是one loop per thread，即epoll事件监控交给一个单独loop，此loop也称为事件分发器（Event Dispatcher），每个连接（Channel）交给单独一个loop，不存在跨线程操作同一个socket，大大减少了竞态（Race Condition）。库中还提供了服务器一些常用组件的经典实现，比如定时器（Timer）、线程池（ThreadPool）、线程安全的单例（Singleton）、异步写日志（Logging），源码很值得初学者进行研究。我自己为了弄懂reactor+one loop per thread + threadpool机制，从只有一个main函数中调用epoll到一步一步实现了muduo的reactor框架，代码详见[这里](https://github.com/mihooke/muduo_study/tree/master/step)，这个仓库里也保存了一份muduo代码，里面加了我自己对muduo的理解。muduo库结合作者的书看效果更佳。作者还在B站有一期《网络编程实战》的课程，是对muduo代码及示例的解读，很值得一看。

值得一提的是，muduo的多线程模型是**reactors+threadpool**，注意这里是**reactors**，即多个reactor。主线程是一个负责accept的主reactor，当主线程accept到连接时，便均匀地放到threadpool里负责的子reactors中，比如threadpool大小是4，则有4个子reactors，分别运行在4个EventLoop中，某时刻收到连接A B C D，则分别放入4个EventLoop中，这样每个reactor负责一个连接，再收到连接时，根据Round-Robin规则，放到对应reactor中，每一个连接的生命周期由当前的EventLoop来维护，也减少了race-condition。缓存数据库memcached也是这个模型。还有另一种IO模型：recator+多进程/多线程，即accept和新连接都放入同一个reactor中，读写事件放到新的进程或线程中处理。至于哪种模型好，只能说各有利弊。像redis就是这种模型的简化版，所有事件都在IO线程中处理，原因是redis操作是纯内存操作，几乎不会有耗时操作，也不会有长时间CPU操作，在一个线程中操作也省了锁的争用。至于多进程和多线程有何区别，我认为对外提供一个服务，首选多线程，毕竟系统调度的基本单位是线程，可比进程节省些资源，如果考虑多线程之间不存在共享资源，或者考虑更稳定的服务，可考虑多进程，理由是假设某个服务掉线，进程的话可以直接重启来恢复服务，线程的话就没办法在用户不感知的情况下重启恢复了，因为进程重启，进程内的所有线程都没了。

[brpc](https://github.com/apache/incubator-brpc)是一个基于epoll（ET）的RPC库，兼容很多主流RPC协议，主打高性能。笔者当初选择brpc学习的一个主要原因是brpc的文档写的非常详细，并且部分文档还介绍了分布式服务器开发中常见的模式，难点，痛点以及如何解决它们，知识覆盖面很广，几乎囊括了后端服务器开发个各个方面，可谓是良心之作，一个真正的工业库，阅读其源码能深有这种感受。另一个优秀的RPC库grpc，Google出品，比brpc稍早开源的，grpc的协议使用HTTP2，RPC架构是基于protobuf3，整体上来说，grpc库要更前沿一些，但它并没有实现分布式系统，这也是我选择brpc学习的最大原因。由于brpc使用了protobuf2的协议，因此整个RPC通信架构也是基于protobuf2的，代码风格是经典的继承式面向对象。

brpc还大量造了很多轮子，有N:M线程模型的用户态线程（即协程）**bthread**，有内存方面的**ResourcePool/ObjectPool**，**原子操作**的，文件操作的，**应用层多生产者单消费者无锁队列**，Work-Staling ThreadPool，各种改进的数据结构。印象特别深的是定时器（TimerThread），由于RPC中定时器场景和一般服务器中场景不太一样，比如一般服务器中定时器场景都是定时回调注册函数，注意这里是这些回调函数都会去执行；但RPC中就不一样了，基本上每一次RPC调用都会设置timeout，而绝大多数RPC调用都不会timeout，那么系统在RPC调用结束后还得负责从内存中把无效的注册函数删掉，如此做法在RPC中属于无用功。muduo中的定时器实现就是把注册回调保存在红黑树上，频繁操作时间复杂度也是O(lgN)，brpc从两个方面来提升，第一是降低锁的调用，通过把注册回调散列到多个bucket中来降低线程间的竞争；第二是不进行或很少进行删除操作，需要删除时通过修改标记位，并且删除操作不参与全局竞争，这么做相当于定制了一版RPC场景的定时器。

 brpc中这样的情况很多，完全是根据场景业务定制的开发，当然，这里的场景是分布式的高并发场景。**服务发现**（Service Discovery）、**负载均衡**（Load Balancing）、**熔断机制**（Circuit Breaker）、**一致性哈希**（Consistent Hashing）、**限流**（Concurrency Limiter）等都可以找到典型的解决办法。

这里简单谈一下我对协程的理解，我最开始了解协程是Python中的协程，但Python的协程一开始需要自己用yield关键字来实现，3.5之后需要借助第三方库async和await来实现，仍然是N:1的线程模型，即N个协程在同一个线程中运行，这样有一个弊端，就是其中某一个协程阻塞，其他协程也就阻塞了，毕竟协程的调度是由用户自己来调度的，目的就是节省系统层面的昂贵耗时的上下文切换。bthread不一样，它是N:M线程模型，即N个协程可运行在M个线程中，Golang也是这样的线程模型，做到真正的协程并发。bthread毕竟是C++语言实现的，和Golang的croutine是不一样的，bthread是用汇编来实现协程的上下文切换的（原版是boost中实现），并在此基础上封装了work-stealing pool，在应用层来调度协程运行，当某个线程中没有协程任务时，可从其他任务多的线程中窃取任务协程到自己，当然，这其中细节很多，比如如何窃取及窃取多少。

两个库都是实现的后端server，要承受高性能并发请求的业务场景，可以发现它们各自都有实现一些基础库，比如任务队列、定时器、线程池、IO缓冲区。其实服务端组件常用的也就那么几种，只是业务场景不同，需要根据业务场景实现一个高效的组件，毕竟任何脱离业务的设计都是耍流氓。