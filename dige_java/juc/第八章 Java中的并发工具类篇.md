## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 等待多线程完成的CountDownLatch

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4573d1b17d343848fda2f6f4f4a7b38~tplv-k3u1fbpfcp-zoom-1.image)

CountDownLatch的构造函数接收一个int类型的参数作为计数器，如果你想等待N个点完成，这里就传入N。

当我们调用CountDownLatch的countDown方法时，N就会减1，CountDownLatch的await方法会阻塞当前线程，直到N变成零。由于countDown方法可以用在任何地方，所以这里说的N个点，可以是N个线程，也可以是1个线程里的N个执行步骤。用在多个线程时，只需要把这个CountDownLatch的引用传递到线程里即可。

如果有某个解析sheet的线程处理得比较慢，我们不可能让主线程一直等待，所以可以使用另外一个带指定时间的await方法――await (long time，TimeUnit unit)，这个方法等待特定时间后，就会不再阻塞当前线程。join也有类似的方法。

# 同步屏障CyclicBarrier

CyelicBarrier的字面意思是可循环使用(Cyclic〉的屏障(Barrier)。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

CyclicBarrier默认的构造方法是CyclicBarier (int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。示例代码如代码清单8-3所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df7a55006b38462a9c08e4a7bcfae22a~tplv-k3u1fbpfcp-zoom-1.image)

输出： 1 2 或 2 1

如果把new CyclicBarrier(2)修改成new CyclicBarrier(3)，则主线程和子线程会永远等待，因为没有第三个线程执行await方法，即没有第三个线程到达屏障，所以之前到达屏障的两个线程都不会继续执行。

CyelicBarrier还提供一个更高级的构造函数CyclicBarrier (int parties，Runnable barrier-Action)，用于在线程到达屏障时，优先执行barrierAction，方便处理更复杂的业务场景，如代码清单8-4所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47dd5ece6cd1438e8ae3b109d6424903~tplv-k3u1fbpfcp-zoom-1.image)

3 2 1 或 3 1 2

# CyclicBarrier和CountDownLatch的区别

CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置。所以CyclicBarrier能处理更为复杂的业务场景。例如，如果计算发生错误，可以重置计数器，并让线程重新执行一次。

CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量。isBroken()方法用来了解阻塞的线程是否被中断。代码清单8-5执行完之后会返回true，其中isBroken的使用代码如代码清单8-6所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/861d60bddf4f4b2e9296cd2aa91dcca3~tplv-k3u1fbpfcp-zoom-1.image)

# 控制并发线程数的Semaphore

Semaphore可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接。假如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程并发地读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这时我们必须控制只有10个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连接。这个时候，就可以使用Semaphore来做流量控制，如代码清单8-7所示。

```
public class SemaphoreTest {
 private static final int THREAD_COUNT = 30;
 private static ExecutorServicethreadPool = Executors.newFixedThreadPool(THREAD_COUNT);
 private static Semaphore s = new Semaphore(10);
 public static void main(String[] args) {
 	for (inti = 0; i< THREAD_COUNT; i++) {
		 threadPool.execute(new Runnable() {
		 	@Override
 			public void run() {
 				try {
 					s.acquire();
					System.out.println("save data");
 					s.release();
 					} catch (InterruptedException e) {
 						}
 				}
 			});
	}
	threadPool.shutdown();
 }
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0db80a0e04414339ae89497019c80c7e~tplv-k3u1fbpfcp-zoom-1.image)

