# version2

## 相关结论

1. lambda 不是匿名内部类的另一种实现, jdk 会自动生成一个私有静态方法[效率比匿名内部类高]
2. lambda + stream 只有在数据量少下, 性能比原生的差, 但可读性变好
3. lambda 会生成放入内存中的接口实现类, 如果大量使用会影响内存, 进而影响 GC
4. lambda 只能使用外部 final 的变量: `可以访问自身外部类的私有变量{final(inline) 之后就不能改了保证外部类的稳定}`

---

## Lambda 表达式

- 原因: `主要是精简代码(可读性): ~~如匿名内部类相关的代码~~ 和匿名内部类是完全不同的东西`
- 条件: 需要函数式接口的支持

### 语法初始

1. 无参数无返回值[Runnable]

   ```java
   // JDK 1.7 之前必须是 final; JDK 1.8 默认加了 final: 不能让改变 num.
   final int num = 0;

   // 不使用 Lambda 表达式
   Runnable runnable =new Runnable() {
       @Override
       public void run() {
           System.out.println("hello lambda" + num);
       }
   };

   // 使用 Lambda 表达式
   Runnable runnable1 = () -> System.out.println("hello lambda2");
   ```

2. 有一个参数无返回值[Consumer]

   ```java
   // (x) -> System.out.println(x) // ()可以不写
   Consumer<String> consumer = (x) -> System.out.println(x);
   Consumer<String> consumer = x -> System.out.println(x);
   consumer.accept("练顺大傻逼！");
   ```

3. 有两个参数有返回值[Comparator]

   ```java
   // (x, y) -> x - y;
   Comparator<Integer> comparator = (x, y) -> {
       System.out.println("sa");
       return Integer.compare(x, y);
   };
   ```

4. 参数类型可以不写: JVM 可以进行 `类型推断`

### 函数式接口

1. 定义: **只有一个未实现的方法 + @FunctionalInterface**

2. Consumer<T>: void accept(T t)

   ```java
   Consumer<Integer> consumer = (money) -> System.out.println(money + " 元.");
   consumer.accept(200);
   ```

3. Function<T, R>: R apply(T t)

   ```java
   // 处理字符串: 获取字符串长度
   public Integer strHandler(String str, Function<String, Integer> func) {
       return func.apply(str);
   }

   @Test
   public void testFunction() {
       Integer size = strHandler("And this file just interprets the directory information at that level.",
               (str) -> str.length());
       System.out.println(size);
   }
   ```

4. Supplier<T>: T get()

   ```java
   @Test
   public void testSupplier2() {
       Set set = new HashSet();
       int size = 50;
       Supplier<Integer> supplier = () -> (int)(Math.random() * 100);
       for (int i = 0; i < size; i++) {
           Integer n = supplier.get();
           set.add(n);
       }
   }
   ```

5. Predicate<T>: bool test(T t)

   ```java
   public List<String> filterStrings(List<String> strs, Predicate<String> predicate) {
       List<String> sts = new ArrayList<>();
       for (String str : strs) {
           if (predicate.test(str)) sts.add(str);
       }
       return sts;
   }
   @Test
   public void testAddList() {
       List<String> strings = Arrays.asList("hello", "zack", "logo", "fans");
       List<String> strs = filterStrings(strings, (str) -> str.contains("a"));
       strs.forEach(System.out::println);
   }
   ```

6. 其他常见函数式接口

   - BiFunction<T, U, R>: R apply(T t, U u)
   - BinaryOperator<T>: T apply(T t1, T t2)
   - UnaryOperator<T>: T apply(T t)
   - BiConsumer<T, U>: void accept(T t, U u)
   - ToIntFunction<T> ToLongFunction<T> ToDoubleFunction<T>: int applyAsInt(T value)
   - IntFunction<R> LongFunction<R> DoubleFunction<R>: R apply(int value);

### 柯里化和部分求值[Currying]

1. `将一个多参数的函数, 转换为一系列单参数函数`

2. sample

   ```java
   import java.util.function.*;

   public class CurryingAndPartials {
       // 未柯里化:
       static String uncurried(String a, String b) {
           return a + b;
       }
       public static void main(String[] args) {
           System.out.println(uncurried("Hi ", "Ho"));

           // 柯里化的函数:
           Function<String, Function<String, String>> sum = a -> b -> a + b;
           Function<String, String> sumHi = sum.apply("Hup ");
           System.out.println(sumHi.apply("Ho"));
           System.out.println(sumHi.apply("Hey"));
       }
   }
   ```

### notice

1. [var in lambda must be final](https://fangshixiang.blog.csdn.net/article/details/80355490)

### usage sample

1. 定制排序 Employee: 年龄-姓名

   ```java
   Collections.sort(employees, (employee1, employee2) -> {
       if (employee1.getAge() == employee2.getAge()) return Double.compare(employee1.getSalary(), employee2.getSalary());
       else return Integer.compare(employee1.getAge(), employee2.getAge());
   });
   employees.forEach(System.out::println);
   ```
