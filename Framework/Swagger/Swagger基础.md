# 常用注解

| 注解                 | 作用            | 属性                   | 示例                                                          |
|--------------------|---------------|----------------------|-------------------------------------------------------------|
| @Api               | 介绍类主要作用       | value：名称<br/>tags：简介 | `@Api(value = "接口名称", tags = "接口简介")`                       |
| @ApiOperation      | 介绍方法作用        | value：名称             | `@ApiOperation("课程查询接口")`                                   |
| @ApiParam          | 单个参数描述        |                      |                                                             |
| @ApiModel          |               |                      |                                                             |
| @ApiModelProperty  | 介绍接口参数实体类属性信息 | value：名称             | `@ApiModelProperty("当前页码")`<br/>`private Long pageNo = 1L;` |
| @ApiResponse       |               |                      |                                                             |
| @ApiResponses      |               |                      |                                                             |
| @ApiImplicitParam  |               |                      |                                                             |
| @ApiImplicitParams |               |                      |                                                             |
|                    |               |                      |                                                             |
| @ApiError          |               |                      |                                                             |
| @ApiIgnore         |               |                      |                                                             |

# SpringBoot集成

1.添加依赖

```xml
<!-- Spring Boot 集成 swagger -->
<dependency>
    <groupId>com.spring4all</groupId>
    <artifactId>swagger-spring-boot-starter</artifactId>
    <version>1.9.0.RELEASE</version>
</dependency>
```

2.添加配置

```yaml
# swagger
swagger:
  title: "学习内容管理系统"
  description: "内容系统管理系统对课程相关信息进行管理"
  base-package: com.stackyu.study.content
  enabled: true
  version: 1.0.0
```

3.访问

http://localhost:63040/content/swagger-ui.html