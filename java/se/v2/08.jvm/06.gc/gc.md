## 堆参数调优入门

- diagram
  ![avatar](/static/image/java/jvm.png)

1. -Xms: heap start, default 1/64 of physical memory
2. -Xmx: heap max, default 1/4 of physical memory
3. -Xmn: new
4. -XX:+PrintGCDetail:
5. -XX:MaxTenuringThreshold: 设置存过次数后移到 `老年区`
6. JDK1.7:

   - -XX:PermSize
   - -XX:MaxPermSize

7. JDK1.8:

   - pro env -Xms avlue is equals to -Xmx to avoid GC compare Memory for Application, which will lead to some odd question

8. code

   ```java
   long xms = Runtime.getRuntime().totalMemory();
   long xmx = Runtime.getRuntime().maxMemory();
   ```

## GCDetails

- diagram
  ![avatar](/static/image/java/FullGC.png)
  ![avatar](/static/image/java/GC.png)
- config

```xml
-Xms10m -Xmx10m -XX:+PrintGCDetails
```

- detail

```java
[GC (Allocation Failure)[PSYoungGen: 2048K{YGC前内存占用}->488K{YGC后内存占用}(2560K{新生区总内存})] 2048K{YGC前堆内存占用}->754K{YGC后堆内存占用}(9728K{JVM堆总大小}), 0.0010957 secs{YGC耗时}] [Times: user=0.00{YGC用户耗时} sys=0.00{YGC系统耗时}, real=0.00 secs{YGC实际耗时}]
[Full GC 区名: GC前大小 -> GC后大小(该区总空间), 耗时]

[GC (Allocation Failure) [PSYoungGen: 2536K->488K(2560K)] 2802K->1039K(9728K), 0.0015812 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
[Full GC (Ergonomics) [PSYoungGen: 1211K->0K(1536K)] [ParOldGen: 6525K->3418K(7168K)] 7736K->3418K(8704K), [Metaspace: 4761K->4761K(1056768K)], 0.0058887 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
[GC (Allocation Failure) [PSYoungGen: 40K->32K(2048K)] 6273K->6264K(9216K), 0.0003920 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
[Full GC (Ergonomics) [PSYoungGen: 32K->0K(2048K)] [ParOldGen: 6232K->4603K(7168K)] 6264K->4603K(9216K), [Metaspace: 4762K->4762K(1056768K)], 0.0139514 secs] [Times: user=0.05 sys=0.00, real=0.01 secs]
[GC (Allocation Failure) [PSYoungGen: 20K->32K(2048K)] 6030K->6042K(9216K), 0.0008025 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
[Full GC (Allocation Failure) Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
[PSYoungGen: 0K->0K(2048K)] [ParOldGen: 5284K->5219K(7168K)] 5284K->5219K(9216K), [Metaspace: 4763K->4763K(1056768K)], 0.0103089 secs] [Times: user=0.02 sys=0.03, real=0.01 secs]
    at java.util.Arrays.copyOf(Arrays.java:3332)
Heap
 PSYoungGen      total 2048K, used 61K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
    at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
  eden space 1024K, 5% used [0x00000000ffd00000,0x00000000ffd0f548,0x00000000ffe00000)
    at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:674)
  from space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
    at java.lang.StringBuilder.append(StringBuilder.java:208)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
    at OOM.main(OOM.java:18)
 ParOldGen       total 7168K, used 5219K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 72% used [0x00000000ff600000,0x00000000ffb18c68,0x00000000ffd00000)
 Metaspace       used 4797K, capacity 4992K, committed 5248K, reserved 1056768K
  class space    used 529K, capacity 560K, committed 640K, reserved 1048576K

Process finished with exit code 1
```

## GC 4 算法: 没有最好的算法, 只能根据每代采用最合适的算法`[分代收集]`

1. ~~引用计数法~~: `无法解决循环依赖问题`, 维护计数器本身有消耗

   - 原理: 被引用的次数为 0, 就可以使用 System.gc() 进行回收
   - code

     ```java
     public class RefCountGC
     {
       private byte[] bigSize = new byte[2 * 1024 * 1024];//这个成员属性唯一的作用就是占用一点内存
       Object instance = null;

       public static void main(String[] args)
       {
         RefCountGC objectA = new RefCountGC();
         RefCountGC objectB = new RefCountGC();
         objectA.instance = objectB;
         objectB.instance = objectA;
         objectA = null;
         objectB = null;

         System.gc();
       }
     }
     ```

2. [GCRoot]复制算法(Copying): `新生代`

   - 原理: 从根集合 GCRoot 开始, 通过 Tracing 从统计是否存活
   - 过程:

     1. eden + From 复制到 To 区, 年龄 +1
     2. 清空 Eden + From
     3. 互换 From 和 To 区

   - feature:

     1. 没有碎片[YGC 时 eden+from 全空]
     2. **耗内存**[需要将幸存区数据复制到 To 区]

3. [GCRoot]标记清除(Mark-Sweep): `养老区`

   - 原理:

     1. 标记要回收的对象
     2. 统一回收

   - feature:

     1. 耗时少, 内存占用少
     2. 产生内存碎片
     3. GC 时程序不动

4. [GCRoot]标记[清除]压缩(Mark-Compact): `养老区`

   - 原理: 标记 + 清除 + 将存活对象整理到一端
   - feature:

     1. **无碎片**
     2. 需要移动对象的成本
     3. **耗时最长**
     4. GC 时程序不动

5. 种算法比较

   - 内存效率: 复制算法 > 标记清除算法 > 标记整理算法[此处的效率只是简单的对比时间复杂度, 实际情况不一定如此]
   - 内存整齐度: 复制算法 = 标记整理算法 > 标记清除算法
   - 内存利用率: 标记整理算法 = 标记清除算法 > 复制算法
