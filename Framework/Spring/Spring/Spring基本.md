# 1. 概述

> Spring是整合框架、简化开发的框架，主要的技术为IOC、DI、AOP和事务等。可以理解为它是一个管家，管理者项目中的对象，集精华框架不重复造轮子。

框架架构图：

![](../../img/spring0.png)

# 2. 简单入门

## 2.1 IOC

> IOC称之为控制反转，大致的意思是将项目中有自己来维护的对象创建由Spring进行来创建管理，自己维护时是用什么创建什么，现在是需要什么就直接从容器中拿什么。

示例：
1. 创建项目，添加核心依赖：
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```

2. 创建需要管理类：
```java
public interface UserDao {
    void save();
}

public class UserDaoImpl implements UserDao {
    public void save() {
        System.out.println("UserDao save run");
    }
}

public interface UserService {
    void add();
}

public class UserServiceImpl implements UserService {

    private UserDao userDao = new UserDaoImpl();

    public void add() {
        userDao.save();
        System.out.println("UserService add run...");
    }
}
```

3. 创建配置文件：

application.xml
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl"/>

</beans>
```

4. 使用容器获得类：
```java
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
    }
}
```

## 2.2 DI
> 类之间可能存在依赖，DI技术会把需要的依赖一并纳入容器管理。

1. 修改`UserServiceImpl`类
```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void add() {
        userDao.save();
        System.out.println("UserService add run...");
    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

2. 配置依赖关系
```xml
<beans>

    <!--配置需要管理的类
        id：唯一标识
        class：类的全路径名
    -->
    <bean id="userDao" class="cn.fishland.dao.impl.UserDaoImpl"/>

    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl">
        <!--设置属性
            name：属性名称
            ref：关联的类（其他配置类id）
        -->
        <property name="userDao" ref="userDao"/>
    </bean>

</beans>
```

3. 调用
```java
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
    }
}
```

# 3. IOC和Bean

> 在Spring中每个类对象被称为Bean，Bean通过一系列的属性配置，容器可以通过一些列配置信息进行管理。

```xml
<beans>
    <bean .../>
    <bean .../>
    ...
</beans>
```
## 3.1 基础属性

### 3.1.1 id

> bean的唯一标识，主要用于bean查找。
```xml
<beans>
    <bean id="userDao" .../>
    <bean id="userService" .../>
    ...
</beans>
```

### 3.1.2 class

> bean需要实例化类的全路径名

```xml
<beans>
    <bean id="userDao" class="cn.fishland.dao.impl.UserDaoImpl"/>
    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl" />
    ...
</beans>
```

### 3.1.3 name

> 为bean起的别名，引用bean也可以通过name中名称完成，多个name名称可以通过逗号（,）分号（;）和空格来实现

```xml
<beans>
    <bean id="userDao" class="cn.fishland.dao.impl.UserDaoImpl" name="dao ud"/>
    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl" name="service us">
        <!--可以引用别名-->
        <property name="userDao" ref="dao"/>
    </bean>
    ...
</beans>
```

### 3.1.4 scope

> bean的作用范围，表示bean是单例还是多例。可选值为singleton和prototype。bean默认是单例的，所以可能出现线程安全问题。所以容器中管理的对象一般
> 为无状态类，此类不存储数据。例如表现层，服务层，数据层，工具类。

```xml
<beans>
    <!--单例对象-->
    <bean id="userDao" class="cn.fishland.dao.impl.UserDaoImpl" name="dao ud" scope="singleton"/>
    <!--多例对象-->
    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl" name="service us" scope="prototype">
        <!--可以引用别名-->
        <property name="userDao" ref="dao"/>
    </bean>
    ...
</beans>
```

## 3.2 bean实例化

> 容器通过配置就可以实例化类，它是通过构造方法进行类的创建，也可以通过bean配置来改变示例化方式。总结有三种方式来实例化构造方法、静态工厂和实例工厂。

### 3.2.1 构造方法

1. 准备的类
```java
public class UserDaoImpl implements UserDao {
    // 构造函数权限为private，Spring还是可以实力化对象的
    public UserDaoImpl() {
        System.out.println("UserDaoImpl init...");
    }

