---
title: spring-cloud之gateway使用
date: 2019-05-21 12:05:31
tags:网关
---

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

## 一 . 相关概念:

- Route（路由）：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。
- Predicate（断言）：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。
- Filter（过滤器）：这是`org.springframework.cloud.gateway.filter.GatewayFilter`的实例，我们可以使用它修改请求和响应。

## 二  .案例

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

 

```Java
@SpringBootApplication
@EnableDiscoveryClient
public class MjyGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(MjyGatewayApplication.class, args);
    }

}
```

```Java
spring.application.name=mjy-gateway

server.port =9999

# 设为 true 便开启通过服务中心的自动根据 serviceId 创建路由的功能。
# spring.cloud.gateway.discovery.locator.enabled= true

# service-id 设置为小写ID
# spring.cloud.gateway.discovery.locator.lower-case-service-id=true

spring.cloud.gateway.routes[0].id=order_router
spring.cloud.gateway.routes[0].uri=lb://mjy-order  #mjy-order为注册的服务名，多个实现负载均衡
spring.cloud.gateway.routes[0].predicates[0]=Path=/order/**

spring.cloud.gateway.routes[1].id=product_router
spring.cloud.gateway.routes[1].uri=lb://mjy-product
spring.cloud.gateway.routes[1].predicates[0]=Path=/product/**
```

- id：我们自定义的路由 ID，保持唯一
- uri：目标服务地址
- predicates：路由条件，Predicate 接受一个输入参数，返回一个布尔值结果。该接口包含多种默认方法来将 Predicate 组合成其他复杂的逻辑（比如：与，或，非）。
- filters：过滤规则，本示例暂时没用。

也可以在代码当中加入

```Java
@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
				.route("path_route", r -> r.path("/about")
						.uri("http://ityouknow.com"))
				.build();
	}
```

## 三. 路由规则

### 通过时间匹配

Predicate 支持设置一个时间，在请求进行转发的时候，可以通过判断在这个时间之前或者之后进行转发。比如我们现在设置只有在2019年1月1日才会转发到我的网站，在这之前不进行转发，我就可以这样配置：

```
spring:
  cloud:
    gateway:
      routes:
       - id: time_route
        uri: http://ityouknow.com
        predicates:
         - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

Spring 是通过 ZonedDateTime 来对时间进行的对比，ZonedDateTime 是 Java 8 中日期时间功能里，用于表示带时区的日期与时间信息的类，ZonedDateTime 支持通过时区来设置时间，中国的时区是：`Asia/Shanghai`。

After Route Predicate 是指在这个时间之后的请求都转发到目标地址。上面的示例是指，请求时间在 2018年1月20日6点6分6秒之后的所有请求都转发到地址`http://ityouknow.com`。`+08:00`是指时间和UTC时间相差八个小时，时间地区为`Asia/Shanghai`。

添加完路由规则之后，访问地址`http://localhost:8080`会自动转发到`http://ityouknow.com`。

Before Route Predicate 刚好相反，在某个时间之前的请求的请求都进行转发。我们把上面路由规则中的 After 改为 Before，如下：

```
spring:
  cloud:
    gateway:
      routes:
       - id: after_route
        uri: http://ityouknow.com
        predicates:
         - Before=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

就表示在这个时间之前可以进行路由，在这时间之后停止路由，修改完之后重启项目再次访问地址`http://localhost:8080`，页面会报 404 没有找到地址。

除过在时间之前或者之后外，Gateway 还支持限制路由请求在某一个时间段范围内，可以使用 Between Route Predicate 来实现。

```
spring:
  cloud:
    gateway:
      routes:
       - id: after_route
        uri: http://ityouknow.com
        predicates:
         - Between=2018-01-20T06:06:06+08:00[Asia/Shanghai], 2019-01-20T06:06:06+08:00[Asia/Shanghai]
```

这样设置就意味着在这个时间段内可以匹配到此路由，超过这个时间段范围则不会进行匹配。通过时间匹配路由的功能很酷，可以用在限时抢购的一些场景中。

### 通过 Cookie 匹配

Cookie Route Predicate 可以接收两个参数，一个是 Cookie name ,一个是正则表达式，路由规则会通过获取对应的 Cookie name 值和正则表达式去匹配，如果匹配上就会执行路由，如果没有匹配上则不执行。

```
spring:
  cloud:
    gateway:
      routes:
       - id: cookie_route
         uri: http://ityouknow.com
         predicates:
         - Cookie=ityouknow, kee.e
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080 --cookie "ityouknow=kee.e"
```

则会返回页面代码，如果去掉`--cookie "ityouknow=kee.e"`，后台汇报 404 错误。

### 通过 Header 属性匹配

Header Route Predicate 和 Cookie Route Predicate 一样，也是接收 2 个参数，一个 header 中属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```
spring:
  cloud:
    gateway:
      routes:
      - id: header_route
        uri: http://ityouknow.com
        predicates:
        - Header=X-Request-Id, \d+
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "X-Request-Id:666666" 
```

则返回页面代码证明匹配成功。将参数`-H "X-Request-Id:666666"`改为`-H "X-Request-Id:neo"`再次执行时返回404证明没有匹配。

