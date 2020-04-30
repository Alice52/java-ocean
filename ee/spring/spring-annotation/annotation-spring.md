## spring annotation

### qiuck start

1. pom

   ```xml
   <!-- it contain beans, core, aop,expression jar -->
   <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
   </dependency>
   ```

2. @Configuration is equivalent to bean.xml

   - xml config

   ```xml
   <bean id="person" class="cn.edu.ntu.javaee.annotation.model.Person">
       <property name="age" value="18"/>
       <property name="name" value="zack"/>
   </bean>
   ```

   ```java
   @Test
    public void testIocContainer() {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        Person person = (Person) applicationContext.getBean("person");
        log.info(String.valueOf(person));
    }

    @Test
    public void testIocObjects() {

        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        String[] definitionNames = applicationContext.getBeanDefinitionNames();

        Arrays.stream(definitionNames).forEach(System.out::println);
    }
   ```

   - annotation config

   ```java
   /*  method name is bean name in IOC container, but if specify value, which will be bean name.</br> */
   @Bean(value = "person")
   public Person injectPerson() {
     return new Person(19, "alice52");
   }
   ```

   ```java
   @Test
   public void testIocContainer() {
       ApplicationContext applicationContext =
           new AnnotationConfigApplicationContext(HelloConfig.class);
       Person person = applicationContext.getBean(Person.class);
       log.info(String.valueOf(person));
   }

   @Test
   public void testIocObjects() {

       ApplicationContext applicationContext =
               new AnnotationConfigApplicationContext(HelloConfig.class);
       String[] definitionNames = applicationContext.getBeanDefinitionNames();

       Arrays.stream(definitionNames).forEach(System.out::println);
   }
   ```

3. create singleton object and put it to IOC container

   - need can be scanned: **`<context:component-scan>`** or `@ComponentScan`

     ```xml
     <context:component-scan base-package="cn.edu.ntu.javaee.annotation" >
         <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     <!-- config use-default-filters="false" to use include-filter -->
     <!--
         <context:component-scan base-package="cn.edu.ntu.javaee.annotation" use-default-filters="false">
             <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
         </context:component-scan>
     -->
     ```

     ```java
     /* <code>@ComponentScan</code> includeFilters usage is: useDefaultFilters = false. <br> */
     @ComponentScan(
         basePackages = {"cn.edu.ntu.javaee.annotation"},
         excludeFilters = {
            @ComponentScan.Filter(
                type = FilterType.ANNOTATION,
                classes = {Controller.class})
        })
     ```

   - tell IOC to control it: `beans.xml` or `@Component`

---

## Bean

### @Bean

1. 注入两个相同名字的 bean, 第二个不会执行: 因为 IOC 内已经有了对应的 bean;
2. 注入两个不同名字的 bean, 即使 Type 一样也都会创建并放入 IOC 容器: 不同的名字是不同的;

3. code

   ```java
   @Configuration
   public class BeanAnno {
       @Bean(value = "person")
       public Person injectPerson() { return new Person(19, "alice52"); }

       /* This method will not execute due to IOC container already has person bean<br> */
       @Bean(value = "person")
       public Person injectPerson2() { return new Person(190, "alice520"); }

       /* This method will execute and put object to IOC container due to it has different bean name. */
       @Bean(value = "alice52")
       public Person injectPerson3() { return new Person(190, "alice52"); }
   }
   ```

4. junit test

   ```java
   @Slf4j
   public class BeanAnnoTest {
       private ApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanAnno.class);

       @Test
       public void testIocContainer() {
           Person person = (Person) applicationContext.getBean("person");
           log.info(String.valueOf(person));
           Person alice52 = (Person) applicationContext.getBean("alice52");
           log.info(String.valueOf(alice52));
       }

       @Test
       @Ignore
       /* This will throw {@link org.springframework.beans.factory.NoUniqueBeanDefinitionException} exception due to IOC container has two beans. <br> */
       public void testIoc() {
           Person person = applicationContext.getBean(Person.class);
           log.info(String.valueOf(person));
       }
   }
   ```

### @ComponentScan: class marked by this will be put IOC container, and it has nothing with @ComponentScan

