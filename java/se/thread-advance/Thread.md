## [JUC](./juc.md)

## Thread

### 相关概念: `3 - 3 - 3 - 4`

1. 进程:

   - 是程序的一次执行过程
   - 是程序执行的`基本单位`, 因此进程是动态的.
   - 一个程序即是从`创建, 运行到消亡`的过程.

2. 线程

   - 是`程序`执行的最小单元
   - 是一个比进程更小的`执行单位`
   - 一个进程在其执行的过程中可以产生多个线程.

3. 区别

   - 系统在产生一个线程, 或是在各个线程之间作切换工作时, 负担要比进程小得多: `同类的多个线程共享同一块内存空间和一组系统资源`
   - `线程也被称为轻量级进程`: `线程 ID + 当前指令指针 PC + 寄存器集合 + 堆栈`
   - **使用多线程而不是用多进程去进行并发程序的设计, 是因为线程间的切换和调度的成本远远小于进程.**

4. 多线程:

   - 多线程就是多个线程`同时或交替`运行.
   - 单核 CPU 的话是顺序执行, 也就是交替运行: `时间片`.
   - 多核 CPU 的话, 因为每个 CPU 有自己的运算器, 所以在多个 CPU 中可以同时运行.
   - 多线程是开发高并发系统的基础, 利用好多线程机制可以大大提高系统整体的并发能力以及性能.

5. 线程中断&线程阻塞

   - 线程阻塞: 代码在执行过程中, 在某个地方暂停了
   - 线程中断: 机器能自动停止正在运行的程序并转入处理新情况的程序, 处理完毕后又返回原被暂停的程序继续运行

### 线程

1. 实现线程的 4 种方法: `前三种都不用, new Thread(){} 相当于每次都招聘一个员工, 干完活就辞退`

   - `java.lang.Thread`:

   ```java
   public static void main(String[] args) {
       log.info("main method start ...");

       CThread cThread = new CThread();
       cThread.start();

       log.info("main method end ...");
   }

   public static class CThread extends  Thread {
       @Override
       public void run() {
       log.info("thread-id: {}", Thread.currentThread().getId());
       int i = 10 / 2;
       log.info("logic result: {}", i);
       }
   }
   ```

   - `java.lang.Runnable`

   ```java
   @SneakyThrows
   public static void main(String[] args) {
       log.info("main method start ...");
       Thread cThread = new Thread(new CRunnable());
       cThread.start();
       log.info("main method end ...");

       log.info("main method start 2...");
       Integer i=0 ;
       FutureTask<Integer> futureTask = new FutureTask<>(cThread, i);
       new Thread(futureTask).start();
       Integer integer = futureTask.get();
       log.info("main method end 2...");
   }

   public static class CRunnable implements Runnable {
       @Override
       public void run() {
       log.info("thread-id: {}", Thread.currentThread().getId());
       int i = 10 / 2;
       log.info("logic result: {}", i);
       }
   }
   ```

   - `Callable + FeatureTask`: 可以拿到线程执行结果, **`每个 futureTask 只能有效的与一个线程绑定`**

   ```java
   @SneakyThrows
   public static void main(String[] args) {
       log.info("main method start ...");
       FutureTask<Integer> futureTask = new FutureTask<>(new CCallabel());
       new Thread(futureTask, "Callable Thread1").start();
       new Thread(futureTask, "Callable Thread2").start(); // 无效的

       // block
       Integer integer = futureTask.get();
       log.info("main method end ...");
   }

   public static class CCallabel implements Callable<Integer> {
       @Override
       public Integer call() throws Exception {
           log.info("thread-id: {}", Thread.currentThread().getId());
           int i = 10 / 2;
           log.info("logic result: {}", i);
           return i;
       }
   }
   ```

   - `线程池: 2`: 所有的多线程都应该使用线程池管理, 向线程池中提交任务, 等待执行; 应该整个系统持有几个线程池: `核心业务, 非核心业务`

   ```java
   /**
   * Thread pool has two method to create thread:
   *
   * <pre>
   *     1. use {@link java.util.concurrent.ExecutorService }
   *     2. use {@link java.util.concurrent.ThreadPoolExecutor}
   * </pre>
   *
   * @author zack <br>
   * @create 2021-01-13<br>
   * @project javase <br>
   */
   @Slf4j
   public class PoolCreate {
       @Deprecated private static ExecutorService service = Executors.newScheduledThreadPool(10);

       private static ThreadPoolExecutor executor =
           initThreadPoolExecutor(
               5,
               200,
               10,
               TimeUnit.SECONDS,
               new LinkedBlockingDeque<>(10_0000),
               Executors.defaultThreadFactory(),
               new ThreadPoolExecutor.AbortPolicy());

       public static void main(String[] args) {
           log.info("main method start ...");
           Future<?> submit = service.submit(new RunnableCreate.CRunnable());
           log.info("main method end ...");

           log.info("main method start 2 ...");
           Future<?> submit1 = executor.submit(new RunnableCreate.CRunnable());
           log.info("main method end 2 ...");
       }

       public static ThreadPoolExecutor initThreadPoolExecutor(
           int corePoolSize,
           int maximumPoolSize,
           long keepAliveTime,
           TimeUnit unit,
           BlockingQueue<Runnable> blockingQueue,
           ThreadFactory threadFactory,
           RejectedExecutionHandler handler) {

           return new ThreadPoolExecutor(
               corePoolSize, maximumPoolSize, keepAliveTime, unit, blockingQueue, threadFactory, handler);
       }
   }

   ```

   - 共性区别

     1. 每个线程都只能 start() 一次, start() 不会立即执行
     2. `1,2`不能得到线程的返回值, `3`可以
     3. [稳定]: `1,2,3` 不能做到资源控制, `4` 可以

   - 线程池的好处

     1. 降低资源的消耗: 通过`重复`利用已经创建好的线程降低线程创建和销毁带来的消耗
     2. 提高响应速度: `线程池中有就绪的线程, 可以直接拿来执行任务`
     3. 提高线程的可管理性: `线程池会根据当前系统的状态合理的优化池内线程, 可以减少创建和销毁的开销, 提高了稳定性`

