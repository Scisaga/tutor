## 基本概念
### 多线程是什么

在多线程程序下，同步能控制对共享资源的访问。如果没有同步，当一个Java线程在修改一个共享变量时，另外一个线程正在使用或者更新同一个变量，这样容易导致程序出现错误的结果。

### 实现多线程的方法

Java线程可通过实现Runnable接口或者继承Thread类来实现，当你打算多重继承时，优先选择实现Runnable。

Thread.start()方法(native)启动线程，使之进入就绪状态，当cpu分配时间该线程时，由JVM调度执行run()方法。

需要run()&start()这两个方法是因为JVM创建一个单独的线程不同于普通方法的调用，所以这项工作由线程的start方法来完成，start由本地方法实现，需要显示地被调用，使用这俩个方法的另外一个好处是任何一个对象都可以作为线程运行，只要实现了Runnable接口，这就避免因继承了Thread类而造成的Java的多继承问题。

### ThreadLocal类

ThreadLocal是一个线程级别的局部变量，并非“本地线程”。ThreadLocal为每个使用该变量的线程提供了一个独立的变量副本，每个线程修改副本时不影响其它线程对象的副本(译者注)。

下面是线程局部变量(ThreadLocal variables)的关键点：

* 一个线程局部变量(ThreadLocal variables)为每个线程方便地提供了一个单独的变量。
* ThreadLocal实例通常作为静态的私有的(private static)字段出现在一个类中，这个类用来关联一个线程。
* 当多个线程访问ThreadLocal实例时，每个线程维护ThreadLocal提供的独立的变量副本。
* 常用的使用可在DAO模式中见到，当DAO类作为一个单例类时，数据库链接(connection)被每一个线程独立的维护，互不影响。(基于线程的单例)

### InvalidMonitorStateException异常

调用wait()/notify()/notifyAll()中的任何一个方法时，如果当前线程没有获得该对象的锁，那么就会抛出IllegalMonitorStateException的异常(也就是说程序在没有执行对象的任何同步块或者同步方法时，仍然尝试调用wait()/notify()/notifyAll()时)。

由于该异常是RuntimeExcpetion的子类，所以该异常不一定要捕获(尽管你可以捕获只要你愿意).作为RuntimeException，此类异常不会在wait(),notify(),notifyAll()的方法签名提及。

### sleep / suspend / wait

Thread.sleep()使当前线程在指定的时间处于“非运行”（Not Runnable）状态。线程一直持有对象的监视器。比如一个线程当前在一个同步块或同步方法中，其它线程不能进入该块或方法中。如果另一线程调用了interrupt()方法，它将唤醒那个“睡眠的”线程。

注意：sleep()是一个静态方法。这意味着只对当前线程有效，一个常见的错误是调用t.sleep()，（这里的t是一个不同于当前线程的线程）。即便是执行t.sleep()，也是当前线程进入睡眠，而不是t线程。t.suspend()是过时的方法，使用suspend()导致线程进入停滞状态，该线程会一直持有对象的监视器，suspend()容易引起死锁问题。

object.wait()使当前线程出于“不可运行”状态，和sleep()不同的是wait是object的方法而不是thread。调用object.wait()时，线程先要获取这个对象的对象锁，当前线程必须在锁对象保持同步，把当前线程添加到等待队列中，随后另一线程可以同步同一个对象锁来调用object.notify()，这样将唤醒原来等待中的线程，然后释放该锁。基本上wait()/notify()与sleep()/interrupt()类似，只是前者需要获取对象锁。

### 静态方法上使用同步

同步静态方法时会获取该类的`Class`对象，所以当一个线程进入同步的静态方法中时，线程监视器获取类本身的对象锁，其它线程不能进入这个类的任何静态同步方法。它不像实例方法，因为多个线程可以同时访问不同实例同步实例方法。

### 同步方法已经执行，线程能够调用对象上的非同步实例方法