1. type

   - FilterType.ANNOTATION
   - FilterType.ASSIGNABLE_TYPE
   - FilterType.REGEX
   - FilterType.ASPECT
   - FilterType.CUSTOM

2. CUSTOM should be custom class implementation TypeFilter

   ```java
   @Slf4j
   public class CustomTypeFilter implements TypeFilter {

      /**
        * If class name contains dao will be include to IOC container. <br>
        *
        * @param metadataReader the metadata reader for the target class
        * @param metadataReaderFactory a factory for obtaining metadata readers for other classes (such
        *     as superclasses and interfaces)
        * @return whether this filter matches
        * @throws IOException in case of I/O failure when reading metadata
        */
       @Override
       public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
           throws IOException {
           // get target annotation metadata
           AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
           // get target class metadata: subClass, superClass, interface
           ClassMetadata classMetadata = metadataReader.getClassMetadata();
           String className = classMetadata.getClassName();
           log.info(className);
           // get target class resource info
           Resource resource = metadataReader.getResource();

           return className.contains("dao");
       }
   }
   ```

3. code

   - CustomTypeFilter

   ```java
   @Slf4j
   public class CustomTypeFilter implements TypeFilter {

       /**
       * If class name contains dao will be include to IOC container. <br>
       *
       * @param metadataReader the metadata reader for the target class
       * @param metadataReaderFactory a factory for obtaining metadata readers for other classes (such
       *     as superclasses and interfaces)
       * @return whether this filter matches
       * @throws IOException in case of I/O failure when reading metadata
       */
       @Override
       public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
           throws IOException {
           // get target annotation metadata
           AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
           // get target class metadata: subClass, superClass, interface
           ClassMetadata classMetadata = metadataReader.getClassMetadata();
           String className = classMetadata.getClassName();
           log.info(className);
           // get target class resource info
           Resource resource = metadataReader.getResource();

           return className.contains("dao");
       }
   }
   ```

   - configuration

   ```java
   @Configuration
   @ComponentScan(
       basePackages = {"cn.edu.ntu.javaee.annotation"},
       includeFilters =  {
       @ComponentScan.Filter(
           type = FilterType.ANNOTATION,
           classes = {Controller.class}),
       @ComponentScan.Filter(
           type = FilterType.ASSIGNABLE_TYPE,
           classes = {PersonService.class}),
       @ComponentScan.Filter(type = FilterType.REGEX, pattern = ".*Service"),
       @ComponentScan.Filter(
           type = FilterType.CUSTOM,
           classes = {CustomTypeFilter.class})
       }, useDefaultFilters = false)
   public class ComponentScanConfig {

   }
   ```

### @Scope:

1. type

   - ConfigurableBeanFactory#SCOPE_PROTOTYPE:
     - will create object and put it to IOC container when get this bean from IOC **`for each time`** rather than IOC created.
   - ConfigurableBeanFactory#SCOPE_SINGLETON:
     - will create object and put it to IOC container when IOC created.
   - org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
   - org.springframework.web.context.WebApplicationContext#SCOPE_SESSION

2. xml configuration

   ```xml
   <bean id="person" class="cn.edu.ntu.javaee.annotation.model.Person" scope="prototype">
       <property name="age" value="18"/>
       <property name="name" value="zack"/>
   </bean>
   ```

3. annotation

   ```java
   @Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
   @Bean(value = "person")
   public Person injectPerson() {
       return new Person(19, "alice52");
   }
   ```

4. SCOPE_SINGLETON bean lazy load

   - donot created when IOC container created, created in first used

   - java

   ```java
   @Lazy
   @Bean(value = "person")
   public Person injectPerson() {
       return new Person(19, "alice52");
   }
   ```

### @Conditional: 满足条件时给容器注入 bean

