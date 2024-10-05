# 1. 什么是注解

Java注解，也称为元数据。是1.5之后引入的新特性，是一种代码级别的说明。

# 2. 元数据

对数据说明的数据，例如47，单看只是到时一个数值，但是加上元，万元就不一样了。

> 注解本身并没有什么作用，只是一种标记或说明，需要通过反射的配合来实现相应功能。

# 3. 注解格式

> 注解只有属性（成员变量），是以无参方法的形式声明。方法名为属性名，返回值为属性值。

```java
// 创建
// @元注解
// public @interface 注解名称{
//     public 数据类型 属性名称() default 默认值; 
// }

// 使用
// @注解名称
// public class TestClass{
//     ...
// }
```

# 4. 注解使用分类

1. 无参注解（标记注解）

语法：@注解名称

不包括参数，所以直接使用，不需要添加括号

2. 单参注解

语法：@注解名称(value="***")或@注解名称("***")

单参数注解默认属性名称为value，在赋值时不需要显示属性名称。

3. 多参注解

语法：@注解名称(value="***",属性名称="***",...)

多属性需要声明名称，中间用逗号隔开。

# 5. 元注解

@Retention ：修饰注解的生命周期

- RetentionPolicy.SOURCE：注解只保留在源文件，当Java文件编译成class文件的时候，注解被遗弃；也就是编译时有效。
- RetentionPolicy.CLASS：注解被保留到class文件，但jvm加载class文件时候被遗弃，这是默认的生命周期；加载时被抛弃。
- RetentionPolicy.RUNTIME：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在；一直有效！

@Documented ：标记注解，只能用于注解中。修饰注解是否会在JavaDoc中显示

@Target ：参数注解，只能用于注解中。修饰注解使用范围。

- @Target(ElementType.TYPE) —— 接口、类、枚举、注解
- @Target(ElementType.FIELD) —— 字段、枚举的常量
- @Target(ElementType.METHOD) —— 方法
- @Target(ElementType.PARAMETER) —— 方法参数
- @Target(ElementType.CONSTRUCTOR) —— 构造函数
- @Target(ElementType.LOCAL_VARIABLE) —— 局部变量
- @Target(ElementType.ANNOTATION_TYPE) —— 注解
- @Target(ElementType.PACKAGE) —— 包

@Inherited ：修饰注解，被注解修饰类的子类会继承此注解。

@Repeatable ：修饰注解，表示该注解可以重复使用。

# 6. 自定义注解

```java
@Documented
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})  //可以在字段、枚举的常量、方法
@Retention(RetentionPolicy.RUNTIME)
public @interface Init {
    String value() default "";
}
```