[toc]

## intros

1. intros

   - 由 @interface 修饰: `public @interface Anno {}`
   - 使用方法作为元素属性: `int age() default 0`
   - 可以`在编译(@Data) | 类加载 | 运行时(@Transactional)`被读取, 并执行相应的处理
   - 可以修饰: 包, 类, 构造器, 方法, 成员变量, 参数, 局部变量
   - 标记注解: 没有成员定义的 Annotation
   - 元数据注解: 包含成员变量的 Annotation
   - 提取 Annotation 信息: 反射获取

2. 分类: `解析注解方式: 编译期扫描 | 运行期反射`

   - 声明时注解: 6 个常见应用
   - 编译时注解
   - 动态设置注解的属性

3. 可用修饰符: 6

   - string
   - enum: can be logic
   - annotation
   - basic type
   - array
   - Class

4. 本质: `注解是继承了 Annotation 的个接口, @interface仅仅是个语法糖`

   - 实现类是由动态代理产生的: `-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true`

5. pros

   - 扩展性: 不改变原有逻辑的情况下, 在源文件中嵌入一些补充信息
   - 做功能声明, 提高代码的复用性, 简洁性

6. 常见内置注解

   ```java
   @Override // 限定重写父类方法, 该注释只能用于方法
   @Deprecated // 用于表示某个程序元素(类, 方法等)已过时
   @SuppressWarnings // 抑制编译器警告
   @SafeVarargs  // 将抑制与varargs使用相关的未检查的警告
   @FunctionalInterface
   ```

## 注解

1. 元注解: 修饰注解的注解

   - @Retention: 可以保留的时长
   - @Target: 可以修饰的元素
   - @Documented: 将被 java-doc 工具提取成文档
   - @Inherited: **类上的**继承相关
   - @Repeatable: 可重复使用
   - || 也可以是 @Import

2. @Retention

   - [Default] RetentionPolicy.CLASS: 编译器将把注解记录在 class 文件中. 当运行 Java 程序时, JVM 不会保留注解
   - RetentionPolicy.RUNTIME: 编译器将把注解记录在 class 文件中. 当运行 Java 程序时, JVM 会保留注解. 程序可以通过反射获取该注解
   - RetentionPolicy.SOURCE: 编译器直接丢弃这种策略的注解

3. @Target

   | 属性                        | 说明                   |
   | :-------------------------- | :--------------------- |
   | **ElementType.FIELD**       | 可以应用于字段或属性   |
   | **ElementType.METHOD**      | 可以应用于方法级注释   |
   | ElementType.TYPE            | 应用于类的任何元素     |
   | ElementType.PARAMETER       | 应用于方法的参数       |
   | ElementType.CONSTRUCTOR     | 可以应用于构造函数     |
   | ElementType.ANNOTATION_TYPE | 应用于注解类型         |
   | ElementType.PACKAGE         | 应用于包声明           |
   | ElementType.LOCAL_VARIABLE  | 可以应用于一个局部变量 |

4. @Inherited

   - 该继承性值发生于类上, 方法等其他的地方都不行[即使是 override]
   - 找到没有注解|方法信息, 且没有使用 @Inherited: AnnotatedElementUtils.findMergedAnnotation(child.getClass(), MyAnnotation.class)))

## 编译时注解

1. 注解处理器: 注解处理器是 javac 的一个工具, 用来在**编译时扫描和处理**注解

   - 以 Java 代码(字节码)作为输入, 生成文件(通常是.java 文件)作为输出

2. ~~maven 处理 plugin~~

   ```xml
   <properties>
      <!--main-->
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <compiler.level>1.8</compiler.level>
      <!--plugin-->
      <plugin.compiler.version>3.2</plugin.compiler.version>
      <maven-surefire-plugin.version>2.18.1</maven-surefire-plugin.version>
   </properties>

   <build>
      <plugins>
         <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>${plugin.compiler.version}</version>
               <configuration>
                  <source>${compiler.level}</source>
                  <target>${compiler.level}</target>
                  <!-- Disable annotation processing for ourselves.-->
                  <compilerArgument>-proc:none</compilerArgument>
                  <encoding>${project.build.sourceEncoding}</encoding>
               </configuration>
         </plugin>
      </plugins>
   </build>
   ```

