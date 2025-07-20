[toc]

1. 反射、注解、泛型 是框架层面非常常用的解决代码重复和通用性的方案

## overview

![avatar](/static/image/java/se/se-overview.png)
![avatar](/static/image/java/javase.png)

## [常见的对象](https://github.com/private-repoes/Alice52/issues/154)

1. Integer
2. Object
3. String
4. Threadlocal
5. Serialize
6. ClassLoader

## collection & map

1. List: CopyOnWriteList
2. Queue: XxDeque
3. Set
4. Stack: Vector
5. Map: chm

## enum: grammar + theory + best practice

1. 继承 enum 的 final 类
2. api: values()/UserStatus.valueOf(name);
3. 序列化单例[禁用 readObject/WriteObject] (一般是通过 resolveObject 解决的)
4. 同一类型的常量
5. 单例: 线程安全[classloader 阶段的保证的 static final]
6. 状态机[状态 CREATED 可以 CANCELD 和 CONFIRMED]: Order(state+OrderStateContext) 内有相关操作 cancel 接口 + OrderStateContext[将 state 与 order 耦合起来, 修改 Order 的 state 方法] + IOrderState + OrderState[最终的枚举控制, 通过 OrderStateContext 类修改 order 的 state]
7. 责任链: 在责任链模式中, 程序可以使用多种方式来处理一个问题, 然后把他们链接起来, 当一个请求进来后, 他会遍历整个链, 找到能够处理该请求的处理器并对请求进行处理
   - Message
   - MessageTypeEnum
   - IMessageHandler#handle
   - MessageHandlersEnum#dohandle 且继承 IMessageHandler 并实现 handle: message.getType() == this.acceptType && doHandle(message);
   - MessageHandlerChain#handle 使用 MessageHandlersEnum 的 value 作为 for 去执行 handle
8. 分发器: new EnumMap<MessageType, MessageHandler>(MessageType.class); 就是一个正常的 map key 是 enum, 对外不暴露这么多 handler, 根据入参确定使用谁的 handler, 就是策略模式

## interface & abstract

1. design pattern relative

## generic

1. 本质: 1
2. 实现原理: 2 - 2
3. 好处: 4
4. 泛型使用: 3[3*1]
5. 泛型参数:8
6. 泛型擦除:
   - 过程: 2
   - 影响: 5
7. conlusion: 2
8. 查看编译之后的代码: arthas

## juc

1. basic

   - 状态
   - 创建/打断
   - 操作: 顺序/交替

2. ThreadPoolExecutor

   - blockingqueue
   - reject strategy

3. ExecutorService

   - ForkJoinPool
   - CompletableFuture

4. volatile - CAS - AtomicInteger
5. AQS

6. lock

   - ReentrantLock
   - synchronized
   - ReadWriteLock
   - 升级
   - 类型

7. tools

   - CountDownLatch
   - CyclicBarrier
   - Semaphore

8. Threadlocal
9. collection thread safe

## jvm

1. basic

   - 内存分布

2. 四大引用
3. jmm

   - 内存泄露/溢出
   - OOM
   - Happened-before

4. classloader

   - spi

5. GC

   - GC 算法

6. 调优经验: 20 例
   - 工具的使用

## exception

## reflect

1. jdk proxy - aop

## [io](./IO.md)

1. 磁盘 IO: 由于 SSD 的普及, 这里的优化空间再收缩
   - async
2. 内存 IO
3. 网络 IO: 异步处理
4. - 发送数据: 同步发送就可以了, 没有必要异步: 想将数据缓存, 通过网卡将缓存中的数据发送
   - 接受数据: 需要有一个线程一直阻塞, 直到有数据时, 写入缓存, 然后给接收数据的线程发一个通知， 线程收到通知后结束等待， 开始读取数据: `周而复始`; 大量线程时就 频繁切换 CPU ...

## java8

1. stream:

   - forkjoin 工作窃取模式
   - RecursiveTask
   - RecursiveAction
   - List 性能不好; ArrayList/IntStream.range 性能好
   - api + parallel 数据不安全

2. stream API

   - stream/of/generate
   - 筛选切片: filter/distinct/limit/skip
   - 映射: map
   - 排序: sorted:
   - 匹配: any/all/noneMatch
   - 查找: find/max
   - forEach
   - reduce
   - count/collect

