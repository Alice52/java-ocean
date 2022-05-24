## disruptor

### 简介

1. 无锁, 高并发, 使用唤醒 Buffer, 直接覆盖旧数据, 降低 GC
2. 实现了基于事件的生产者消费者模式: 观察者模式
3. 底部是环形数组: size 为 2 的次幂数 + `index=num&(size-1)`