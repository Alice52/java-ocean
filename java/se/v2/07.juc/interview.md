## 1.谈谈对 volatile 的理解

1. volatile 是 java 虚拟机提供的轻量级的同步机制: 乞丐版的 `synchronized`

   - [内存屏障]保证可见性: JMM 的可见性[MESI CPU 的缓存一致性协议]
   - [内存屏障]禁止指令重排: volatile 实现禁止指令重拍优化, 避免多线程下程序出现乱序执行的现象[loadfence/storefence 原语指令]
   - 不保证原子性[会出现写丢失]: synchronized/Lock/AtomicInteger[cas]

2. 是对 `JMM` 规定的一种实现, 但是不保证原子性

3. 单例模式: synchronized/Double Check Lock + volatile

   - 非线程安全

     ```java
     public class Singleton {
       private static Singleton instance = null;

       // 可以使用 synchronized 可以解决: 性能不好
       public static Singleton getInstance() {
         if (instance == null) {
           instance = new Singleton();
         }

         return instance;
       }

       public static void main(String[] args) {
         IntStream.rangeClosed(0, 100)
             .forEach(x -> new Thread(() -> Singleton.getInstance(), "AAA" + x).start());
       }
     }
     ```

   - 线程安全: DCL[Double Check Lock + volatile]

     ```java
     // 主要用例禁止指令重排序, 否则可能会获取到未被初始完成的对象, 从而引发安全问题
     private static volatile Singleton instance = null;
     /**
     * DCL: 双端检锁机制, 在加锁前后都检查 + volatile<br>
     * 还是线程不安全的[未初始化的对象]: 指令重排导致的
     *
     * <pre>
     *     1. new Singleton()
     *         - 分配内存空间
     *         - 初始化对象
     *         - 设置 instance 指向刚分配的内存地址
     *     2. 由于指令重排的话可能是1-3-2, 此时返回的只是内存空间还没有初始化
     *     3. 在变量前加 volatile 禁止指令重排
     * </pre>
     *
     * @return
     */
     public static Singleton getInstanceV2() {
       // 外层加判空的目的是为了避免每次获取实例的时候都需要获取锁和释放锁, 这样会带来很大的性能消耗, 外层判空可以在已经初始化完成后, 直接返回实例对象.
       if (instance == null) {
         synchronized (Singleton.class) {
           // 内层判空是为了保证对象的单例, 因为在多线程情况下, 如果没有内层判空的话, 那么多个线程可能在竞争锁之前都已经通过了外层判空逻辑
           // 那么在这种情况下, 会出现多个实例对象.
           // 所以加上内层判空, 那么另一个线程进来后, 再次判空的时候对象已经被之前释放锁的线程初始化完成, 那么自然不会进入new对象的逻辑中, 从而保证了对象的单一
           if (instance == null) {
             instance = new Singleton();
           }
         }
       }

       return instance;
     }
     ```

## 2.JMM: 线程安全性获得保证

## 3.cas: `自旋锁` + `UnSafe`

1. 定义:

   - 本质: CAS 的全称为 Compare-And-Swap, 它是一条 CPU 并发原语
   - 作用: 判断内存某个位置的值是否为预期值, 如果是则更新为新的值, `过程是原子的`

