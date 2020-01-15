# 1. 线程基础、线程之间的共享和协作

## 1.1 基础知识

### CPU核心数和可运行线程数的关系

核心数:线程数=1:1  使用了超线程技术后---> 1:2

### CPU时间片轮转机制

又称RR调度，会导致上下文切换

### 什么是进程和线程

- 进程：程序运行资源分配的最小单位，进程内部有多个线程，会共享这个进程的资源
- 线程：CPU调度的最小单位，必须依赖进程而存在。

### 并行和并发

- 并行：同一时刻，可以同时处理事情的能力
- 并发：与单位时间相关，在单位时间内可以处理事情的能力

### 高并发编程的意义、好处和注意事项

- 好处：充分利用cpu的资源、加快用户响应的时间，程序模块化，异步化

- 问题：1.线程共享资源，存在冲突, 2.容易导致死锁 3.启用太多的线程，就有搞垮机器的可能

### 认识Java里的线程

- Class extends Thread
- Class implements Runnable / Callable

### 让Java里的线程安全停止工作

- 线程自然终止：自然执行完或抛出未处理异常

- stop()，resume(),suspend()已不建议使用，stop()会导致线程不会正确释放资源，suspend()容易导致死锁。
java线程是协作式，而非抢占式 => 主要是为了让每个线程有充分的时间做自己的清理工作。

- 调用一个线程的interrupt() 方法中断一个线程，并不是强行关闭这个线程，只是跟这个线程打个招呼，将线程的中断标志位置为true，线程是否中断，由线程本身决定。
isInterrupted() 判定当前线程是否处于中断状态。可利用IsInterrupted()来判断其他线程是否对本线程发出interrupt，再做进一步处理。

- static方法interrupted() 判定当前线程是否处于中断状态，同时中断标志位改为false。**方法里如果抛出InterruptedException，线程的中断标志位会被复位成false，如果确实是需要中断线程，要求我们自己在catch语句块里再次调用interrupt()**。

- 当一个方法抛出 InterruptedException 时，意味着几件事情: 除了它可以抛出一个特定的检查异常, 它还告诉你它是一种阻塞方法，它会尝试解除阻塞并提前返回。

