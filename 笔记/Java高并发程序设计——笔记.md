# Java高并发程序设计——笔记

------

#### 1.2 一些基本概念

##### 1.2.1 同步和异步

​	同步方法调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。异步方法调用更像一个消息传递，一旦开始，方法调用会立即返回，调用者可以继续后续的操作。异步方法通常会在另外一个线程中“真实”地执行。比如后面的FutureTask就是异步。同步和异步，类似于去实体店购物和网购，去实体店（同步）你需要去店里自己拿东西，而网购（异步）你只需要在网上下单，然后就可以去做其他事情了，等到快递送到，用凭证（短信）去取件就行。

##### 1.2.2并发（Concurrency）和并行(Parallelism)

​	并发偏重于多个任务交替执行，而多个任务之间有可能还是串行的。并行是真正意义上的“同时执行”。从严格意义上讲，并行是多个任务真的同时执行，而对于并发来说，这个过程是交替的，一会执行任务A，一会执行任务B，系统会不停地在两者之前切换。但对于外部观察者来说，即使多个任务之间是串行并发的，也会造成多任务间并行执行的错觉。真正的并行只可能出现在拥有多个CPU的系统中。

##### 1.2.3 临界区

​	临界区用来表示一种公共资源或者说共享数据，可以被多个线程使用，但是每一次只能由一个线程使用，一旦临界区被占用，其他线程想要使用这个资源就必须等待。

##### 1.2.4 阻塞（Blocking）和非阻塞(Non-Blocking)

​	阻塞和非阻塞用来形容多线程间的相互影响。比如一个线程占用了临界区资源 ，那么其他所有需要这个资源的线程就必须在这个临界区中等待。等待会导致线程挂起，这种情况就是阻塞。如果占用资源的线程一直不愿意释放资源，那么其他所有阻塞在这个临界区上的线程都不能工作。

​	非阻塞强调没用一个线程可以妨碍其他线程执行，所有线程都会尝试不断向前执行。

##### 1.2.5 死锁（DeadLock）、饥饿（Starvation）和活锁（Livelock）

​	死锁、饥饿、活锁都属于多线程的活跃性问题。如果出现上述情况，那么相关线程可能就不在活跃，也就是说它可能很难再执行下去了。

​	死锁：线程之间互相占用对方需要的临界区资源，每个线程将会一直阻塞。

​	饥饿：一个或多个线程因种种原因无法获得所需要的资源，导致一直无法执行。与死锁相比，饥饿还是有可能解决的（比如高优先级的线程完成了任务，释放资源）。

​	活锁：如果线程都秉承着“谦让”的原则，主动将资源释放给其他线程，那么资源将不断地在线程之间跳动，没有一个线程可以拿到资源正常执行。类似于行人相向而行，都首先让对方走，谦让之后反而会继续拥堵。

#### 1.3 并发级别

​	并发级别分为阻塞、无饥饿、无障碍、无锁、无等待几种。

##### 1.3.1 阻塞

​	线程若是阻塞的，那么在其他线程释放它所需要的资源之前，当前线程无法继续执行。使用synchronized或重入锁，得到的就是阻塞的线程。

##### 1.3.2 无饥饿

​	如果线程之间有优先级，那么对同一个资源的分配，是不公平。对于非公平锁来说，可能会产生饥饿现象；对于公平锁，则不会有饥饿现象出现。

##### 1.3.3 无障碍

​	无障碍是一种最弱的非阻塞调度。非阻塞的调度是一种乐观的策略。当检测到冲突时，线程对自己所做的修改进行回滚。检测可以通过“一致性标记”来实现。线程在操作之前，先读取并保存这个标记，操作完成后再读取，检查这个标记是否修改了。一致则访问没有冲突，不一致则重试操作。在修改数据前，都需要更新这个一致性标记，表示数据不再安全。（如果只有一个线程，那么更新标记之后还会重试操作吗？还是说无障碍不允许修改数据？）

##### 1.3.4 无锁(CAS,Compare And Swap)

​	无锁的并行都是无障碍的。无锁的情况下，所有的线程都能尝试对临界区访问，但不同的是，无锁的并发保证必然有一个线程能在有限步内完成操作离开临界区。

