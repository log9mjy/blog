---
title: spring-cloud之eureka
date: 2019-05-14 11:43:08
tags: 
categories: springcloud
---



springboot与springcloud版本对应

| Spring Boot   | Spring Cloud             |
| ------------- | ------------------------ |
| 1.2.x         | Angel版本                |
| 1.3.x         | Brixton版本              |
| 1.4.x stripes | Camden版本               |
| 1.5.x         | Dalston版本、Edgware版本 |
| 2.0.x         | Finchley版本             |
| 2.1.x         | Greenwich.SR2            |



### 1.添加依赖

```Java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.mjy</groupId>
        <artifactId>mjy-cloud</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>mjy-eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>mjy-eureka</name>
    <description>注册中心</description>


    <dependencies>
        <!--服务中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <!--security-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>

        <!--web 模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



### 2.配置

```Java

spring.application.name=mjy-eureka
server.port=5761

spring.security.user.name=mjy
spring.security.user.password=mjy



eureka.instance.hostname=localhost
#表示是否将自己注册到Eureka Server，默认为true
eureka.client.register-with-eureka=false
#不需要同步其他的Eureka Server节点的数据
eureka.client.fetch-registry=false
#查询服务和注册服务都需要依赖这个地址,最前面需要加上安全认证的用户名和密码
eureka.client.service-url.defaultZone=http://mjy:mjy@${eureka.instance.hostname}:${server.port}/eureka/
#心跳比例
eureka.server.renewal-percent-threshold=0.49
#暴露所有的监控端
management.endpoints.web.exposure.include=*



```

### 3.安全配置

```java
package com.mjy.eureka.security;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

/**
 * @Author jiangyong
 * @Description TODO
 * @Date 2019/9/12 15:13
 **/
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                //开放监控端口
                .antMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
                .and()
                //开启表单认证
                .httpBasic();
    }
}
```

### 4.开启注册中心

```Java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```

### 注册中心的集群