    public void save() {
        System.out.println("UserDao save run");
    }
}
```

2. 配置bean

`<bean id="userDao" class="cn.fishland.dao.impl.UserDaoImpl"/>`

3. 测试
```java
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserDao userDao = (UserDao) context.getBean("userDao");
        userDao.save();
    }
}
```

**_通过以上控制台显示可以看出是调用构造函数，但是当我们把构造函数权限变成私private有也是可以实例化对象的。_**
这是因为Spring是通过反射来实现初始化的。但是类必须有无参构造函数，否则无法实例化。

### 3.2.2 静态工厂

> 用工厂类来实现类的实例化

```java
public class UserDaoFactory {
    public static UserDao userDao() {
        // 实例化前一些必须操作
        System.out.println("init UserDao pre work...");
        return new UserDaoImpl();
    }
}
```

配置：
```xml
<bean id="userDao" class="cn.fishland.tool.UserDaoFactory" factory-method="userDao"/>
```

### 3.2.3 实例化工厂

> 通过工厂实例化bean与静态工厂不同的是需要先实例化工厂才可以使用。

1. 实例化工厂实现

类：
```java
public interface BookDao {
    void show();
}

public class BookDaoImpl implements BookDao {

    public BookDaoImpl() {
        System.out.println("BookDao init...");
    }

    public void show() {
        System.out.println("BookDao run...");
    }
}
```
工厂类：
```java
public class BookDaoFactory {
    private BookDao bookDao() {
        return new BookDaoImpl();
    }
}
```

配置：
```xml
<beans>
    <bean id="bookDaoFactory" class="cn.fishland.tool.BookDaoFactory"/>
    <bean id="bookDao" class="cn.fishland.dao.impl.BookDaoImpl" factory-bean="bookDaoFactory" factory-method="bookDao"/>
</beans>
```

测试：
```java
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        BookDao bookDao = (BookDao) context.getBean("bookDao");
        bookDao.show();
    }
}
```

2. 实现FactoryBean创建实例化工厂

创建工程类：
```java
public class BookDaoFactoryBean implements FactoryBean<BookDao> {
    /** 获得实例化对象 */
    public BookDao getObject() throws Exception {
        return new BookDaoImpl();
    }

    /** 实例化对象类型 */
    public Class<?> getObjectType() {
        return null;
    }

    /** 是否为单例 */
    public boolean isSingleton() {
        return true;
    }
}
```

配置文件：
```xml
<!--如此简单-->
<bean id="bookDaoBean" class="cn.fishland.tool.BookDaoFactoryBean"/>
```


## 3.3 bean生命周期

> 生命周期即bean从创建到销毁的过程，在整个过程都干了什么事（执行了哪些方法以便于我们自己实现来进行修改）。

1. 简单实现

类：
```java
public interface MenuDao {
    void save();
}

public class MenuDaoImpl implements MenuDao {
    public MenuDaoImpl() {
        System.out.println("MenuDaoImpl construct...");
    }

    public void save() {
        System.out.println("MenuDaoImpl save run...");
    }

    public void init() {
        System.out.println("MenuDaoImpl init...");
    }

    public void destory() {
        System.out.println("MenuDaoImpl destory...");
    }
}
```

配置：`<bean id="menuDao" class="cn.fishland.dao.impl.MenuDaoImpl" init-method="init" destroy-method="destory"/>`

测试：
```java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        MenuDao menuDao = (MenuDao) context.getBean("menuDao");
        menuDao.save();
        // 关闭容器
        context.close();
    }
}
```

> 执行顺序构造、init、destory

2. 通过实现接口简化配置

> 实现接口InitializingBean完成afterPropertiesSet方法，为bean的init-method。实现接口DisposableBean完成destroy方法，为bean的destroy-method

类：
```java
public class MenuDaoImpl implements MenuDao, InitializingBean, DisposableBean {
    public MenuDaoImpl() {
        System.out.println("MenuDaoImpl construct...");
    }

    public void save() {
        System.out.println("MenuDaoImpl save run...");
    }

    /** destroy方法 */
    public void destroy() throws Exception {
        System.out.println("MenuDaoImpl destroy...");
    }