一个非同步方法总是可以被调用而不会有任何问题。实际上，Java没有为非同步方法做任何检查，锁对象仅仅在同步方法或者同步代码块中检查。如果一个方法没有声明为同步，即使你在使用共享数据Java照样会调用，而不会做检查是否安全，所以在这种情况下要特别小心。一个方法是否声明为同步取决于临界区访问(critial section access),如果方法不访问临界区(共享资源或者数据结构)就没必要声明为同步的。

### 一个对象上两个线程可以调用两个不同的同步实例方法

不能，因为一个对象已经同步了实例方法，线程获取了对象的对象锁。所以只有执行完该方法释放对象锁后才能执行其它同步方法。

### 死锁

死锁就是两个或两个以上的线程被无限的阻塞，线程之间相互等待所需资源。这种情况可能发生在当两个线程尝试获取其它资源的锁，而每个线程又陷入无限等待其它资源锁的释放，除非一个用户进程被终止。就JavaAPI而言，线程死锁可能发生在一下情况。

* 当两个线程相互调用Thread.join()
* 当两个线程使用嵌套的同步块，一个线程占用了另外一个线程必需的锁，互相等待时被阻塞就有可能出现死锁。

### 参考

基础
* Java语言包含两种内在的同步机制：[同步块（或方法）](http://enetor.iteye.com/blog/986623)和[volatile变量](http://www.ibm.com/developerworks/cn/java/j-jtp06197.html)。这两种机制的提出都是为了实现代码线程的安全性。其中volatile变量的同步性较差（但有时它更简单并且开销更低）。
* [Java同步锁](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=3616041)
* [Java Concurrency / Multithreading Tutorial](http://tutorials.jenkov.com/java-concurrency/index.html)
* [java.util.concurrent介绍](http://www.cnblogs.com/sarafill/archive/2011/05/18/2049461.html) | [Java 理论与实践: 并发集合类](http://www.ibm.com/developerworks/cn/java/j-jtp07233/) | [Java concurrency (multi-threading) - Tutorial](http://www.vogella.com/tutorials/JavaConcurrency/article.html)
* [wait()/notify()](http://www.javamex.com/tutorials/wait_notify_how_to.shtml)
* [sychronized](http://uule.iteye.com/blog/1104562)
* [Latch/Barrier](http://janeky.iteye.com/blog/769965) | [Semaphore/FutureTask/Exchanger](http://janeky.iteye.com/blog/770393) | [ScheduledThreadPoolExecutor](http://janeky.iteye.com/blog/770441) | [BlockingQueue](http://janeky.iteye.com/blog/770671)
* [BlockingQueue ConcurrentLinkedQueue](http://blog.csdn.net/madun/article/details/20313269) | [Java并发包分析——BlockingQueue](http://www.cnblogs.com/qifengshi/p/6808065.html)
* [Timer](http://www.cnblogs.com/liangdelin/archive/2012/12/04/2801338.html)

高级
* [java thread state](http://www.cnblogs.com/zhengyun_ustc/archive/2013/03/18/tda.html)
* [Starvation and Fairness](http://tutorials.jenkov.com/java-concurrency/starvation-and-fairness.html)
* [Concurrency Pattern: The Producer Consumer Pattern in Java](http://dhruba.name/2011/05/21/the-producer-consumer-pattern-in-java/)
* [Non-blocking Algorithms](http://tutorials.jenkov.com/java-concurrency/non-blocking-algorithms.html)

## synchronized

在实际编程中有两种方式实现同步，分别是同步方法(synchronized methods)或同步块(synchronized block/synchronized statement)。

Java的线程同步是通过synchronized()来实现的，需要说明的是，Java的synchronized()方法类似于操作系统概念中的互斥内存块，在Java中的Object类型中，都是带有一个内存锁的，在有线程获取该内存锁后，其它线程无法访问该内存，从而实现Java中简单的同步、互斥操作。synchronized就是针对内存区块申请内存锁。

同步方法是在方法前加synchronized, 如果该方法是static的，则认为锁是相对于Class的，其他线程操作该类的任何对象时，遇到static同步方法或者方法内同步该Class时，需要等待；若该方法不是static的，则认为锁是相对于自身对象(this)的，其他线程操作此对象时，遇到同步方法(非static)，需要等待。

同步块一般写在函数里，形式如下：
`synchronized ( Expression ) Block`

Expression需是引用类型(对象，类)， Block则是代码块。同步块开始时需要获得Expression的一个互斥锁（mutual-exclusion lock）。当该锁没被其他线程占用时，获取该锁并则执行Block里内容，在Block结束后释放此锁。在执行Block时，任何其他想要获得该锁的线程需要阻塞等待。

* 可以认为锁是属于引用类型的, 同步的操作需要获取锁之后才进行，否则一直等待。编程时需注意锁(synchronized)的对象。
* 线程在wait后会释放持有的锁。
* 各线程同步时遵守先触发，先得锁原则（happens-before relationship）。
* 构造函数无法被synchronized。

## 使用wait与notify/notifyAll使得多线程协作

Obj.wait()，与Obj.notify()必须要与synchronized(Obj)一起使用，也就是wait,与notify是针对已经获取了Obj锁进行操作，从语法角度来说就是Obj.wait(),Obj.notify必须在`synchronized(Obj){...}`语句块内。从功能上来说wait就是说线程在获取对象锁后，主动释放对象锁，同时本线程休眠。直到有其它线程调用对象的notify()唤醒该线程，才能继续获取对象锁，并继续执行。相应的notify()就是对对象锁的唤醒操作。

但有一点需要注意的是notify()调用后，并不是马上就释放对象锁的，而是在相应的synchronized(){}语句块执行结束，自动释放锁后，JVM会在wait()对象锁的线程中随机选取一线程，赋予其对象锁，唤醒线程，继续执行。这样就提供了在线程间同步、唤醒的操作。Thread.sleep()与Object.wait()二者都可以暂停当前线程，释放CPU控制权，主要的区别在于Object.wait()在释放CPU同时，释放了对象锁的控制。

调用sleep()和yield()的时候锁并没有被释放，而调用wait()将释放锁。这样另一个任务（线程）可以获得当前对象的锁，从而进入它的synchronized方法中。可以通过notify()/notifyAll()，或者时间到期，从wait()中恢复执行。

只能在同步控制方法或同步块中调用wait()、notify()和notifyAll()。如果在非同步的方法里调用这些方法，在运行时会抛出IllegalMonitorStateException异常。

* wait()执行后，线程状态会变为disabled, 想继续执行除非以下事件中的一个发生:
   - 其他线程在此同步的引用上执行了notify()或者notifyAll()，注意是和wait相同引用上执行的notify()。
   - 线程被其他线程中断，会报InterruptedException。
   - wait可以指定timeout, 当timeout时间过去时。
* wait继续执行需要重新获得该引用的锁，若有其他线程占有着此锁，则仍然无法恢复。
* notify是随机唤醒一个wait中的线程，notifyAll是把所有wait中的线程全部唤醒。notify的对象仍旧是其持有的锁的引用。
* 如果没有线程wait, notify将会被忽略。

### 例子1：模拟线程之间的协作

Game类有2个同步方法prepare()和go()。标志位start用于判断当前线程是否需要wait()。Game类的实例首先启动所有的Athele类实例，使其进入wait()状态，在一段时间后，改变标志位并notifyAll()所有处于wait状态的Athele线程。

```java
class Athlete implements Runnable {
    private final int id;
    private Game game;

    public Athlete(int id, Game game) {
      this.id = id;
      this.game = game;
    }

    public boolean equals(Object o) {
      if (!(o instanceof Athlete))
        return false;
      Athlete athlete = (Athlete) o;
      return id == athlete.id;
    }

    public String toString() {
      return "Athlete<" + id + ">";
    }

    public int hashCode() {
      return new Integer(id).hashCode();
    }

    public void run() {
      try {
        game.prepare(this);
      } catch (InterruptedException e) {
        System.out.println(this + " quit the game");
      }
    }
}

public class Game implements Runnable {
private Set<Athlete> players = new HashSet<Athlete>();
private boolean start = false;

    public void addPlayer(Athlete one) {
      players.add(one);
    }

    public void removePlayer(Athlete one) {
      players.remove(one);
    }

    public Collection<Athlete> getPlayers() {
      return Collections.unmodifiableSet(players);
    }

    public void prepare(Athlete athlete) throws InterruptedException {
      System.out.println(athlete + " ready!");
      synchronized (this) {
        while (!start)
        wait();
        if (start)
          System.out.println(athlete + " go!");
      }
    }

    public synchronized void go() {
      notifyAll();
    }
    
    public void ready() {
      Iterator<Athlete> iter = getPlayers().iterator();
      while (iter.hasNext())
        new Thread(iter.next()).start();
    }

    public void run() {
      start = false;
      System.out.println("Ready......");
      ready();
      start = true;
      System.out.println("Go!");
      go();
    }

    public static void main(String[] args) {
      Game game = new Game();
      for (int i = 0; i < 10; i++)
        game.addPlayer(new Athlete(i, game));
      new Thread(game).start();
    }
}
```

结果：
```
Ready......
Athlete<0> ready!
Athlete<1> ready!
Athlete<2> ready!
Athlete<3> ready!
Athlete<4> ready!
Athlete<5> ready!
Athlete<6> ready!
Athlete<7> ready!
Athlete<8> ready!
Athlete<9> ready!
Go!
Athlete<9> go!
Athlete<8> go!
Athlete<7> go!
Athlete<6> go!
Athlete<5> go!
Athlete<4> go!
Athlete<3> go!
Athlete<2> go!
Athlete<1> go!
Athlete<0> go!
```

## 使用Volatile变量

Java 语言中的 volatile 变量可以被看作是一种 “程度较轻的 synchronized”；与 synchronized 块相比，volatile 变量所需的编码较少，并且运行时开销也较少，但是它所能实现的功能也仅是 synchronized 的一部分。

### 例子2：模拟等待过程

```java
class MyObject implements Runnable {
    private Monitor monitor;

    public MyObject(Monitor monitor) {
      this.monitor = monitor;
    }

    public void run() {
      try {
        TimeUnit.SECONDS.sleep(3);
        System.out.println("i'm going.");
        monitor.gotMessage();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
}

class Monitor implements Runnable {
private volatile boolean go = false;

    public void gotMessage() throws InterruptedException {
      go = true;
    }

    public void watching() {
      while (go == false)
        ;
      System.out.println("He has gone.");
    }

    public void run() {
      watching();
    }
}

public class BusyWaiting {
    public static void main(String[] args) {
        Monitor monitor = new Monitor();
        MyObject o = new MyObject(monitor);
        new Thread(o).start();
        new Thread(monitor).start();
    }
}
```

## Concurrent 对象
### Lock
```java
Lock lock = new ReentrantLock();
lock.lock();
lock.tryLock();
lock.unlock();
```
参考：
* [Java并发编程：Lock](https://www.cnblogs.com/dolphin0520/p/3923167.html)

### Blocking Queue

#### ArrayBlockingQueue
ArrayBlockingQueue顾名思义是一种数组形式的阻塞队列，其自然就有数组的特点，即队列的长度不可改变，只有初始化的时候指定。

put方法与take方法均是阻塞的方法。当队列已经满的时候，就会阻塞放入方法，当队列为空的时候，就会阻塞取出方法。

#### LinkedBlockingQueue
LinkedBlockingQueue顾名思义是一个链表形式的阻塞队列，不同于ArrayBlockingQueue。如果不指定容量，则默认是Integer.MAX_VALUE。也就是说他是一个无界阻塞队列。

这里阻塞的本质实现也是通过Condition类的等待/通知机制。但是有几点不同：
- 这里用了一个原子类的count计数，官方的给的注释是即使没有锁来提供保护，也能保证线程安全，实现wait guard。
- ArrayBlockingQueue的通知是在入队与出队的方法中，LinkedBlockingQueue则不是，并且插入之后不满的时候，还有通知其他await的线程。
- ArrayBlockingQueue的lock一直是一个，也就是put/take是用的一个锁，放与取无法实现并行。但是LinkedBlockingQueue是两个锁，放一个锁，取一个锁，可以实现put/take的并行，要高效一些。

#### SynchronousQueue

SynchronousQueue顾名思义是同步队列，特点不同于上面的阻塞队列，他是一个无界非缓存的队列，准确说他不存储元素，放入的元素，只有等待取走元素之后才能放入。也就是说任意时刻：
- isEmpty()法永远返回是true
- remainingCapacity() 方法永远返回是0
- remove()和removeAll() 方法永远返回是false
- iterator()方法永远返回空
- peek()方法永远返回null
- 元素并不会被生产者存在队列中，而是直接生产者与消费者进行交互
- 其实现是利用无锁算法，可以参考SynchronousQueue实现
- 同步队列支持公平性与非公平性。公平性是利用队列来管理多余生产者与消费者，非公平性是利用栈来管理多余生产者与消费者。

### CountdownLatch

### Phaser

### 信号量 Semaphore
单个信号量的Semaphore对象可以实现互斥锁的功能，并且可以是由一个线程获得了“锁”，再由另一个线程释放“锁”，这可应用于死锁恢复的一些场合。

## 线程池使用

* [JAVA线程池的分析和使用](http://www.infoq.com/cn/articles/java-threadPool)
* [Executors and ThreadPoolExecutor](http://www.journaldev.com/1069/java-thread-pool-example-using-executors-and-threadpoolexecutor)

```java
public List<Future<T>> executeTasks(Collection<Callable<T>> tasks) {
    // create an ExecutorService
    final ExecutorService executorService = Executors.newSingleThreadExecutor();

    // execute all tasks
    final List<Future<T>> executedTasks = executorService.invokeAll(tasks);

    // shutdown the ExecutorService after all tasks have completed
    executorService.shutdown();

    return executedTasks;
}
```
可以定制的Executors:
- newCachedThreadPool() — 创建一个 ExecutorService，根据需要创建新线程，并重用现有线程来处理传入的任务。
- newFixedThreadPool(int numberOfThreads) — 创建一个 ExecutorService，重用固定数量的线程。
- newScheduledThreadPool(int corePoolSize) — 创建一个 ExecutorService，调度命令在给定延迟后运行（或定期执行）。
- newSingleThreadExecutor() — 创建一个使用单个工作线程的 ExecutorService。
- newSingleThreadScheduledExecutor() — 创建一个单线程的 ExecutorService，调度命令在给定延迟后运行（或定期执行）。
- newWorkStealingPool() — 创建一个使用多个任务队列以减少竞争的 ExecutorService。

提交任务的方法:
- void execute(Runnable)
- Future<T> submit(Callable<T>)
- Future<?> submit(Runnable)

终止方法:
- Shutdown() // 等待之前的任务执行完 不再接受新的任务
- awaitTermination()

使用 CompletableFuture 检验任务执行结果

每个线程池包含以下几部分:
- 一个任务队列 BlockingQueue
- 工作线程集合 WorkerThread
- 线程工厂 ThreadFactory
- 元信息

参考:
* [An introduction to thread pools in Java](https://allegro.tech/2015/05/thread-pools.html)

### 为线程池中的线程命名
方便堆栈dump时分析
```java
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
        .setNameFormat("my-sad-thread-%d").build();
Executors.newSingleThreadScheduledExecutor(namedThreadFactory);
```

### ThreadPoolExecutor queue占满自动阻塞submit()
#### 自动抛弃
```java
executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
```

#### 重写rejectedExecution()，调用queue的put方法
```java
executor.setRejectedExecutionHandler(new RejectedExecutionHandler() {
@Override
public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
	if (!executor.isShutdown()) {
		try {
			executor.getQueue().put(r);
			} catch (InterruptedException e) {
			}
		}
	}
});
```
有可能导致死锁
#### CallerRunsPolicy
```java
executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy())
```
当使用这种拒绝策略时，如果workQueue满了，ThreadPoolExecutor就会在调用者线程（即生产者线程）中执行要提交的task——生产者线程帮消费者线程干了自己“不应该干的活”。

#### 重写BlockingQueue的offer()方法

#### 参考
* [让ThreadPoolExecutor的workQueue占满时自动阻塞submit()方法](https://www.codelast.com/%E5%8E%9F%E5%88%9B-%E8%AE%A9threadpoolexecutor%E7%9A%84workqueue%E5%8D%A0%E6%BB%A1%E6%97%B6%E8%87%AA%E5%8A%A8%E9%98%BB%E5%A1%9Esubmit%E6%96%B9%E6%B3%95/)

## Future
基本使用框架：
```java
Future<Void> taskFuture = executor.submit(new Wrapper(task));
try {
	taskFuture.get(timeout, TimeUnit.MILLISECONDS);
}
// 等待过程中，被其它线程终止
catch (InterruptedException e) {
}
// 运行时异常
catch (ExecutionException ex) {
}
// 超时异常
catch (TimeoutException e) {
	// 手动终止Future
	taskFuture.cancel(true);
}
```
参考：
* http://www.baeldung.com/java-future
* http://www.baeldung.com/java-runnable-callable

### ListenableFuture 
Guava中提供，基本使用框架：
```java
ListenableFuture<Void> initFuture = executorService.submit(new TestCall());

Futures.addCallback(initFuture, new FutureCallback<Void>() {
@Override
public void onSuccess(Void result) {
System.err.println("onSuccess end.");
}

	@Override
	public void onFailure(Throwable t) {
		logger.error(t); // java.util.concurrent.CancellationException: Task was cancelled.
		System.err.println("onFailure end.");
	}

}, executorService);

try {
   initFuture.get(2000L, TimeUnit.MILLISECONDS);
}
catch (InterruptedException e) {
    // 外部终止
    logger.error(e);
}
catch (ExecutionException e) {
    // 运行时错误
    logger.error(e);
}
catch (TimeoutException e) {
    // 此处异常不用处理，直接停止future
    logger.error(e);
    initFuture.cancel(true);
}
```

### CompletableFuture
参考：
* [Java CompletableFuture 详解](http://colobu.com/2016/02/29/Java-CompletableFuture/)

## 线程的控制
### 线程优先级
总而言之, 在linux系统, 使用root执行方有效果, 具体参考:
* [What is Java thread priority?](http://www.javamex.com/tutorials/threads/priority_what.shtml)


### 线程执行时间控制
总之，线程中执行的操作可以被interrupt，才能通过异常机制提前终止线程的执行
* [How to Stop Execution After a Certain Time in Java](https://www.baeldung.com/java-stop-execution-after-certain-time)
* [IO vs. NIO – Interruptions, Timeouts and Buffers](https://squirrel.pl/blog/2012/06/05/io-vs-nio-interruptions-timeouts-and-buffers/)
* [basinilya/InterruptibleInputStream.java](https://gist.github.com/basinilya/a5392de106cd890a28742960bcc5cf8c)
* [How to read a blocking InputStream yet being sensitive to Interrupts](https://coderanch.com/t/731635/java/read-blocking-InputStream-sensitive-Interrupts)
* [Interruptible network I/O in Java](https://stackoverflow.com/questions/19380576/interruptible-network-i-o-in-java)