1. spring

   - custom condition

   ```java
   @Slf4j
   public class CustomCondition implements Condition {
       /**
       * If the env is window, it will be inject to IOC container. <br>
       *
       * @param context the condition context
       * @param metadata metadata of the {@link org.springframework.core.type.AnnotationMetadata class},
       *     which is annotation info
       */
       @Override
       public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

           ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
           ClassLoader classLoader = context.getClassLoader();
           log.info(String.valueOf(classLoader));
           Environment environment = context.getEnvironment();
           BeanDefinitionRegistry registry = context.getRegistry();
           ResourceLoader resourceLoader = context.getResourceLoader();
           String osName = environment.getProperty("os.name");

           return StrUtil.containsIgnoreCase(osName, "windows");
       }
   }
   ```

   - usage: 实现当运行环境是 Window 时才注入这个 bean

   ```java
   @Conditional({CustomCondition.class})
   @Bean(value = "alice52")
   public Person injectPerson() {
       return new Person(19, "alice52");
   }
   ```

2. spring boot

   - application.xml

   ```yml
   swagger2:
     enable: true
   ```

   - configuration: when swagger2 is enabled, then do IOC about OnProperty

   ```java
   /**
   * swagger2 in config is true, then will execute follow code. <br>
   */
   @Configuration
   @ConditionalOnProperty(
       prefix = "swagger2",
       value = {"enable"},
       havingValue = "true")
   public class OnProperty {}
   ```

### @Import: 快速的给 IOC 容器内添加组件

1. code

   ```java
   /**
   * <code>@Import</code> will put class to IOC container, and bean name is full class name. <br>
   * And call NoArgsConstructor of Person. <br>
   */
   @Configuration
   @Import({Person.class})
   public class ImportAnno {}
   ```

2. use ImportSelector interface

   - selector

   ```java
   /**
   * import classes[full class name] from ImportSelector to IOC container. <br>
   */
   public class CustomImportSelector implements ImportSelector {

       /**
       * @param importingClassMetadata Metadata info about class marked by @Import annotation, such as
       *     other annotation, annotation info
       * @return String[] return the names of which class(es) should be imported
       */
       @Override
       public String[] selectImports(AnnotationMetadata importingClassMetadata) {
           String className = importingClassMetadata.getClassName();
           return new String[] {"cn.edu.ntu.javaee.annotation.common.model.Employee"};
       }
   }
   ```

   - usage

   ```java
   @Configuration
   @Import({Person.class, CustomImportSelector.class})
   public class ImportAnno {}
   ```

3. use ImportBeanDefinitionRegistrar interface

   - register

   ```java
   public class CustomImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

       /**
       * manual add bean to IOC container. <br>
       *
       * @param metadata annotation metadata of the importing class
       * @param registry current bean definition registry
       */
       @Override
       public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
           MergedAnnotations annotations = metadata.getAnnotations();
           annotations.stream().forEach(System.out::println);

           boolean containsStudentBeanDefinition = registry.containsBeanDefinition("student");
           if (!containsStudentBeanDefinition) {
               registry.registerBeanDefinition("student", new RootBeanDefinition(Student.class));
           }
       }
   }
   ```

   - usage

   ```java
   @Configuration
   @Import({Person.class, CustomImportSelector.class, CustomImportBeanDefinitionRegistrar.class})
   public class ImportAnno {}
   ```

4. use Spring FactoryBean

   - factory

   ```java
   public class DogFactoryBean implements FactoryBean<Dog> {

       @Override
       public boolean isSingleton() {
           return true;
       }

       @Override
       public Dog getObject() throws Exception {
           return new Dog();
       }

       @Override
       public Class<?> getObjectType() {
           return Dog.class;
       }
   }
   ```

   - usage

   ```java
   @Configuration
   public class ImportAnno {
       @Bean
       public DogFactoryBean dogFactoryBean() {
           return new DogFactoryBean();
       }
   }
   ```

   - junit test

   ```java
   @Test
   public void testGetObjectFromIocFactoryBean() {

       Object bean = applicationContext.getBean("dogFactoryBean");
       // class cn.edu.ntu.javaee.annotation.common.model.Dog
       log.info(String.valueOf(bean.getClass()));

       // Used to dereference a FactoryBean instance and distinguish it from beans created by the FactoryBean. <br>
       // For example, if the bean named dogFactoryBean is a FactoryBean, getting &dogFactoryBean will return the factory, not the instance returned by the factory.
       Object bean2 = applicationContext.getBean("&dogFactoryBean");
       // class cn.edu.ntu.javaee.annotation.impor.factory.DogFactoryBean
       log.info(String.valueOf(bean2.getClass()));

       Object bean3 = applicationContext.getBean(DogFactoryBean.class);
       // class cn.edu.ntu.javaee.annotation.impor.factory.DogFactoryBean
       log.info(String.valueOf(bean3.getClass()));
   }
   ```

