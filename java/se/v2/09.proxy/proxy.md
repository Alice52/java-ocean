[toc]

## 代理

1. intros

   - 使用一个代理将对象包装起来, 然后用该代理对象取代原始对象
   - `任何对原始对象的调用都要通过代理`
   - **代理对象决定是否以及何时将方法调用转到原始对象上**.

2. 分类

   - 静态代理

     - cons: 强耦合, 通用性低
     - impl: 通过代码层面持有被代理对象
     - pros: 可以访问、控制或扩展被代理类

   - 动态代理:

     - jdk(接口代理): Proxy 提供了创建代理对象的静态方法(通用性)
     - cglib(接口&类代理)

3. 分类区别

   - jdk 代理和 CGLIB 代理

     1. jdk 代理接口, CGLIB 代理非 final 类;
     2. jdk 使用 Proxy 创建代理类, CGLIB 底层采用 ASM 字节码生成代理类;
     3. 效率: 在 jdk1.8 下少量调用时, java 反射效率要高, 否则 CGLIB 效率高

   - 动态代理 和 静态代理

     1. 动态代理接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理: InvocationHandler.invoke
     2. 动态代理具有通用性, 静态代理是手动**代码代理**每个方法: 增加了代码维护的复杂度 | 重复度

4. pros

   - 通用性, 可以处理一些共有逻辑: aop
   - 代理对象可以**扩展**目标对象的功能
   - 代理模式能将客户端与目标对象分离, 在一定程度上降低了系统的**耦合**度
   - 代理模式在客户端与目标对象之间起到一个**中介作用和保护**目标对象的作用: 不同级别权限

5. cons

   - 性能一般: 底层是反射实现
   - 增加了系统的复杂度: 反射

6. 保存生成的代理类

   - jdk: `System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");`
   - cglib: `System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "d:\\code");`

7. ~~理解图~~

   ~~![avatar](/static/image/java/proxy.png)~~

8. code

   ```java
   public interface Calculator {
       int add(int i, int j);
       int sub(int i, int j);
       int mult(int i, int j);
       int div(int i, int j);
   }

   public class CalculatorImpl implements Calculator {
       @Override
       public int add(int i, int j) {
           int result = i + j;
           return result;
       }
       @Override
       public int sub(int i, int j) {
           int result = i - j;
           return result;
       }
       @Override
       public int mult(int i, int j) {
           int result = i * j;
           return result;
       }
       @Override
       public int div(int i, int j) {
           int result = i / j;
           return result;
       }
   }
   ```

## 静态代理

1. 本事是没有通用性的代码耦合的: 创建一个代理对象, 包含被代理对象, 对代理对象做访问、控制或扩展
2. sample

   ```java
   public class CalculatorProxy implements Calculator {

       private Calculator calculator = new CalculatorImpl();

       @Override
       public int add(int a, int b) {
           log.info("The method add begins with [" + a + ", " + b + "]");
           int result = calculator.add(a, b);
           log.info("The method add ends with [" + result + "]");
           return result;
       }
   }

   public static void main(String[] args) {
       CalculatorProxy proxy = new CalculatorProxy();
       proxy.add(1, 1);
       log.info("{}", proxy.getClass());
   }
   ```

## 动态代理-jdk 代理

1. core
   - 使用 Proxy 动态创建代理对象, 之后对代理对象进行代理
   - 基于接口实现的, 创建接口的代理类实现对目标的增强, 不能修改字节码(运行时)