##### 1.3.5 无等待(Wait-Free)

​	无等待要求所有线程都必须在有限步内完成，这样不会引饥饿问题。一种典型的无等待是RCU(Read Copy Update)，所有的读线程是无等待的，但在写数据时，先取得原始数据的副本，接着只修改副本数据(这就是为什么读可以不控制)，修改完成之后，在合适的时机写回数据。

#### 1.5 回到Java：JMM

​	JMM，Java的内存模型，一种规则，保证多个线程间可以有效地、正确地协同工作。JMM的关键技术点都是围绕多线程的原子性、可见性和有序性来建立的。

##### 1.5.1 原子性(Atomicity)

​	原子性是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

##### 1.5.2 可见性(Visibility)

​	可见性是指当一个线程修改了某一个共享变量的值时，其他线程能够立即知道这个修改。

##### 1.5.3 有序性(Ordering)

​	并发时，程序的执行可能就会出现乱序，看起来好像写在前面的代码，会在后面执行。有序性问题是程序在执行时，可能会进行指令重排，重排后的指令与原指令的顺序未必一致。指令重排有一个基本前提，就是保证串行语义的一致性，指令重排不会使串行的语义逻辑发生问题。

###### 哪些指令不能重排：Happen-Before规则

​	1.程序顺序原则：一个线程内保证语义的串行性。

​	2.volatile规则：volatile变量的写先于读发生，这保证了volatile变量的可见性。

​	3.锁规则：解锁(unlock)必然发生在随后的加锁(lock)之前。

​	4.传递性：A先于B，B先于C，那么A必然先于C。

​	5.线程的start()方法先于它的每一个动作。

​	6.线程的所有操作先于线程的终结(Thread.join())。

​	7.线程的中断(interrupt())先于被中断线程的代码。

​	8.对象的构造函数的执行、结束先于finalize()方法。

## 第2章 Java并行程序基础

#### 2.2 线程的基本操作

##### 2.2.1 新建线程

​	线程Thread有一个run()方法，启动的start()方法会新建一个线程并让这个线程执行run()方法。run()方法也能编译通过和正常执行，但是却不会新建一个新线程，而是在当前线程中调用run()方法，只是作为一个普通的方法调用。(不应该用run()方法来开启信线程，它只会在当前线程中串行执行run()方法中的代码)

##### 2.2.2 终止线程

​	Thread类提供了一个stop()方法立刻终止线程，但是不推荐使用，因为stop()方法过于暴力，强力把执行到一半的线程终止，可能会引起一些数据不一致的问题。

##### 2.2.3 线程中断

​	线程中断并不会使线程立即退出，而是给线程发送一个通知，告知目标线程，希望你退出。至于目标线程接到通知后如何处理，则由目标线程自行决定。如果中断后线程立刻无条件退出，又回到stop()方法了。

##### 2.2.4 等待(wait)和通知(notify)

​	当在一个对象实例上调用wait()方法后，当前线程就会在这个对象上等待。等待到其它线程调用了obj.notify()方法为止。工作原理：如果一个线程A调用了object.wait()方法，A就会进入object对象的等待队列。这个等待队列中，可能会有多个线程；当object.notify()方法被调用时，它会从等待队列中随机选择一个线程将其唤醒，这个唤醒不是公平的，是完全随机的。

​	Object.wait()方法和Thread.sleep()方法都可以让线程等待若干时间。除wait()方法可以被唤醒外，另外一个主要区别就是wait()方法会释放目标对象的锁，而Thread.sleep()方法不会释放任何资源。(注意，这两个方式是定义在Object类中的)

##### 2.2.6 等待线程结束(join)和谦让(yield)

###### 1.join()方法

​	有时候，一个线程的输入可能非常依赖于另一个或多个线程的输出，此时这个线程就需要等待依赖线程执行完毕，才能继续执行。

​	join()方法的本质是让调用线程wait()方法在当前线程对象实例上。当线程执行完成后，被等待的线程会在退出前调用nofifyAll()方法通知所有的等待线程继续执行。

​	我目前个人的理解是，A.join()代表当前运行的线程要加入(join)到A线程，所以要等A线程完成以后跟着它一起运行。如下代码所示，addThread线程调用了join()方法，让主线程加入它，所以主线程要等到addThread线程完成后再接着向下运行。