### 通过 Host 匹配

Host Route Predicate 接收一组参数，一组匹配的域名列表，这个模板是一个 ant 分隔的模板，用`.`号作为分隔符。它通过参数中的主机地址作为匹配规则。

```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Host=**.ityouknow.com
```

使用 curl 测试，命令行输入:

```
curl http://localhost:8080  -H "Host: www.ityouknow.com" 
curl http://localhost:8080  -H "Host: md.ityouknow.com" 
```

经测试以上两种 host 均可匹配到 host_route 路由，去掉 host 参数则会报 404 错误。

### 通过请求方式匹配

可以通过是 POST、GET、PUT、DELETE 等不同的请求方式来进行路由。

```
spring:
  cloud:
    gateway:
      routes:
      - id: method_route
        uri: http://ityouknow.com
        predicates:
        - Method=GET
```

使用 curl 测试，命令行输入:

```
# curl 默认是以 GET 的方式去请求
curl http://localhost:8080
```

测试返回页面代码，证明匹配到路由，我们再以 POST 的方式请求测试。

```
# curl 默认是以 GET 的方式去请求
curl -X POST http://localhost:8080
```

返回 404 没有找到，证明没有匹配上路由

### 通过请求路径匹配

Path Route Predicate 接收一个匹配路径的参数来判断是否走路由。

```
spring:
  cloud:
    gateway:
      routes:
      - id: host_route
        uri: http://ityouknow.com
        predicates:
        - Path=/foo/{segment}
```

如果请求路径符合要求，则此路由将匹配，例如：/foo/1 或者 /foo/bar。

使用 curl 测试，命令行输入:

```
curl http://localhost:8080/foo/1
curl http://localhost:8080/foo/xx
curl http://localhost:8080/boo/xx
```

经过测试第一和第二条命令可以正常获取到页面返回值，最后一个命令报404，证明路由是通过指定路由来匹配。

### 通过请求参数匹配

Query Route Predicate 支持传入两个参数，一个是属性名一个为属性值，属性值可以是正则表达式。

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=smile
```

这样配置，只要请求中包含 smile 属性的参数即可匹配路由。

使用 curl 测试，命令行输入:

```
curl localhost:8080?smile=x&id=2
```

经过测试发现只要请求汇总带有 smile 参数即会匹配路由，不带 smile 参数则不会匹配。

还可以将 Query 的值以键值对的方式进行配置，这样在请求过来时会对属性值和正则进行匹配，匹配上才会走路由。

```
spring:
  cloud:
    gateway:
      routes:
      - id: query_route
        uri: http://ityouknow.com
        predicates:
        - Query=keep, pu.
```

这样只要当请求中包含 keep 属性并且参数值是以 pu 开头的长度为三位的字符串才会进行匹配和路由。

使用 curl 测试，命令行输入:

```
curl localhost:8080?keep=pub
```

测试可以返回页面代码，将 keep 的属性值改为 pubx 再次访问就会报 404,证明路由需要匹配正则表达式才会进行路由。

### 通过请求 ip 地址进行匹配

Predicate 也支持通过设置某个 ip 区间号段的请求才会路由，RemoteAddr Route Predicate 接受 cidr 符号(IPv4 或 IPv6 )字符串的列表(最小大小为1)，例如 192.168.0.1/16 (其中 192.168.0.1 是 IP 地址，16 是子网掩码)。

```
spring:
  cloud:
    gateway:
      routes:
      - id: remoteaddr_route
        uri: http://ityouknow.com
        predicates:
        - RemoteAddr=192.168.1.1/24
```

可以将此地址设置为本机的 ip 地址进行测试。

```
curl localhost:8080
```

果请求的远程地址是 192.168.1.10，则此路由将匹配。

### 组合使用

上面为了演示各个 Predicate 的使用，我们是单个单个进行配置测试，其实可以将各种 Predicate 组合起来一起使用。

例如：

```
spring:
  cloud:
    gateway:
      routes:
       - id: host_foo_path_headers_to_httpbin
        uri: http://ityouknow.com
        predicates:
        - Host=**.foo.org
        - Path=/headers
        - Method=GET
        - Header=X-Request-Id, \d+
        - Query=foo, ba.
        - Query=baz
        - Cookie=chocolate, ch.p
        - After=2018-01-20T06:06:06+08:00[Asia/Shanghai]
