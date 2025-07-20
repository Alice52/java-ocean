[toc]

## intros

1. java 允许在类的内部定义类-嵌套(内部)类

   - 嵌套类: 静态内部类 + 内部类(非静态成员类 | 匿名类 | 局部类)
   - `内部类相当于类的一个成员`: 封装隐匿
   - 可以使用 `public/private/protected | static | final | abstract` 修饰

2. 分类

   - **静态内部类**
   - 常规内部类
   - 匿名内部类
   - 局部内部类

3. 区分维度

   - 是否依赖于外部类
   - 成员变量种类
   - 外部类对内部类的属性访问
   - 内部类对外部类的属性访问
   - 创建: 在外部类中 | 在其他类中

   |   type    | 赖于外部类 | 成员变量类型 | 静属性 o->i | 非静属性 o->i | 静属性 i->o | 非静属性 i->o | 是否可创建 |
   | :-------: | :--------: | :----------: | :---------: | :-----------: | :---------: | :-----------: | :--------: |
   |  common   |     Y      | 不可 static  |     _Y_     |       N       |      Y      |       Y       |     Y      |
   |  static   |     N      |  可 static   |      Y      |       N       |      Y      |       N       |     Y      |
   | anonymous |    _Y_     | 不可 static  |     _Y_     |       N       |      Y      |       Y       |    _Y_     |
   |   local   |    _Y_     | 不可 static  |     _Y_     |       N       |      Y      |       Y       |     N      |