3. 方法引用

   - 对象::实例方法
   - 类::静态方法
   - 类::实例方法
   - 构造器::new

4. DateTime

   - 线程安全
   - LocalDateTime
   - Duration
   - Period
   - Instant
   - clock

5. Functional: 简化编码

   - Function
   - Comusner
   - Suppiler
   - Predicate
   - lambda

6. Optional 显示提醒 null 问题

   - Optional 可以包裹 null
   - api: ifp, isp, orelse, orleseG, orelseT, of, ofnullable, filter, map

## others

1. design pattern in jdk
2. this
3. static
4. final
5. try.catch
6. switch
7. instanceof/isAssignableFrom
8. 高 CPU 分析
9. JWT:

   - JWT 是一个自包含的访问令牌[带有声明和过期的受保护数据结构], 去中心化的思想[Validate]
   - JWT 不可回收所以不要带有敏感信息
   - 作用: 用来在身份提供者和服务提供者间传递安全可靠的信息
   - Header[算法+类型] + Payload[签发人/过期时间/主题/受众/生效时间/标号] + Signature[前两部分的签名]
   - 流程: 先校验 签名, 之后校验 payload
   - 自验证: 非对称加密, 可以利用私钥进行签名, 生成令牌, 然后公钥校验令牌**签证的合法性**

10. social-oauth:

    - 创建 App: 填写 callback, 会得到 clientId
    - 使用 callback+clientId 拼接对用的 uri
    - 点击 uri 会跳转社交平台的登录页面
    - 用户登录
    - 成功跳转 callback
    - 进入 callback 时会带有 code
    - code 换 access_token
    - token 换受保护信息

11. sso

    - 基于 web cookie 实现的
    - 核心点是在 oss-server 留下登录的痕迹
    - 第一次授权请求 oss-server 结束是 会有一个 domain 是 sso-server 的 cookie
    - 第二次发现已经有了 cookie, 则直接发放 code, 无序授权登录

12. 开放平台的使用

    - 可以获取 accessKey, accessKeySecret
    - accessKey 在请求中
    - accessKeySecret 一般用于构建签名[对整个请请求进行签名]
    - uri + method + request-body + accessKey + sign

13. 分布式 session

    - 传统的 jssesionId
    - session 复制: 每个 server 都要有所有的 session 信息耗内存, 同步好带宽; tomcat 支持, 好配
    - 客户端存储: 放在 cookie 里面, 客户端每次请求都带上, 有 cookie 大小限制, 安全风险, 不用
    - hash 一致性算法: Ip Hash 使得一个用户的请求落到同一台 sevre 上[nginx], 且克水平扩展; 但是 rehash 会出现部分用户重登录[serever 重启问题]
    - redis 统一存储: 水平扩展 + 安全 + 重启 server 不用重新登录; 但是会增加一次 http 的 redis 请求[比内存慢]

14. Cross-domain

    - 同源策略: **协议, 域名, 端口**都要相同, 其中有一个不同都会产生跨域[同一域名下不同文件 夹不会跨域]/ IP 与 域名也是跨域
    - 解决: 使用 nginx 部署为同一域/nginx 配置允许: Access-Control-Allow-Origin/Methods/Credentials/Headers
    - Optainal 的预检请求

15. http

    - GET 请求在 URL 中传送的参数是有长度限制的, 而 POST 么有
    - GET 在浏览器回退时是无害的, 而 POST 会再次提交请求
    - GET 参数通过 URL 传递, POST 放在 Request body 中

16. quartz

    - Job 接口任务的内容， org.quartz.job && 有无参构造函数
    - JobDetail: 每次都会反射创建 name 用于 mark, group mark 个管理 [两个相同 group 的 trigger 具有不同的 name(auto/manual)]
    - Trigger: simple./cron
    - scheduler: b 绑定 d&t
    - durable: jdbc， lock, jobdetail, triggers, schedulerstate
    - cluster

17. gRPC:

    - protobuf 的协议, 小快方便， 跨平台 + 多语言 +可扩展
    - protoc  生成代码
    - flow
      1. 创建隧道: 重复使用
      2. 创建 client：重复使用
      3. build & send 请求
      4. server response
    - 消息类型
      1. 一元消息
      2. server streaming： 流视频
      3. client streaming： 上传
      4. 双向 streaming
    - 5. 为什么 grpc
      1. Nginx
      1. 生态好
      1. 速度快
      1. 性能好
