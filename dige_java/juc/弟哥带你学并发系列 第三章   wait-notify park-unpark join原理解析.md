## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# wait-notify

## wait notify流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/204f3efb12e24881bb42ae233e31d807~tplv-k3u1fbpfcp-zoom-1.image)

1.  使用wait()、notify()和notifyAll()时需要先对调用对象加锁。
1.  调用wait()方法后，线程状态由RUNNING变为WAITING，并将当前线程放置到对象的等待队列（会释放锁）。

<!---->

3.  notify()或notifyAll)方法调用后，等待线程依旧不会从wait()返回，需要调用notify()或notifAll()的线程释放锁之后，等待线程才有机会从wait()返回。
3.  notify()方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而notifyAll()方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由WAITING变为BLOCKED。

<!---->

5.  从wait()方法返回的前提是获得了调用对象的锁。



## sleep(long n) 和 wait(long n) 的区别

1.  sleep 是 Thread 方法，而 wait 是 Object 的方法 
1.  sleep 不需要强制和 synchronized 配合使用，但 wait 需要和 synchronized 一起用 

<!---->

3.  sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁 

\


## wait notify正确用法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81398899470a4332a4373703ba33cf06~tplv-k3u1fbpfcp-zoom-1.image)



# park-unpark

## 原理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dba02f9f33274e62ae2ff33e9e0f242d~tplv-k3u1fbpfcp-zoom-1.image)




## park

1. 当前线程调用 Unsafe.park() 方法

2. 检查 _counter ，本情况为 0，这时，获得 _mutex 互斥锁

3. 线程进入 _cond 条件变量阻塞

4. 设置 _counter = 0

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f9418855f274acf9febedc37437debd~tplv-k3u1fbpfcp-zoom-1.image)

## unpark

1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1

2. 唤醒 _cond 条件变量中的 Thread_0

3. Thread_0 恢复运行

4. 设置 _counter 为 0

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab567fb84a0e4dc5a0b47b83d16ac220~tplv-k3u1fbpfcp-zoom-1.image)

## 不会阻塞

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c16d7ff9c0b46b299281fdf9a31b70c~tplv-k3u1fbpfcp-zoom-1.image)

# Join

join用于让当前执行线程等待join线程执行结束。其实现原理是不停检查join线程是否存活，如果join线程存活则让当前线程永远等待。其中，wait (0）表示永远等待下去，代码片段如下。

```
while (isAlive()){
    wait (O) ;
}
```

直到join线程中止后，线程的this.notifyAll()方法会被调用，调用notifyAll()方法是在JVM里实现的，所以在JDK里看不到，大家可以查看JVM源码。