---

## lifecycle

- **bean create -- bean init[`对象创建完成且赋值好之后`] -- use -- destroy**

  - singleton:
    - init: 容器创建时创建对象, 并调用 Init() 方法
    - destroy: 容器关闭时调用
  - prototype:
    - init: 第一次使用时才会创建对象, 并调用 Init() 方法
    - destroy: 容器只会创建这个 Bean 但是不会销毁[管理], 如果需要则自己手动销毁

### @Bean

1. specify init and destroy method

   - xml comfig

   ```xml
   <bean id="person" class="cn.edu.ntu.javaee.annotation.common.model.Person" init-method="" destroy-method="">
       <property name="age" value="18"/>
       <property name="name" value="zack"/>
   </bean>
   ```

   - annotation

   ```java
   @Bean(value = "person", initMethod = "init", destroyMethod = "destroy")
   public Person injectPerson() {
       return new Person();
   }
   ```

   - test

   ```java
   Person person = applicationContext.getBean(Person.class);
   log.info(String.valueOf(person));

   ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) this.applicationContext;
   context.close();

   AnnotationConfigApplicationContext context = (AnnotationConfigApplicationContext) this.applicationContext;
   context.close();
   ```

### InitializingBean#afterPropertiesSet && DisposableBean#destroy

1. Dog model

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @ToString
   @Slf4j
   public class Dog implements InitializingBean, DisposableBean {
       private Integer age;
       private String name;
       private String color;

       @Override
       public void afterPropertiesSet() throws Exception {
           log.info("1. Person object afterPropertiesSet method execute.");
       }

       public void init() {
           log.info("2. Person object init method execute.");
       }

       @Override
       public void destroy() throws Exception {
           log.info("3. Person object destroy[DisposableBean] method execute.");
       }

       public void destroy0() {
           log.info("4. Person object destroy0 method execute.");
       }
   }
   ```

2. usage
   ```java
   @Bean(value = "dog", initMethod = "init", destroyMethod = "destroy0")
   public Dog injectPerson() { return new Dog();  }
   ```

### @PostConstruct[在 bean 创建完成并赋值之后执行] && @PreDestroy[bean 被从容器移除之前]

1. Dog model

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @ToString
   @Slf4j
   public class Dog implements InitializingBean, DisposableBean {
       private Integer age;
       private String name;
       private String color;

       @PostConstruct
       public void init1() {
           log.info("1. Person object init1 method execute.");
       }

       @Override
       public void afterPropertiesSet() throws Exception {
           log.info("2. Person object afterPropertiesSet method execute.");
       }

       public void init() {
           log.info("3. Person object init method execute.");
       }

       @PreDestroy
       public void destroy1() {
           log.info("4. Person object destroy1 method execute.");
       }

       @Override
       public void destroy() throws Exception {
           log.info("5. Person object destroy[DisposableBean] method execute.");
       }

       public void destroy0() {
           log.info("6. Person object destroy0 method execute.");
       }
   }
   ```

2. usage
   ```java
   @Bean(value = "dog", initMethod = "init", destroyMethod = "destroy0")
   public Dog injectPerson() { return new Dog();  }
   ```

### BeanPostProcessor: postProcessBeforeInitialization && postProcessAfterInitialization

- processor flow

```java
BeanPostProcessor

populateBean(beanName, mbd, instanceWrapper); // set value of bean
initializeBean
{
   applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   invokeInitMethods(beanName, wrappedBean, mbd); // execute custom init
   applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}
```

