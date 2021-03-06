# 问题
1. 快速请求造成崩溃问题
    g5对每个请求都做了fd与请求对象的绑定， fd => task(epollfd, fd)的关联关系。每次fd事件到来都会开新线程去处理。那么同一fd发送大量的请求时，线程池维护的任务队列情况可能是这样（实际是task*，这里记与task绑定的fd）: 6, 7, 6, 6， 8....
      1. 这队列中对相邻fd=6事件处理存在时间差，因为第一次处理完请求后便释放了内存， 所以后边的事件处理就会出现问题。解决这个办法就是在线程池，执行任务部分加了个判断:
    
         ``` c++
         if (task != NULL) // 任务没有被释放才执行任务
               task->exec();
         ```
    
      1. 除上面问题之外， 还会存在其他问题。