## Throwable

### Error

1. NoSuchMethodError
2. OutOfMemoryError

### Exception

1. RuntimeException
2. CompileException
3. usage:

   - 在方法参数列表后面使用 `throws` 关键字声明抛出异常
   - 异常在当前的方法内不处理，而是抛给调用这个方法的方法
   - 可以声明抛出多个异常，使用 , 分割
   - `运行时异常不需要使用 throws 关键字进行显示的抛出`
   - `重写的方法不能抛出比被重写方法范围更大的异常`
   - try{}catch(){}finally{}

   ```java
   try {
       int a=5/0;
   } catch (java.lang.Exception e) {
       e.getMessage();
       return 10;
   }finally{
       //一定会执行，在return 10之前
       System.out.println("finally...");
   }
   ```

4. 异常的层次:
   ![avatar](/static/image/java/throwable.png)

5. 异常 & 参数校验
   - 如果函数是 private 类私有的, 只在类内部被调用, 自己保证在调用这个 private 函数的时候, 不要传递 NULL 值或空字符串就可以了
   - 如果函数是 public 的, 为了尽可能提高代码的健壮性, 最好是在 public 函数中做 NULL 值或空字符串的判断

### diff between Error and Exception