```

各种 Predicates 同时存在于同一个路由时，请求必须同时满足所有的条件才被这个路由匹配。

## 四：过滤器使用

## 修改请求路径的过滤器

**StripPrefix Filter**

StripPrefix Filter 是一个请求路径截取的功能，我们可以利用这个功能来做特殊业务的转发。

application.yml 配置如下：

```
spring:
  cloud:
    gateway:
      routes:
      - id: nameRoot
        uri: http://nameservice
        predicates:
        - Path=/name/**
        filters:
        - StripPrefix=2
```

上面这个配置的例子表示，当请求路径匹配到`/name/**`会将包含name和后边的字符串接去掉转发， `StripPrefix=2`就代表截取路径的个数，这样配置后当请求`/name/bar/foo`后端匹配到的请求路径就会变成`http://nameservice/foo`。

我们还是在 cloud-gateway-eureka 项目中进行测试，修改 application.yml 如下：

```
spring:
  cloud:
     routes:
     - id: nameRoot
       uri: lb://spring-cloud-producer
       predicates:
       - Path=/name/**
       filters:
       - StripPrefix=2
```

配置完后重启 cloud-gateway-eureka 项目，访问地址：`http://localhost:8888/name/foo/hello`页面会交替显示：

```
hello world!
hello world smile!
```

和直接访问地址 `http://localhost:8888/hello`展示的效果一致，说明请求路径中的 `name/foo/` 已经被截取。

**PrefixPath Filter**

PrefixPath Filter 的作用和 StripPrefix 正相反，是在 URL 路径前面添加一部分的前缀

```
spring:
  cloud:
    gateway:
      routes:
      - id: prefixpath_route
        uri: http://example.org
        filters:
        - PrefixPath=/mypath
```

大家可以下来去测试，这里不在演示。

## 限速路由器

限速在高并发场景中比较常用的手段之一，可以有效的保障服务的整体稳定性，Spring Cloud Gateway 提供了基于 Redis 的限流方案。所以我们首先需要添加对应的依赖包`spring-boot-starter-data-redis-reactive`

```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

配置文件中需要添加 Redis 地址和限流的相关配置

```
spring:
  application:
    name: cloud-gateway-eureka
  redis:
    host: localhost
    password:
    port: 6379
  cloud:
    gateway:
     discovery:
        locator:
         enabled: true
     routes:
     - id: requestratelimiter_route
       uri: http://example.org
       filters:
       - name: RequestRateLimiter
         args:
           redis-rate-limiter.replenishRate: 10
           redis-rate-limiter.burstCapacity: 20
           key-resolver: "#{@userKeyResolver}"
       predicates:
         - Method=GET
```

- filter 名称必须是 RequestRateLimiter
- redis-rate-limiter.replenishRate：允许用户每秒处理多少个请求
- redis-rate-limiter.burstCapacity：令牌桶的容量，允许在一秒钟内完成的最大请求数
- key-resolver：使用 SpEL 按名称引用 bean

项目中设置限流的策略，创建 Config 类。

```
public class Config {

    @Bean
    KeyResolver userKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
    }
}
```

根据请求参数中的 user 字段来限流，也可以设置根据请求 IP 地址来限流，设置如下:

```
@Bean
public KeyResolver ipKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}
```

这样网关就可以根据不同策略来对请求进行限流了。

## 熔断路由器

在之前的 Spring Cloud 系列文章中，大家对熔断应该有了一定的了解，如过不了解可以先读这篇文章：[熔断器 Hystrix](http://www.ityouknow.com/springcloud/2017/05/16/springcloud-hystrix.html)

Spring Cloud Gateway 也可以利用 Hystrix 的熔断特性，在流量过大时进行服务降级，同样我们还是首先给项目添加上依赖。

```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

配置示例

```
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: http://example.org
        filters:
        - Hystrix=myCommandName
```

配置后，gateway 将使用 myCommandName 作为名称生成 HystrixCommand 对象来进行熔断管理。如果想添加熔断后的回调内容，需要在添加一些配置。

```
spring:
  cloud:
    gateway:
      routes:
      - id: hystrix_route
        uri: lb://spring-cloud-producer
        predicates:
        - Path=/consumingserviceendpoint
        filters:
        - name: Hystrix
          args:
            name: fallbackcmd
            fallbackUri: forward:/incaseoffailureusethis
```

`fallbackUri: forward:/incaseoffailureusethis`配置了 fallback 时要会调的路径，当调用 Hystrix 的 fallback 被调用时，请求将转发到`/incaseoffailureuset`这个 URI。

## 重试路由器

RetryGatewayFilter 是 Spring Cloud Gateway 对请求重试提供的一个 GatewayFilter Factory

配置示例：

```
spring:
  cloud:
    gateway:
      routes:
      - id: retry_test
        uri: lb://spring-cloud-producer
        predicates:
        - Path=/retry
        filters:
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY
```

Retry GatewayFilter 通过这四个参数来控制重试机制： retries, statuses, methods, 和 series。

- retries：重试次数，默认值是 3 次
- statuses：HTTP 的状态返回码，取值请参考：`org.springframework.http.HttpStatus`
- methods：指定哪些方法的请求需要进行重试逻辑，默认值是 GET 方法，取值参考：`org.springframework.http.HttpMethod`
- series：一些列的状态码配置，取值参考：`org.springframework.http.HttpStatus.Series`。符合的某段状态码才会进行重试逻辑，默认值是 SERVER_ERROR，值是 5，也就是 5XX(5 开头的状态码)，共有5 个值。

**添加参数过滤器**

**application.yml配置示例**

```
spring:
  cloud:
    gateway:
      routes:
      - id: add_request_parameter_route
        uri: http://example.org
        filters:
        - AddRequestParameter=foo, bar
```

这样就会给匹配的每个请求添加上`foo=bar`的参数和值。