1. Dog model

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   @ToString
   @Slf4j
   public class Dog implements InitializingBean, DisposableBean {
       private Integer age;
       private String name;
       private String color;

       @PostConstruct
       public void init1() {
           log.info("2. Person object init1 method execute.");
       }

       @Override
       public void afterPropertiesSet() throws Exception {
           log.info("3. Person object afterPropertiesSet method execute.");
       }

       public void init() {
           log.info("4. Person object init method execute.");
       }

       @PreDestroy
       public void destroy1() {
           log.info("6. Person object destroy1 method execute.");
       }

       @Override
       public void destroy() throws Exception {
           log.info("7. Person object destroy[DisposableBean] method execute.");
       }

       public void destroy0() {
           log.info("8. Person object destroy0 method execute.");
       }
   }
   ```

2. CustomBeanPostProcessor

   ```java
   @Slf4j
   public class CustomBeanPostProcessor implements BeanPostProcessor {

       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName)
           throws BeansException {
           log.info("1. postProcessBeforeInitialization, bean: {}, beanName: {}", bean, beanName);
           return bean;
       }

       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
           log.info("5. postProcessAfterInitialization, bean: {}, beanName: {}", bean, beanName);
           return bean;
       }
   }
   ```

3. usage

   ```java
   @Configuration
   @ComponentScan(
       basePackages = "cn.edu.ntu.javaee.annotation",
       includeFilters = {
       @ComponentScan.Filter(
           type = FilterType.ASSIGNABLE_TYPE,
           classes = {CustomBeanPostProcessor.class})
       })
   public class BeanInitAndDestroyInter {

   @Bean(value = "dog", initMethod = "init", destroyMethod = "destroy0")
       public Dog injectPerson() {
           return new Dog();
       }
   }
   ```

4. junit test

   ```java
   @Test
   public void testBeanInitAndDestroy() {
       Dog dog = applicationContext.getBean(Dog.class);
       log.info(String.valueOf(dog));

       AnnotationConfigApplicationContext context =
           (AnnotationConfigApplicationContext) this.applicationContext;
       context.close();
   }
   ```

## functional

### put object to IOC container

1. @Component/@Controller + @ComponentScan

   - 如果 @ComponentScan 使用自定义的方式或者指定包含某些类时[ASSIGNABLE_TYPE, REGEX, CUSTOM], 没有 `@Component/@Controller` 也可以成功注入 IOC
   - ANNOTATION 必须使用 `@Component/@Controller` 才能成功注入 IOC

2. @Bean 将第三方包中的组件归置到 IOC 管理

3. @Import

   - @Import
   - ImportSelector
   - ImportBeanDefinitionRegistrar
   - Spring FactoryBean

---

### bean lifecycle

- **bean create -- bean init[`对象创建完成且赋值好之后`] -- use -- destroy**

  - singleton:
    - init: 容器创建时创建对象, 并调用 Init() 方法
    - destroy: 容器关闭时调用
  - prototype:
    - init: 第一次使用时才会创建对象, 并调用 Init() 方法
    - destroy: 容器只会创建这个 Bean 但是不会销毁[管理], 如果需要则自己手动销毁

1. BeanPostProcessor
2. **`@PostConstruct && @PreDestroy`**
3. **InitializingBean && DisposableBean**
4. `@Bean(value = "person", initMethod = "init", destroyMethod = "destroy")`

### get IOC container

1. configuration

   ```java
   @Configuration
   public class IocContainer implements ApplicationContextAware {
       private ApplicationContext applicationContext;

       public ApplicationContext getApplicationContext() {
           return this.applicationContext;
       }

       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           this.applicationContext = applicationContext;
       }

       @Bean
       public Person person() {
           return new Person();
       }
   }
   ```

2. junit test

   ```java
   @Slf4j
   public class IocContainerTest {
       private ApplicationContext applicationContext = new AnnotationConfigApplicationContext(IocContainer.class);

       @Test
       public void testGetIocContainer() {
           IocContainer contextBean = applicationContext.getBean(IocContainer.class);
           ApplicationContext applicationContext = contextBean.getApplicationContext();

           Person person0 = applicationContext.getBean(Person.class);
           log.info(String.valueOf(person0));

           Person person = applicationContext.getBean(Person.class);
           log.info(String.valueOf(person));

           Assert.isTrue(person0 == person);
           Assert.isFalse(applicationContext == contextBean);
       }
   }
   ```

### common annotation

1. @Component

2. @Service

3. @Repository

4. @Controller

5. @Autowired

   - byType

6. @Resource

   - byName

7. @Inject
