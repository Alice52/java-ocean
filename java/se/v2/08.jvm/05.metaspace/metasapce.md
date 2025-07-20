### Method Area: PermGen space / Meta space

![avatar](/static/image/java/javase-jvm-jdk8-metadata.png)
![avatar](/static/image/java/javase-jvm-jdk7-metadata.png)

1. 非堆内存[元空间]和永久代通过使用 方法区 实现
2. jdk8 移除永久代的原因

   - 因为永久代使用的是虚拟机的内存, 为永久代设置空间大小是很难确定的; 元空间直接使用物理内存且可以子调整[在 max 范围内]
   - 对永久代进行调优是很困难的
   - 更容易导致 Java 程序更容易 OOM, 永久代仍然使用的是 Java 虚拟机的内存
   - 为融合 HotSpot JVM 与 JRockit VM 而做出的努力，因为 JRockit 没有永久代， 不需要配置永久代
   - 可以在 GC 不进行暂停的情况下并发地释放类数据

3. 方法区的大小决定了系统可以加载多少个类:

   - 如果系统定义的类太多可能会产生 OOM
   - 关闭 JVM 就会释放方法区的内存

4. 特性

   - 方法区（Method Area）同堆区一样, 是各个**线程共享**的内存区域
   - 方法区内存可以是不连续的: 永久代逻辑和物理上都属于 heap
   - 方法区的大小和堆空间一样可以动态调整[`元空间-XX:MaxMetaspaceSize=-1 无限制`]或者固定[永久代]

5. [存放内容](https://blog.csdn.net/Xu_JL1997/article/details/89433916): 类的元数据, 但是 Class 对象是存放在 heap 中的

   - 运行时常量池
     1. [常量池表]运行时常量池[引用]: **但是字符串常量池[1 个]在 heap 中, 字符串常量池本质是一个 hash 的 StringTable, value 还是引用**
     2. JIt 的缓冲
   - 类信息

     1. 类型信息[存在于 Class 对象中]: 对每个加载的类型[Class/interface/enum/annotation], JM 必须在方法区中存储以下类型信息
        - 全类名: 包名.类名
        - 直接父类的全类名: interface/java.lang.Object 没有父类
        - 类型的修饰符: public, abstract, final
        - 类型直接接口的一个有序列表
     2. 域信息: 类的属性/成员变量, 静态属性的值在 heap 中
        - 类所有的成员变量相关信息[赋值语句]及声明顺序
        - 域名称/域类型/域修饰符[pυblic/private/protected/static/final/volatile/transient]
     3. 方法信息: 所有方法信息[包含构造方法], 声明顺序
        - 方法名称
        - 返回类型或 void
        - 方法参数的数量和类型[按顺序]
        - 修饰符: [public/private/protected/static/final/synchronized/native/abstract]
        - 方法的内容字节码 bytecodes + 操作数栈 + 局部变量表及大小[abstract 和 native 方法除外]
        - 异常表[abstract 和 native 方法除外]:每个异常处理的开始位置, 结束位置, 代码处理在程序计数器中的偏移地址, 被捕获的异常类的常量池索引
     4. 类初始化代码: 静态初始化代码块 + 动态代码块
     5. 方法的符号引用

6. 方法区的 GC 问题

   - 在 Java7 及之前, HotSpot 虚拟机中将 GC 分代收集扩展到了方法区, 主要是针对常量池的回收和对类型的卸载
   - 而在 Java8 中, 已经彻底没有了永久代, 通过一个存储于堆内的 DirectByteBuffer 对象作为对元空间引用的操作
     1. 分配空间时: 虚拟机维护了一个阈值默认 windows-21M, 如果 Metaspace 的空间大小超过了这个阈值, 那么在新的空间分配申请时, 虚拟机首先会通过收集可以卸载的类加载器来达到复用空间的目的, 而不是扩大 Metaspace 的空间, 这个时候会触发 GC。这个阈值会上下调整, 和 Metaspace 已经占用的操作系统内存保持一个距离
     2. Metaspace 的总使用空间达到了 MaxMetaspaceSize 设置的阈值, 或者 Compressed Class Space 被使用光了, 如果这次 GC 真的通过卸载类加载器腾出了很多的空间, 否则的话, 我们会进入一个糟糕的 GC 周期, 即使我们有足够的堆内存

7. 运行时常量池 vs 常量池

   - 方法区中, 内部包含了**运行时常量池**
   - 字节码文件中, 内部包含了**常量池**
   - 常量池: 存放编译期间生成的各种字面量与符号引用/可以看做是一张表, 虚拟机指令根据这张常量表找到要执行的类名/方法名/参数类型/字面量等类型
   - 运行时常量池: 常量池表在运行时的表现形式
   - 编译后的字节码文件中包含了类型信息, 域信息, 方法信息等, 通过 ClassLoader 将字节码文件的常量池中的信息加载到内存中, 存储在了方法区的运行时常量池中

---

## reference

1. https://blog.csdn.net/weixin_45505313/article/details/114089053
2. [arthas oom](https://zhuanlan.zhihu.com/p/359329753)
