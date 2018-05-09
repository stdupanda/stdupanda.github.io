---
layout: post
title: SpringBoot 系列 12 Spring Boot 集成 swagger
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, mybatis
---

集成 swagger 方便生成 API 文档。

# 集成步骤

- `pom.xml`
```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

- `main` 入口 和 配置类

```java
@SpringBootApplication
@EnableSwagger2
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@Configuration
public class SwaggerConfig {

    private static final Logger log = LoggerFactory.getLogger(SwaggerConfig.class);

    @Value("${swagger.enable:true}")
    private boolean isSwaggerEnabled;

    @Bean
    public Docket docket() {

        Predicate<String> paths;
        paths = PathSelectors.none();

        log.debug("swagger.enable: {}", isSwaggerEnabled);
        if (isSwaggerEnabled) {
            log.debug("swagger is enabled, will generate api docs.");
            paths = PathSelectors.any();
        } else {
            log.debug("swagger 未启用");
        }

        return new Docket(DocumentationType.SWAGGER_2).enable(isSwaggerEnabled).apiInfo(genApiInfo()).select()
                .apis(RequestHandlerSelectors.basePackage("cn.xz")).paths(paths).build();
    }

    private ApiInfo genApiInfo() {
        String title = "项目 API 文档";
        String description = "描述接口定义";
        String version = "1.0";
        // String termsOfServiceUrl = "market@cecpay.com";
        // String license = "Copyright © All Rights Reserved";
        // String licenseUrl = "http://www.cecpay.com/";
        return new ApiInfoBuilder().title(title).description(description).version(version)
                // .termsOfServiceUrl(termsOfServiceUrl).license(license).licenseUrl(licenseUrl)
                .build();
    }
}
```

- Controller 使用方式

```java
package cn.xz.ctrl;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import cn.xz.dto.UserInfo;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;

@Api(value = "/user", tags = "用户接口")
@RestController
@RequestMapping("/user")
public class UserCtrl {

    private static final Logger log = LoggerFactory.getLogger(UserCtrl.class);

    @ApiOperation(value = "测试用户信息", notes = "测试接口", response = UserInfo.class)
    @ApiImplicitParam(name = "user", value = "User请求信息", required = true ,dataType = "UserInfo")
    @PostMapping("/hello")
    public UserInfo index(@ApiParam(value = "The id of the city" ,required=true )@RequestBody UserInfo user) {
        log.debug("{}", System.nanoTime());
        return new UserInfo();
    }
}
```

## 详细使用介绍

```
*   @Api：用在类上，说明该类的作用
*   @ApiOperation：用在方法上，说明方法的作用
*   @ApiImplicitParams：用在方法上包含一组参数说明
*   @ApiImplicitParam：用在@ApiImplicitParams注解中，指定一个请求参数的各个方面

    *   paramType：参数放在哪个地方

        *   header --> 请求参数的获取：@RequestHeader
        *   query -->请求参数的获取：@RequestParam
        *   path（用于restful接口）--> 请求参数的获取：@PathVariable
        *   body（不常用）
        *   form（不常用）

        *   name：参数名
    *   dataType：参数类型
    *   required：参数是否必须传
    *   value：参数的意思
    *   defaultValue：参数的默认值

*   @ApiResponses：用于表示一组响应
*   @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息

    *   code：数字，例如400
    *   message：信息，例如"请求参数没填好"
    *   response：抛出异常的类

*   @ApiModel：描述一个Model的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）

    *   @ApiModelProperty：描述一个model的属性
```

# 生产环境配置开关

```java
// 参考上述的 SwaggerConfig 内部使用示例。
@Value("${swagger.enable:true}")
private boolean isSwaggerEnabled;

if (isSwaggerEnabled) {
    log.debug("swagger is enabled, will generate api docs.");
    paths = PathSelectors.any();
} else {
    log.debug("swagger 未启用");
}
```

# 关于配合 lombok 的问题

假如使用 `lombok` 的时候，若直接使用 eclipse 等 IDE 的启动方式，可能会存在 请求和相应 `Model` 的字段信息列出失败，原因应该是开发编译阶段生产的 class 并未包括 get、set 方法，打 jar 包后即可正常显示。

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")
