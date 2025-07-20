# interview

## 枚举

1. final/线程安全/继承 Enum
2. values(责任链-HandlersEnum)/valueOf/getEnumConstants
3. 单例(DCL)/序列化: writeObject
4. 状态机(内部有允许的操作): 实现接口
5. 分发器(责任链): 实现接口

## cl

1. cl 图
2. cl 作用: 3
3. cl 种类 & jdk 实现集成
4. cl 加载过程: 加载 2 & 链接 [3] - 初始化 [4-3]
5. 初始化和实例化
6. 热加载
7. cl 加载顺序
8. 双亲委派机制
9. spi

## jvm

1. 图 1 + 线程图 2
2. 堆体系结构
3. 堆参数调优入门: -Xms==-Xmm 为什么要一样
4. GCRoot
5. 4 大垃圾回收算法: 特点 + 使用
6. 如何盘点查看 JVM 系统默认值
   - 的参数类型: 3
   - 盘点家底指令: 2
   - 查看默认的逻辑回收器
7. 常用的优化参数
8. GCDetails 的查看
9. 强引用/软引用/弱引用/虚引用
10. OOM 的认识: 8
11. 垃圾收集器分类
12. 垃圾收集器有哪些
13. 如果选择垃圾收集器
14. jvm 配置垃圾回收器
15. 生产环境服务器变慢
16. CPU 过高怎么排查具体的排查过程

## 生产遇到的问题

1. oom: heap | metaspace

---

## reference

1. [oom analysis](https://zhuanlan.zhihu.com/p/561704557)
