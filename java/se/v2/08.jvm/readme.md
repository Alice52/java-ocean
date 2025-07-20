## Introduce

1. JVM 体系结构图
   ![avatar](/static/image/java/GC.bmp)

   - 栈是运行时的单位, 堆是存储时的单位

     ```java
     Class Files  ==>  类装载器子系统 Class loader
                       ||       ||
     ---------------------------------------------------------------
     |            运行时数据区[Runtime Data Area]                   |
     | 方法区[share]          Java 栈 NoGC             本地方法栈    |
     |  Method Area          Java Stack     Native Method Stack   |
     | 永久区是对其的实现                                            |
     |                                                            |
     |   堆[share]                      程序计数器                  |
     |     Heap              Program Counter Register             |
     |------------------------------------------------------------|
           || ||                          ||  ||
           执行引擎           ==>            本地方法接口     <== 本地方法库
     Execution Engine      <==         Native Interface
     ```

2. [ClassLoader](./ClassLoader.md)

   ![avatar](/static/image/java/class-loader.png)

   ![avatar](/static/image/java/javase-jvm-thread.png)
