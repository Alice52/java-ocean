## outline

1. 本质: 1
2. 实现原理: **2 - 2**
3. 好处: 4
4. 泛型使用: 3[3*1]
5. 泛型参数:8
6. 泛型擦除:
   - 过程: 2
   - 影响: 5
7. conlusion: 2
8. 查看编译之后的代码: arthas

## subject

1. Generic: `参数化类型 = 编译检查 + 强转`

---

## dimension

### 1.实现原理

1. 模式: Code Share
2. 原理:

   - 为每个泛型类型创建一个`唯一`的字节码: `共享静态属性`
   - 将所有的该泛型类型的实例都映射到这个唯一的字节码表示上: `泛型擦除`

   ```java
   // 当泛型内包含静态变量
   public class StaticTest{
       public static void main(String[] args){
           GT<Integer> gti = new GT<Integer>();
           gti.var=1;
           GT<String> gts = new GT<String>();
           gts.var=2;
           System.out.println(gti.var); // 2, 所有的泛型共享静态变量
       }
   }
   class GT<T>{
       public static int var=0;
       public void nothing(T x){}
   }
   ```

### 2.好处

1. 扩展性
2. OOP
3. 阅读性
4. 编译检查

### 3.泛型使用

1. 泛型类: <E> 只有在非静态方法上才可以使用

   ```java
   public class GenericWildcard<E> {

       @Deprecated
       public static <T extends Person> void mergeSameType(List<T> dest, List<T> src) {
           src.forEach(x -> dest.add(x));
       }

       public <T extends E> void mergeSameType2(List<T> dest, List<T> src) {
           src.forEach(x -> dest.add(x));
       }

       public static <T> void merge(List<? super T> dest, List<T> src) {
           src.forEach(x -> dest.add(x));
       }
   }
   ```

2. 泛型接口: `interface & abstract` 获取子类的具体泛型类型

   ```java
   public abstract class BaseGenericMethod<T> {

       public T produce() {
           Type type = this.getClass().getGenericSuperclass();
           ParameterizedType parameterizedType = (ParameterizedType) type;
           Class<T> clazz = (Class<T>) parameterizedType.getActualTypeArguments()[0];

           return createInstance(clazz);
       }

       @SneakyThrows
       public T createInstance(Class<T> clazz) {

           return clazz.newInstance();
       }

       public abstract int hash();
   }

   public class GenericMethodImpl extends BaseGenericMethod<Person> {
       @Override
       public int hash() {
           Person instance = this.createInstance(Person.class);
           return Objects.hash(instance);
       }
   }

   //private GenericMethodImpl genericMethod = new GenericMethodImpl();
   // Person person = genericMethod.produce();
   ```

3. 泛型方法: 创建实例

   ```java
   (T[]) Array.newInstance(clazz, length);
   clazz.newInstance();
   (T) constructor.newInstance(init)
   ```

### 4.泛型参数

1. `E, K, V, S, T, ?`
2. `T` 是明确类型,且可以确保类型一致性 ; `?` 表示不确定类型, 且为局部变量时没有意义
3. 只有 `T super A` Error

   ```java
   T extends A // OK
   T super A // error
   ? extends A // OK
   ? super A // OK
   ```

4. `T` 可以多限定: `&`; `?` 不可以

   ```java
   public <T extends Number & Object> void merge(List<T> dest, List<T> src)
   ```

5. `T` 可以操作; `?` 不可以, 提供了一种只读的无关类型的限制

   ```java
   // 提供了一种只读的无关类型的限制
   public void change(Collection<?> collection) {
       collection.add(null);//在這個方法中, 传入任何数据都是错误的, 除了null
   }
   ```

6. `T` 可以定义变量; `?` 不可以

   ```java
   T t = operate(); // 可以
   ? car = operate(); // 不可以
   ```

7. `?` 可以作为限制类型使用; `T` 不可以

   ```java
   public class A {
       public Class<?> clazz;
       public Class<T> clazzT; // error
   }
   public class B<T> {
       public Class<?> clazz;
       public Class<T> clazzT;
   }
   ```

8. `?` 常用语形参和方法调用; `T` 泛型类和泛型方法的定义

   ```java
   // 通过 T 来 确保 泛型参数的一致性
   public <T extends Number> void merge(List<T> dest, List<T> src)

   //通配符是 不确定的, 所以这个方法不能保证两个 List 具有相同的元素类型
   public void merge(List<? extends Number> dest, List<? extends Number> src) // error
   ```

### 5.泛型擦除

1. 过程

   - 将所有的泛型参数用其最左边界[边界值]类型替换
   - 移除所有的参数泛型(所说的 Java 泛型在字节码中会被擦除，并不总是擦除为 Object 类型，而是擦除到上限类型)
   - 如果是继承基类而来的泛型，就用 getGenericSuperclass() , 转型为 ParameterizedType 来获得实际类型
   - 如果是实现接口而来的泛型，就用 getGenericInterfaces() , 针对其中的元素转型为 ParameterizedType 来获得实际类型