2. CAS: `自旋锁` + `UnSafe`

   ```java
   // 满足 JMM 的规范: volatile + cas
   public class AtomicInteger extends Number implements java.io.Serializable {
       private static final Unsafe unsafe = Unsafe.getUnsafe();
       private static final long valueOffset;

       static {
           try {
               valueOffset = unsafe.objectFieldOffset
                   (AtomicInteger.class.getDeclaredField("value"));
           } catch (Exception ex) { throw new Error(ex); }
       }

       // value 使用 volatile 修饰保证了线程间的可见性: 通知其他线程主内存中的值已修改
       private volatile int value;

       public final int getAndIncrement() {
           // this 表示当前对象, valueOffset 表示内存偏移量[引用地址]
           return unsafe.getAndAddInt(this, valueOffset, 1);
       }
   }

   public final class Unsafe {
       public native int getIntVolatile(Object var1, long var2);
       public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

       public final int getAndAddInt(Object var1, long var2, int var4) {
           int var5;
           do {
               // 获取 var1[AtomicInteger对象] 对象中 var2[valueOffset] 地址的值: 从主内存中获取值
               var5 = this.getIntVolatile(var1, var2);
           } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4)); // compareAndSwapInt 再次获取如果还是 var5, 就修改: 这一步是 os 的并发原语, 具有原子性

           return var5;
       }
   }

   // - 假设线程 A 和线程 B 同时执行 getAndAddInt 操作[跑在不同的 CPU 上]
   // 1. AtomicInteger 中的 value 的原始值时 3, 即主内存中的 AtomicInteger 中的 value 是 3, 根据 JMM 模型, 线程 A 和线程 B 各自持有一份值为 3 的副本在各自的各自空间
   // 2. 线程 A 通过 getAndAddInt(var1, var2) 拿到 value 值为 3, 此时线程 A 挂起
   // 3. 线程 B 通过 getAndAddInt(var1, var2) 拿到 value 值为 3, 线程 B 没有挂起, 执行了 compareAndSwapInt 方法, 比较主内存中的值也是 3, 则成功修改主内存中的值为 4, 线程 B 结束
   // 4. 此时线程 A 恢复, 接着执行 compareAndSwapInt 方法进行比较发现获取的主内存的值为 4 不同于 var5[3], 说明该值被其他线程抢先异步修改了, 那么线程 A 本次修改失败, 只能重新读取重新来一遍*5*
   // 5. 线程 A 重新获取 A 的值, 因为 value 是被 volatile 修饰的, 所以其他线程可见修改, 线程 A 继续执行 compareAndSwapInt 进行比较替换, 直到成功
   ```

3. UnSafe: 可以直接原子的操作主内存中的数据

   - 是 CAS 的核心类[并发原语的体现], 由于 Java 方法无法直接访问底层系统, 需要通过 native 方法
   - UnSafe 类存在于 `sun.misc`[rt.jar]
   - Unsafe 类中所有的方法都是 native 的, 本质都是直接调用操作系统底层资源执行任务的, 直接操作指定内存的数据[主内存]
   - 调用 Unsafe 类中的 CAS 方法, JVM 会帮我们实现出 CAS 汇编指令: `完全依赖硬件`, 实现原子操作

4. CPU 并发原语

   - 并发原语是操作系统的语言
   - 有若干条指令组成, 用于实现指定功能
   - 原语的执行必须是连续的, 在执行过程中不能中断, `具有原子性`

5. **`cas vs synchronized`**

   - synchronized 加锁`同一时间内只有一个线程访问`, 一致性得到保障, 但是`并发性下降`
   - cas 没有加锁, 反复比较直到更新完后, 保证一致性和并发性

6. cas 的缺点

   - 循环时间长开销大: 如果 CAS 失败就会一直尝试, 如果 CAS`长时间一直不成功`会给 CPU 带来很大的开销
   - 只能保证`一个`共享变量[`可以是对象 AtomicReference`]的原子操作: 一个共享变量时可以使用循环 CAS 保证原子性, 多个共享变量时循环 CAS 无法保证原子性[需要使用 Lock]

     ```java
     User z3 = new User("z3", 15);
     User l4 = new User("l4", 25);
     AtomicReference<User> atomicReference = new AtomicReference<>();
     atomicReference.set(z3);
     boolean success = atomicReference.compareAndSet(z3, l4);
     ```

   - ABA 问题

