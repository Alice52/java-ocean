## version 2

1. findFirst() 选取的第一个元素为空则会 NPE: **原因是 Optional.of() 不允许为空**

   ```java
   // NullPointerException - if the element selected is null
   findFirst();

   List<String> strings = new ArrayList<>();
   strings.add(null);
   strings.add("test");

   String firstString = strings.stream()
           .findFirst/finAny() // Exception thrown here
           .orElse("StringWhenListIsEmpty");
   ```

---

### introduce

- `用 Optional 来包装一个可能为 null 值的变量, 其最大意义其实仅仅在于给了调用者一个明确的警示`
- Optional 的好处在于可以**简化一系列判断 null 的操作**, 使得用起来的时候看着不需要判断 null, 表现出来好像用 Optional 就不需要关心空指针的情况
- Optional<T> 类(java.util.Optional) 是一个容器类, 代表一个值存在或不存在, 原来用 null 表示一个值不存在, 现在 Optional 可以更好的表达这个概念. 并且可以避免空指针异常.
- 常用方法
  > Optional.of(T t): 创建一个 Optional 实例
  > Optional.empty(): 创建一个空的 Optional 实例
  > Optional.ofNullable(T t): 若 t 不为 null, 创建 Optional 实例, 否则创建空实例
  > ~~isPresent~~(): 判断是否包含值
  > **orElse**(T t): 如果调用对象包含值, 返回该值, 否则返回 t
  > **orElseGet**(Supplier s): 如果调用对象包含值, 返回该值, 否则返回 s 获取的值
  > **map**(Function f): 如果有值对其处理, 并返回处理后的 Optional, 否则返回 Optional.empty()
  > **flatMap**(Function mapper): 与 map 类似, 要求返回值必须是 Optional

### Optional(T value), empty(), of(T value), ofNullable(T value)

- 通过 of(T value)函数所构造出的 Optional 对象, 当 Value 值为空时, 依然会报 NullPointerException; 通过 of(T value)函数所构造出的 Optional 对象, 当 Value 值不为空时, 能正常构造 Optional 对象.
- Optional 类内部还维护一个 value 为 null 的对象, empty() 的作用就是返回 EMPTY 对象
- ofNullable(T value): value can be null

  ```java
  public static <T> Optional<T> ofNullable(T value) {
      return value == null ? empty() : of(value);
  }
  ```

### ~~orElse(T other)~~, orElseGet(Supplier<? extends T> other), orElseThrow(Supplier<? extends X> exceptionSupplier)

- 都是在构造函数传入的 value 值为 null 时, 进行调用的. orElse 和 orElseGet 的用法如下所示, 相当于 value 值为 null 时, 给予一个默认值:

  ```java
  @Test
  public void test() {
      User user = null;
      user = Optional.ofNullable(user).orElse(createUser());
      user = Optional.ofNullable(user).orElseGet(() -> createUser());

  }
  public User createUser(){
      User user = new User();
      user.setName("zhangsan");
      return user;
  }
  ```

- 当 user 值不为 null 时, **orElse 函数依然会执行 createUser()方法, 而 orElseGet 函数并不会执行 createUser()方法**
- orElseThrow, 就是 value 值为 null 时,直接抛一个异常出去, 用法如下所示

  ```java
  User user = null;
  Optional.ofNullable(user).orElseThrow(()->new Exception("用户不存在"));
  ```

### map(Function<? super T, ? extends U> mapper) and flatMap(Function<? super T, Optional<U>> mapper)

- map: envople result with Optional, `做值的转换, 上一步的值 null 则直接返回上一步中的单例 Optional包装对象`

  ```java
  public class User {
      private String name;
      public String getName() {
          return name;
      }
  }

  String city = Optional.ofNullable(user).map(u-> u.getName()).get();
  ```

- flatMap: no envople result with Optional, and just return R

  ```java
  public class User {
      private String name;
      public Optional<String> getName() {
          return Optional.ofNullable(name);
      }
  }

  String city = Optional.ofNullable(user).flatMap(u-> u.getName()).get();
  ```

### ~~isPresent()~~和 ifPresent(Consumer<? super T> consumer)

- isPresent 即判断 value 值是否为空, 而 ifPresent 就是在 value 值不为空时, 做一些操作

  ```java
  Optional.ofNullable(user).ifPresent(u->{
      // do something
  });
  ```

### **filter**(Predicate<? super T> predicate)

- filter 方法接受一个 Predicate 来对 Optional 中包含的值进行过滤, 如果包含的值满足条件, 那么还是返回这个 Optional; 否则返回 Optional.empty.

  ```java
  // 如果user的name的长度是小于6的, 则返回. 如果是大于6的, 则返回一个EMPTY对象.
  Optional<User> user1 = Optional.ofNullable(user).filter(u -> u.getName().length()<6);
  ```

### sample

```java
public String getCity(User user)  throws Exception{
        if(user!=null){
            if(user.getAddress()!=null){
                Address address = user.getAddress();
                if(address.getCity()!=null){
                    return address.getCity();
                }
            }
        }
        throw new Excpetion("取值错误");
    }

// java 8
public String getCity(User user) throws Exception{
    return Optional.ofNullable(user)
                   .map(u-> u.getAddress())
                   .map(a->a.getCity())
                   .orElseThrow(()->new Exception("取指错误"));
}
```

```java
if(user!=null){
    dosomething(user);
}

// java 8
Optional.ofNullable(user)
    .ifPresent(u->{
        dosomething(u);
});
```

```java
public User getUser(User user) throws Exception{
    if(user!=null){
        String name = user.getName();
        if("zhangsan".equals(name)){
            return user;
        }
    }else{
        user = new User();
        user.setName("zhangsan");
        return user;
    }
}

// java 8
public User getUser(User user) {
    return Optional.ofNullable(user)
                   .filter(u->"zhangsan".equals(u.getName()))
                   .orElseGet(()-> {
                        User user1 = new User();
                        user1.setName("zhangsan");
                        return user1;
                   });
}
```

```java
@Test
public void givenOptional_whenFlatMapWorks_thenCorrect2() {
    Person person = new Person("john", 26);
    Optional<Person> personOptional = Optional.of(person);

    Optional<Optional<String>> nameOptionalWrapper
      = personOptional.map(Person::getName);
    Optional<String> nameOptional
      = nameOptionalWrapper.orElseThrow(IllegalArgumentException::new);
    String name1 = nameOptional.orElse("");
    assertEquals("john", name1);

    String name = personOptional
      .flatMap(Person::getName)
      .orElse("");
    assertEquals("john", name);
}
```

```JAVA
@Test
public void givenTwoEmptyOptionals_whenChaining_thenDefaultIsReturned() {
    String found = Stream.<Supplier<Optional<String>>>of(
      () -> createOptional("empty"),
      () -> createOptional("empty")
    )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst()
      .orElseGet(() -> "default");

    assertEquals("default", found);
}
```

---

## refernece

1. https://mp.weixin.qq.com/s/R0OK1KKX9EXAh2DnHWpoHw