2. 影响

   - `ArrayList<Integer> --> ArrayList`
   - `m(T t) --> getDeclaredMethod("m", Object.class)`
   - `ArrayList<Integer> 可以放入 String 值`
   - `ArrayList<Object> 不是 ArrayList<Integer> 的子类`
   - `#question#1`

   ```java
   List<Integer> list = new ArrayList<Integer>();
   list.add(66);
   int num = list.get(0);
       list.getClass(); // ArrayList
   // javac
   List list = new ArrayList();
   list.add(Integer.valueOf(66));
   int num = ((Integer) list.get(0)).intValue();

   public static <A extends Comparable<A>> A max(Collection<A> xs) {
       Iterator<A> xi = xs.iterator();
       A w = xi.next();
       while (xi.hasNext()) {
           A x = xi.next();
           if (w.compareTo(x) < 0)
               w = x;
       }
       return w;
   }
   // javac
   public static Comparable max(Collection xs){
       Iterator xi = xs.iterator();
       Comparable w = (Comparable)xi.next();
       while(xi.hasNext())
       {
           Comparable x = (Comparable)xi.next();
           if(w.compareTo(x) < 0)
               w = x;
       }
       return w;
   }

   public class Erasure <T>{
       public void add(T object) {

       }
   }

   // add(T object) 泛型擦除之后时add(Object object)
   ```

   ```java
   List<Integer> ls = new ArrayList<>();
   ls.add(23);
   // ls.add("text"); // Error
   Method method = ls.getClass().getDeclaredMethod("add",Object.class);
   method.invoke(ls,"test");
   method.invoke(ls,42.9f);
   ```

3. common error

   - instanceof T; // Error
   - new T(); // Error
   - new T[10]; // Error

4. suggestion

   - pass T
   - and pass Class for type

---

## conclusion

1. 泛型设计的精髓就是忽略泛型, 尽量不要想着 new: `new 就需要知道具体的类型, 泛型就失去了意义`
2. `使用时需要指定 T, 和 T 的 Class: 因为泛型被擦除了`
3. 清除 T 类型是错误的想法，请将 Class 作为参数传递
4. Generic 的 Subtle 没有类型，所以不要尝试清除 T 类型
5. [应用](https://github.com/Alice52/tutorials-sample/blob/master/db/jdbc/jdbc-sample/src/main/java/cn/edu/ntu/jdbc/sample/generics/dao/BaseDAO.java)

---

## task

1. 创建泛型对象

   ```java
   (T[]) Array.newInstance(clazz, length);
   clazz.newInstance();
   (T) constructor.newInstance(init)
   ```

2. Array.newInstance(Class, length)

   ```java
   // T[] array = new T[]; // compile error
   public static <T> T[] createArray(Class<T> componentType, int length) {
   return (T[]) Array.newInstance(componentType, length);
   }
   // Integer[] array = CreateWithGeneric.createArray(Integer.class, 2);
   ```

3. create generic list

   - 创建一个普通对象 + add to list: Arrays.asList(array)

     ```java
     @SneakyThrows
     public static <T> List<T> createList(Class<T> componentType) {
          List<T> list = new ArrayList<>();
          T t;
          if (wrapperPrimitiveMap.containsKey(componentType)) {
            t = null;
          } else {
            t = componentType.newInstance();
          }
          list.add(t);

          return list;
      }

      // List<Person> people = CreateWithGeneric.createList(Person.class); // [Person{name='null', age=0}]

      @SneakyThrows
      public static <T> List<T> createList(Class<T> componentType, int length) {
          return Arrays.asList((T[]) Array.newInstance(componentType, length));
     }
     // List<Integer> list1 = CreateWithGeneric.createList(Integer.class, 2); // [null, null]
     ```

## question

- [x] 1. create instance without pass Class as args, `It cannot implement`

  ```java
  public class Gen<T> {
      // 想创建一个T对象并返回
      public T createInstance() {
          // 这里怎么写?
      }
  }

  // javac
  public class Gen<Object> {
      // 想创建一个T对象并返回
      public Object createInstance() {
          // T 是 Object, 不知道具体的类型, 所以无法创建T对象
      }
  }

  public static void main(string []args) {
      Gen<Person> pg = new Gen<>();
      Person p = pg.createInstance();
  }
  ```

- [ ] 2. 在泛型翻出之后这段代码怎么可以取到泛型的值

  ```java
  Type type = this.getClass().getGenericSuperclass();
  ParameterizedType parameterizedType = (ParameterizedType) type;
  Class<T> clazz = (Class<T>) parameterizedType.getActualTypeArguments()[0];
  ```

- [x] 3. 不能使用以下方式
  - ~~instanceOf T~~
  - ~~new T()~~
  - ~~new T []~~
