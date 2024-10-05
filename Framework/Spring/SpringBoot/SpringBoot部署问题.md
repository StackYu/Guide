# nested exception is java.lang.NoClassDefFoundError: javax/xml/bind/DatatypeConverter] with root cause

> 运行SpringBoot的jar包报错，Jdk版本11。因为高版本缺少jaxb-api包

解决：引入依赖
```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <!--  版本根据项目确定  -->
    <version>XXX</version>
</dependency>
```