---
title: Spring Security使用
date: 2018-07-12 11:43:08
tags: [Spring Security]
---
### Spring Security使用

#### 过滤器

WebAsyncManagerIntegrationFilter 
SecurityContextPersistenceFilter 
HeaderWriterFilter 
CorsFilter 
LogoutFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
AnonymousAuthenticationFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter



#### 核心组件

  SecurityContextHolder：提供对SecurityContext的访问
  SecurityContext,：持有Authentication对象和其他可能需要的信息
  AuthenticationManager 其中可以包含多个AuthenticationProvider
  ProviderManager对象为AuthenticationManager接口的实现类
  AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
  Authentication：Spring Security方式的认证主体
  GrantedAuthority：对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
 UserDetails：构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
 UserDetailsService：通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）  

---------------------
