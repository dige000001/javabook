## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 一 基本概念

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4910e1d0a55c4e7787c522092499264a~tplv-k3u1fbpfcp-zoom-1.image)

## 线程创建和启动

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ec852eb81494bd583a4aa7ae774e013~tplv-k3u1fbpfcp-zoom-1.image)

现在应该有四种，还有通过线程池, callable

注：在main方法里启动其他线程后，其他线程进入就绪态，此时cpu会继续执行main还是执行其他线程都有可能

方法一：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c166d006eed1491ba93bb61d447debe9~tplv-k3u1fbpfcp-zoom-1.image)

main和r所属线程并发

方法二：略




## 线程方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc1ab8129ad54607a2678b056cc06f3d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1b4dc8e55154440e96715387b27f1d0d~tplv-k3u1fbpfcp-zoom-1.image)

sleep抛出了一个异常，故调用时需要catch

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b603bf5880243a39fbd557ebe42a6bf~tplv-k3u1fbpfcp-zoom-1.image)

结果：main把t1合并到自己的线程里，故只有t1执行完后main才开始进入循环

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8410cfd2a6f94f36b10a248fc2d0010b~tplv-k3u1fbpfcp-zoom-1.image)

注：低优先级不意味没有机会执行，只是机会较小

让线程结束的较好的方法：设置一个flag，flag为true时run里面的内容执行，为false时run方法结束，还有一个shutdown函数，shutdown用于将flag设为false，当run方法结束时，线程就结束了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95af6c61c9164b06878312bf6149bf47~tplv-k3u1fbpfcp-zoom-1.image)

\


# 线程同步

例子：线程不同步

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/595fe8abba2c4cb7bc79af9737f6ec12~tplv-k3u1fbpfcp-zoom-1.image)

结果：两个都显示num为2

解决方法：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/67abcc4bab1b49d29edc3d388e79441c~tplv-k3u1fbpfcp-zoom-1.image)

或

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/999e59ed9935474baf06af20f43e2611~tplv-k3u1fbpfcp-zoom-1.image)

效果都为：锁定当前对象，不能同时访问同一对象被锁住的方法，但依旧可以访问未锁住的方法

注意：对于不同线程，只要传入synchronized()方法形参的对象是同一个，就能实现同步，若不是同一个对象，就不行，如在该例中，t1和t2都是同一个Testsync，也是同一个Timer，故能同步，若是在run()方法里new一个Timer，则t1 t2不能实现同步

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc0fe0d7704e49fdaf374b942846a2d4~tplv-k3u1fbpfcp-zoom-1.image)

