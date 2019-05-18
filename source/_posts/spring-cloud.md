---
title: spring-cloud之eureka
date: 2019-05-14 11:43:08
tags: [微服务]
---

### 1.添加依赖

```Java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<groupId>com.example</groupId>
<artifactId>eureka</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>eureka</name>
<description>Demo project for Spring Boot</description>

<properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



### 2.配置

```Java
#端口号
server:
  port: 5761
eureka:
  instance:
    hostname: localhost
  client:
# eureka.client.registerWithEureka ：表示是否将自己注册到Eureka Server，默认为true。
# 由于当前这个应用就是Eureka Server，故而设为false
    register-with-eureka: false
# eureka.client.fetchRegistry ：表示是否从Eureka Server获取注册信息，默认为true。因为这是一个单点的Eureka Server，
# 不需要同步其他的Eureka Server节点的数据，故而设为false。
    fetch-registry: false
    service-url:
# eureka.client.serviceUrl.defaultZone ：设置与Eureka Server交互的地址，
#查询服务和注册服务都需要依赖这个地址。默认是
      defaultZone: http://${eureka.instance.hostname}:5761/eureka/

  server:
    renewal-percent-threshold: 0.49
```

### 3.开启

```Java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```