2. Runnable 和 Thread

   - 本质: **`Runnable 接口的实现类并不是真正的线程类, 只是线程运行的目标类`**, 需要借助 Thread#start() 才能执行
   - `Runnable` 将线程执行单元从控制中剥离出来, 只是一个可运行的逻辑单元, 一个标记接口, 而不是线程一个是实现, 一个是接口
   - 共享变量:
     1. Thread 实现变量共享, 则需要 resource 是静态的: 因为多线程会 `new XxThread()`;
     2. Runnable 接口`适合`于资源的共享, 是不需要的将资源定义成 static, **将 resource 写在方法外部会成为共享变量**
   - Runnable 接口的实现就像是一种策略: `调用 start 方法, 会在执行一些固定的逻辑后调用一个可变的策略方法`

   ```java
   @Slf4j
   public class RunnableShareVariable {

       public static void main(String[] args) {
           AppleRunnable appleRunnable = new AppleRunnable();
           new Thread(appleRunnable).start();
           new Thread(appleRunnable).start();
           new Thread(appleRunnable).start();
       }

       public static class AppleRunnable implements Runnable {
           // 这个变量是共享的: new AppleRunnable() 多个时彼此独立
           private int appleCount = 500;
           // 一次拿一个
           @SneakyThrows
           private synchronized boolean getApple() {
               if (appleCount > 0) {
                   appleCount--;
                   TimeUnit.MICROSECONDS.sleep(1000);
                   log.info(Thread.currentThread().getName() + " 拿走了一个苹果, 还剩下" + appleCount + "个苹果！");
                   return true;
               } else {
                   log.info(Thread.currentThread().getName() + " 已经死了！");
                   return false;
               }
           }

           /** 不停的拿, 拿到没有结束 */
           @Override
           public void run() {
               boolean flag = getApple();
               // 多线程这里一定要写 true, 不能时 if
               while (flag) {
                   flag = getApple();
               }
           }
       }
   }
   ```

   ```java
   @Slf4j
   public class ThreadShareVariable {

       public static void main(String[] args) {
           new AppleThread("0001").start();
           new AppleThread("0002").start();
       }

       @NoArgsConstructor
       public static class AppleThread extends Thread {
           // 此处必须是 static 修饰, 才能达到共享资源的效果
           private static int i;

           @Override
           public void run() {
           for (; i <= 1000; i++) {
               log.info("{}: {}", Thread.currentThread().getName(), i);
           }
           }

           public AppleThread(String name) {
           super(name);
           }
       }
   }
   ```

   - 可以继承父类: 继承 Thread 的就不能继承父类了(Java 是单继承的)
   - Runnable 接口必须实现 run 方法

