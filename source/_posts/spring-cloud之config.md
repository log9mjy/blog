---
title: spring-cloud之config
date: 2019-05-18 16:28:01
tags: 
categories: springcloud

---

##  简介

Spring Cloud Config为分布式系统中的外部配置提供服务器和客户端支持。使用Config Server，您可以为所有环境中的应用程序管理其外部属性。它非常适合spring应用，也可以使用在其他语言的应用上。随着应用程序通过从开发到测试和生产的部署流程，您可以管理这些环境之间的配置，并确定应用程序具有迁移时需要运行的一切。服务器存储后端的默认实现使用git，因此它轻松支持标签版本的配置环境，以及可以访问用于管理内容的各种工具。

## config-server 

### 加入依赖

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 启动类上

```java
@EnableDiscoveryClient
@EnableConfigServer
@SpringBootApplication
public class MjyConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(MjyConfigApplication.class, args);
    }

}
```

### 配置

可以使git的文件也可以是本地文件

```Java
server.port=8888

spring.application.name=mjy-config
spring.cloud.config.server.git.uri=https://gitee.com/log9mjy/spring-cloud-config.git
spring.cloud.config.server.git.search-paths=test-config
spring.cloud.config.server.git.username=log9mjy@gmail.com
spring.cloud.config.server.git.password=MJY777489mjy

eureka.client.service-url.defaultZone=http://mjy:mjy@localhost:5761/eureka/

# 暴露actuator的监控断点
management.endpoints.web.exposure.include=*
```

为本地classpath中的文件

```
#native会默认去resource中去查找
spring.profiles.active=native 
# 配置中心
spring.cloud.config.server.native.search-locations =classpath:/config/
```

其中mjy:mjy为eureka注册中心的安全验证用户名和密码

spring-cloud-config会自动将配置文件根据名字生成restful链接

规则:lable为分支名称

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

## config-client

### 导入依赖

```Java
<!--config客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

### 配置

```Java
spring.cloud.config.name=order
spring.cloud.config.profile=dev
spring.cloud.config.uri=http://localhost:8888/
spring.cloud.config.label=master
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=mjy-config

eureka.client.service-url.defaultZone=http://mjy:mjy@localhost:5761/eureka/
# 允许相同key情况下beanDefinition实例的覆盖
spring.main.allow-bean-definition-overriding=true
```

在bootstrap.properties中配置,必须先加载该文件 .然后会去config-server中去查找对应的配置文件.

另外:在依赖中有start的依赖,一般不需要自己去配置了.

## 刷新

当我们改变配置文件时.希望项目中可以即使更新

在controller上加上注解

@RefreshScope

手动刷新

http://localhost:5763/actuator/refresh

2.0之前是http://localhost:5763/refresh

就会得到改变后的配置文件