7. ABA 问题

   - CAS 算法实现的一个重要前提需要取出内存中取出某个时刻的数据并与当下时刻进行比较替换, 那么在这个`时间差内会导致数据变化`
   - eg. t1 从主内存中 V 位置取出 A[后挂起], t2 也取出 A, t2 将值变成 B, t2 又将值变成 A; 此时 t1 进行 CAS 操作时发现内存中 V 位置是 A, 然后`t1操作成功`

     ```java
     public class ABATest {
         static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
         public static void main(String[] args) {
             new Thread(() -> {
                         atomicReference.compareAndSet(100, 101);
                         atomicReference.compareAndSet(101, 100);
                     }, "AAA").start();

             new Thread(() -> {
                         // 保证AAA线程完成一次ABA操作
                         TimeUnit.SECONDS.sleep(1);
                         atomicReference.compareAndSet(100, 102);
                     }, "BBB").start();

             // 等待 AAA, BBB 线程执行结束
             while (Thread.activeCount() > 2) {
                 Thread.yield();
             }
             log.info("main thread atomicReference value: {}", atomicReference.get());
         }
     }
     ```

   - soulution: 加修改版本号: `AtomicStampedReference`

     ```java
     static int initVersion = 1;
     static AtomicStampedReference<Integer> atomicReference = new AtomicStampedReference<>(100, initVersion);

     public static void main(String[] args) {
       new Thread(() -> {
                 // 需要 sleep 一下, 使得 BBB 可以拿到最初的版本号
                   TimeUnit.SECONDS.sleep(1);
                   int stamp = atomicReference.getStamp();
                   atomicReference.compareAndSet(100, 101, stamp, ++stamp);
                   atomicReference.compareAndSet(101, 100, stamp, ++stamp);
               }, "AAA").start();

       new Thread(() -> {
                   int stamp = atomicReference.getStamp();
                   // 保证AAA线程完成一次ABA操作
                   TimeUnit.SECONDS.sleep(3);
                   atomicReference.compareAndSet(100, 102, stamp, ++stamp); // false
               }, "BBB").start();

       // 等待 AAA, BBB 线程执行结束
       while (Thread.activeCount() > 2) {
         Thread.yield();
       }
     }
     ```

## 4. Collection 的线程不安全问题

1. 证明线程不安全: ConcurrentModificationException

   ```java
   // 同时读写一个 List 会出现 ConcurrentModificationException 异常
   // 无锁的并发修改导致
   public static void threadSafe() {
       ArrayList<String> unsafeList = new ArrayList<>();
       IntStream.rangeClosed(1, 1000).forEach(
               i -> new Thread(() -> {
                               String uuid = UUID.fastUUID().toString();
                               unsafeList.add(uuid);
                               log.info("{}", unsafeList);
                           }, "AAA" + i).start());
   }
   ```

2. ArrayList 是线程不安全的

   - 多线程操作 {@link ArrayList } 会出现 {@link java.util.ConcurrentModificationException}
   - solution[3]: Vector/Collections#synchronizedList/CopyOnWriteArrayList

     ```java
     private transient volatile Object[] array;

     // CopyOnWriteArrayList: `ReentrantLock + volatile` = `jmm`, `并发读`
     private E get(Object[] a, int index) {
         return (E) a[index];
     }

     // 写时赋值容器: 读写分离思想
     //   1. 往一个容器 Object[] 追加元素时, 不直接往当前元素追加,
     //   2. 而是先将当前容器的元素进行 copy 到新的容器中 Object [] new, 之后再新的容器中加锁的追加元素
     //   3. 之后将原数组的地址指向新的数组
     //   4. 好处: 可以无锁的并发读
     public boolean add(E e) {
         final ReentrantLock lock = this.lock;
         lock.lock();
         try {
             // 获取主内存中的数据
             Object[] elements = getArray();
             int len = elements.length;
             Object[] newElements = Arrays.copyOf(elements, len + 1);
             newElements[len] = e;
             // 写回主内存, 并通知其他持有该变量的线程
             setArray(newElements);
             return true;
         } finally {
             lock.unlock();
         }
     }
     ```

   - [读写分离的思想]CopyOnWrite 容器即写时赋值容器。 add(Element e) 时将当前容器进行 copy, 赋值出一个新的容器, 然后向新的容器内添加元素, 完成后将原来容器的引用指向新的容器
   - feature: 可以对 CopyOnWrite 容器并发的读, 且不需要加锁, 因为当前容器不会添加值； CopyOnWrite 容器也是一种读写分离的思想, 读数据和写数据在不同的容器进行