4. [作用](https://www.codercto.com/a/4917.html)

   - **实现隐藏, 封装性**
   - _利用内部类实现**多重继承**_:
     1. 相当于持有属性(组合原则)
     2. 并不友好, 不要这么使用
   - 实现**回调**功能:
     1. 内部类调用外部实例方法
     2. 被继承和要实现接口有同一个方法-继承类且内部类实现病持有接口
   - 非静态内部类拥有其所在外部类的所有元素的访问权限

## 静态内部类

1. intros

   - 不依赖于外部类存在: 相当于外部类的一个静态对象属性
   - 静态内部类可以有非静态属性
   - 静态内部类只能操作外部类的非静态属性权限
   - 外部类可以操作静态内部类的静态属性

2. 创建

   ```java
   // 在外部类中：内部类名 name = new 内部类名();
   InnerClass innerClass = new InnerClass()
   // 在非外部类中: 外部类名.内部类名 name = new 外部类名.内部类名();
   OuterClass.InnerClass inner = new OuterClass.InnerClass();
   ```

3. sample

   ```java
   public class OuterClass {
       private int op;
       private static int sop;
       private InnerClass innerClass = new InnerClass();
       @Data
       public static class InnerClass {
           static int sp;
           private int ip;
           public void accessOp() {
               log.info("out class properties: {}", sop);
           }
           public void method() {
               int a = 3;
               // 访问方法内部变量
               log.info("{}", a);
               // 访问内部类的成员变量
               log.info("{}", this.a);
               // 访问外部内的成员变量: error
               // log.info("{}", OuterClass.this.a);
          }
       }
   }

   public static void main(String[] args) {
       val a = OuterClass.InnerClass.sp;
       OuterClass.InnerClass inner = new OuterClass.InnerClass();
       inner.accessOp();
       int ip = inner.getIp();
   }
   ```

## 常规内部类

1. intros

   - **内部类依赖于外部类存在**, 不能独立存在: 相当于非静态成员属性
   - **内部类不能有静态成员**
   - 内部类可以操作外部类属性
   - ~~外内部类可以操作内部类属性~~: 不能吧

2. 创建

   ```java
   // 在外部类中：内部类名 name = new 内部类名();
   InnerClass innerClass = new InnerClass()
   // 在非外部类中: 外部类名.内部类名 name = new 外部类名().new 内部类名();
   OuterClass.InnerClass inner = new OuterClass().new InnerClass();
   ```

3. sample

   ```java
   public class OuterClass {
       private int op;
       private static int sop;
       private InnerClass innerClass = new InnerClass();

       public class InnerClass {
           // inner class cannot contains static properties.
           // private static int p;
           private int ip;
           public void accessOp() {
               log.info("out class properties: {}-{}", op, sop);
           }
           public void method() {
               int a = 3;
               // 访问方法内部变量
               log.info("{}", a);
               // 访问内部类的成员变量
               log.info("{}", this.a);
               // 访问外部内的成员变量
               log.info("{}", OuterClass.this.a);
          }
       }
   }

   public class Test {
       OuterClass oclass = new OuterClass();
       OuterClass.InnerClass inner = oclass.new InnerClass();
   }
   ```

## 匿名内部类-anonymous

1. intros

   - 匿名内部类是内部类的一种, 匿名类可以作为实现**接口**的参数: _不能创建和实例化_
   - 匿名内部类可以包含静态属性和非静态属性: 只能知己访问
   - 匿名内部类可以操作外部类属性
   - 外部类不能操作匿名内部类属性

2. 创建

   ```java
   // 实现接口且没有类名, 比如 Consumer
   (Object a) -> log.info(a.toString())
   ```

3. sample

   ```java
   interface AnonymousInterface {
      void accept(String tag, Consumer consumer);
   }

   AnonymousInterface anonymousInterface = new AnonymousInterface() {
       // can define property in anonymous class，and can only used in local.
       int field = 1;

       @Override
       public void accept(String tag, Consumer consumer) {
           consumer.accept(tag);
           LOG.info("Outer class field2: " + outerFiled);
       }
   };

   // create anonymous for each interface method call.
   anonymousInterface.accept("obj1", (Object a) -> LOG.info(a.toString()));
   ```

## 局部内部类

1. intros

   - 局部内部类是内部类的一种, 是存在于方法内的类
   - 不能使用 public/protected/private 修饰, final / abstract 可以

2. sample

   ```java
   @Slf4j
   public class OuterClass {
       public int op;
       public void getXx() {
           int mp = 1;
           final class LocalInner {
               public int ip;
               public void getX() {
                   log.info("{}-{}-{}", op, mp, ip);
               }
           }
       }
   }
   ```

## 内部类与字节码

## 内部类与多继承

## 内部类与内存泄漏

---

## reference

1. https://blog.csdn.net/hacker_zhidian/article/details/82193100

---

## 6. Deep Understanding Inner Class

1. test sample

   ```java
   public class InnerClassTest {

       int field1 = 1;
       private int field2 = 2;

       public InnerClassTest() {
           InnerClassA inner = new InnerClassA();
           int v = inner.x2;
       }

       public class InnerClassA {
           int x1 = field1;
           private int x2 = field2;
       }
   }
   ```

2. run `javac InnerClassTest.java` command in cmd, then can get two file: `InnerClassTest.class`, `InnerClassTest$InnerClassA.class`

3. run `javap -c InnerClassTest` to decompile file, can get follow

   ```java
   Compiled from "InnerClassTest.java"
   public class InnerClassTest {
       int field1;

       public InnerClassTest();
           Code:
           0: aload_0
           1: invokespecial #2                  // Method java/lang/Object."<init>":()V
           4: aload_0
           5: iconst_1
           6: putfield      #3                  // Field field1:I
           9: aload_0
           10: iconst_2
           11: putfield      #1                  // Field field2:I
           14: new           #4                  // class InnerClassTest$InnerClassA
           17: dup
           18: aload_0
           19: invokespecial #5                  // Method InnerClassTest$InnerClassA."<init>":(LInnerClassTest;)V
           22: astore_1
           23: aload_1
           24: invokestatic  #6                  // Method InnerClassTest$InnerClassA.access$000:(LInnerClassTest$InnerClassA;)I
           27: istore_2
           28: return

       // access$100 method accept InnerClassTest as parameter to get InnerClassTest field
       // it will be called by inner class
       // expose own property
       static int access$100(InnerClassTest);
           Code:
           0: aload_0
           1: getfield      #1                  // Field field2:I
           4: ireturn
   }
   ```

- `getfield` make sense for InnerClassTest to get outer class field
- `24: invokestatic`: it calls inner class `access$000` method to get inner class field

4. run `javap -c InnerClassTest$InnerClassA.class` to decompile file, can get follow content

   ```java
   Compiled from "InnerClassTest.java"
   public class InnerClassTest$InnerClassA {
       int x1;
       // common inner class will have reference of outer class.
       final InnerClassTest this$0;

       // constructor will accept a outer class parameter
       public InnerClassTest$InnerClassA(InnerClassTest);
           Code:
           0: aload_0
           1: aload_1
           2: putfield      #2                  // Field this$0:LInnerClassTest;
           5: aload_0
           6: invokespecial #3                  // Method java/lang/Object."<init>":()V
           9: aload_0
           10: aload_0
           11: getfield      #2                  // Field this$0:LInnerClassTest;
           14: getfield      #4                  // Field InnerClassTest.field1:I
           17: putfield      #5                  // Field x1:I
           20: aload_0
           21: aload_0
           22: getfield      #2                  // Field this$0:LInnerClassTest;
           25: invokestatic  #6                  // Method InnerClassTest.access$100:(LInnerClassTest;)I
           28: putfield      #1                  // Field x2:I
           31: return

       // expose own property
       static int access$000(InnerClassTest$InnerClassA);
           Code:
           0: aload_0
           1: getfield      #1                  // Field x2:I
           4: ireturn
   }
   ```

- `getfield` in `access$000` method make sense for `InnerClassTest$InnerClassA` to get inner class field
- `25: invokestatic`: it is call outer class `access$000` method to get outer class field

5. static inner class content

   ```java
   Compiled from "InnerClassTest.java"
   public class InnerClassTest$InnerClassA {
       // no call outer class access method, so it has no access to outer class property
       public InnerClassTest$InnerClassA();
           Code:
           0: aload_0
           1: invokespecial #2                  // Method java/lang/Object."<init>":()V
           4: aload_0
           5: iconst_0
           6: putfield      #1                  // Field x2:I
           9: return

       // this makes sense to expose inner class property
       static int access$000(InnerClassTest$InnerClassA);
           Code:
           0: aload_0
           1: getfield      #1                  // Field x2:I
           4: ireturn
   }
   ```

## 7. Inner Class and Multiple Inheritance

1. not allow multiple inheritance
2. use class D to generate A, B, C: I think it more like factory rather than multiple inheritance

   ```java
   class A {}
   class B {}
   class C {}

   public class D extends A {
       class InnerClassB extends B {}
       class InnerClassC extends C {}

       public B makeB() {
           return new InnerClassB();
       }

       public C makeC() {
           return new InnerClassC();
       }

       public static void testA(A a) {}

       public static void testB(B b) {}

       public static void testC(C c) {}

       public static void main(String[] args) {
           D d = new D();
           testA(d);
           testB(d.makeB());
           testC(d.makeC());
       }
   }
   ```

3. feature
   - broken class structure
   - Unless there is a very clear dependency between the two classes (such as a certain car and its special model of wheels), or one class exists to assist another class (such as the HashMap and its internal HashIterator class used to traverse its elements), then using the inner class at this time will have a better code structure and implementation effect.
   - In other cases, separate classes will have better code readability and code maintainability.

## 8. Internal Class and Memory Leaks

1. Memory leak: there are some objects that can be recycled but not recycled for some reason

2. how to avoid memory leak

   - Use static inner classes whenever possible
   - For some custom class objects, use the static keyword with caution

3. code

   ```java
   import org.slf4j.Logger;
   import org.slf4j.LoggerFactory;

   import java.util.concurrent.TimeUnit;

   /**
   * function: this class is created for test memory leak.<br>
   *      1. MyComponent, OnClickListener, MyWindow will be not recycled<br>
   *      2. due to OnClickListener is static, so it will not be recycled;<br>
   *           and OnClickListener has a final member to point to MyComponent outer class,
   *           so MyComponent will not also be recycled;<br>
   *       and MyComponent has MyWindow member, so MyWindow will also not recycled
   * // TODO: why not recycle memory? for static or lambda
   */
   public class MemoryLeakTest {
       private static final Logger LOG = LoggerFactory.getLogger(MemoryLeakTest.class);

       abstract static class Component {

           final void create() {
           onCreate();
           }

           final void destroy() {
           onDestroy();
           }

           /** This is for subClass overwrite. */
           abstract void onCreate();

           /** This is for subClass overwrite. */
           abstract void onDestroy();
       }

       static class MyComponent extends Component {
           static OnClickListener clickListener;
           MyWindow myWindow;

           @Override
           void onCreate() {
               clickListener = obj -> LOG.info("Object " + obj + " onclick.");
               myWindow = new MyWindow();
               myWindow.setClickListener(clickListener);
           }

           @Override
           void onDestroy() {
               myWindow.removeClickListener();
           }
       }

       static class MyWindow {
           OnClickListener clickListener;

           void setClickListener(OnClickListener clickListener) {
               this.clickListener = clickListener;
           }

           void removeClickListener() {
               this.clickListener = null;
           }
       }

       public interface OnClickListener {
           void onClick(Object obj);
       }

       public static void main(String[] args) throws InterruptedException {
           MyComponent myComponent = new MyComponent();
           myComponent.create();
           myComponent.myWindow.clickListener.onClick(new Object());
           myComponent.destroy();
           // this operation will donot recycled memory
           myComponent = null;
           System.gc();
           System.out.println("");
           TimeUnit.HOURS.sleep(5);
       }
   }
   ```

---

---

## conclusion

2. understand about inner class:

   - 在外部类访问非静态内部类私有成员的时候, 会持有一个指向外部类引用的成员变量, 对应的内部类会生成一个静态方法, 用来返回对应私有成员的值,而外部类对象通过调用其内部类提供的静态方法来获取对应的私有成员的值.
   - 在非静态内部类访问外部类私有成员的时候, 对应的外部类会生成一个静态方法, 用来返回对应私有成员的值, 而内部类对象通过调用其外部类提供的静态方法来获取对应的私有成员的值.
   - 静态内部类访问外部类的成员时, 对应的外部类会生成一个静态方法, 用来返回对应私有成员的值, 而内部类对象不没有调用其外部类提供的静态方法来获取对应的成员的值.
