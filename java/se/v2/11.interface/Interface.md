## V2

### 接口: 抽象方法和常量的集合

1. 定义: `Java 接口是一系列方法的声明, 是一些方法特征的集合`, 一个接口只有方法的特征没有方法的实现, 因此这些方法可以在不同的地方被不同的类实现, 而这些实现可以具有不同的行为(功能).
2. 接口(Interface), `在JAVA编程语言中是一个抽象类型(Abstract Type)`, 它被用来要求类(Class)必须实现指定的方法, 使不同类的对象可以利用相同的界面进行沟通. 接口通常以 interface 来宣告.
3. 它仅能包含`方法签名`(Method Signature)以及`常数宣告`(变量宣告包含了 static 及 final), 一个接口不会包含方法的实现(仅有定义).
4. 一个方法的特征仅包括方法的 `名字` , `参数的数目` 和 `种类`, 而不包括方法的返回类型, 参数的名字以及所抛出来的异常

### notice

1. 使用 interface 来定义接口, java8 中可以有默认实现 default
2. 接口中所有的`成员变量`默认都是 `public static final` 修饰的, 在`声明时必须赋值`
   常量标识符的书写要求：字母都大写, 多个单次使用\_连接
3. 接口中所有的方法默认都是使用 `public abstract` 修饰, 接口没有构造方法
4. 实现接口用 `implements` 关键字, 若一个类即实现了接口又继承了父类, 则把 extends 关键字放在前面; [先继承后实现]
   即`先继承父类, 后实现多个接口`, 一个类可以实现多个无关接口, 使用 `,` 分割[可以利用他们模拟多重继承]
5. `接口也可以继承另一个接口, 使用 extends 关键字`
6. 实现接口中的类必须提供接口中所有方法的具体实现; 若为 `abstract` 则另当别论
7. 多个无关的类可以实现同一接口
8. `与继承类似, 接口和实现类之间存在多态性`
9. 接口无法被实例化, 但是可以被实现

   ```java
   Comparable x; //这是允许的.
   ```

10. 在 Java 中, 接口类型可用来宣告一个变量, 他们可以`成为一个空指针`, 或是被绑定在一个`以此接口实现的对象`.

### 接口[has-a] & 抽象[is-a]

1. **接口**: 抽象方法和常量的集合, java8 中引入 default
   - 接口是一系列方法的声明, 是一个抽象类型, 没有实现{`public abstract` 修饰}[扩展性{不同的实现}]
   - 仅方法签名 + 常数宣告{不能包含属性}
   - 类实现接口的时候, 必须实现接口中声明的所有方法
   - 可以多实现, **接口间可以多继承**
   - 接口可以声明变量: `Comparable x;`
   - **本质: 就是一组协议或者约定, 是功能提供者提供给使用者的一个`功能列表`**
2. 抽象**类**
   - 不能实例化
   - 含属性和方法
   - 子类继承抽象类, 必须实现抽象类中的所有抽象方法
   - 只能单继承
   - 作用: 代码复用 + 模板中必须重写的方法 + 多态的优雅实现方案
3. 区别
   - 使用接口来实现面向对象的**抽象**特性、**多态**特性, 基于接口而非实现的**设计原则**
   - 使用抽象类来实现面向对象的**继承特性**和**模板设计模式**等
   - 使用规则: `抽象: is-a; 接口: has-a/can-do/behaves like`
   - 抽象: 代码复用; 接口: 解耦{行为的抽象}
4. 特征标
   - 仅包括方法的 `名字`, `参数的数目` 和 `种类`
   - 不包括方法的**返回类型**, **参数的名字**以及所抛出来的**异常**
5. others
   - extends, implements: 先继承后实现
   - 类优先于接口. `如果一个子类继承的父类和接口有相同的方法实现. 那么子类继承父类的方法`
   - `子类型中的方法优先于父类型中的方法[就近原则]`
6. 如果做到面向接口编程
   - **设计初衷: 将接口和实现相分离, 封装不稳定的实现, 暴露稳定的接口{解耦&扩展性}**
   - 函数的命名不能暴露任何实现细节
   - 封装具体的实现细节
   - 为实现类定义抽象的接口

### 继承 & 组合

1. 解释继承的优缺点
   - 优点: 复用代码/模板定义 + 表示 is-a 关系 + 支持多态特性 + 简单
   - 缺点: 层次复杂带来的阅读和扩展性问题 + 破坏了封装性/灵活性
