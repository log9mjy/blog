---
title: spring-cloud之链路追踪
date: 2019-04-01 12:12:28
tags: spring cloud
---

#### zipkin 服务端

```java
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-autoconfigure-ui</artifactId>
    <version>${zipkin.version}</version>
</dependency>
<dependency>
    <groupId>io.zipkin.java</groupId>
    <artifactId>zipkin-server</artifactId>
    <version>${zipkin.version}</version>
</dependency>
```

```java 
@SpringCloudApplication
@EnableZipkinServer
public class MjyZipkinApplication {

    public static void main(String[] args) {
        SpringApplication.run(MjyZipkinApplication.class, args);
    }

}
```

```java
spring.application.name=mjy-zipkin
spring.profiles.active=dev
server.port=5765


eureka.client.service-url.defaultZone=http://mjy:mjy@localhost:5761/eureka/

# 暴露actuator的监控断点
management.endpoints.web.exposure.include=*

# 允许相同key情况下beanDefinition实例的覆盖
spring.main.allow-bean-definition-overriding=true

#关闭不然会报错
management.metrics.enable.http=false
# logging.level.org.springframework= DEBUG
```

#### zipkin 客户端

```java
<!--服务链路追踪-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```java
spring.zipkin.base-url=http://mjy-zipkin
spring.zipkin.enabled=true
spring.sleuth.web.client.enabled= true
# 默认的采样比率为0.1，不能看到所有请求数据
# 更改采样比率为1，就能看到所有的请求数据了，但是这样会增加接口调用延迟
spring.sleuth.sampler.probability =1.0
```

访问地址 http://localhost:5765/zpikin