[浅析线程的正确停止](https://juejin.im/post/5d6c8e23f265da03c23eedb8)

### 线程常用方法和线程的状态

![](./img/thread_status.png)

线程只有5种状态。整个生命周期就是这几种状态的切换。

### run()、start()、yield()

- run方法就是普通对象的普通方法，只有调用了start()后，Java才会将线程对象和操作系统中实际的线程进行映射，再由新的线程来执行run方法。
- yield: 让出cpu的执行权，将线程从运行转到可运行状态，但是下个时间片，该线程依然有可能被再次选中运行。

### 线程的优先级

取值为1\~10，缺省为5，但线程的优先级不可靠，有些操作系统甚至会忽略线程优先级，不建议作为线程开发时候的手段。

### 守护线程

守护线程有：
Monitor Ctrl-Break
Signal Dispatcher
Finalizer
Reference Handler
GC

- 他们和主线程共死。如果给自定义的子线程setDaemon(true)，和主线程共死。同时，守护进程里的finally不能保证一定执行。所以，子进程想用finally做必要的清理时，不能作为守护进程。

## 1.2 线程间的共享

- synchronized内置锁：
	- 对象锁，锁的是类的对象实例，不同的实例之间互不影响。
	- static synchronized - “类”锁，锁的是每个类的的Class对象，每个类的的Class对象在一个虚拟机中只有一个，所以类锁也只有一个。

- volatile关键字：虚拟机提供的最轻同步机制，对某变量加上volatile，意味着每次get都会从主内存读取，set完后，一定要成员写入主内存，保证其他线程get到最新的值（缓存失效），读写不会加锁。但是，这不是线程安全的，因为它只能确保内存可见性，不能确保原子性。适合于只有一个线程写，多个线程读的场景。

- ThreadLocal线程变量。每个线程只使用自己的拷贝。可以理解为是个map，类型Map<Thread,Integer>，尽量存储小一些数据，否则内存消耗很大，链接池用的比较多。

## 1.3 线程间协作

- 轮询：难以保证及时性，资源开销很大，

- 等待和通知
	- wait()    对象上的方法
	- notify/notifyAll  对象上的方法

### 等待和通知的标准范式：

等待方：
- 1、	获取对象的锁；
- 2、	循环里判断条件是否满足，不满足调用wait方法：**wait方法会释放锁，并等待信号通知，在收到通知后，会去尝试重新获得这个锁，所以在wait方法退出之前会重新获取这把锁，只有获取了这把锁才会继续执行。整个wait方法是原子的。**
- 3、	条件满足执行业务逻辑

通知方来说：
- 1、	获取对象的锁；
- 2、	改变条件
- 3、	通知所有等待在对象的线程: notify/notifyAll

notify和notifyAll应该用谁？应该尽量使用notifyAll，使用notify只会发送一个信号到同一对象上等待队列中的一个wait，并不一定是自己期望的那个wait，所以有可能发生信号丢失的的情况。

**相关文章**

[sleep和wait的区别](https://www.zhihu.com/question/23328075/answer/665978836)

[你知道wait/notify的这些知识点吗？](https://juejin.im/post/5da03850e51d4577f706198b)


### 等待超时模式实现一个连接池：

假设等待时间时长为T，当前时间now+T以后超时：

```java
long  overtime = now+T;
long remain = T;//等待的持续时间
while(result不满足条件&& remain>0){
	wait(remain);
	remain = overtime – now;//等待剩下的持续时间
}
return result;
```
### join()方法

主要用途：线程A，执行了线程B的join方法，线程A必须要等待B执行完成了以后，线程A才能继续自己的工作。


### 调用yield()、sleep()、wait()、notify()等方法对锁有何影响？ 

- 线程在执行yield()以后，持有的锁是不释放的
- sleep()方法被调用以后，持有的锁是不释放的
- 调动方法之前，必须要持有锁。调用了wait()方法以后，锁就会被释放，当wait方法返回的时候，线程会重新持有锁
- 调动方法之前，必须要持有锁，调用notify()方法本身不会释放锁的


# 2. 线程的并发工具类

## Fork-Join 分而治之

### 什么是分而治之

规模为N的问题，N<阈值，直接解决，N>阈值，将N分解为K个小规模子问题，子问题互相对立，与原问题形式相同，将子问题的解合并得到原问题的解。但在动态规划中k个子问题相互有关系。

比如：加速二分查找、快速排序、比较排序

### 工作密取 workStealing

类似golang调度系统中偷取G的方式，提前完成任务的线程，从工作繁重线程的Task队列中把工作偷过来运行。

### Fork/Join使用的标准范式

![](./img/fork_join_struct.png)

- Fork/Join的同步用法同时演示返回结果值：统计整形数组中所有元素的和
- Fork/Join的异步用法同时演示不要求返回值：遍历指定目录（含子目录）寻找指定类型文件

### CountDownLatch

作用：是一组线程等待其他的线程完成工作以后在执行，加强版的join，使用await用来等待，countDown负责计数器的减一。

例子：主线程需要等待三个子线程全部完成才能继续走：主线程声明countDownLatch(3)，然后调用await。子线程完成调用countDown让计数器减一，当扣除完毕（扣除数达到3次时），主线程从await的阻塞中恢复。

### CyclicBarrier

让一组线程达到某个屏障，被阻塞，一直到组内最后一个线程达到屏障时，屏障开放，所有被阻塞的线程会继续运行CyclicBarrier(int parties)
CyclicBarrier(int parties, Runnable barrierAction)，屏障开放，barrierAction定义的任务会执行，任务执行后，各个线程中的await才返回。

CountDownLatch和CyclicBarrier辨析：
- 1、countdownlatch放行由第三者控制，CyclicBarrier放行由一组线程本身控制，所以CyclicBarrier中的await到达约定数量前都会被阻塞，达到后一起放行。
- 2、countdownlatch放行条件 >= 线程数，CyclicBarrier放行条件=线程数

### Semaphore

控制同时访问某个特定资源的线程数量，用在流量控制

### Exchange

两个线程间的数据交换，用得比较少。第一个到达exchange时，会阻塞，等到第二个线程也到达exchange时，双方就交换对应的数据。

### Callable、Future和FutureTask 

![](./img/FuturTask.png)

- isDone，结束，正常还是异常结束，或者自己取消，返回true，正在运行则返回false。
- isCancelled 任务完成前被取消，返回true；
- cancel（boolean）：
	- 任务还没开始，返回false
	- 任务已经启动，cancel(true)，中断正在运行的任务，中断成功，返回true，cancel(false)，不会去中断已经运行的任务。Future.cancel(false) is only useful to avoid starting tasks that hadn't already been started.
	- 任务已经结束，返回false

**相关文章**
[Use case for Future.cancel(false)?](https://stackoverflow.com/questions/3271564/use-case-for-future-cancelfalse)

使用场景：包含图片和文字的文档的处理：图片（云上），可以用future去取图片，主线程继续解析文字。

























