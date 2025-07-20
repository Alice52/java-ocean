## 介绍

1. 本质: 为了打破双亲委派机制模型而创建出来的机制, 属于没有办法的事情[SpringBoot 其实也是借助类加载器利用工厂机制进行一个全 jar 包加载并实例化的过程]
2. 比如 Springboot 注解能扫描到 main 函数同路径及包下的所有的类[所以 SpringBoot 就做了一个全局自顶向下的一个可扩展的加载机制], 那么像一些第三方的 jar 如果也需要实例化, 只有要配置里定义后, 通过 SPI 机制扫描, 手动的加载到容器中
3. se/jvm/classloader.md: `META-INF/interface-full-name` + impl content + 可插拔
4. spring boot: `META-INF/spring.factories` + impl content

## usage

1. jdk 的 spi: ServiceLoader
2. boot 的 spi
3. dubbo 的 spi
4. mysql 中的使用

## spi

1. 配置 `resource/META-INF/services/cn.edu.ntu.javase.classloader.SpiInterface` 接口名文件, 内容是实现类

   ```js
   // 必须是实现类
   cn.edu.ntu.javase.classloader.SpiImpl;
   ```

2. ServiceLoader 会加载: 不需要自己写反射和 find 的代码

   ```java
   public static void main(String[] args) {
       SpiInterface spi = null;
       // load(class, loader) 进行制定类加载器进行加载
       ServiceLoader<SpiInterface> spis = ServiceLoader.load(SpiInterface.class);
       Iterator<SpiInterface> iterator = spis.iterator();

       if (iterator.hasNext()) {
         spi = iterator.next();
       }

       Optional.ofNullable(spi).ifPresent(System.out::println);
   }
   ```

---

## reference

1. https://mp.weixin.qq.com/s/0sFQ88Qiop-ElizcxA9huA
2. https://mp.weixin.qq.com/s/EHxnzjANziuxq86XOQCElQ
3. https://mp.weixin.qq.com/s/CGAzsC4wyrR68MOhZd0aLg
4. https://mp.weixin.qq.com/s?__biz=MzUxOTAxODc2Mg==&mid=2247490338&idx=1&sn=e653a4bc5f78b9fcc99b0188cec4d46c