3. diff between Callable and Runnable

   - **泛型**
   - **返回值**
   - 方法名不同
   - **异常抛出**
   - **异步回调[FutureTask]**

   ```java
   public class CallableDemo {
       private static final Logger LOG = LoggerFactory.getLogger(CallableDemo.class);

       public static void main(String[] args) throws ExecutionException, InterruptedException {
       FutureTask<Integer> future = new FutureTask<>(() -> {
                   TimeUnit.SECONDS.sleep(4);
                   return 200;
               });

       // call() will just execute one time, unless use many new FutureTask<>(new Callable() {});
       new Thread(future, "Callable Thread1 implement").start();
       new Thread(future, "Callable Thread2 implement").start();

       LOG.info(Thread.currentThread().getName() + ": Main Thread execute");

       Integer integer = future.get();
       LOG.info("Callable Thread response: " + integer);
       }
   }

   class RThread implements Runnable {
       @Override
       public void run() {}
   }

   class CThread implements Callable<Integer> {
       @Override
       public Integer call() throws Exception {
       TimeUnit.SECONDS.sleep(4);
       return 200;
       }
   }
   ```

4. FutureTask/Callable

   - one `new FutureTask<>(new CThread())`, no matter how many thread, Callable call() method will just execute one time.
   - FutureTask often used to calculate time-consuming task
   - thread method should be in last, when execute thread method will block main thread.
   - use get() to get FutureTask result, if finish calcutate, it will nerver recalculate and cannot be canceled
   - if call get() method, calculate donot finish, the thread will be blocked until finish calculation

#### 线程方法

1. java 应用程序的 main 函数是一个线程, 被 jvm 启动时调用, 线程名字就是 main
2. jvm 启动后实际上是有多个线程, 但至少有一个是非守护线程
   - 守护线程可以应用为 health check
   - 守护线程依赖于非守护线程存在

##### 构造方法

- init(ThreadGroup group, Runnable targer, String name, long stackSize, AccessControlContext accessor)
- 如果没有传递 Runnable 接口或者复写 Thread 的 run 方法, start() 则不会执行 run 方法[控制逻辑]
- 如果没有传入 ThreadGroup, 新建的线程会使用父线程的 Group 作为该线程的 Group, 此时子线程和父线程将会在同一个 ThreadGroup
- 如果没有传入 stackSize[表示该线程占用的 stack 大小], 则默认是 0[表示忽略该参数]. stackSize 平台相关, 会被 JNI 函数使用: `-Xss10M`

1. Thread():

   - 创建线程 Thread 对象, 默认有一个线程名, 以 `Thread-`开头, 从 0 开始

2. Thread(String name)
3. Thread(Runnable able)
4. Thread(Ruunable able, String able)
5. Thread(ThreadGroup group, String name)
6. Thread(ThreadGroup group, Runnable target)
7. Thread(ThreadGroup group, Runnable target, String name)
8. Thread(ThreadGroup group, Runnable target, String name, long stackSize)

##### 其他方法

1. run(): 直接调用 Thread 的 `run()` 方法, 不会产生新的线程, 只是会执行相关的逻辑

   ```java
   public static void main(String[] args) {
       Thread t = new Thread(() -> log.info(Thread.currentThread().getName()));

       t.run(); // log's thread name is main thread
   }
   ```

2. start():

   - 调用 `start()` 方法会在执行一些模板化的逻辑之后, 调用 run() 方法
   - 这里使用了 模板方法 的概念
   - start() 方法是 `synchronized` 修饰的
   - 底层是使用 start0 的 native 方法

3. setDaemon(bool)

   - 设置是否为守护线程
   - 必须在 start() 之前