    /** init方法 */
    public void afterPropertiesSet() throws Exception {
        System.out.println("MenuDaoImpl afterPropertiesSet...");
    }
}
```

配置：

`<bean id="menuDao" class="cn.fishland.dao.impl.MenuDaoImpl"/>`

测试：
```java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        MenuDao menuDao = (MenuDao) context.getBean("menuDao");
        menuDao.save();
        // 关闭容器
        context.close();
    }
}
```

**总结：** bean的生命周期
1. 创建对象
2. 调用构造
3. 执行setter
4. 执行bean的init
5. 使用bean执行业务方法
6. 关闭销毁执行bean销毁方法

# 4. DI

> DI称为依赖注入，就是通过配置将类属性进行设置。依赖注入分为两种setter和构造方法，注入的内容为引用类型和基础数据类型。

## 4.1 setter

### 4.2.1 注入引用数据类型
类：
```java
public interface UserDao {
    void save();
}

public class UserDaoImpl implements UserDao {
    public UserDaoImpl() {
        System.out.println("UserDaoImpl init...");
    }

    public void save() {
        System.out.println("UserDao save run");
    }
}

public interface UserService {
    void add();
}

public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void add() {
        userDao.save();
        System.out.println("UserService add run...");
    }

    // 需要添加属性set方法
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

配置信息：
```xml
<beans>
    <bean id="userDao" class="cn.fishland.tool.UserDaoFactory" factory-method="userDao"/>

    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl">
        <!--设置属性
            name：属性名称
            ref：关联的类（其他配置类id）
        -->
        <property name="userDao" ref="userDao"/>
    </bean>
</beans>
```

测试：
```java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.add();
        // 关闭容器
        context.close();
    }
}
```

### 4.2.1 注入基础数据类型

类：
```java
public interface TagDao {
    void show();
}

public class TagDaoImpl implements TagDao {
    private String name;
    private String flag;

    public void show() {
        System.out.println("TagDaoImpl show... name=" + name + " flag=" + flag);
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setFlag(String flag) {
        this.flag = flag;
    }
}
```

配置：
```xml
<bean id="tagDao" class="cn.fishland.dao.impl.TagDaoImpl">
    <!--
        name：需要注入的属性名称
        value：注入的值，spring会自动类型转换，需要确保在转换中不会出错。
    -->
    <property name="name" value="fish"/>
    <property name="flag" value="1"/>
</bean>
```

测试：
```java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        TagDao tagDao = (TagDao) context.getBean("tagDao");
        tagDao.show();
        // 关闭容器
        context.close();
    }
}
```

## 4.2 构造注入

### 4.2.1 引用类型注入

类：
```java
public interface TagService {
    void add();
}

public class TagServiceImpl implements TagService {

    private TagDao tagDao;

    public TagServiceImpl(TagDao tagDao) {
        this.tagDao = tagDao;
    }

    public void add() {
        tagDao.show();
        System.out.println("TagServiceImpl add...");
    }
}
```

配置：
```xml
<bean>
    <bean id="tagDao" class="cn.fishland.dao.impl.TagDaoImpl">
        <!--
            name：需要注入的属性名称
            value：注入的值，spring会自动类型转换，需要确保在转换中不会出错。
        -->
        <property name="name" value="fish"/>
        <property name="flag" value="1"/>
    </bean>

    <bean id="tagService" class="cn.fishland.service.impl.TagServiceImpl">
        <constructor-arg name="tagDao" ref="tagDao"/>
    </bean>
</bean>
```

测试：
```java
public class Application {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        TagService tagService = (TagService) context.getBean("tagService");
        tagService.add();
        // 关闭容器
        context.close();
    }
}
```

### 4.2.2 基础数据类型

```java
public class TagServiceImpl implements TagService {

    private TagDao tagDao;
    private String name;
    private Integer flag;

    public TagServiceImpl(TagDao tagDao) {
        this.tagDao = tagDao;
    }

    public TagServiceImpl(TagDao tagDao, String name, Integer flag) {
        this.tagDao = tagDao;
        this.name = name;
        this.flag = flag;
    }

    public void add() {
        tagDao.show();
        System.out.println("TagServiceImpl add...");
        System.out.println("TagServiceImpl name=" + name);
        System.out.println("TagServiceImpl flag=" + flag);
    }
}
```

