[toc]

## features

1. [优化底层 Hash 和 内存空间](./feature/Hash-Modify.md)
2. [Lambda 表达式](./feature/Lambda.md)

   - 函数式编程
     1. 函数式接口实例
     2. Java 内置四大核心函数式接口
        > Consumer<T> 消费型接口
        > Supplier<T> 供给型接口
        > Function<T, R> 函数型接口
        > Predicate<T> 断定型接口
   - Lambda 表达式
     1. 函数接口: `@FunctionalInterface`
     2. 类型检查、类型推断: Java 编译器根据 Lambda 表达式上下文信息就能推断出参数的正确类型。
     3. 局部变量限制
        > Lambda 表达式也允许使用自由变量 `外层作用域中定义的变量`, 就像匿名类一样. 它们被称作`捕获 Lambda`.
        > Lambda 可以有限制地捕获 `也就是在其主体中引用` 实例变量和静态变量. 限制为: 局部变量 `必须` 显式声明为 `final` , 或`事实上是 final[java8 默认为 final]` [demo](./feature/Lambda.md#语法).

3. [方法引用与构造器引用](./feature/Reference.md)

   - 方法引用
   - 构造器引用
   - 数组引用

4. [Stream 数据流](./feature/Stream.md)

   - Stream 流介绍
   - 使用流-筛选切片

     ```java
     filter
     distinct
     limit
     skip
     ```

   - 使用流-映射

     ```java
     - map
     - flatMap
     ```

   - 使用流-匹配

     ```java
      - anyMatch
      - allMatch
      - noneMatch
     ```

   - 使用流-查找

     ```java
      - noneMatch
      - findFirst
     ```

   - 使用流-归约

     ```java
      - reduce
      - max/min
     ```

   - 使用流-汇总统计

     ```java
      - collect
      - count
     ```

   - 使用流-遍历

     ```java
      - foreach
     ```

5. [并行流与串行流](./feature/parallel.md)

   - Fork/Join 框架: 这里面的中间值的确定很有问题: 好的值会比 `并行` 快: `工作窃取`算法: 算法是指某个线程从其他队列里窃取任务来执行
   - 并行串行切换: `parallel()/sequential()`

6. [接口](./feature/Interface.md)

   - 类优先于接口。 `如果一个子类继承的父类和接口有相同的方法实现。 那么子类继承父类的方法`
   - `子类型中的方法优先于父类型中的方法[就近原则]`
   - 如果以上条件都不满足, 则必须显示覆盖/实现其方法, 或者声明成 abstract。
   - 默认方法/静态方法/私有方法

7. [JAVA8 全新的时间包](./feature/DateTime.md)

8. [Optional 类](./feature/Optional.md)

   - 没看出有什么优点, 以后再看看

9. 重复注解与类型注解

10. [Base64](./feature/Base64.md)

## others

1. [demo-code](https://github.com/Alice52/DemoCode/tree/master/java/javase/java8-feature)
2. functional -- [`more-lambda`](https://github.com/PhantomThief/more-lambdas-java)
3. **stream**: 自己实现(原理)
