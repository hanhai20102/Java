# Java多线程

## 多线程的实现方式
>参考文章：[https://www.cnblogs.com/felixzh/p/6036074.html](https://www.cnblogs.com/felixzh/p/6036074.html)
### 继承Thread类
```java
public class MyThread extends Thread {  
　　public void run() {  
　　 System.out.println("MyThread.run()");  
　　}  
}  
 
MyThread myThread1 = new MyThread();  
MyThread myThread2 = new MyThread();  
myThread1.start();  
myThread2.start();
```
### 实现Runnable接口创建线程
```java
public class MyThread extends OtherClass implements Runnable {  
　　public void run() {  
　　   System.out.println("MyThread.run()");  
　　}  
}


MyThread myThread = new MyThread();  
Thread thread = new Thread(myThread);  
thread.start();  
```
### 实现callable接口

线程实现callable接口需要重写call（）函数，而接受运算结果需要FutureTask实现类的支持
```
Future<String> res = new FutureTask<>();
String str = res.get();
```
```java

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;
 
/*
 * 一、创建执行线程的方式三：实现 Callable 接口。 相较于实现 Runnable 接口的方式，方法可以有返回值，并且可以抛出异常。
 *
 * 二、执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。  FutureTask 是  Future 接口的实现类
 */
public class TestCallable {
 
    public static void main(String[] args) {
        ThreadDemo td = new ThreadDemo();
 
        //1.执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。
        FutureTask<Integer> result = new FutureTask<>(td);
 
        new Thread(result).start();
 
        //2.接收线程运算后的结果
        try {
            Integer sum = result.get();  //FutureTask 可用于 闭锁 类似于CountDownLatch的作用，在所有的线程没有执行完成之后这里是不会执行的
            System.out.println(sum);
            System.out.println("------------------------------------");
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
 
}
 
class ThreadDemo implements Callable<Integer> {
 
    @Override
    public Integer call() throws Exception {
        int sum = 0;
 
        for (int i = 0; i <= 100000; i++) {
            sum += i;
        }
 
        return sum;
    }
 
}
```
## 线程池
>参考文章[https://blog.csdn.net/lipc_/article/details/52025993](https://blog.csdn.net/lipc_/article/details/52025993)
**ThreadPoolExecutor类**
代码示例：
```java
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 20, 60, TimeUnit.MINUTES, new LinkedBlockingQueue<Runnable>());
		final AtomicInteger num = new AtomicInteger(0);
		for (int i = 0; i < 255; i++) {
			final String ip = netWork+"."+(i+1);
			threadPool.execute(new Runnable() {
				
				@Override
				public void run() {
					// TODO Auto-generated method stub
					try {
						if(judgeIPValid(ip)){
							ipList.add(ip);
						    System.out.println(ip);
						}
					} catch (IOException e) {
						e.printStackTrace();
					}
					synchronized (num) {
						System.out.println("已经完成："+num.incrementAndGet()+"个IP测试");
					}
					
				}
			});
		}
		//线程结束后关闭线程池 并等待一个小时等线程池中的线程把所有的任务都结束了
		threadPool.shutdown();
		if(threadPool.awaitTermination(1, TimeUnit.HOURS)){
			//循环输出所有的IP地址
			System.out.println("共有以下IP地址可以连接");
			for (String sb : ipList) {
				System.out.println(sb);
			}
		}


```
ThreadPoolExecutor构造函数各参数的意义：
+ corePoolSize 核心线程的个数
+ maximumPoolSize 线程池中线程的个数
+ keepAliveTime 在非核心线程执行完任务，在过了这个时间段没有任务执行时销毁非核心线程
+ timeUnit 配合keepAliveTime使用,是时间的刻度值 如：TimeUnit.Hours
+ workQueue 任务等待队列的类型有三种

### 线程池的逻辑结构：
![线程池的逻辑结构](https://img-blog.csdn.net/20160724192641221)

● 等待队列：顾名思义，就是你调用线程池对象的submit()方法或者execute()方法，要求线程池运行的任务（这些任务必须实现Runnable接口或者Callable接口）。但是出于某些原因线程池并没有马上运行这些任务，而是送入一个队列等待执行。

● 核心线程：线程池主要用于执行任务的是“核心线程”，“核心线程”的数量是你创建线程时所设置的corePoolSize参数决定的。如果不进行特别的设定，线程池中始终会保持corePoolSize数量的线程数（不包括创建阶段）。

● 非核心线程：一旦任务数量过多（由等待队列的特性决定），线程池将创建“非核心线程”临时帮助运行任务。你设置的大于corePoolSize参数小于maximumPoolSize参数的部分，就是线程池可以临时创建的“非核心线程”的最大数量。这种情况下如果某个线程没有运行任何任务，在等待keepAliveTime时间后，这个线程将会被销毁，直到线程池的线程数量重新达到corePoolSize。

● maximumPoolSize参数也是当前线程池允许创建的最大线程数量。那么如果设置的corePoolSize参数和设置的maximumPoolSize参数一致时，线程池在任何情况下都不会回收空闲线程。keepAliveTime和timeUnit也就失去了意义。

● keepAliveTime参数和timeUnit参数也是配合使用的。keepAliveTime参数指明等待时间的量化值，timeUnit指明量化值单位。例如keepAliveTime=1，timeUnit为TimeUnit.MINUTES，代表空闲线程的回收阀值为1分钟。
#### 线程池的任务等待队列
+ 任务：就是你调用线程池对象的submit()方法或者execute()方法，要求线程池运行的任务（这些任务必须实现Runnable接口或者Callable接口）。但是出于某些原因线程池并没有马上运行这些任务，而是送入一个队列等待执行。(任务即是平时实现线程的方式，就是run方法要执行的代码块，通过线程来执行)
+ 任务等待队列 就是queue存放线程要执行的任务，由操作系统调度线程执行队列中要执行的任务

#### 四种线程池

1. new CachedThreadPool
核心线程为0，最大线程为整数的最大值，线程存活的时间为60s，任务等待队列为同步队列SynchronousQueue

2. newFixedThreadPool
核心线程nThread，最大线程数nThread，线程的存活时间无限期，任务等待队列为基于链表的无限队列LinkedBlockingQueue
线程池会自己维护此数量的核心线程的数量。
3. newSingleThreadPool
只有一个线程的线程池，线程的存活时间是无限的，线程繁忙时，新来的任务进入无限的阻塞队列中。
4. newSchehuledThreadPool
固定大小的线程池，存活时间无限制，线程池可以支持定时以及周期任务执行，如果所有线程进入繁忙状态，新的任务进入delayedWorkQueue，这是一种按照超时时间排队的队列
### 线程池处理一个任务的过程

1. 如果当前线程池中运行的线程数量还没有达到corePoolSize大小时，线程池会创建一个新的线程运行你的任务，无论之前已经创建的线程是否处于空闲状态。 
2. 如果当前线程池中运行的线程数量已经达到设置的corePoolSize大小，线程池会把你的这个任务加入到等待队列中。直到某一个的线程空闲了，线程池会根据设置的等待队列规则，从队列中取出一个新的任务执行。 
3. 如果根据队列规则，这个任务无法加入等待队列。这时线程池就会创建一个“非核心线程”直接运行这个任务。注意，如果这种情况下任务执行成功，那么当前线程池中的线程数量一定大于corePoolSize。 
4. 如果这个任务，无法被“核心线程”直接执行，又无法加入等待队列，又无法创建“非核心线程”直接执行，且你没有为线程池设置RejectedExecutionHandler。这时线程池会抛出RejectedExecutionException异常，即线程池拒绝接受这个任务。（实际上抛出RejectedExecutionException异常的操作，是ThreadPoolExecutor线程池中一个默认的RejectedExecutionHandler实现：AbortPolicy，这在后文会提到） 
5. 一旦线程池中某个线程完成了任务的执行，它就会试图到任务等待队列中拿去下一个等待任务（所有的等待任务都实现了BlockingQueue接口，按照接口字面上的理解，这是一个可阻塞的队列接口），它会调用等待队列的poll()方法，并停留在哪里。 
6. 当线程池中的线程超过你设置的corePoolSize参数，说明当前线程池中有所谓的“非核心线程”。那么当某个线程处理完任务后，如果等待keepAliveTime时间后仍然没有新的任务分配给它，那么这个线程将会被回收。线程池回收线程时，对所谓的“核心线程”和“非核心线程”是一视同仁的，直到线程池中线程的数量等于你设置的corePoolSize参数时，回收过程才会停止。

**任务等待队列有三种分为两类：** 

无限队列 LinkedBlockingQueue;

有限队列 SynchronousQueue、ArrayBlockingQueue

*使用方式*：在初始化的时候要存放runnable这种形式(任务) 例如：new ArrayBlockingQueue<Runnable>();
+ SynchronousQueue是一种阻塞队列：任何一次插入操作的元素都要等待相对的删除/读取操作，否则进行插入操作的线程就要一直等待，反之亦然。
+ ArrayBlockingQueue：此队列按 FIFO（先进先出）原则对元素进行排序。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。这是一个典型的“有界缓存区”，固定大小的数组在其中保持生产者插入的元素和使用者提取的元素。
+ LinkedBlockingQueue 具有无线容量的特性既可以指定容量也可以不指定容量，基于链表结构

### Java线程池的取消
>参考文章：[https://blog.csdn.net/jiyiqinlovexx/article/details/51002427](https://blog.csdn.net/jiyiqinlovexx/article/details/51002427)

## 多线程相关

### AtomicInteger类
>参考文章：[https://blog.csdn.net/u012734441/article/details/51619751](https://blog.csdn.net/u012734441/article/details/51619751)

在使用Integer的时候，必须加上synchronized保证不会出现并发线程同时访问的情况，而在AtomicInteger中却不用加上synchronized，在这里AtomicInteger是提供原子操作的。
```java
	AtomicInteger num = new AtomicInteger(0);
    num.incrementAndGet();
```
+ num.incrementAndGet(); 先执行加操作再返回
+ num.GetAndincrement(); 先返回再执行加操作

### 使用Collections的方法转换ArrayList为线程安全的类
```java
		final List<String> ipList = Collections.synchronizedList(new ArrayList<String>());  //ArrayList是线程不安全的，转换为线程安全的类

```