## 5.锁: `volatile + lock/cas+spin`

## 6.CountDownLatch/CyclicBarrier/Semaphore

### CountDownLatch

1. 解释: 所有人都离开才能锁门
2. 让一些线程阻塞直到另外一些完成后才被唤醒: await()/countDown()
3. code

   ```java
   public static void main(String[] args) {
       int count = 20;
       CountDownLatch cdl = new CountDownLatch(count);
       for (int i = 0; i < count; i++) {
       new Thread(
           () -> {
               LOG.info(Thread.currentThread().getName() + " leave room.");
               cdl.countDown();
           },
           String.valueOf(i)).start();
       }
       cdl.await();
       LOG.info("no person in room, can close door!");
   }
   ```

### CyclicBarrier

1. 解释: 集齐 7 颗龙珠就能召唤神龙
2. 让一组线程达到一个屏障(同步点)时被阻塞, 直到最后一个达到屏障时, 屏障才会打开, 所有的被阻塞的线程才会继续: `await()`
3. code

   ```java
   public class CyclicBarrierDemo {
       private static final int NUMBER = 10;
       public static void main(String[] args) {
           CyclicBarrier cb = new CyclicBarrier(NUMBER, () -> LOG.info("all things is ok, open barrier!"));

           for (int i = 0; i < NUMBER; i++) {
           final int times = i;
           new Thread(
               () -> {
                    LOG.info(Thread.currentThread().getName() + " on condition, and will await for open barrier, and now have " + times + " in wait.");
                    cb.await();
               },
               String.valueOf(i)).start();
           }
       }
   }
   ```

### Semaphore

1. 解释: 停车场
2. first in first get, leaved then others in
3. code

   ```java
   public class SemaphoreDemo {
       private static final int PARK_NUMBER = 5;
       private static final Logger LOG = LoggerFactory.getLogger(SemaphoreDemo.class);

       public static void main(String[] args) {
           // mockup park number
           Semaphore semaphore = new Semaphore(PARK_NUMBER);

           for (int i = 0; i < PARK_NUMBER * 2; i++) {
              new Thread(
                  () -> {
                      semaphore.acquire();
                      LOG.info(Thread.currentThread().getName() + " get access to park");

                      TimeUnit.SECONDS.sleep(new Random().nextInt(10)); // park random second, then leave
                      LOG.info(Thread.currentThread().getName() + " leave park");
                  }, String.valueOf(i)).start();
          }
      }
   }
   ```

## 7.阻塞队列[BlockingQueue]

1. 定义

   - 当阻塞队列是空时,从队列中获取元素的操作将会被阻塞.
   - 当阻塞队列是满时,往队列中添加元素的操作将会被阻塞.

2. 好处

   - 无需关心阻塞线程/唤醒线程,因为 BlockingQueue 在做
   - 阻塞也有好的一面: 比如餐厅的候客区
   - 不得不阻塞时的管理: 去体检[必须的]