配置：方式一
```xml
<beans>
    <bean id="tagDao" class="cn.fishland.dao.impl.TagDaoImpl">
        <!--
            name：需要注入的属性名称
            value：注入的值，spring会自动类型转换，需要确保在转换中不会出错。
        -->
        <property name="name" value="fish"/>
        <property name="flag" value="1"/>
    </bean>

    <bean id="tagService" class="cn.fishland.service.impl.TagServiceImpl">
        <!--
            name:为形参的名称
        -->
        <constructor-arg name="tagDao" ref="tagDao"/>
        <constructor-arg name="name" value="fish"/>
        <constructor-arg name="flag" value="1"/>
    </bean>
</beans>
```

配置：方式二，主要输解决构造方法形参名称改变导致的一些列问题
```xml
<beans>
    <bean id="tagDao" class="cn.fishland.dao.impl.TagDaoImpl">
        <!--
            name：需要注入的属性名称
            value：注入的值，spring会自动类型转换，需要确保在转换中不会出错。
        -->
        <property name="name" value="fish"/>
        <property name="flag" value="1"/>
    </bean>

    <bean id="tagService" class="cn.fishland.service.impl.TagServiceImpl">
        <!--
            type：为形参的数据类型，当存在相同数据类型就会出现问题
        -->
        <constructor-arg type="cn.fishland.dao.TagDao" ref="tagDao"/>
        <constructor-arg type="java.lang.String" value="fish"/>
        <constructor-arg type="java.lang.Integer" value="1"/>
    </bean>
</beans>
```

配置：方式三，解耦又可以避免相同类型问题
```xml
<beans>
    <bean id="tagDao" class="cn.fishland.dao.impl.TagDaoImpl">
        <!--
            name：需要注入的属性名称
            value：注入的值，spring会自动类型转换，需要确保在转换中不会出错。
        -->
        <property name="name" value="fish"/>
        <property name="flag" value="1"/>
    </bean>

    <!--
        index：形参的位置，从0开始
    -->
    <bean id="tagService" class="cn.fishland.service.impl.TagServiceImpl">
        <constructor-arg index="0" ref="tagDao"/>
        <constructor-arg index="1" value="fish"/>
        <constructor-arg index="2" value="1"/>
    </bean>
</beans>
```
## 4.3 自动装配

> 前面两种方式都较麻烦，需要大量的配置文件，Spring还提供了自动装配设置，更为简单。IOC容器会根据一些条件自动在容器中寻找符合条件的依赖bean。
> 装配的条件为按照类型、按照名称。

1. 按类型注入

**自动装配必须条件：**
1. 被注入类必须在容器管理中
2. 注入属性需要有setter方法
3. 容器中存在多个相同类型按照类型注入就会失败

```java
public class UserDaoImpl implements UserDao {
    public UserDaoImpl() {
        System.out.println("UserDaoImpl init...");
    }

    public void save() {
        System.out.println("UserDao save run");
    }
}

public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void add() {
        userDao.save();
        System.out.println("UserService add run...");
    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

```xml
<beans>
    <bean class="cn.fishland.tool.UserDaoFactory" factory-method="userDao"/>
    <!--
        autowire；开启自动装配，byType表示根据类型装配
    -->
    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl" autowire="byType" />
</beans>
```


2. 按名称注入

> 存在一个问题，当容器存在多个相同类型，就会导致装配失败，这时可以使用根据名称装配。按照名称注入如果容器找不到对应的类就会注入null

名称：指的是注入setter方法去掉set后首字母小写名称，按照默认set规则，最后的结果就是属性名称。

```xml
<beans>
    <bean id="userDao" class="cn.fishland.tool.UserDaoFactory" factory-method="userDao"/>
    <!--
        byName：开启根据名称注入
    -->
    <bean id="userService" class="cn.fishland.service.impl.UserServiceImpl" autowire="byName" />
