1. 在你的项目中, 那些数据是数据库和 redis 缓存双写一份的? 如何保证双写一致性?
2. 系统上线 redis 缓存系统是如何部署的?
3. 系统上线 redis 缓存给了多大的总内存? 命中率有多高? 抗住了多少 QPS? 数据流回源会有多少 QPS?
4. **热 key 大 value 问题, 某个 key 出现了热点缓存导致集群中的某台机器负载过高? 如何发现并解决?**
5. **超大 value 打满网卡问题如何规避?**
6. 你过往的工作经验中是否出现过缓存集群的事故? 说说细节和包高可用的保证方案
7. 平时如何监控缓存集群的 QPS 个容量?
8. 缓存集群如何扩容?
9. 说下 redis 集群的原理和选举机制?
10. **key 的寻址算法都有哪些?**
11. redis 线程模型, 画图解释?
12. **redis 内存模型, 画图解释?**
13. redis 底层数据结构?
14. redis 单线程特性的优缺点?
15. 如何解决缓存穿透, 雪崩问题?
16. redis 基本数据结构的使用场景?
17. 生产上 redis 内存陪多大, 怎么修改
18. redis 内存满了怎么办
19. redis 内存清理方式
20. redis lru