3. API

   | function | 抛出异常  | 特殊值   | 阻塞   | 超时                 |
   | -------- | --------- | -------- | ------ | -------------------- |
   | 插入     | add(e)    | offer(e) | put(e) | offer(e, time, unit) |
   | 移除     | remove()  | poll()   | take() | poll(time, unit)     |
   | 检查     | element() | peek()   | --     | --                   |

   - 抛出异常:

     1. 当阻塞队列满时, 再往队列里面 add 插入元素会抛 IllegalStateException: Queue full
     2. 当阻塞队列空时, 再往队列 remove 元素时候回抛出 NoSuchElementException

   - 特殊值:

     1. 插入方法, 成功返回 true 失败返回 false
     2. 移除方法, 成功返回元素, 队列里面没有就返回 null

   - 一直阻塞:

     1. 当阻塞队列满时, 生产者继续往队列里面 put 元素, 队列会一直阻塞直到 put 数据 or 响应中断退出
     2. 当阻塞队列空时, 消费者试图从队列 take 元素, 队列会一直阻塞消费者线程直到队列可用.

   - 超时退出

     1. 当阻塞队列满时, 队列会阻塞生产者线程一定时间, 超过后限时后生产者线程就会退出

4. digram

   ![avatar](/static/image/java/javase-collection-queue-struct.png)

   - **[循环数组+cacheline]ArrayBlockingQueue**: 由数组结构组成的有界阻塞队列.
   - **[put/take 两把锁]LinkedBlockingQueue**: 由链表结构组成的有界(但大小默认值 Integer>MAX_VALUE)阻塞队列.
   - PriorityBlockingQueue:支持优先级排序的无界阻塞队列.
   - DelayQueue: 使用优先级队列实现的延迟无界阻塞队列.
   - **SynchronousQueue**:不存储元素的阻塞队列,也即是**单个元素**的队列: `每个put操作必须要等待一个take操作,否则不能继续添加元素,反之亦然`

     ```java
     BlockingQueue<Integer> bq = new SynchronousQueue<>();

     new Thread(
             () -> {
                 bq.put(50);
                 bq.put(51);
                 bq.put(52);
             }, "AA").start();

     new Thread(
             () -> {
                 TimeUnit.SECONDS.sleep(5);
                 Optional.of(bq.take()).ifPresent(System.out::println);

                 TimeUnit.SECONDS.sleep(5);
                 Optional.of(bq.take()).ifPresent(System.out::println);

                 TimeUnit.SECONDS.sleep(5);
                 Optional.of(bq.take()).ifPresent(System.out::println);
             }, "BB").start();
     ```

   - LinkedTransferQueue:由链表结构组成的无界阻塞队列.
   - LinkedBlocking**Deque**:由了解结构组成的双向阻塞队列.

5. ArrayBlockingQueue 原理: ReentrantLock

   - **本质的数字是循环数组**
   - 元素不允许为空, 为空会抛 NullPointException 异常；
   - 插入数据和取出数据分别用 putIndex 和 takeIndex 来定位位置的
   - 对于方法的访问是通过 ReentrantLock 来实现同步的
   - 对于取数据和写数据实现阻塞的方式是通过 Condition 来实现的, 而且分别使用不同的 Condition: notEmpty, notFull
   - 对于实现生产者/消费者模式, ArrayBlockingQueue 是一个非常好的数据结构

6. LinkedBlockingQueue
   - 链表数据
   - take/put 两个锁, 分别控制生产和消费: 并发度提高了
7. 代码实现

   - 生产者消费者模式: ABAB 的三种实现
   - 多个线程无序操作资源: 3 seller sale 30 tickets
   - 多线程间的顺序调用 ABCABC

## 8.synchronized 和 Lock 的区别

1. 原始构成

   - synchronized 是 JVM 的关键字: monitorenter/monitorexit&monitorexit[异常退出 ]
   - synchronized 是重量级的锁: 线程阻塞 + 上下文切换 + 线程调度
   - Lock 是具体实现类的 API

2. 使用方法

   - synchronized 不需要手动释放[两个 monitorexit], 代码执行完成之后系统会让线程自动释放锁
   - Lock 需要手动的加锁和释放锁, 否则可能会死锁

3. 等待是否可以中断

   - synchronized 不可以中断, 除非正常执行完成或者出异常
   - Lock 可以中断: tryLock(time, unit)/lockInterruptibly()[调用 interrupt() 可以中断]

4. 加锁是否公平

   - synchronized 是非公平锁
   - Lock 可以使非公平锁, 也可以是公平锁

