## ClassLoader

1. 主要作用

   - 通过类的全限定名获取该类的二进制字节{来源可以使二进制文件,字节码, 网络等}, 并加载进内存{转换为运行时数据结构}
   - 双亲委派
   - 沙箱安全

2. overview

   - 基础
     1. 作用
     2. 类加载体系分类
     3. 自定义类加载器
     4. 双亲委派: 数据隔离
     5. 沙箱安全{保护 JDK 源码}
     6. Properties
   - 加载过程
     1. 加载
     2. 连接
     3. 初始化: **顺序**
     4. 热加载: dev-tools
   - [spi](../spi/readme.md)
   - others
     1. 初始化 vs 实例化
     2. 成员 vs 属性
     3. agent
     4. tomcat 类的隔离与共享
     5. arthas 类的隔离与共享
     6. arthas 如果动态修改一个类{热加载原理}

---

## reference

1. https://github.com/Alice52/Alice52/issues/21
2. https://github.com/Alice52/Alice52/issues/84
3. https://mp.weixin.qq.com/s/F3Ng8JIOrjyHoR2rZYzL2Q
4. https://mp.weixin.qq.com/s/Q0MqcvbeI7gAcJH5ZaQWgA
5. https://mp.weixin.qq.com/s?__biz=Mzg4NTcyNjczNg==&mid=2247504223&idx=1&sn=b963e639e11c701d1dd011d9e4cb1169&source=41
