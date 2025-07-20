## JMM

### 概念

1. JMM 概念:

   - Java 内存模型, 本身时一种抽象概念, `不真实存在`,
   - 描述一组`规则和规范`, 通过这组规范定义了程序中的各个变量`[包含实例字段, 静态字段和构成数组对象的元素]`的访问方式
   - 为了屏蔽各种硬件和操作系统的内存访问差异, 以实现让 Java 程序在各种平台下都能达到一致的内存访问效果
   - JMM 规定了所有的变量都存储在主内存[Main Memory]中, 每条线程还有自己的工作内存[Working Memory]
   - feature: 可见性, 原子性, 有序性

2. JMM 规定

   - 线程解锁前, 必须把共享变量的值刷新汇主内存
   - 线程加锁前, 必须读取主内存的最新值到自己的工作内存
   - 加锁解锁时同一把锁

3. flow

   - JVM 运行程序的实体时线程, 而每个线程创建时 JVM 都会为其创建一个工作内存[栈空间], 工作内存时每个线程私有的数据区域
   - JMM 中规定所有的变量都存储在`主内存`: `主内存` 是共享内存区域, 所有线程都可以访问
   - 但是线程对变量的操作[读取修改], 都必须在自己的工作内存中: 所以首先需要将变量从主内存拷贝到自己的工作内存空间, 然后对变量进行操作, 完成操作后在写回主内存, `不能直接操作主内存中的变量`
   - 各个线程中工作内存中存储着主内存中变量副本的拷贝, 因此不同的线程件无法访问对方的工作内存,
   - 线程间的通信[传值]必须通过主内存来完成

   ![avatar](/static/image/java/JMM.bmp)

4. 可见性:

   - [通过内存通信]各个线程对`主内存`中`共享`变量的操作都是各个线程各自拷贝到自己的`工作内存`操作后`再写回主内存`中的, 之后会通知其他线程的机制就是可见性
   - 如 AA 线程修改了共享变量 X 的值还未写回主内存中时, 另外一个线程 BB`又`对内存中的一个共享变量 X 进行操作, 但此时 AA 线程工作内存中的共享比那里 X 对线程 BB 来说并不不可见
   - 验证不可见性: **创建一个新的线程去修改值, 另一个线程判断修改过后的值是否可得**

     ```java
     static int number = 0;
     public static void main(String[] args) {
       new Thread(() -> ++number, "AAA").start();  // AAA thread will copy number to itself workspace, and do add 1 then write to main memory

       // main thread will copy number[0] to itself workspace, after AAA change number value , the number in main thread will aslo be 0
       while (number == 0) { // code will block here
       }

       log.info("thread: {} get number value is {}", Thread.currentThread().getName(), number); // unreachable
     }
     ```

   - 解决: volatile || sunchronized || final

5. 原子性[两个操作之间不能加塞 || 两个操作要么一起成功要么一起失败**且顺序执行**]: `完整性, 不可分割`

   - 定义: 某个线程正在做某个具体的业务时, 中间不可以被加塞或者分割, 需要整体完整, 要么成功, 要么失败
   - 解决: 锁 || CAS || OS 做
   - 验证原子性: 50 个线程执行 +1 方法 1000 次, 如果结果不是 5w 则证明`非原子性`

     ```java
     static volatile int number = 0;
     static AtomicInteger atomicInteger = new AtomicInteger(0);

     public static void main(String[] args) {

       for (int i = 0; i < 50; i++) {
         new Thread(  () -> IntStream.rangeClosed(1, 1000).forEach(x -> { number++; atomicInteger.addAndGet(1); })).start();
       }

       // 需要等待上面 50 个线程都执行完成后 在 main 线程中去 number
       while (Thread.activeCount() > 2) { // main + gc 两个线程
         Thread.yield(); // 放弃执行, 把当前线程重新置入抢 CPU 时间的队列
       }
     }

     // number++ 为什么是非原子的:
     //    1. number++ 在字节码层面是三个步骤: 拿到值, 修改值, 返回主内存
     //    2. 在只有 volatile 修饰时, 线程 1 写入主内存后通知线程2[此时被挂起], 所以没收到通知, 线程2[挂起态结束] 就也写入主内存
     ```

6. 有序性: 禁止指令重排

   - 指令重排序导致乱序执行[遵循 as-if-serial: 不管怎么重排序, 单线程环境下程序的执行结果不能被改变]
   - CPU 和编译器不会对存在数据依赖关系的操作做重排序
   - 不同 CPU 之间和不同线程之间的数据依赖性是不被 CPU 和编译器考虑的
   - 解决: volatile || sunchronized || Happens-before

   - 内存屏障: 内存栅栏, 是一个 CPU 的指令, 具有以下 2 个特性和作用: `顺序执行`, `某些变量的内存可见性[volatile可见性]`, **强制刷出各种 CPU 缓存数据, 保证 CPU 上的线程都能读取到最新的数据**
   - 如果在指令间插入一条 Memory Barrier 则会告诉编译器和 CPU, 无论什么指令都不能和这条 MB 指令重排: 通过插入内存屏障禁止在内存屏障前后的指令重排优化

### 指令重排: 编译器重排 + CPU 重排

1. 分类
   - 编译器重排: 主要解决阻塞时先去执行其他命令
   - CPU 重排: 指令并行的重排, 内存系统重排
     1. CPU 特性: 流水线, 乱序执行
     2. 主要满足 as-if-serial 就可以, 压榨 CPU 资源
     3. 指令并行的重排: 执行间不存在数据依赖时可以修改顺序执行
     4. 内存系统重排[伪重排]: CPU 和主存之间存在一个**高速缓存[减少主存和 CPU 的交互]**, 会优先使用缓存数据进行交互, 从而导致缓存一致问题
2. 指令重排: 计算机在执行程序时为了提高性, 编译器和处理器常常会对指令做重排

   - **as-if-serial:** 单线程里确保`最终执行结果`和`顺序执行结果一致`
   - 处理器在进行重排时必须考虑指令间的`数据依赖性`
   - 多线程环境中线程交换执行, 由于编译器优化重排的存在, `两个线程中使用的同一变量`**能否保持一致性**是无法确定的, `结果无法预测`
   - dcl 中的指令重排

     ```java
     public void sort() {
           int x = 10;  // 1
           int y = 20;  // 2
           x = x + 5;   // 3
           y = x * x;   // 4
     }
     // 指令重排之后可能是 1234, 2134, 1324
     ```

     ```java
     public class CommandResort {
        int a = 0;
        boolean flag = false;

        /**
         * 这里可能存在指令重排, 则在超多线程都在执行 m0, m2 时会出问题, 导致结果不唯一
        *
        * <pre>
        *      1. flag 在第一句: 就会导致线程执行 m2 时, 且在 a=1 执行之前执行, a=5
        *      2. flag 在第二句: 就会导致线程执行 m2 时,  a=6
        *      3. solution: 在 a 和 flag 之前都加上 volatile 指定涉及到这两个的都不循序指令重排, 则 a=6
        * </pre>
        */
        public void m0() {
           a = 1;
           flag = true;
        }

        public void m2() {
           if (flag) {
                 a = a + 5;
                 log.info("a: {}", a);
           }
        }
     }
     ```

3. 内存屏障可以禁止指令重排

### Happens-before

1. 简介
   - 它是判断数据是否存在竞争，线程是否安全的非常有用的手段
   - 是理解 JMM 的关键

---

## 补充

1. 物理磁盘, 内存, CPU[只负责计算]
2. 主内存是物理内存条