3. @Setter 原理

   ![avatar](/static/image/java/se/annotation-processor.png)

4. [应用]: `@Serial` | `统计 jar 版本的使用比例`

   - 统计 jar 版本的使用比例: jar 中写成常量后上报 | gradle 全局替换(编译时) | 注解

   ```java
   // origin code
   @TrisceliVersion
   public static final String version = "";

   // compiled code
   @TrisceliVersion
   public static final String version = "1.0.31-SNAPSHOT";
   ```

5. 天坑

   - 与 lombok 一起使用时, 需要注意一定要优先执行直接的(pom 中依赖的顺序)
   - processor 外不能有 <build/> 脚本

## 应用

1. **功能声明 1**: `@Transactional` | `@RedisLimitRequest`

   - [link](https://github.com/Alice52/common-api/blob/master/common-redis/src/main/java/common/redis/aspect/LimitRequestAspect.java)

   ```java
   // @Aspect(cglib/proxy + asm) + @Pointcut + @Around

   // 01. anno
      @Target({TYPE, METHOD})
      @Retention(RetentionPolicy.RUNTIME)
      public @interface RedisLimitRequest {
         int count() default 0; /** 可以访问的次数 */
         long time() default 1;      /** 超时时长，默认1分钟 */
         TimeUnit timeUnit() default TimeUnit.MINUTES;    /** 超时时间单位，默认 分钟 */
         String message() default "调用频繁";  /** 错误提示 */
      }

   // 02. aspect
      @Aspect
      @Component
      public class LimitRequestAspect {
         @Pointcut("@annotation(redisLimitRequest)")
         public void pointCut(RedisLimitRequest redisLimitRequest) {}
         @Around("pointCut(redisLimitRequest)")
         public Object doPoint(ProceedingJoinPoint point, RedisLimitRequest redisLimitRequest){}
      }

   // 03. usage
      @RedisLimitRequest(count = 200)
      public R<String> save(@RequestBody String body) {}
   ```

2. **功能声明 2**: `@RocketMQBasedDelay`

   - [link](https://github.com/Alice52/practice/blob/main/backend/project-cloud-custom/practice-job/job-delay/src/main/java/top/hubby/job/delay/domain/order/service/delay/v5/dcpmq/support/annotation/RocketMQBasedDelay.java)

   ```java
   // MethodInterceptor + DefaultPointcutAdvisor

   // 01. anno
     @Target(ElementType.METHOD)
     @Retention(RetentionPolicy.RUNTIME)
     public @interface RocketMQBasedDelay {
        String topic(); // RocketMQ topic
        int delayLevel();  // 延时级别
        String consumerGroup();  // 消费者组信息
     }

   // 02. method interceptor
     @Slf4j
     public class SendMessageInterceptor implements MethodInterceptor {
        @Autowired private RocketMQTemplate rocketMQTemplate;

        @Override
        public Object invoke(MethodInvocation methodInvocation) throws Throwable {
           Method method = methodInvocation.getMethod();
           // 1. 获取 方法上的注解信息
           RocketMQBasedDelay rocketMQBasedDelay = method.getAnnotation(RocketMQBasedDelay.class);

           // 2. 将请求参数 转换为 MQ
           Object[] arguments = methodInvocation.getArguments();
           String argData = serialize(arguments);
           Message<String> message = MessageBuilder.withPayload(argData).build();

           // 3. 发送 MQ
           this.rocketMQTemplate.syncSend(
                    rocketMQBasedDelay.topic(), message, 200, rocketMQBasedDelay.delayLevel());
           log.info("success to sent Delay Task to RocketMQ for {}", Arrays.toString(arguments));
           return null;
        }
     }

     @Bean
     public SendMessageInterceptor messageSendInterceptor() { // 声明 AOP 拦截器 在调用 @RocketMQBasedDelay 注解方法时，自动拦截，将请求发送至 RocketMQ
        return new SendMessageInterceptor();
     }

     @Bean
     public PointcutAdvisor pointcutAdvisor( //对 @RocketMQBasedDelay 标注方法进行拦截
              @Autowired SendMessageInterceptor sendMessageInterceptor) {
        return new DefaultPointcutAdvisor(
                 new AnnotationMatchingPointcut(null, RocketMQBasedDelay.class),
                 sendMessageInterceptor);
     }

   // 03. usage
     @RocketMQBasedDelay(
              topic = "delay-task-topic-ann",
              delayLevel = 2,
              consumerGroup = "CancelOrderGroup")
     public void cancelOrder(Long orderId) {}
   ```

3. request advice: `@ControllerAdvice + RequestBodyAdviceAdapter`

   - [link](https://github.com/Alice52/common-api/blob/master/common-crypt/src/main/java/common/encrypt/advice/DecryptRequestAdvice.java)

   ```java
   @ControllerAdvice
   public class DecryptRequestAdvice extends RequestBodyAdviceAdapter {
      @Resource private ObjectMapper objectMapper;
      @Resource private AES decryptAes;

         // 方法上有DecryptionAnnotation注解的，进入此拦截器
      @Override
      public boolean supports( MethodParameter methodParameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
         return methodParameter.hasMethodAnnotation(Decrypt.class);
      }

      // 转换之后，执行此方法，解密，赋值
      @SneakyThrows
      @Override
      public Object afterBodyRead( Object body, HttpInputMessage inputMessage, MethodParameter parameter, Type targetType,  Class<? extends HttpMessageConverter<?>> converterType) {

         Decrypt decrypt = parameter.getMethodAnnotation(Decrypt.class);
         String field = decrypt.field();
         HttpServletRequest request = WebUtil.getCurrentRequest();

         // 获取数据
         ServletInputStream inputStream = request.getInputStream();
         JsonNode node = objectMapper.readValue(inputStream, JsonNode.class);
         JsonNode dnode;
         if (!node.has(field) || !(dnode = node.get(field)).isTextual()) {
               throw new BusinessException(DECRYPT_FIELD_NOT_FOUND);
         }
         String origin = dnode.asText();

         // 放入解密之前的数据
         request.setAttribute(CryptConstant.INPUT_ORIGINAL_DATA, origin);

         // 解密
         String decryptText;
         try {
               decryptText = AesUtils.decrypt(decryptAes, origin);
               // 放入解密之后的数据
               request.setAttribute(CryptConstant.INPUT_DECRYPT_DATA, decryptText);
               // 获取结果
               return objectMapper.readValue(decryptText, body.getClass());
         } catch (Exception e) {
               log.error("decrypt request occurs errors:  ", e);
               throw new BusinessException(CryptExceptionEnums.DECRYPT_FAILED);
         }
      }
   }
   ```

4. validator filed: `@Constraint(validatedBy = MobileDescriptor.class)`

   - [link](https://github.com/Alice52/common-api/blob/master/common-core/src/main/java/common/core/annotation/Mobile.java)

   ```java
   // @Constraint(validatedBy = MobileDescriptor.class)

   // 01. anno
      @Target({FIELD})
      @Retention(RUNTIME)
      @Constraint(validatedBy = MobileDescriptor.class)
      public @interface Mobile {
         boolean required() default false;
         String message() default "Invalid phone number, please check again";
         Class<?>[] groups() default {};
         Class<? extends Payload>[] payload() default {};
      }

   // 02. logic
      public class MobileDescriptor implements ConstraintValidator<Mobile, String> {
         private boolean required = false;
         @Override
         public void initialize(Mobile constraint) {
            this.required = constraint.required();
         }
         @Override
         public boolean isValid(String obj, ConstraintValidatorContext context) {
            if (!required && StrUtil.isBlank(obj)) {
                  return true;
            }
            return ValidatorUtil.validateMobile(obj);
         }
      }

   // 03. usage
      public class Person {
         @Mobile private String mobile;
      }
   ```

5. 作为策略模式的 key: 在 BeanFactoryPostProcessor 过程中收集该注解和修饰类信息(ClassScanner#scanPackageByAnnotation)

   - [link](https://github.com/Alice52/practice/blob/main/backend/project-cloud-custom/practice-coding/src/main/java/top/hubby/coding/elseif/strategy1/annotation/HandlerType.java)

   ```java
   // 01. anno and must implement by self
      @Target({ElementType.TYPE})
      @Retention(RetentionPolicy.RUNTIME)
      public @interface HandlerType {
         String source();
         String pay() default "";
      }

   // 02. context
      @Component
      public class OrderHandlerContext {
         private Map<HandlerType, AbstractOrderHandler> handlerMap;
         // Use HandlerType as key.
         @Autowired
         public OrderHandlerContext(List<AbstractOrderHandler> orderHandlers) {
            handlerMap = orderHandlers.stream()
                              .collect(Collectors.toMap(orderHandler -> AnnotationUtils.findAnnotation(orderHandler.getClass(), HandlerType.class),
                                             v -> v, (v1, v2) -> v1));
         }

         // Get different bean for order handle.
         public AbstractOrderHandler getInstance(OrderDTO order) {
            return handlerMap.get(new HandlerTypeImpl(order.getType(), order.getPay()));
         }
      }

   // 03. handler
   public abstract class AbstractOrderHandler {
      public abstract String handle(OrderDTO order);
   }
   @Component
   @HandlerType(source = OrderConstants.TYPE_GROUP_ORDER, pay = OrderConstants.PAY_WECHAT)
   public class GroupAliOrderHandler extends AbstractOrderHandler {
      @Override
      public String handle(OrderDTO order) {
         return OrderConstants.TYPE_GROUP_ORDER;
      }
   }
   @Component
   @HandlerType(source = OrderConstants.TYPE_GROUP_ORDER, pay = OrderConstants.PAY_ALI)
   public class GroupWechatOrderHandler extends AbstractOrderHandler {
      @Override
      public String handle(OrderDTO order) {
         return OrderConstants.TYPE_GROUP_ORDER;
      }
   }

   // 04. usage
   context.getInstance(order).handle(order);
   ```

6. 对 vo 脱敏: `@JsonSerialize(using = SensitiveJsonSerializer.class)`

   - [link](https://github.com/Alice52/common-api/blob/master/common-core/src/main/java/common/core/annotation/DeSensitive.java)

   ```java
   // 01. anno
      @JacksonAnnotationsInside
      @JsonSerialize(using = SensitiveJsonSerializer.class)
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.FIELD)
      public @interface DeSensitive {
         SensitiveStrategy strategy();
      }

   // 02. serialize
      public class SensitiveJsonSerializer extends JsonSerializer<String>
            implements ContextualSerializer {
         private SensitiveStrategy strategy;

         @Override
         public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
            gen.writeString(strategy.getDeSensitiver().apply(value));
         }

         // 用来获取实体类上的 @DeSensitive 注解并根据条件初始化对应的 JsonSerializer对象
         @Override
         public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property)
                  throws JsonMappingException {

            DeSensitive annotation = property.getAnnotation(DeSensitive.class);
            if (Objects.nonNull(annotation) && Objects.equals(String.class, property.getType().getRawClass())) {
                  this.strategy = annotation.strategy();
                  return this;
            }
            return prov.findValueSerializer(property.getType(), property);
         }
      }

   ```

7. 启用某功能: `@Import({EncryptConfig.class})`

   - [link](https://github.com/Alice52/common-api/blob/master/common-crypt/src/main/java/common/encrypt/annotation/EnableDecrypt.java)

   ```java
   // 01. anno
      @Import({DecryptConfig.class})
      @Target({ElementType.TYPE})
      @Retention(RetentionPolicy.RUNTIME)
      @Documented
      public @interface EnableDecrypt {}
   ```

8. 动态设置注解的属性: TestNG 中的 @DataProvider 注解, 拥有 dataProvider() 属性用来指定数据源

   ```java
   // 01. anno
      @Retention(RetentionPolicy.RUNTIME)
      @Target({ElementType.TYPE, ElementType.METHOD, ElementType.LOCAL_VARIABLE})
      public @interface Tag {
         String value() default "";
      }

   // 02. logic
      Tag tag = method.getAnnotation(Tag.class);
      InvocationHandler h = Proxy.getInvocationHandler(tag);
      Field hField = h.getClass().getDeclaredField("memberValues");  // memberValues 是注解的元属性
      hField.setAccessible(true);
      Map memberMethods = (Map) hField.get(h);  // 注解的所有属性方法后可修改
   ```

9. 编译时注解

   - [@SeData](https://github.com/Alice52/tutorials-sample/blob/master/java/javase/javase-annotation/annotation-processor/src/main/java/cn/edu/ntu/java/annotations/SeData.java)
   - [@Serial](https://github.com/Alice52/tutorials-sample/blob/master/java/javase/javase-annotation/annotation-processor/src/main/java/cn/edu/ntu/java/annotations/Serial.java)

10. 相关工具类

    - AnnotatedElementUtils: 可以帮助我们找出 父类和接口、父类方法和接口方法上的注解, 并可以处理桥接方法, 实现一键找到继承链的注解
    - ClassScanner: scanPackageByAnnotation 获取 package 下被指定注解修饰的数据
    - AnnotationUtils.findAnnotation

## sample

1. quick start: basic usage(reflect)

   - anno

     ```java
     @Target(value = {ElementType.TYPE, ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
     @Retention(RetentionPolicy.RUNTIME)
     public @interface HelloAnnotation {
        String major();

        int age();

        @Deprecated
        String school() default "NanTong";
     }
     ```

   - usage

     ```java
     @Data
     @HelloAnnotation(age = 12, major = "class")
     public class HelloAnnotationUsage {

        @HelloAnnotation(age = 12, major = "field")
        private String anno;

        @HelloAnnotation(age = 12, major = "method")
        public void test(@HelloAnnotation(age = 12, major = "parameter") String anno) {
        }
     }
     ```

   - test

     ```java
     @Slf4j
     public class HelloAnnotationTest {

        @SneakyThrows
        public static void main(String[] args) {
           HelloAnnotationUsage obj = new HelloAnnotationUsage();
           Class<?> clazz = obj.getClass();

           // 获取对象上的注解
           HelloAnnotation anno = clazz.getAnnotation(HelloAnnotation.class);
           log.info("class annotation:{}", anno);

           // 获取属性上的注解
           Field field = clazz.getDeclaredField("anno");
           anno = field.getAnnotation(HelloAnnotation.class);
           log.info("field annotation:{}", anno);

           // 获取方法上的注解
           Method method = clazz.getMethod("test", String.class);
           anno = method.getAnnotation(HelloAnnotation.class);
           log.info("method annotation:{}", anno);

           // 获取方法上的注解
           Annotation[][] pAnnos = method.getParameterAnnotations();
           log.info("parameter annotation:{}", pAnnos);
        }
     }
     ```

## Reference

1. https://blog.csdn.net/weixin_33768481/article/details/88606099
2. https://blog.csdn.net/ryo1060732496/article/details/80891058
3. https://mp.weixin.qq.com/s/_VzwbsYUgbY53bc8d9AJEA