代码举例：

```java
public class Join_Demo {
    public volatile static int i = 0;
    public static class AddThread extends Thread {
        @Override
        public void run() {
            for (i = 0; i < 10000000; i++);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AddThread addThread = new AddThread();
        addThread.start();
        addThread.join();
        //调用了join方法后，会输出10000000，如果没有join，则会输出i的初始值0
        System.out.println(i);
    }
}
```

###### 2.yield()方法

​	这是一个Thread类的静态方法，一旦执行，它会使当前线程让出CPU。但要注意，让出CPU并不表示当前线程不执行了。当前线程让出CPU以后，还会进行CPU资源的争夺，但能否被再次分配到就不一定了。

​	当你觉得一个线程不那么重要，或者优先级比较低，而且又担心它会占用太多的CPU资源，那么可以在适当的时候调用Thread.yield()方法，给与其他的重要线程更多的工作机会。

#### 2.3 volatile于Java内存模型(JMM)

​	用volatile关键字去声明一个变量时，就等于告诉计算机，这个变量极有可能会被某些程序或者线程修改，需要保持该变量的可见性。

​	volatile关键字可以保证操作的原子性，数据的可见性和有序性。但是volatile并不能代替锁，也无法保证一些复合操作的原子性。

```java
public class Volatile_Demo {
    private static volatile boolean ready;
    private static int number;

    private static class ReaderThread extends Thread {
        @Override
        public void run() {
            while (!ready);
            System.out.println(number);
        }
    }
    //当ready变量用volatile声明时，可以看见输出，如果没有，则一直在while(!ready)处死循环
    //此例也说明，我的虚拟机是Server模式，因为Client模式的JIT优化不够，即使没有volatile声明，也能看见输出
    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();
        Thread.sleep(1000);
        number = 42;
        ready = true;
        Thread.sleep(1000);
    }
}
```

#### 2.5 驻守后台：守护线程(Deamon)

​	守护线程是一种比较特殊的线程，就和它的名字一样，是系统的守护者，在后台默默完成一些系统性的服务，比如垃圾回收线程、JIT线程就可以理解为守护线程。与之对应的就是用户线程，用户线程可以认为是系统的工作线程，它会完成这个程序应该要完成的业务操作。当一个Java应用内只有守护线程时，Java虚拟机就会自然退出。

```java
public class Daemon_Demon {
    public static class DaemonT extends Thread {
        @Override
        public void run() {
            while (true) {
                System.out.println("I am alive ~~~");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        DaemonT t = new DaemonT();
        t.setDaemon(true);
        t.start();
        //因为将t设置为了守护线程，所以会在主线程sleep的6s中内执行t的run()方法
        Thread.sleep(6000);
    }
}
```

#### 2.6 线程优先级

​	Thread类中定义了三个静态标量，数字越大表示优先级越高，且范围为1-9。

```java
/**
  * The minimum priority that a thread can have.
  */
 public final static int MIN_PRIORITY = 1;

/**
  * The default priority that is assigned to a thread.
  */
 public final static int NORM_PRIORITY = 5;

 /**
  * The maximum priority that a thread can have.
  */
 public final static int MAX_PRIORITY = 10;
```

代码演示：

```java
public class Priority_Demo {
    public static void main(String[] args) {
        HighPriority highPriority = new HighPriority();
        LowPriority lowPriority = new LowPriority();
        highPriority.setPriority(Thread.MAX_PRIORITY);
        lowPriority.setPriority(Thread.MIN_PRIORITY);
        lowPriority.start();
        highPriority.start();
    }

    public static class HighPriority extends Thread {
        static int count = 0;
        @Override
        public void run() {
            while (true) {
                synchronized (Priority_Demo.class) {
                    count++;
                    if (count > 10000000) {
                        System.out.println("high priority completed ~~~");
                        break;
                    }
                }
            }
        }
    }

    public static class LowPriority extends Thread {
        static int count = 0;
        @Override
        public void run() {
            while (true) {
                synchronized (Priority_Demo.class) {
                    count++;
                    if (count > 10000000) {
                        System.out.println("low priority completed ~~~");
                        break;
                    }
                }
            }
        }
    }
}
```

