## static: 静态的

1. 如需要一个类的多个对象共享一个变量, 则改变量需要使用 static 修饰.
2. 因为 static 修饰的变量为类的所有是实例所共享, 所以 static 成员属于整个类, 而不属于某个类的实例. 所以在访问权限允许的情况下可以使用 "类名." 的方式直接访问 static 静态成员(`成员包括属性和方法`).
3. `在类的静态方法只能直接调用同类中的其他静态成员(成员包括属性和方法), 而不能直接访问类中的非静态成员`. 原因: 对于非静态成员的方法和变量, 需要创建类实例的对象后才能使用, 而静态方法在使用前不需要常见任何对象.
4. 同 3 中的道理, `静态方法不能以任何方式引用 this 和 super 关键字`.
5. `非静态方法中可以直接访问静态成员`.
6. main() 方法是静态的, 因此 JVM 执行 main()方法时不需要创建 main 方法所在的类的实例对象.
7. `静态初始化指对类的静态成员进行初始化`.

   - 不应该在 constructor 中对静态成员进行初始化: `静态成员不因类的改变而改变`.

   ```java
   // 非静态代码块: 先于 constructor 执行, 没创建一个对象都会执行一次.
   {
       System.out.println("非静态代码块");
   }
   ```

   - 静态代码块: 使用 static 修饰的代码块:

   ```java
   // 在类被加载时执行一次, 且只执行一次. 可以在静态代码块中对静态成员进行初始化.
   static {
    System.out.println("静态代码块");
   }
   ```

8. 所谓的类的 `单态设计模式`, 就是采取一定的方法保证在整个软件系统中, 对某个类只能存在一个对象实例.

   - 不能在类的外部通过 new 方式来创建新的实例: `构造器私有化`.
   - 只能是 `内部创建实例`
   - 为了让类的外部能够访问内部创建的实例, 所以该实例要是有 `static` 修饰
   - `不能允许在类的外部修改内部创建的实例引用`: `SingleInstance instance = null;`所以属性需要 private 修饰
   - 为了让外部可以读取, 所以要添加 get 方法

9. Java 代码初始化时的执行顺序

   - **`静态优先, 父类优先, 初始化实例变量, 动态代码块, 构造函数`**

   ```java
   1、 初始化父类静态属性
   2、 执行父类静态代码块
   3、 初始化子类静态属性
   4、 执行子类静态代码块
   5、 初始化父类实例变量
   6、 执行父类动态代码块
   7、 执行父类构造方法
   8、 初始化子类实例变量
   9、 执行子类动态代码块
   10、执行子类构造方法
   ```

## sample

```java
public class TestThis {
    private int a = 30;
    public void test(int b) {  // b = 5
        {
            int a = 26;
            {
                int c = 560;
                this.a = c;
                System.out.println(a);  // 26
                System.out.println(this.a);  // 560
                System.out.println(this instanceof TestThis);  //true
            }
            System.out.println(this.a);  //560
        }
        System.out.println(a);  // 560
    }

    public static void main(String[] args) {
        TestThis test1 = new TestThis();
        test1.test(5);
        System.out.println(test1.a);  //560
    }
}
```

## inteview

1. code

   ```java
   public class StaticTest {
       public static void main(String[] args) {
           staticFunction();
       }

       // 静态变量[有实例化的过程,这就是本题的重点]
       static StaticTest st = new StaticTest();
       static {
           // System.out.println(b); // 编译报错
           System.out.println(st.b); // 0
           System.out.println("1");
       }

       {
           System.out.println(b); // 0
           System.out.println(a); // 编译报错, 如果 a 定义再前面既不会报错
           System.out.println("2");
       }

       // 执行构造函数之前, 必须初始化实例属性
       StaticTest() {
           System.out.println("3");
           System.out.println("a=" + a + ",b=" + b);
       }

       public static void staticFunction() {
           System.out.println("4");
       }

       int a = 110;
       static int b = 112;
   }

   0
   2
   3
   a=110,b=0
   0
   1
   4
   ```

2. explain
   - 静态变量会再类加载时被加载到内存中, 且赋值初始值, 非静态变量不会
   - 动态代码块会在 `构造函数` 之前执行, 和赋值语句代码顺序执行
   - static 相关的会在 load 进内存时执行, 之后 new 的逻辑不会触发
   - 静态变量从上到下初始化
   - 顺序: 静态变量, 静态代码块 是顺序执行的; 但是父子类是[静态成员变量, 静态代码块]
   - 执行顺序:
     1. 一个类中: 静态变量, 静态代码块是顺序执行的[代码顺序]; 非静态变量赋值, 动态代码块是顺序执行的
     2. 父子类中: 所有的静态变量先执行, 静态代码块执行, 动态代码块, 成员变量, 构造函数