2. 解决方案: 组合 + 接口 + 委托
   - 继承改写成组合意味着要做更细粒度的类的拆分: **我们要定义更多的类和接口**
3. 继承与组合的选用
   - 继承: 类之间的继承结构稳定, 继承层次比较浅, 继承关系不复杂 + **重写外部类的具体实现**
   - 组合: 系统越不稳定, 继承层次很深, 继承关系复杂
4. 设计模式相关
   - 组合: 装饰者模式、策略模式、组合模式等
   - 继承: 模板模式
5. 继承 & 组合: 鸟 -- 能飞 -- 能叫 -- 能下蛋
   - 继承: `3 * 2 * 1 =6 个类`
   - 组合: 3 个能力接口

### 结论

1. class always win, sub-interface win,
2. implements is always sub-class have high prior
3. 类优先于接口. `如果一个子类继承的父类和接口有相同的方法实现. 那么子类继承父类的方法`
4. `子类型中的方法优先于父类型中的方法[就近原则]`
5. 如果以上条件都不满足, 则必须显示覆盖/实现其方法, 或者声明成 abstract.

## V1

### 接口继承多个父接口

```java
+---------------+         +------------+
|  Interface A  |         |Interface B |
+-----------^---+         +---^--------+
            |                 |
            |                 |
            |                 |
            +-+------------+--+
              | Interface C|
              +------------+
```

```java
interface A {
    default String say(String name) {
        return "hello " + name;
    }
}
interface B {
    default String say(String name) {
        return "hi " + name;
    }
}
interface C extends A,B{
    // 这里编译就会报错: error: interface C inherits unrelated defaults for say(String) from types A and B
}

interface C extends A,B{
    default String say(String name) {
        return "greet " + name;
    }
}
```

### 接口多层继承

```java
+---------------+
|  Interface A |
+--------+------+
         |
         |
         |
+--------+------+
|  Interface b |
+-------+-------+
        |
        |
        |
+-------+--------+
|   Interface C  |
+----------------+
```

- 很容易知道 C 会继承 B 的默认方法, 包括直接定义的默认方法, 覆盖的默认方法, 以及隐式继承于 A1 接口的默认方法.

  ```java
  interface A {
      default void run() {
          System.out.println("A.run");
      }

      default void say(int a) {
          System.out.println("A");
      }
  }
  interface B extends A{
      default void say(int a) {
          System.out.println("B");
      }

      default void play() {
          System.out.println("B.play");
      }
  }
  interface C extends B{

  }
  ```

### 多层多继承

```java
 +---------------+
|  Interface A1 |
+--------+------+
         |
         |
         |
+--------+------+         +---------------+
|  Interface A2 |         |  Interface B  |
+-------+-------+         +---------+-----+
        |       +---------+---------^
        |       |
        |       |
+-------+-------++
|   Interface C  |
+----------------+

```

```java
interface A1 {
    default void say(int a) {
        System.out.println("A1");
    }
}

interface A2 extends A1 {

}

interface B {
    default void say(int a) {
        System.out.println("B");
    }
}
// 必须重新写具有相同特征标的方法
interface C extends A2,B{
    default void say(int a) {
        B.super.say(a);
    }
}
```

### 复杂的

```java
+--------------+
 | Interface A1 |
 +------+------++
        |      ^+-------+
        |               |
+-------+-------+       |
|  Interface A2 |       |
+------------+--+       |
             ^--++      |
                 |      |
              +--+------+-----+
              |  Interface C  |
              +---------------+
```

```java
interface A1 {
    default void say() {
        System.out.println("A1");
    }
}
interface A2 extends A1 {
    default void say() {
        System.out.println("A2");
    }
}
interface C extends A2,A1{

}
static class D implements C {

}
public static void main(String[] args) {
    D d = new D();
    d.say(); // A2
}
```

### 类和接口的复合

- `子类优先继承父类的方法, 如果父类没有相同签名的方法, 才继承接口的默认方法`

```java
+-------------+       +-----------+
| Interface A |       |  Class B  |
+-----------+-+       +-----+-----+
            ^-+    +--+-----^
              |    |
          +---+----+-+
          |  Class C |
          +----------+
```

```java
interface A {
    default void say() {
        System.out.println("A");
    }
}
static class B {
    public void say() {
        System.out.println("B");
    }
}
static class C extends B implements A{

}
public static void main(String[] args) {
    C c = new C();
    c.say(); //B
}
```