#### 2.7 线程安全的概念和synchronized

​	volatile关键字并不能保证线程的安全，它只能确保一个线程修改了数据后，其它线程能够看到这个改动，但当两个线程同时修改一个数据时，依然会产生冲突。

​	关键字synchronized的作用是实现线程间的同步。它的工作是对同步的代码加锁，使得每一次，只能有一个线程进入同步块，从而保证线程间的安全性。

​	synchronized有多种用法：

​		1.指定加锁对象：对给定对象加锁，进入同步代码之前要获得给定对象的锁。

​		2.直接作用于实例方法：相当于对当前实例加锁，进入同步代码前要获得当前实例的锁。

​		3.直接作用于静态方法：相当于对当前类加锁，进入同步代码前要获得当前类的锁。

​	synchronized除了用于线程同步，确保线程安全外，还可以保证线程间的可见性和有序性。

​	例如下面这个例子：

​	期待看到的是i = 20000，但是我一次运行的结果是13778。因为线程1和线程2同时读取i = 0，并各自计算得到i = 1，并先后写入这个结果，因此，虽然i++被执行了两次，但实际上i的值只增加了1。

```java
public class Synchronized_Demo1 implements Runnable{
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(instance);
        Thread thread2 = new Thread(instance);
        thread1.start();thread2.start();
        thread1.join();thread2.join();
        System.out.println(i);
    }
    
    static Synchronized_Demo1 instance = new Synchronized_Demo1();
    static volatile int i = 0;

    public static void increase() {
        i++;
    }

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            increase();
        }
    }
}
```

### 第3章 JDK并发包

#### 3.1 同步控制

##### 3.1.1 重入锁

​	重入锁ReentrantLock主要有以下常使用的方法：

```java
void lock();//请求获得锁
void lockInterruptibly() throws InterruptedException;//请求获得锁，并且可以相应中断
boolean tryLock();//请求获得锁，若请求失败则返回false，成功返回true
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;//在给定的时间内请求获得锁，失败返回false，成功true
void unlock();//释放锁
Condition newCondition();//新建一个信号量，用于实现等待和唤醒的功能(类似于Object.wait()和Object.notify())
```

##### 3.1.2 重入锁的搭档：Condition

​	可以用一个重入锁实例创建一个Contion，用于实现await,signal等停止和唤醒的功能

​	基本方法如下：

```java
//需要注意的是，其他线程调用await()方法时需要获得对应的重入锁，并且一个线程被signal()后，需要获得对应的重入锁才能继续运行，见如下示例
void await() throws InterruptedException;//可以相应中断
void awaitUninterruptibly();//不相应中断
long awaitNanos(long nanosTimeout) throws InterruptedException;
boolean await(long time, TimeUnit unit) throws InterruptedException;
boolean awaitUntil(Date deadline) throws InterruptedException;
void signal();
void signalAll();
```

​	Conditon使用示例:

```java
public class ReenterLockCondition_Demo implements Runnable {

    public static void main(String[] args) throws InterruptedException {
        ReenterLockCondition_Demo conditionDemo = new ReenterLockCondition_Demo();
        Thread thread = new Thread(conditionDemo);
        thread.start();
        //此处如果没有让主线程停止以下，则会发生死锁现象
        Thread.sleep(2000);
        lock.lock();
        condition.signal();
        lock.unlock();
    }

    public static ReentrantLock lock = new ReentrantLock();
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try {
            lock.lock();
            condition.await();
            System.out.println("Thread is going on");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

##### 3.1.4 ReadWriteLock读写锁

​	读写锁，读读之间并不阻塞，而写写和独写之间需要等待和持有锁。对于读操作的次数远大于写操作的程序中，读写锁能够大幅提升性能。如下面代码示例：

```java
public class ReadAndWriteLock_Demo {