</beans>
```

_**总结：**_
1. 自动装配适用于引用类型，不适用基础数据类型
2. 按照类型装配需要确保容器中只有一个类型的bean
3. 按照名称注入耦合较大
4. 自动装配优先级低于setter和构造方法配置注入，同时存在自动装配失效

## 4.4 集合注入

```java
public class CollectionDaoImpl implements CollectionDao {

    private int[] array;
    private List<String> list;
    private Set<String> set;
    private Map<String, String> map;
    private Properties properties;

    public void showData() {
        System.out.println("CollectionDaoImpl showData...");
        System.out.println("CollectionDaoImpl array=" + Arrays.toString(array));
        System.out.println("CollectionDaoImpl list=" + list);
        System.out.println("CollectionDaoImpl set=" + set);
        System.out.println("CollectionDaoImpl map=" + map);
        System.out.println("CollectionDaoImpl properties=" + properties);

    }

    public void setArray(int[] array) {
        this.array = array;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void setProperties(Properties properties) {
        this.properties = properties;
    }
}
```

```xml
<bean id="collectionDao" class="cn.fishland.dao.impl.CollectionDaoImpl">
    <property name="array">
        <array>
            <value>1</value>
            <value>2</value>
            <value>3</value>
        </array>
    </property>
    <property name="list">
        <list>
            <value>itcast</value>
            <value>itheima</value>
            <value>boxuegu</value>
            <value>chuanzhihui</value>
        </list>
    </property>
    <property name="set">
        <set>
            <value>itcast</value>
            <value>itheima</value>
            <value>boxuegu</value>
            <value>boxuegu</value>
        </set>
    </property>
    <property name="map">
        <map>
            <entry key="country" value="china"/>
            <entry key="province" value="henan"/>
            <entry key="city" value="kaifeng"/>
        </map>
    </property>
    <property name="properties">
        <props>
            <prop key="country">china</prop>
            <prop key="province">henan</prop>
            <prop key="city">kaifeng</prop>
        </props>
    </property>
</bean>
```

# 5. 配置第三方Bean

配置Druid数据库连接池:

1. 引依赖
```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.15</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
</dependencies>
```

2. 配置
```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:33060/BookManager"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</bean>
```


3. 测试
```java
public class Application {
    public static void main(String[] args) throws SQLException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        DataSource dataSource = (DataSource) context.getBean("dataSource");
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        // 关闭容器
        context.close();
    }
}
```

配置C3p0数据库连接池:

配置：
```xml
<bean id="dataSourceC3p0" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:33060/BookManager"/>
    <property name="user" value="root"/>
    <property name="password" value="root"/>
    <property name="maxPoolSize" value="1000"/>
</bean>
```

# 6. 加载properties文件

1. 添加properties文件

jdbc.properties
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:33060/BookManager
jdbc.username=root
jdbc.password=root
```

2. 添加context命名空间
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            <!--新增-->
            http://www.springframework.org/schema/context
            <!--新增-->
            http://www.springframework.org/schema/context/spring-context.xsd"
        <!--新增-->
       xmlns:context="http://www.springframework.org/schema/context">
    
</beans>
```

3. 配置添加properties文件
```xml
<beans>
    <!--加载本地文件-->
    <context:property-placeholder location="jdbc.properties"/>
</beans>
```

4. 引用properties属性
```xml
<!--使用${}来获得内容-->
<bean id="dataSourceC3p0" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="maxPoolSize" value="1000"/>
</bean>
```

5. 测试
```java
public class Application {
    public static void main(String[] args) throws SQLException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        DataSource dataSource = (DataSource) context.getBean("dataSourceC3p0");
        System.out.println(dataSource);
        // 关闭容器
        context.close();
    }
}
```

_**注意：**_
1. 本地的username与环境变量冲突

在加载jdbc.properties文件的时候，也会加载系统环境变量。本地与系统发生冲突后，系统优先。

```java
public class Application {
    public static void main(String[] args) throws SQLException {
        /*此方法可以查看系统环境变量*/
        Map<String, String> env = System.getenv();
        System.out.println(env);
    }
}
```

可以配置`system-properties-mode="NEVER"`表示不加载系统环境变量

2. 加载多个文件

```xml
<beans>
    <!--
        在location中配置多个文件名，中间用逗号隔开
    -->
    <context:property-placeholder location="jdbc.properties,jdbc2.properties" system-properties-mode="NEVER"/>
    <!--
        *.properties：表示所有以properties结尾的文件都会被加载
    -->
    <context:property-placeholder location="*.properties" system-properties-mode="NEVER"/>
    <!--
        classpath:*.properties：项目根目录下所有properties文件都会被加载
    -->
    <context:property-placeholder location="classpath:*.properties" system-properties-mode="NEVER"/>
    <!--
        classpath*:*.properties：项目加项目依赖所有的properties文件
    -->
    <context:property-placeholder location="classpath*:*.properties" system-properties-mode="NEVER"/>
