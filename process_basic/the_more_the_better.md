
## 进程越多越好？

前面提到多进程的并行可以提高并发度，那么进程是越多越好？一般遇到这种问题都回答不是，事实上，很多大型项目都不会同时开太多进程。

下面以支持100K并发量的Nginx服务器为例。

## 举个例子: Nginx

Nginx是一个高性能、高并发的Web服务器，也就是说它可以同时处理超过10万个HTTP请求，而它建议的启动的进程数不要超过CPU个数，为什么呢？

我们首先要知道Nginx是Master-worker模型，Master进程只负责管理Worker进程，而Worker进程是负责处理真实的请求。每个Worker进程能够处理的请求数跟内存有关，因为在Linux上Nginx使用了epoll这种多路复用的IO接口，所以不需要多线程做并行也能实现并发。

而多进程有一个坏处就是带来了CPU上下文切换时间，所以一味提高进程个数反而使系统系能下降。当然如果当前进程小于CPU个数，就没有充分利用多核的资源，所以Nginx建议Worker数应该等于CPU个数。

## 特殊情况

我们想想进程数应该等于CPU数，但是如果进程有阻塞呢？这是是应该提高进程数增加并行数的。

在Nginx的例子中，如果Nginx主要负责静态内容的下载，而服务器内存比较小，大部分文件访问都需要读磁盘，这时候进程很容易阻塞，所以建议提高下Worker数目。

## 绑定CPU

一般情况下除了确保进程数等于CPU数，我们还可以绑定进程与CPU，这就保证了最少的CPU上下文切换。

在Nginx中可以这样配置。

```
worker_processes 4;
worker_cpu_affinity 1000 0100 0010 0001;
```

这是同故宫系统调用sched_setaffinity()实现了，感兴趣大家可以自行学习这方面的知识。