# SpringBoot
无需进行繁琐的XML配置，仅需要一个main方法即可进行基于Spring下的项目进行运行。

# SpringBoot自动装配
SpringBoot实现开箱即用，无需任何配置的最核心功能就是自动装配。

## 1. 什么是自动装配
提起自动装配一般都会和SpringBoot关联起来。比如创建RestFul Web项目，在不使用SpringBoot项目时，需要配置JSON和viewResolver。但是在使用
SpringBoot后，无需配置直接启动即可。

SpringBoot启动时会扫描jar包中的META-INF下的spring.factories配置文件，将重要类加载到容器中。再通过简单的配置即可使用。

## 2. 如何实现自动装配
SpringBoot核心注解@SpringBootAppliccation，只需在main方法类上添加即可。

@SpringBootApplication = @Configuration、@ComponentScan、@EnableAutoConfiguration

@EnableAutoConfiguration：启动SpringBoot的自动配置机制
@Configuration：允许在上下文中注册额外的 bean 或导入其他配置类
@ComponentScan：扫描被注解的bean，默认是扫描当前包下的所有类。（容器中将排除`TypeExcludeFilter`和`AutoConfigurationExcludeFilter`）

实现自动配置主要的注解@EnableAutoConfiguration：
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
// 加载主键
@AutoConfigurationPackage
// 加载自动装配类 xxxAutoconfiguration
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

AutoConfigurationImportSelector实现了DeferredImportSelector和ImportSelector接口：
实现ImportSelector接口selectImports方法：主要功能是获取MEAT-INF/spring.factories，
再实现getAutoConfigurationEntry方法进行自动配置，步骤如下：
1. 判断是否开启自动注解。
2. 获取注解中的`exclude` 和`excludeName`配置内容。
3. 获取所有配置在spring.factories文件中的类。
4. 过滤配置文件中需要加载的类，判断条件根据@ConditionalOnXXXX判断。
5. 加载符合条件到容器中即可。

## 3. 如何按需加载
使用@ConditionalOnXXX等注解来判断是否需要加载：
- @ConditionalOnBean：当容器里有指定 Bean 的条件下
- @ConditionalOnMissingBean：当容器里没有指定 Bean 的情况下
- @ConditionalOnSingleCandidate：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- @ConditionalOnClass：当类路径下有指定类的条件下
- @ConditionalOnMissingClass：当类路径下没有指定类的条件下
- @ConditionalOnProperty：指定的属性是否有指定的值
- @ConditionalOnResource：类路径是否有指定的值
- @ConditionalOnExpression：基于 SpEL 表达式作为判断条件
- @ConditionalOnJava：基于 Java 版本作为判断条件
- @ConditionalOnJndi：在 JNDI 存在的条件下差在指定的位置
- @ConditionalOnNotWebApplication：当前项目不是 Web 项目的条件下
- @ConditionalOnWebApplication：当前项目是 Web 项 目的条件下

## 4. 如何实现一个Start
[spring-boot-custom-starter](https://github.com/xiaoyu2017/spring-boot-custom-starter)



# 3. 字段为空返回null字符串

<img src="/Users/yujiangzhong/IdeaProjects/JavaOffer/img/SpringBoot/SpringBoot1.jpg" style="zoom:50%;" />



## 1. 在返回实体类上添加注解

`@JsonInclude(JsonInclude.Include.NON_NULL)`

> 添加到类和类字段上均可

## 2. 全局配置

`spring.jackson.default-property-inclusion=non_null`

## 3. 配置类

在继承了WebMvcConfigurationSupport的类上，进行全局配置。

```java
// JsonInclude全局配置
@JsonInclude(JsonInclude.Include.NON_NULL)
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
            .serializationInclusion(JsonInclude.Include.NON_NULL);
    converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
}
```