</beans>
```


# 7. 核心容器

## 7.1 容器创建

> 容器主要有两种：ClassPathXmlApplicationContext和FileSystemXmlApplicationContext

```java
public class Application {
    public static void main(String[] args) throws SQLException {
        /*
         * 类翻译过来的意思就是类路径下XML配置文件，所以参数就是类路径下xml文件
         * */
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");

        /*
         * 文件系统下的XML配置文件，配置文件在项目的文件系统下
         * */
        FileSystemXmlApplicationContext context1 = new FileSystemXmlApplicationContext("D:\\XXX\\XX\\X\\application.xml");
    }
}
```

## 7.2 Bean获得

> 获得bean有三种方式

```java
public class Application {
    public static void main(String[] args) throws SQLException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        // 通过bean的id获得
        BookDao bookDao = (BookDao) context.getBean("bookDao");
        // 通过名称获得，并且给出结果的类型，免除强制类型转换       
        BookDao bookDao = ctx.getBean("bookDao", BookDao.class);
        // 直接通过类型获得，这种方法存在一些局限性，确保容器中只有一个这种类型实例
        BookDao bookDao = ctx.getBean(BookDao.class);
    }
}
```

## 7.3 BeanFactory

创建BeanFactory：
```java
public class Application {
    public static void main(String[] args) throws SQLException {
        XmlBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("application.xml"));
        CollectionDao collectionDao = beanFactory.getBean("collectionDao", CollectionDao.class);
        collectionDao.showData();
    }
}
```

ApplicationContext和BeanFactory区别：
1. BeanFactory是懒加载，使用时才会加载
2. ApplicationContext支持懒加载，但是默认是容器创建就会加载bean

ApplicationContext实现懒加载：
```xml
<!--
    lazy-init:true开启加载，false为不开启，default默认
-->
<bean id="menuDao" class="cn.fishland.dao.impl.MenuDaoImpl" lazy-init="true"/>
```

# 8. 注解开发

## 8.1 注解定义bean

类：
```java
public interface RoleService {
    void add();
}

@Component("roleService")
public class RoleServiceImpl implements RoleService {
    public void add() {
        System.out.println("RoleServiceImpl add...");
    }
}
```

配置：
```xml
<!--
    开启注解组件扫描
    base-package：当前包及其子包的类
-->
<context:component-scan base-package="cn.fishland"/>
```

测试：
```java
public class Application {
    public static void main(String[] args) throws SQLException {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        RoleService bean = context.getBean(RoleService.class);
        bean.add();
    }
}
```

> @Component("roleService")注解表示当前类为bean，当spring扫描到类带此注解就会将类加载到容器中。@Component注解后面也可以什么都不加，这时
> 此类的名称就是类首字母小写的名称。**@Component注解不可以放在接口类上**，接口无法直接创建。

@Component衍生注解：同@Component同样效果
1. @Controller：表现层
2. @Service：服务层
3. @Repository：数据层

## 8.2 存注解开发

> 通过以上的示例，发现使用Spring还是挺麻烦，xml文件配置较为繁琐。Spring在3.0后就支持存注解开发了，去掉xml配置文件直接用类来进行配置。

配置类：
```java
// 标识当前类为配置类，取代xml配置文件
@Configuration
// 开启包扫描
@ComponentScan("cn.fishland")
public class AppConfig {
}
```

bean类：
```java
public interface RoleService {
    void add();
}