5. 锁绑定多个条件

   - synchronized 不行: 要么随机唤醒一个, 要么唤醒全部
   - Lock 可以通过 Condition 来实现分组唤醒和精准唤醒

## 9.sleep 与 wait 的区别

1. sleep 是 Thread 的方法, wait 是 Object 的方法
2. sleep 不会释放锁[monitor], 不会交出控制权，当睡眠时间结束后，该进程会立即继续运行; wait 会释放锁并加入获取锁[monitor]的队列, 会交出控制权，之后会与其他进程抢占 CPU
3. sleep 不依赖与锁[monitor], wait 需要[必须在同步代码块内调用]
4. sleep 不需要被唤醒[时间到就会再次执行], wait 需要[notify]

## 10.创建线程的方法与区别

1. 4 种创建方式: Thread/Runnable/Callable+FeatureTask/pool
2. 区别: 线程池的好处

## 11.线程池的理解

1. 使用线程池的好处
2. Executors 中的常见方法
3. ThreadPoolExecutor

   - flow,
   - 7 参数
   - 线程池的拒绝策略: 自定义+ 激进线程池
   - CompletetableFuture
   - 提交到线程池的任务在放入 queue 时是不会执行的
   - threadPool.prestartAllCoreThreads();

## 12.合理配置线程池

1. CPU 密集型: 大量计算

   ```java
   //查看CPU核数
   System.out.println(Runtime.getRuntime().availableProcessors());
   ```

   - 只有在多核 CPU 上才能实现加速
   - 线程池数量: CPU 核心数 + 1

2. IO 密集型

   - IO 在堵塞[大量的 CPU 计算能力被浪费]可以多配置一些线程
   - 线程池数量 1: CPU 核心数 \* 2
   - 线程池数量 2: CPU 核心数 / (1 - 阻塞系数[0.8-0.9])

## 13.LockSupport: 线程等待唤醒机制

1. 定义: 用于创建`锁和其他同步类` 的基本线程的阻塞原语

   - 是线程阻塞唤醒的工具类, 所有方法都是静态的[UNSAFE]
   - 可以让线程在任意地方阻塞, 也可以在任意地方唤醒

2. 作用:

   - LockSupport 的 park() 和 unpark() 实现阻塞线程和解除阻塞
   - LockSupport 使用 Permit[许可证] 来实现阻塞和唤醒线程功能, 每个线程都有一个 Permit[0/1]

3. LockSupport: 类似以只有一个容量 Semaphore

   ```java
   // permit 默认值为 0, 调用 park() 方法就会阻塞线程,
   // 直到别的线程将 permit 设置为 1 则唤醒 park 方法[放行], 然后将 permit 再次设置为0
   public static void park() {
       UNSAFE.park(false, 0L);
   }

   // 调用 unpark 方法会将线程的 permit 设置为 1[多从调用还是1]
   // 之后会自动唤醒线程, 即之前 park 的方法会执行
   public static void unpark(Thread thread) {
       if (thread != null)
           UNSAFE.unpark(thread);
   }
   ```

## 14.wait&notify 与 await&signal 与 park&unpark 的区别

1. wait&notify

   - 是 Object 的方法
   - 必须在同步代码块内执行[依赖 monitor]
   - notify 在 wait 之前执行则不能唤醒: 先等待后唤醒
   - notify 执行随机唤醒一个线程

2. await&signal

   - Lock api 的方法
   - 也必须在 lock()/unlock() 的代码块内执行
   - signal 在 await 之前执行则不能唤醒: 先等待后唤醒
   - signal 可以精准唤醒
   - lock/unlock 方法必须成对出现

3. park&unpark

   - LockSupport static api 的方法: UNSAFE
   - 先 unpark 之后 park 也是可以的
   - 可以不再同步代码块内执行

## 15. AQS: AbstractQueuedSynchronizer/AbstractOwnableSynchronizer/AbstractQueuedLongSynchronizer