4. yield(): `执行状态`可以把当前线程重新置入抢 CPU 时间的队列[获取锁]
5. wait(): 使线程进入阻塞态, 放弃锁
6. sleep(): 通知 CPU 在指定的时间内不参与 CPU 的竞争[使得低优先级的也能执行], 但是线程不会失去任何监视器的所有权, `不会放弃锁`

   - sleep 是 Thread 的方法, wait 是 Object 的方法
   - sleep 不会释放锁[monitor], wait 会释放锁并加入获取锁[monitor]的队列
   - sleep 不依赖与锁[monitor], wait 需要
   - sleep 不需要被唤醒, wait 需要[notify]

7. join(): `当前线程调用其他线程`
8. [interrupt()](https://www.zhihu.com/question/41048032/answer/252905837): interrupt() 并不能真正的中断线程, interrupt 之后的代码还是会执行的
9. isAlive(): 判断线程是否存活
10. `setPriority(MAX_PRIORITY)`: 不准确, 不使用

    ```java
    @Slf4j
    public class Test {

    public static void main(String[] args) {
        new JoinSleepThread().start();
        new Thread(new YieldThread()).start();

        testInterrupt();
        interruptThread();
    }

    /**
    * https://www.zhihu.com/question/41048032/answer/252905837
    *
    * <pre>
    *     1. 线程调用 interrupt() 时: 线程执行不收影响
    *        - 如果线程处于被阻塞状态[例如处于sleep, wait, join 等状态]，那么线程将立即退出被阻塞状态, 并抛出一个 {@link InterruptedException }, 仅此而已
    * 　　    - 如果线程处于正常活动状态, 那么会将该线程的中断标志设置为 true, 仅此而. 被设置中断标志的线程将继续正常运行， 不受影响
    * </pre>
    */
    @SneakyThrows
    private static void testInterrupt() {
        // 次线程, 主线程是 main[函数本身]
        Thread thread = new JoinSleepThread();
        // 判断线程是否活着: false
        log.info("before start: {}", thread.isAlive());
        thread.start();
        // true
        log.info("after start: {}", thread.isAlive());

        thread.interrupt();
        // true
        log.info("after interrupt: {}", thread.isAlive());

        thread.join();
        // false
        log.info("after join: {}", thread.isAlive());
        // 已经执行过的线程, 再次调用会出现异常, IllegalThreadStateException
        // thread.start();
    }

    /**
    * Real interrupt thread: interrupt 之后的代码还是会执行的
    *
    * <pre>
    *    1. interrupt() 并不能真正的中断线程, 需要被调用的线程自己进行配合才行. 中断线程:
    *        - 在正常运行任务时, 经常检查本线程的中断标志位, 如果被设置了中断标志就自行停止线程
    *        - 在调用阻塞方法时正确处理 {@link InterruptedException }, 例如, catch异常后就结束线程
    * </pre>
    */
    @SneakyThrows
    private static void interruptThread() {
        Thread thread =
            new Thread(
                () -> {
                    while (!Thread.interrupted()) {
                        // 正常任务代码
                        log.info("{} interrupt.", Thread.currentThread().getName());
                    }
                    // 中断处理代码
                    // 可以在这里进行资源的释放等操作
                });
        thread.start();
        TimeUnit.MICROSECONDS.sleep(1000);
        thread.interrupt();
    }

    public static class JoinSleepThread extends Thread {
        @SneakyThrows
        @Override
        public void run() {
             for (int i = 0; i <= 100; i++) {
                 if (i == 10) {
                    TimeUnit.SECONDS.sleep(1000);
                 }
                 log.info("{}: {}", Thread.currentThread().getName(), i);
             }
        }
    }

    /**
    * Test Sleep method:
    *
    * <pre>
    *      1. 使当前线程从执行状态[运行状态]变为可执行态[就绪状态]
    *      2. cpu 会从众多的可执行态里选择, 所以还是可能选择执行当前线程
    *      3. yield 的本质是把当前线程重新置入抢 CPU 时间的队列
    *  </pre>
    */
    public static class YieldThread implements Runnable {
        // shared variable
        int i;

        @Override
        public void run() {
            for (; i < 100; i++) {
                if (i == 10) {
                    // 执行状态下的线程可以调用 yield 方法, 该方法用于主动出让 CPU 控制权.
                    Thread.yield();
                }

                log.info("{}: {}", Thread.currentThread().getName(), i);
            }
        }
    }

    /** 设置优先级, 但是不一定准, 所以不用 */
    private static void testPriority() {
        Thread priority = new JoinSleepThread();
        priority.start();

        priority.setPriority(MAX_PRIORITY);
        for (int i = 0; i < 10; i++) {
        log.info("{}: {}", Thread.currentThread().getName(), i);
        }

        log.info("priority thread: {}", priority.getPriority());
        log.info("main: {}", Thread.currentThread().getPriority());
    }

    /**
     * 在非 Main 函数中调用线程的 start 方法, 则 run 函数不会被执行<br>
     *
     * <pre>
     *     1. 可以调用 thread.run() 去运行线程
     *     2. 可以使用 sleep 使得 new 的线程有执行的时间
     * </pre>
     */
    @org.junit.Test
    public void testNoMainSleep() {
        Thread thread = new JoinSleepThread();
        thread.start();
        // thread.run();
    }

    /**
    * 告诉操作系统: 在未来的多少毫秒内不参与 CPU 竞争
    *
    * <pre>
    *    1. Thread.Sleep(0)的作用: 触发操作系统立刻重新进行一次CPU竞争
    *    2. 这也是我们在大循环里面经常会写一句Thread.Sleep(0), 因为这样就给了其他线程比如Paint线程获得CPU控制权的权力, 这样界面就不会假死在那里
    * </pre>
    */
    public static void testSleep() {
        Thread thread = new JoinSleepThread();
        thread.start();
        for (int i = 0; i < 100; i++) {
        log.info("main - {}: {}", Thread.currentThread().getName(), i);
        }
    }

    /**
    * 作用就是同步, 它可以使得线程之间的并行执行变为串行执行, 本质为: wait <br>
    *
    * <pre>
    *    1. join 的意思是使得放弃当前线程的执行, 并返回对应的线程, 并执行调用 join 的线程
    *    2. 在 A 线程中调用了 B 线程的 join() 方法时, 表示只有当 B 线程执行完毕时, A 线程才能继续执行
    *        - A 线程中调用了 B 线程的 join 方法, 则相当于 A 线程调用了 B 线程的 wait 方法, 在调用了 B 线程的 wait 方法后, A 线程就会进入阻塞状态
    *    3. join(10) 表示 main 线程会等待 t1 线程 10 毫秒, 10 毫秒过去后, main 线程和 t1 线程之间执行顺序由串行执行变为普通的并行执行
    *    4. join 方法可以在 start 方法前调用时, 并不能起到同步的作用
    * </pre>
    */
    public static void testJoin() {
        Thread thread = new JoinSleepThread();
        thread.start();

        for (int i = 0; i < 100; i++) {
        if (i == 10) {
            try {
                // main 执行的输出会被暂停, 等到 thread 执行完了 main 才有机会继续执行
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        log.info("main -- {}: {}", Thread.currentThread().getName(), i);
        }
    }
    ```

11. 关于线程通信

- `wait(), notify(), notifyAll()`: 这些方法要在同步方法中调用
- 当一个线程正在使用同步方法时, 其他线程就不能使用这个同步方法, 而有时涉及一些特殊情况:

  ```js
  当一个人在一个售票窗口排队买电影票时, 如果她给售票员的不是零钱,
  而售票员有没有售票员找她, 那么她必须等待[wait()], 并允许后面的人买票, 以便售票员获取零钱找她,
  如果第 2 个人也没有零钱, 那么她俩必须同时等待.
  ```

#### 线程状态:

1. NEW
2. RUNNABLE[RUNNING/READY]
3. BLOCKED
4. WAITING
5. TIMED_WAITING
6. TERMINATED

   ![avatar](/static/image/java/thread-status.png)

#### 线程池-Executors

1. Executors.newFixedThreadPool(10): `LinkedBlockingQueue`

   - 创建一个定长的线程池: core == pool-max == 10, 都不可以回收
   - 实现控制线程的最大并发数
   - 不使用的原因: LinkedBlockingQueue 超出的任务会在 queue 里等待[queue 无限大]
   - 适用: 执行`一个`长期的`任务`, 性能好很多
   - Queue 无限长会导致 OOM

2. Executors.newCachedThreadPool(): `SynchronousQueue`

   - 创建一个可以缓存的线程池: core = 0, pool-max == Integer.MAX_VALUE 都可以回收
   - 如果线程池长度超过处理需要, 可以灵活的回收线程
   - 若无可回收则新建线程
   - 不使用的原因: 线程数量可以 Integer.MAX_VALUE
   - 适用: 执行很多`短期异步`的小程序或者`负载较轻`的服务器
   - 会创建很多线程导致 CPU 100% 且线程数量过多的话也会导致 OOM

3. Executors.newScheduledThreadPool(10): `DelayedWorkQueue`

   - 创建一个定长的线程池: core == 10, pool-max == Integer.MAX_VALUE, 大于 core 的可以回收
   - 支持定时和周期任务的执行
   - 不使用的原因: 线程数量可以 Integer.MAX_VALUE

4. Executors.newSingleThreadExecutor(): `LinkedBlockingQueue`

   - 创建一个单线程化的线程池: core = max = 1, 不可以回收
   - 只会使用唯一的线程来工作
   - 不使用的原因: LinkedBlockingQueue 超出的任务会在 queue 里等待[queue 无限大]
   - 适用: `一个任务`一个线程执行的任务场景
   - Queue 无限长会导致 OOM

5. Executors.newWorkStealingPool(int):

   - java8 新增, 使用目前机器上可以的处理器作为他的并行级别

6. executor.shutdown();
7. submit()/execute()
   - execute() 参数 Runnable
   - excute(Ruunable x)没有返回值. 可以执行任务
   - submit() 参数(Ruunable)或(Ruunable 和结果)或(callable)
   - submit(Callable x)有返回值, 返回一个 Future 类的对象
   - submit() 方便 Exception 处理

#### 线程池-ThreadPoolExecutor

##### 状态

1. Running: 可以接受任务和处理已添加的任务
2. Shutdown: 不接受新的任务, 可以处理已添加的任务
3. Stop: 不接受新的任务, 不处理已添加的任务, 且中断正在执行的任务
4. Tidying: 所有的任务都已经终止, ctl 记录的任务数量为 0, ctl[负责记录线程池的运行状态和活动线程的数量]
5. Terminated: 线程池彻底终止

   ```java
   // 一个 Integer 32 bit: 前三bit 表示线程池的状态; 后面的29bit表示当前工作线程的数量
   private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
   private static final int COUNT_BITS = Integer.SIZE - 3;
   private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

   // runState is stored in the high-order bits
   private static final int RUNNING    = -1 << COUNT_BITS;  // 111x xxxx xxxx xxxx xxxx xxxx xxxx xxxx
   private static final int SHUTDOWN   =  0 << COUNT_BITS;  // 000x xxxx xxxx xxxx xxxx xxxx xxxx xxxx
   private static final int STOP       =  1 << COUNT_BITS;  // 001x xxxx xxxx xxxx xxxx xxxx xxxx xxxx
   private static final int TIDYING    =  2 << COUNT_BITS;  // 010x xxxx xxxx xxxx xxxx xxxx xxxx xxxx
   private static final int TERMINATED =  3 << COUNT_BITS;  // 011x xxxx xxxx xxxx xxxx xxxx xxxx xxxx
   ```

##### flow

1. 线程池创建, 准备好 core 数量的核心线程, 准备接受任务
2. 新的任务进来, 用 core 准备好的空闲线程执行
   - core 满了, 九江再进来的任务放入阻塞队列, 空闲的 core 就会自己去 queue 中回去任务并执行
   - queue 满了, 才会开新的线程执行, 直到达到线程的最大数量
   - 如果线程数量已是 max, 还有新的任务进来[且 queue 满了, 否则会放入 queue], 就会使用 handler 进行拒绝
   - 任务执行完了, max 个数的线程空闲下来, 则 max - core 个线程会在 keepAliveTime 之后被释放掉， 最终使得线程数量达到 core 个
3. 所有的线程都是由指定的 factory 创建的
4. interview： 一个线程池 core: 7, max: 20, queue: 50, 此时 100 并发

   - 先有 7 个能被直接被执行
   - 50 个进入 queue
   - 之后开 13 个线程继续执行: **这里的执行可能早于 queue 中的**
   - 余下的 30 个使用 handler 进行拒绝
   - **提交优先级: core -- queue -- new thread**
   - **执行优先级: core -- new thread -- queue**

5. core

   - [flow diagram](https://www.processon.com/view/5ffe4c0c7d9c080e58bec180)

   ```java
   public void execute(Runnable command) {
       if (command == null)
           throw new NullPointerException();

       int c = ctl.get();
       // 当前线程总数小于 core-pool-size 则创建线程执行
       if (workerCountOf(c) < corePoolSize) {
           if (addWorker(command, true))
               return;
           c = ctl.get();
       }
       // core-pool-size 最大则入 queue
       if (isRunning(c) && workQueue.offer(command)) {
           int recheck = ctl.get();
           if (! isRunning(recheck) && remove(command))
               reject(command);
           else if (workerCountOf(recheck) == 0)
               addWorker(null, false);
       }
       // 如果入 queue 失败或者使用 SynchronousQueue 则执行 reject
       else if (!addWorker(command, false))
           reject(command);
   }
   ```

   ![avatar](/static/image/java/thread-pool-executor.png)
   ![avatar](/static/image/java/javase-juc-submit.png)
   ![avatar](/static/image/java/javase-juc-execute-flow.png)

#### 7 parameters

1. `@param corePoolSize`:
   - 一直存在[除非线程池销毁或者设置{@code allowCoreThreadTimeOut}],
   - 线程池创建好之后就准备就绪的线程数量, 等到接受异步任务去执行
2. `@param maximumPoolSize`: 最多线程数量, 控制资源
3. `@param keepAliveTime`:
   - 当前线程数大于核心线程数后,
   - 如果线程空闲大于 keepAliveTime 就会释放该线程,
   - 释放的线程时 `maximumPoolSize - corePoolSize`
4. `@param unit`: keepAliveTime 的时间单位
5. `@param blockingQueue`: 如果任务很多, 则多出来的将任务放入 queue 里, 只要有线程空闲了就会从 queue 里取出任务执行

   - {@link SynchronousQueue } 没有容量:
     1. 每一个插入操作都需要等待相应的删除操作, 反之亦然;
     2. 不会真的保存任务, 总是将任务直接交给线程执行, 没有空闲线程则创建新的线程, 线程数量达到最大值则使用 rejectHandler;
     3. 使用时建议设置很大的 pool-size
   - {@link ArrayBlockingQueue(int size) }:
     1. 如果有新的任务进来则就交给空闲线程执行;
     2. < core-pol-size 则创建新的线程执行,
     3. > core-pol-size, 则加入 queue
     4. queue 满了则 reject-handler
   - {@link LinkedBlockingDeque(int capacity)}:
     1. 容量默认是 Integer 的最大值[一定要限制]: 使用时一定要设置容量
     2. 如果有新的任务进来则就交给空闲线程执行;
     3. < core-pol-size 则创建新的线程执行,
     4. > core-pol-size, 则加入 queue
     5. queue 满了则 reject-handler
   - {@link java.util.PriorityQueue(int capacity) }: 控制任务执行顺序

6. `@param threadFactory`: 线程创建工厂
7. `@param handler`: 如果队列满了且达到了最大 pool-size, 就使用指定的策略拒绝向 queue 里放任务
   - [默认]AbortPolicy: 直接丢弃新的任务, throw exception
   - DiscardPolicy: 直接丢弃新的任务, 不 throw exception
   - DiscardOldestPolicy: 丢弃最老的任务
   - CallerRunsPolicy: 转为同步调用
   - 实现 `RejectedExecutionHandler` 进行自定义

#### CompletetableFuture

1. 创建一个异步操作

   - 没有指定 Executor 的方法会使用 `ForkJoinPool.commonPool()` 作为它的线程池执行异步代码

   ```java
   // 1. runAsync: 异步执行没有返回值
   public static CompletableFuture<Void> runAsync(Runnable runnable)
   public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
   // 2. supplyAsync: 异步执行有返回值, get() 获取结果
   public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
   public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
   ```

2. 线程串行化

   ```java
   // 1. thenRun: 不能获取上一步的执行结果
   public CompletableFuture<Void> thenRun(Runnable action)
   public CompletableFuture<Void> thenRunAsync(Runnable action)
   public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor)

   // 2. thenAcceptAsync: 能接受上一步结果, 但是无返回值
   public CompletableFuture<Void> thenAccept(Consumer<? super T> action)
   public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action)
   public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor)

   // 3. thenApplyAsync: 能接受上一步结果, 有返回值
   public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
   public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
   public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
   ```

3. 计算结果完成时的回调方法

   ```java
   // 可以处理异常, 不能处理异常时返回值
   public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
   public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
   public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
   //可以处理异常, 能处理异常时返回值
   public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
   ```

4. handle 方法: handle 是执行任务完成时对结果的处理

   ```java
   // 可以感知异常, 并能处理异常时返回值
   // handle就像finally, 不论正常返回还是出异常都会进入handle, 类似whenComplete
   //   handle: T -> R
   //   whenComplete的返回值和上级任务传入的结果一致, 不能对其转换
   public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
   public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
   public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,  Executor executor)
   ```

   ```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
       DeptService deptService = new DeptService();
       UserService userService = new UserService();

       CompletableFuture<User> future = CompletableFuture
            .supplyAsync(() -> deptService.getById(1)})
            .handle((Dept dept, Throwable throwable) -> {
                    if (throwable != null){
                        System.out.println(throwable.getMessage());
                        return null;
                    }
                    User user = new User(1, "winter", 32, dept.getId(), dept.getName());
                    return userService.save(user);
                }
            );
   }
   ```

5. 组合任务: **被组合的是异步的则需要** join， cf.getNow(null) 得到的就是 null

   ```java
   // 1. thenCombine 组合两个 Future{异步执行}, 获取两个 future 的结果, 并返回当前任务的结果
   public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
   public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)
   public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)

   // 2. thenAcceptBoth 组合两个 Future, 获取两个 future 的结果, 执行当前任务, 无返回值
   public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
   public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
   public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);

   // 3. runAfterBoth 组合两个 Future, 不能获取两个 future 的结果, 执行当前任务, 无返回值
   public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
   public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
   public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);

    // 4. applyToEither: 两个任务有一个之下能够完成后, 获取其返回值, 执行当前任务, 有返回值
    public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn)
    public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn)
    public <U> CompletableFuture<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn, Executor executor)

    // 5. acceptEither: 两个任务有一个之下能够完成后, 获取其返回值, 执行当前任务, 无返回值
    public CompletableFuture<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)
    public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)
    public CompletableFuture<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)

   // 6. runAfterEither 两个任务有一个之下能够完成后, 不能获取其返回值, 执行当前任务, 无返回值
   public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
   public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
   public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);

   // 7. thenCompose 方法允许你对两个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作
   public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
   public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
   public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn,Executor executor);

   // 8. allOf 等待所有任务完成
   public static CompletableFuture<Void> allOf(CompletableFuture<?>... cfs)

   // 9. anyOf 只要有一个任务完成, 可以拿到该执行完成后的结果 get()
   public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
   ```

6. sample

   ```java
   // 异步执行两个任务后得到结果, 做计算并返回
   return supplyAsync(() -> countParticipantInScope(currentMemberId, activity))
           .thenCombine(supplyAsync(() -> getCurrentRank(currentMemberId, activity))
                   ,(count, rank) -> setCR(count, rank, basicInfoVO))
           .get();
   ```

## 扩展

1. 几个重要的概念

   - 同步和异步: 同步和异步通常用来形容一次方法调用.
     同步方法调用一旦开始, 调用者必须等到方法调用返回后, 才能继续后续的行为.
     异步方法调用更像一个消息传递, 一旦开始, 方法调用就会立即返回, 调用者可以继续后续的操作.
   - 关于异步目前比较经典以及常用的实现方式就是消息队列:

   ```java
   在不使用消息队列服务器的时候, 用户的请求数据直接写入数据库, 在高并发的情况下数据库压力剧增, 使得响应速度变慢.
   但是在使用消息队列之后, 用户的请求数据发送给消息队列之后立即 返回, 再由消息队列的消费者进程从消息队列中获取数据, 异步写入数据库.
   由于消息队列服务器处理速度快于数据库(消息队列也比数据库有更好的伸缩性), 因此响应速度得到大幅改善.
   ```

2. 并发(Concurrency)和并行(Parallelism)
   - concurrency: many process compare for one kind resource
   - parallel: one process do many things at same time
