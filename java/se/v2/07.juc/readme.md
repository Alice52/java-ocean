## 简介

1. juc 出现的原因
   - 由于摩尔定律的失效, 想要高速和高性能则需要使用多核心的优势[并发编程]
2. juc 优点
   - 可以充分利用多核, 提高性能,
   - 适应高并发的需求
   - 提高吞吐量{异步/回调}
3. 问题
   - 线程安全问题
   - 线程锁问题
   - 线程性能问题

## synchronized

1. 重入问题
   - synchronized 是可重入锁, 不然不好使用
   - 重入次数是需要记录下来的, 等下解锁次数需要对用
   - 偏向锁/轻量级锁: 记录在线程栈中[hashcode 等 markword 信息也是]的 Lock Record 中, 重入一次就会生成一个 LR[释放则 pop 一个 LR]
   - 重量级锁是记载 objectmonitor 的一个字段上
2. 为什么有了自旋锁之后还有重量级锁?

   - 自旋是消耗资源的, 如果自旋次数过多或者等待时间过长会消耗大量 CPU 资源
   - 重量级锁中有等待队列, 可以将之前那些自旋的线程放入等待队列, 减少资源的消耗

3. 打开偏向锁, 效率一定会提高吗
   - object#10.为什么会有偏向锁

## disruptor

1. 无锁, 高并发, 使用唤醒 Buffer, 直接覆盖旧数据, 降低 GC
2. 实现了基于事件的生产者消费者模式: 观察者模式
3. 底部是环形数组: size 为 2 的次幂数 + `index=num&(size-1)`

## reference

1. [21-lock](https://mp.weixin.qq.com/s?__biz=MzAwMjI0ODk0NA==&mid=2451944398&idx=1&sn=4e9bd51a668ff4be2f1abd3183bc5b9c)
2. [atomicxx](https://www.cnblogs.com/tong-yuan/p/LongAdder.html)