2. Proxy.newProxyInstance(ClassLoader, Class<?>[], InvocationHandler)

   ```java
   public class Proxy implements java.io.Serializable {
       // 获取代理类 <br/>
       // loader: 类加载器
       // interfaces: 类实现的接口
       Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces);

       // 生成代理对象 <br/>
       // loader: 类加载器
       // interfaces: 类实现的接口
       // h: 动态代理回调
       Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h);

       // 判断是否为代理类 <br/>
       // cl: 待判断类
       public static boolean isProxyClass(Class<?> cl);

       // 获取代理对象的 InvocationHandler <br/>
       // proxy : 代理对象
       InvocationHandler getInvocationHandler(Object proxy);
   }
   ```

   - 类加载器: 一般就是被代理对象的类加载器(这里由于存在多态，所以最好用被代理对象实现的接口的类加载器)
   - 获取被代理对象实现的接口的 Class 数组: [代理对象只有被代理对象的接口]target.getClass().getInterfaces() | new Class[]{ ArithmeticCaculator.class}
   - InvocationHandler

3. InvocationHandler: `invoke(proxy, Method method, Object[] args)`

   - proxy: 正在被返回的代理
   - method: 正在被调用的方法
   - args: 调用方法时传入的参数

   ```java
   public interface InvocationHandler {
       /**
        * proxy: 正在被返回的代理对象, 一般不会使用 class com.sun.proxy.$Proxy4
        * method: 调用方法
        * args: 调用方法参数
        **/
       // Processes a method invocation on a proxy instance and returns
       // the result.  This method will be invoked on an invocation handler
       // when a method is invoked on a proxy instance that it is associated with.
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
   }
   ```

4. sample

   ```java
   public class JdkCalculatorProxy {
       public static Calculator getProxy(Calculator calculator) {
           ClassLoader loader = calculator.getClass().getClassLoader();
           Class<?>[] interfaces = calculator.getClass().getInterfaces();
           InvocationHandler h =
                   new InvocationHandler() {
                       @Override
                       public Object invoke(Object proxy, Method method, Object[] args)
                               throws Throwable {
                           Object result = null;
                           try {
                               log.info("The method {} begins with: {}", method.getName(), args);
                               result = method.invoke(calculator, args);
                               log.info("The method {} ends with: {}", method.getName(), result);
                           } catch (Exception e) {
                           } finally {
                           }
                           return result;
                       }
                   };

           Object proxy = Proxy.newProxyInstance(loader, interfaces, h);
           return (Calculator) proxy;
       }
   }

   public static void main(String[] args) {

       System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
       Calculator proxy = JdkCalculatorProxy.getProxy(new CalculatorImpl());
       proxy.add(1, 1);
       log.info("{}", proxy.getClass());
   }
   ```

## 动态代理-cglib 代理

1. core

   - 基于 ASM 的字节码处理库, 在 jvm 启动时动态生成 Java 子类(重写方法并实现增强): aop 为典型应用

2. sample

   ```java
   public class CglibCalculatorProxy implements MethodInterceptor {
       @Override
       public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
           log.info("The method {} begins with: {}", method.getName(), objects);
           Object result = methodProxy.invokeSuper(o, objects);
           log.info("The method {} ends with: {}", method.getName(), result);
           return result;
       }
   }

   public class ClientTest {
       public static void main(String[] args) {
           CglibCalculatorProxy proxy = new CglibCalculatorProxy();
           // 动态代理创建的class文件存储到本地
           System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "d:\\code");
           // 通过cglib动态代理获取代理对象的过程，创建调用的对象,在后续创建过程中EnhanceKey的对象
           // 所以在进行enhancer对象创建的时候需要把EnhancerKey（newInstance）对象准备好,恰好这个对象也需要动态代理来生成
           Enhancer enhancer = new Enhancer();
           // 设置enhancer对象的父类
           enhancer.setSuperclass(CalculatorImpl.class);
           // 设置enhancer的回调对象
           enhancer.setCallback(new CglibCalculatorProxy());
           // 创建代理对象
           CalculatorImpl myCalculator = (CalculatorImpl) enhancer.create();
           // 通过代理对象调用目标方法
           myCalculator.add(1, 1);
           System.out.println(myCalculator.getClass());
       }
   }
   ```

## reference

1. [spring-aop](/java/ee/spring/spring-framework/3.aop.md)