@Component("roleService")
public class RoleServiceImpl implements RoleService {
    public void add() {
        System.out.println("RoleServiceImpl add...");
    }
}
```

测试：
```java
public class Application {
    public static void main(String[] args) throws SQLException {
        // 使用AnnotationConfigApplicationContext加载配置类
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        RoleService bean = context.getBean(RoleService.class);
        bean.add();
    }
}
```

> 从此处开始Spring的使用就简洁很多

## 8.3 注解下bean作用范围和生命周期

bean类：
```java
@Component("roleService")
//@Scope("prototype") 配置bean范围注解，表示多例
@Scope("singleton") // 配置bean范围，表示单例，默认值，不配也行
public class RoleServiceImpl implements RoleService {
    public void add() {
        System.out.println("RoleServiceImpl add...");
    }

    // init注解，表示当前方法为init方法
    @PostConstruct
    public void init() {
        System.out.println("init ...");
    }

    // destroy注解，当前方法为destroy方法
    @PreDestroy
    public void destroy() {
        System.out.println("destroy ...");
    }
}
```

测试：
```java
public class Application {
    public static void main(String[] args) throws SQLException, InterruptedException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        RoleService bean1 = context.getBean(RoleService.class);
        System.out.println(bean1);
        context.close();
    }
}
```

注解和xml配置关系：

![](../../../img/spring1.png)

## 8.4 注解依赖注入

### 8.4.1 自动注入
```java
// 加入容器
@Repository
public class RoleDaoImpl implements RoleDao {
    public void insert() {
        System.out.println("RoleDaoImpl run...");
    }
}

@Component
public class RoleServiceImpl implements RoleService {

    // 根据类型自动注入
    @Autowired
    private RoleDao roleDao;

    public void add() {
        roleDao.insert();
        System.out.println("RoleServiceImpl add...");
    }

}

// 测试
public class Application {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        RoleService bean1 = context.getBean(RoleService.class);
        bean1.add();
        context.close();
    }
}
```

### 8.4.2. 按照名称注入

> 当容器中出现多个同类型实例时就需要按照名称注入

```java
@Repository
public class RoleDaoImpl implements RoleDao {
    public void insert() {
        System.out.println("RoleDaoImpl run...");
    }
}

@Repository("roleDaoImpl1")
public class RoleDaoImpl1 implements RoleDao {
    public void insert() {
        System.out.println("RoleDaoImpl1 run...");
    }
}

@Service("roleService")
public class RoleServiceImpl implements RoleService {

    @Autowired
    // 根据名称注入，不能单独使用必须配合Autowired使用
    @Qualifier("roleDaoImpl1")
    private RoleDao roleDao;

    public void add() {
        roleDao.insert();
        System.out.println("RoleServiceImpl add...");
    }
}

```

总结：
1. @Autowired根类型装配，出现多个相同类型时，就会使用属性变量名进行名称装配，再找不到就会抛异常。
2. @Qualifier不能单独使用，需要配合@Autowried使用。

### 8.4.3 简单数据类型注入

```java
@Repository
public class DataDaoImpl implements DataDao {

    @Value("fish")
    private String name;

    public void shoeData() {
        System.out.println("DataDaoImpl name=" + name);
    }
}
```

@Value配合properties使用

加载配置文件
```java
@Configuration
// 加载配置文件
@PropertySource("classpath:jdbc.properties")
@ComponentScan("cn.fishland")
public class AppConfig {
}
```

使用配置文件中数据：
```java
@Repository
public class DataDaoImpl implements DataDao {

    @Value("fish")
    private String name;

    // 使用配置文件中数据
    @Value("${jdbc.url}")
    private String url;

    public void shoeData() {
        System.out.println("DataDaoImpl name=" + name);
        System.out.println("DataDaoImpl url=" + url);
    }
}
```

> 家在多个配置文件@PropertySource({"jdbc1.properties","jdbc2.properties"})

### 8.4.4 注入第三方bean

新建配置类：
```java

// @Configuration 这个注解可以加也可以在另一个配置中使用@Import配置，示例如下
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    // 在方法上直接添加@Bean注解即可
    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}