    public static void main(String[] args) {
        ReadAndWriteLock_Demo demo = new ReadAndWriteLock_Demo();
        Runnable readThread = new Runnable() {
            @Override
            public void run() {
                try {
                    demo.handleRead(readLock);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        Runnable writeThread = new Runnable() {
            @Override
            public void run() {
                try {
                    demo.handleWrite(writeLock, new Random().nextInt());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        for (int i = 0; i < 18; i++) {
            new Thread(readThread).start();
        }

        for (int i = 18; i <= 20; i++) {
            new Thread(writeThread).start();
        }

    }

    private static Lock lock = new ReentrantLock();
    private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private static Lock readLock = readWriteLock.readLock();
    private static Lock writeLock = readWriteLock.writeLock();
    private int value;

    public int handleRead(Lock lock) throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);
            return value;
        }
        finally {
            lock.unlock();
        }
    }

    public void handleWrite(Lock lock, int value) throws InterruptedException {
        try {
            lock.lock();
            Thread.sleep(1000);
            this.value = value;
        } finally {
            lock.unlock();
        }
    }
}
```

##### 3.1.5 倒计时器：CountDownLatch

​	这个工具用来控制线程等待，比如主线程要想继续运行， 必须要等到有n个(n是设定好的)线程完成工作后才能继续执行。示例如下：

```java
public class CountDownLatch_Demo implements Runnable {

    public static void main(String[] args) throws InterruptedException {
        //设定需要有10个线程完成任务
        ExecutorService exec = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            exec.submit(demo);
        }
        //这里让主线程停止等待了
        end.await();
        //有10个线程完成后，主线程继续执行
        System.out.println("Fire!");
        exec.shutdown();
    }

    static final CountDownLatch end = new CountDownLatch(10);
    static final CountDownLatch_Demo demo = new CountDownLatch_Demo();

    @Override
    public void run() {
        try {
            //模拟检查任务
            Thread.sleep(new Random().nextInt(10) * 1000);
            System.out.println("check complete");
            //执行完成后让待完成线程数量-1
            end.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

##### 3.1.8 限流算法

​	1.漏桶算法：利用一个缓冲区，当有请求进入系统时，无论请求的速率如何，都先在缓存区内保存，然后以固定的速率流出缓存区进行处理。特点：无论外部请求压力如何，漏桶算法总是以固定的流速处理数据。漏桶的容积和流出速率是该算法的两个重要参数。

<img src="D:\深入理解JAVA虚拟机笔记图片\屏幕截图 2020-11-15 091712.png" alt="屏幕截图 2020-11-15 091712" style="zoom:50%;" />

​	令牌桶算法：是一种反向的漏桶算法，桶中存放的不再是请求 ，而是令牌。处理程序只有拿到令牌后，才能对请求进行处理。如果没有令牌，那么处理程序要么丢弃请求，要么等待可用的令牌。该算法在每个单位时间产生一个定量的令牌存入桶中，桶的容量是有限的。

<img src="D:\深入理解JAVA虚拟机笔记图片\屏幕截图 2020-11-15 092027.png" alt="屏幕截图 2020-11-15 092027" style="zoom:50%;" />

​	Guava的RateLimiter采用了令牌桶算法。

#### 3.2 线程复用：线程池

​	线程的创建和关闭依然需要花费时间，如果为每一个任务都创建一个线程，则有可能出现创建和销毁线程所占用的时间大于线程执行任务的时间。

​	线程也需要占用内存空间，如果线程数量过于庞大，可能会导致OutOfMemory异常。即使不发生异常，大量的线程回收也会给GC带来很大的压力，延长GC的停顿时间。

​	线程池中，总有一些活跃的线程，需要使用线程时，从池子中取一个空闲线程，工作完成时，并不关闭线程，而是将这个线程退回到线程池中，便于下次使用。

​	使用了线程池后，创建线程变成了从线程池获得空闲线程，关闭线程变成向线程池归还线程。

##### 3.2.3 刨根问底：核心线程池的内部实现

```java
public ThreadPoolExecutor(	  int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

​	线程池是ThreadPoolExecutor类的封装，构造函数如上代码所示。需要具体学习下workQueue和handler。

​	workQueue指被提交但未被执行的任务队列，是一个BolckQueue的接口，仅用于存放Runable对象。主要有以下几种BlockQueue接口：

​	1.SynchronousQueue：直接提交的队列。它是一个特殊的BlockQueue，没有容量。如果使用SynchronousQueue，提交的任务不会被真实地保存，而是将新任务提交给线程执行，如果没有空闲的进程，则尝试创建新线程。如果线程数量已达到最大值，则执行拒绝策略。因此，使用SynchronousQueue队列，通常要设置很大的maximumPoolSize指，否则很容易执行拒绝策略。

​	2.ArrayBlockQueue：有界的任务队列。该类的构造函数必须有一个参数，`public ArrayBlockingQueue(int apacity)`，表示该队列的最大容量。使用时，若有新的任务需要执行，如果线程池的实际线程数小于corePoolsize，则会优先创建新的线程，若大于corePoolSize，则会将新任务加入等待队列；若等待队列已满，无法加入，则在总线程数量不大于maximumPoolSize的前提下，创建新的线程执行任务；若大于maximumPoolSize，则执行拒绝策略。队列是FIFO的。

​	3.LinkedBlockingQueue：无界的任务队列。除非系统资源耗尽，否则该队列不存在任务入队列失败的情况。当有新任务时，若系统的线程数量小于corePoolSize，线程池会生成新的线程执行任务，但当系统的线程数达到corePoolSize后，就不会继续增加了。若还有任务进入，又没有空闲的线程资源，则任务直接进入队列等待。若任务创建和任务处理的速度差距很大，则无界队列会快速增长，直到耗尽系统内存。队列是FIFO的。

​	4.PriorityBlockingQueue：优先任务队列，是带有执行优先级的队列，可以控制任务的执行先后顺序，是一个特殊的无界队列。

​	newFixedThreadPool()的实现：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

​	返回一个corePoolSize和maximumPoolSize大小一样的，并且使用LinkedBlockingQueue任务队列的线程池。当任务提交非常频繁时，该队列会迅速膨胀，从而耗尽系统资源。

​	newCachedThreadPool()的实现：

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

​	newCachedThreadPool()方法返回corePoolSize为0，maximumPoolSize为无穷大的线程池。意味着没有任务时，线程池内无线程，任务被提交时，该线程池会使用空闲的线程执行任务，若无空闲线程，则将任务加入SynchronousQueue队列，该队列是直接提交的队列，所以它会迫使线程池增加新的线程执行任务。任务执行完毕后，由于corePoolSize为0，因此空闲线程会在指定时间(60s)内被回收。

#### 3.3 不要重复发明轮子：JDK的并发容器

##### 3.3.1 并发集合简介

​	1.ConcurrentHashMap：是一个高效的并发HashMap，可以理解为是线程安全的HashMap。

​	2.CopyOnWriteArrayList：在读多写少的场合，这个List性能非常好，远远优于Vector。

​	3.ConcurrentLinkedQueue：高效的并发队列，使用链表实现。可以看成是一个线程安全的LinkedList。

​	4.BlockingQueue：一个接口，JDK内部通过链表、数组等方式实现了这个接口，表示阻塞队列，非常适合作为数据共享的通道。

​	5.ConcurrentSkipListMap：这是一个Map，使用跳表的数据结构进行快速查找。

​	另外，java.util下的Vector也是线程安全的(性能和上述的没法比)，Collections工具类也可以将任意的集合包装成线程安全的集合。

### 第4章 锁的优化及注意事项

#### 4.1 提高锁性能的措施

​	锁的竞争必然会导致程序的整体性能下降，这里给出一些优化措施。

##### 4.1.1 减少锁持有时间

​	减少锁的持有时间有助于降低锁冲突的可能性，进而提高系统的并发能力。

​	例如下面代码：

```java
public synchronized void synMethod() {
        otherCode1();
    	//只有这个方法需要同步
        mutexMethod();
        otherCode2();
    }
```

​	otherCode1()和otherCode2()并不需要同步，因此可以将代码改写如下：

```java
public void synMethod2() {
        otherCode1();
        synchronized (this) {
            mutexMethod();
        }
        otherCode2();
    }
```

​	改进的代码中，只对mutexMethod()做了同步，锁占用的时间减少了，能有更高的并行度。

​	在JDK源码中的体现：

```java
public Matcher matcher(CharSequence input) {
        if (!compiled) {
            synchronized(this) {
                if (!compiled)
                    compile();
            }
        }
        Matcher m = new Matcher(this, input);
        return m;
    }
```

##### 4.1.2 减小锁粒度

​	减小锁粒度，就是指缩小锁定对象的范围，从而降低锁冲突的可能性，进而提高系统的并发能力。

​	对于HashMap的get()和put()方法，并发时自然能够想到对整个HashMap加锁从而得到一个线程安全的对象，但是这样锁粒度太大。对于ConcurrentHashMap类，它内部细分成了若干个小的HashMap，称之为段(SEGMENT)，默认情况下，一个ConcurrentHashMap类可以被细分为16个段。

​	如果需要在ConcurrentHashMap中添加一个表项，并不是将整个HashMap加锁，而是根据hashcode得到该表项应该放入哪个段中，然后对该段加锁，完成put()方法操作。

​	由于默认有16个段，因此幸运的话，ConcurrentHashMap类可以接受16个线程同时插入(不同的段)，从而大大提高吞吐量。但是当需要获得全局锁时，例如size()方法，首先采用无锁的方式求和，失败后则获得所有段的锁，求和后再释放。因此高并发场合ConcurrentHashMap的size()方法性能差于同步的HashMap。

​	只有在类似于size()方法获取全局信息的方法调用并不频繁时，这种减小锁粒度的方法才能在真正意义上提高系统的吞吐量。

##### 4.1.3 用独写分离锁来替换独占锁

​	使用独写分离锁来替代独占锁是减小锁粒度的一种特殊情况。如果说减小锁粒度是通过分割数据结构实现的，那么独写分离锁则是对系统功能点的分割。

##### 4.1.4 锁分离

​	将读写锁的思想进一步延申，就是锁分离。

​	在LinkedBlockingQueue中，take()和put()方法虽然都对当前队列进行修改，但由于LinkedBlockingQueue是基于链表的，因此两个操作分别作用于队列的前端和尾端，从理论上说，两者不冲突。

​	JDK的实现中，用两把不同的锁分离了take()和put()的操作。

```java
 /** Lock held by take, poll, etc */
    private final ReentrantLock takeLock = new ReentrantLock();

    /** Wait queue for waiting takes */
    private final Condition notEmpty = takeLock.newCondition();

    /** Lock held by put, offer, etc */
    private final ReentrantLock putLock = new ReentrantLock();

    /** Wait queue for waiting puts */
    private final Condition notFull = putLock.newCondition();
```

​	定义了takeLock和putLock，分别在tak()和put()方法中使用，因此两个方法相互独立，它们之间没有锁竞争的关系，只在take()和put()方法自身间存在竞争，从而削弱了锁竞争的可能性。

##### 4.1.5 锁粗化

​	虚拟机在遇到一连串连续地对同一个锁不断进行请求和释放的操作时，会把所有的锁操作整合成对锁的一次请求，从而减少对锁的请求同步次数，这个操作叫做锁的粗化。

​	例如：

```java
for (int i = 0; i < n; i++) {
            synchronized (lock) {
                
            }
        }
```

​	更合理的做法是：

​	

```java
 synchronized (lock) {
            for (int i = 0; i < n; i++) {

            }
        }
```

#### 4.2 Java虚拟机的锁优化

​	这部分内容来自《深入理解Java虚拟机》

##### 4.2.1 自旋锁

​	挂起线程和恢复线程都需要转入内核态中完成完成，会给Java虚拟机的并发性带来很大压力，而有些共享数据的锁定状态只会持续很短时间，为了这段时间去挂起和恢复线程并不值得。在多处理器或多处理器核心的物理机器上，可以让请求锁的那个线程“稍微等一会”，但不放弃处理器的执行时间，看看持有锁的线程是否很快释放锁。为了使线程等待，让线程执行一个自旋，这就是自旋锁。

​	自旋等待需要有一定的限度，如果超过了限定的次数仍然没有获得锁，就应当使用传统的方式去挂起线程。自旋的次数默认是10次，也可以通过-XX:PreBlockSpin来设置。

##### 4.2.2 自适应自旋

​	这是JDK6对自旋锁的优化。自适应意味着自旋的时间不再是固定的了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么就会认为这次自旋也会再次成功获得锁，进而允许自旋等待持续相对更长的时间；如果对于某个锁，自旋很少成功获得过锁，那么以后获取这个锁时可能直接省略自旋过程，避免浪费处理器资源。

##### 4.2.3 锁消除

​	根据逃逸技术分析，判断一段代码中，在堆上的数据都不会逃逸出去被其他线程访问到，那么就可以把它们当作栈上数据对待，认为是线程私有的，同步加锁自然就无须再进行。

​	例如这段代码：

```java
    public String[] createStrings() {
        Vector<String> v = new Vector<>();
        for (int i = 0; i < 100; i++) {
            v.add(Integer.toString(i));
        }
        return v.toArray(new String[]{});
    }
```

 其中用到了Vector.add()方法，看看该方法的源码：

```java
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
```

​	很明显这个方法有一个synchronized，因此每次使用add()时都要求先获得锁。但是在这个例子中，变量v只在createStrings()方法中使用，仅仅是一个局部变量。局部变量是在线程栈上分配的，属于线程私有的数据，不可能被其他线程访问，在这种情况下，Vector内部的所有加锁同步都是没有必要的，因此虚拟机检测后，会将这些无用的锁操作去除。

​	使用-XX:DoEscapeAnalysis参数打开逃逸分析，使用-XX:+EliminateLocks参数打开锁消除。

##### 4.2.4 锁粗化

​	同4.1.5锁粗化。

##### 4.2.5 轻量级锁

​	轻量级锁是为了在没有竞争的前提下减少重量级锁使用操作系统互斥量产生的性能消耗。

​	如果没有竞争，轻量级锁通过CAS操作成功避免了使用互斥量的开销。

​	在代码即将进入同步块时，如果同步对象没有被锁定，虚拟机将在当前线程的栈帧中建立一个锁记录空间，存储锁对象目前Mark Word的拷贝。然后虚拟机使用CAS尝试把对象的Mark Word更新为指定锁记录的指针，如果更新成功就代表该线程拥有了锁，锁标志位变为00，表示处于轻量级锁定状态。

​	如果更新失败就意味着至少存在一条线程与当前线程竞争。虚拟机检查对象的Mark Word是否指向当前线程的栈帧，如果是则说明当前线程已经拥有了锁，直接进入同步块执行，否则说明锁对象被其他线程抢占。如果出现两条以上线程争用同一个锁，轻量级锁就不再有效，将膨胀为重量级锁，锁标志状态变为10，此时Mark Word存储的就是指向重量级锁的指针，后面等待锁的线程也必须阻塞。

​	解锁同样通过CAS进行，如果对象Mark Word仍然指向线程的锁记录，就用CAS把对象当前的Mark Word和线程复制的Mark Word替换回来。假如替换成功，同步过程就顺利完成了，如果失败则说明有其他线程尝试过获取该锁，就要在释放锁的同时唤醒被挂起的线程。

##### 4.2.6 偏向锁

​	如果说轻量级锁是在无竞争的情况下使用CAS操作去消除同步使用的互斥量，那偏向锁就是在无竞争的情况下，把整个同步都消除掉，连CAS操作都不做了。

​	偏向锁是为了在没有竞争的情况下减少锁开销，锁会偏向于第一个获得它的线程，如果在执行过程中锁一直没有被其他线程获取，那持有偏向锁的线程将不需要进行同步。

​	当锁对象第一次被线程获取时，虚拟机会将对象头中的偏向模式设为1，同时使用CAS把获取到锁的线程ID记录在对象的Mark Word中。如果CAS成功，持有偏向锁的线程以后每次进入锁相关的同步块都不再进行任何同步操作。

​	一旦有其他线程尝试获取锁，偏向模式立即结束，根据锁对象是否处于锁定状态决定是否撤销偏向，后续同步按照轻量级锁那样进行。

#### 4.5 死锁

​	死锁就是两个或者多个线程相互占用对方需要的资源，都不进行释放，导致彼此之间相互等待对方释放资源，产生了无限制等待的现象。死锁一旦发生，如果没有外力介入，这种等待将永远存在，从而对程序产生严重的影响。

​	死锁的通常表现就是相关的进程不再工作，并且CPU占用率为0。如果想避免死锁，除使用无锁的函数之外，还有一种有效的做法是使用重入锁，通过重入锁的中断或者限时等待可以有效规避死锁带来的问题。