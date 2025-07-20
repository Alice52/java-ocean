## try.catch.finally.return

1. 执行流程(go-defer 一致): `执行到 return 是会先去执行 finally`

   - 先计算返回值, 并将返回值存储起来, 等待返回
   - 执行 finally 代码块
   - 将之前存储的返回值, 返回出去

2. tcf: `catch 内 return`

   - 基本类型: 则返回的是 catch 时的值
   - 引用类型: 则返回的是 finally 都执行之后的值

3. tcf: `finally 直接 return`

   - 直接覆盖之前我们在 catch 或 try 返回的值

## finally 的不执行场景

1. try 未执行则 finally 也不会执行
2. try 内调用 System.exit(0) 终止 JVM 的, finally 不会执行
3. 非守护线程终结或退出时, 守护线程的 finally 可能不会被执行

## 异常

1. 流程

   - 触发异常时, JVM 会从上到下遍历异常表中所有的条目
   - 比较触发异常的行数是否在 from-to 范围内
   - 范围匹配之后, 会继续比较抛出的异常类型和异常处理器所捕获的异常类型 type 是否相同
   - 如果类型相同, 会跳转到 target 所指向的行数开始执行
   - 如果类型不同, 会弹出当前方法对应的 java 栈帧, 并对调用者重复操作
   - 最坏的情况下 JVM 需要遍历该线程 Java 栈上所有方法的异常表

2. 代价

   - 如果没有异常, TCF 的执行时间可以忽略不计
   - 如果出现异常, 则需要编辑异常表, 构建异常实例和所需的栈轨迹
   - 该操作会逐一访问当前线程的栈帧, 记录各种调试信息{包括类名、方法名、触发异常的代码行数等}
   - 所以执行效率会大大降低

---

## reference

1. https://github.com/Alice52/Alice52/issues/98