// 主配置类
@Configuration
@PropertySource("classpath:jdbc.properties")
@ComponentScan("cn.fishland")
// 添加另一个配置类
@Import({JdbcConfig.class})
public class AppConfig {
}
```

测试：
```java
public class Application {
    public static void main(String[] args) throws SQLException, InterruptedException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        DataSource dataSource = context.getBean(DataSource.class);
        System.out.println(dataSource.getConnection());
        context.close();
    }
}
```

> 当@Bean配置的类需要注入其他bean怎么办了，Spring给出的解决办法是，直接设置形参就像，Spring会自动根据形参进行自动注入。示例如下：

```java
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    // 形参中直接设置即可，容器会自动根据类型进行注入，爽^_^
    @Bean
    public DataSource dataSource(DataDao dataDao) {
        dataDao.shoeData();
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

# 9. 第三方框架整合

## 9.1 Mybatis整合

添加依赖：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.15</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.6</version>
    </dependency>
    <dependency>
        <!--Spring操作数据库需要该jar包-->
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
        <!--Spring与Mybatis整合的jar包 这个jar包mybatis在前面，是Mybatis提供的 -->
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>1.3.0</version>
    </dependency>    
</dependencies>
```

创建数据源配置类：
```java
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

创建Mybatis配置类：
```java
public class MybatisConfig {

    // 设置mybatis主要配置
    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        // 配置别名
        sqlSessionFactoryBean.setTypeAliasesPackage("cn.fishland.pojo");
        // 配置数据源
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }

    // 配置mapper扫描包，mybatis需要加载的类
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        // 需要加载的类
        mapperScannerConfigurer.setBasePackage("cn.fishland.mapper");
        return mapperScannerConfigurer;
    }

}
```

主配置类：
```java
// 表明为配置类
@Configuration
// 扫描组件包
@ComponentScan("cn.fishland")
// 导入配置文件
@PropertySource("classpath:jdbc.properties")
// 导入其他配置类
@Import({JdbcConfig.class, MybatisConfig.class})
public class AppConfig {
}
```

测试：
```java
public class Application {
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        TagService bean = context.getBean(TagService.class);
        bean.findAll();
    }
}
```

## 9.2 整合Junit

导入依赖：
```xml
<!--需要注意版本问题-->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

创建测试类运行即可：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {AppConfig.class})
public class TagServiceTset {

    @Autowired
    private TagService tagService;

    @Test
    public void tagServiceTest() {
        tagService.findAll();
    }
}
```

# 10. 注解总结

|注解|说明|对比XML|
|---|---|---|
|@Component|标记当前类为bean，组件类，会被容器加载，衍生注解@Service，@Controller，@Repository|bean标签|
|@Configuration|当前类为配置类，代替application.xml配置文件|beans标签|
|@ComponentScan("cn.fishland")|放在配置类上，设置组件扫描包|`<context:component-scan base-package="cn.fishland"/>`|
|@Scope("prototype")|配置bean的范围，值为prototype何singleton|bean标签scope属性|
|@PostConstruct|配置加载类的init方法|bean标签init-method属性|
|@PreDestroy|配置bean加载的destroy方法|bean标签destroy-method属性|
|@Autowired|根据类型自动注入类，同类型多个采用属性名查找，未找到抛异常|bean标签autowired属性|
|@Qualifier("roleDaoImpl1")|根据名称注入bean，需要同@Autowired注解一同使用|bean标签autowired属性的byName值|
|@Value("xx")|注入普通值到属性中，也可以引用外部配置值，例如：@Value("${jdbc.url}")|`<property name="url" value="jdbc:mysql://localhost:33060/BookManager"/>`|
|@PropertySource("classpath:jdbc.properties")|加载外部properties文件，classpath表示项目根目录开始|`<context:property-placeholder location="jdbc.properties"/>`|
|@Bean|使用在方法上，会将返回值加入容器中，名称为方法名|bean标签|
|@Import({JdbcConfig.class})|用在配置类上，加载其他配置类||
|@RunWith(SpringJUnit4ClassRunner.class)|测试类注解||
|@ContextConfiguration(classes = {AppConfig.class})|测试类加载的配置类||

# 11.  断言



