---
title: spring-cloud之权限
date: 2019-04-21 12:16:31
tags: 
categories: springcloud
---

 

#### 对外提供的接口的权限:使用oauth2

oauth2除了验证token,还验证客户端的合法性 . access_token是为了调用接口需要的验证,而client_id和client_secret是验证颁发token的客户端.在每个资源服务器都会配client_id和client_secret,相当于不是每一个资源服务器都可以来我这里认证,必须要我认证的才可以 .不要误解为颁发给这个客户端的token,只能调用这个客户端相同的资源服务器接口.而是只要这个client_id和client_secret认证正确都可以调用,起到权限的作用还是token.使用

```java
  @RequestMapping(value = "/list", method = RequestMethod.GET)
    @PreAuthorize("hasRole('ADMIN')")
    public R list() {
        List<User> list = userService.list();
        return R.ok(list);
    }
```



授权服务器

```java
package com.mjy.auth.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {


    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated();
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```



```java
package com.mjy.auth.config;

import com.mjy.common.core.constants.SecurityConstants;
import com.mjy.common.security.entity.UserDetails;
import com.mjy.common.security.service.ClientDetailsService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.common.DefaultOAuth2AccessToken;
import org.springframework.security.oauth2.common.exceptions.OAuth2Exception;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.error.WebResponseExceptionTranslator;
import org.springframework.security.oauth2.provider.token.TokenEnhancer;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.redis.RedisTokenStore;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

@EnableAuthorizationServer
@Configuration
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    private DataSource dataSource;

    private RedisConnectionFactory redisConnectionFactory;

    private UserDetailsService userDetailsService;

    private AuthenticationManager authenticationManager;

    private WebResponseExceptionTranslator<OAuth2Exception> exceptionTranslator;


    public AuthorizationServerConfiguration(DataSource dataSource,
                                            RedisConnectionFactory redisConnectionFactory,
                                            UserDetailsService userDetailsService,
                                            AuthenticationManager authenticationManager,
                                            WebResponseExceptionTranslator<OAuth2Exception> exceptionTranslator) {
        this.dataSource = dataSource;
        this.redisConnectionFactory = redisConnectionFactory;
        this.userDetailsService = userDetailsService;
        this.authenticationManager = authenticationManager;
        this.exceptionTranslator = exceptionTranslator;
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients()//开启过滤器认证客户端信息
                .checkTokenAccess("permitAll()")//开启/oauth/token_key验证端口无权限访问
                .tokenKeyAccess("permitAll()");//开启/oauth/check_token验证端口认证权限访问
        // .passwordEncoder(new BCryptPasswordEncoder()); //编码
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        ClientDetailsService clientDetailsService = new ClientDetailsService(dataSource);
        clientDetailsService.setSelectClientDetailsSql(SecurityConstants.DEFAULT_SELECT_STATEMENT);
        clientDetailsService.setFindClientDetailsSql(SecurityConstants.DEFAULT_FIND_STATEMENT);
        clients.withClientDetails(clientDetailsService);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST)
                .tokenStore(tokenStore())
                .tokenEnhancer(tokenEnhancer())
                .userDetailsService(userDetailsService)
                .authenticationManager(authenticationManager)
                .exceptionTranslator(exceptionTranslator)
                .reuseRefreshTokens(false);//refresh token不需要需要重新生成；
    }

    @Bean
    public TokenStore tokenStore() {
        RedisTokenStore tokenStore = new RedisTokenStore(redisConnectionFactory);
        tokenStore.setPrefix(SecurityConstants.PROJECT_PREFIX + SecurityConstants.OAUTH_PREFIX);
        return tokenStore;
    }

    @Bean
    public TokenEnhancer tokenEnhancer() {
        return (accessToken, authentication) -> {
            final Map<String, Object> additionalInfo = new HashMap<>(1);
            UserDetails userDetails = (UserDetails) authentication.getUserAuthentication().getPrincipal();
            additionalInfo.put(SecurityConstants.DETAILS_USER_ID, userDetails.getId());
            ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
            return accessToken;
        };
    }
}
```

资源服务器

RemoteTokenServices,将headers中的

Authorization-----Bearer 40f94906-72b4-4e37-8ac4-1c322e00f5e1,调用loadAuthentication()封装token参数,并且将client_id,client_secret

```java
String creds = String.format("%s:%s", clientId, clientSecret);

try {
    return "Basic " + new String(Base64.encode(creds.getBytes("UTF-8")));
} catch (UnsupportedEncodingException var5) {
    throw new IllegalStateException("Could not convert String");
}
```

远程调用授权服务器验证token以及客户端的合法性

```java
package com.mjy.common.security.config;

import com.mjy.common.security.component.IgnorePropertiesConfig;
import com.mjy.common.security.handler.AccessDeniedHandler;
import com.mjy.common.security.handler.EntryPointHandle;
import lombok.SneakyThrows;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.security.oauth2.resource.ResourceServerProperties;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.http.HttpStatus;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.ExpressionUrlAuthorizationConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.token.RemoteTokenServices;
import org.springframework.web.client.DefaultResponseErrorHandler;
import org.springframework.web.client.RestTemplate;


@EnableResourceServer
@Configuration
@ConditionalOnProperty("security.oauth2.client.client-id")
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {


    private IgnorePropertiesConfig propertiesConfig;

    private AccessDeniedHandler accessDeniedHandler;

    private EntryPointHandle entryPointHandle;

    private ResourceServerProperties resource;


    public ResourceServerConfiguration(IgnorePropertiesConfig propertiesConfig,
                                       AccessDeniedHandler accessDeniedHandler,
                                       EntryPointHandle entryPointHandle,
                                       ResourceServerProperties resource) {
        this.propertiesConfig = propertiesConfig;
        this.accessDeniedHandler = accessDeniedHandler;
        this.entryPointHandle = entryPointHandle;
        this.resource = resource;
    }


    @Bean
    @Primary
    @LoadBalanced// 开启负载均衡,使用服务名(mjy-auth)
    public RestTemplate lbRestTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setErrorHandler(new DefaultResponseErrorHandler() {
            @Override
            @SneakyThrows
            public void handleError(ClientHttpResponse response) {
                if (response.getRawStatusCode() != HttpStatus.BAD_REQUEST.value()) {
                    super.handleError(response);
                }
            }
        });
        return restTemplate;
    }

    @Bean
    public RemoteTokenServices remoteTokenServices() {
        RemoteTokenServices services = new RemoteTokenServices();
        services.setCheckTokenEndpointUrl(resource.getTokenInfoUri());
        services.setClientId(resource.getClientId());
        services.setClientSecret(resource.getClientSecret());
        services.setRestTemplate(lbRestTemplate());
        return services;
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.accessDeniedHandler(accessDeniedHandler)
                .authenticationEntryPoint(entryPointHandle)
                .tokenServices(remoteTokenServices());
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>
                .ExpressionInterceptUrlRegistry registry = http.csrf()
                .disable()
                .authorizeRequests();
        propertiesConfig.getUrls().forEach(s -> registry.antMatchers(s).permitAll());
        registry.anyRequest().authenticated();
    }
}
```



#### 服务之间的调用的权限

所有调用接口都由网关转发,所以在部署时,只需要暴露网关的IP和端口,其他服务所有服务器可以使用内部局域网,这样外部网络无法直接访问这些服务的接口,只能通过网关.在服务间调用,我们可以给服务调用添加一个内部服务调用的header,这些提供服务的接口不需要oauth2的权限,直接放开.并且在网关是指拦截器,将我们设置特殊的header中的值清空,这样就不会服务调用服务的时候带上特殊的header,那些内部调用的接口就不会让调用.这一公共我们可以使用注解,加上aop可以统一处理.

```java
@Slf4j
@Aspect
@Component
@AllArgsConstructor
public class SecurityInnerAspect implements Ordered {
   private final HttpServletRequest request;

   @SneakyThrows
   @Around("@annotation(inner)")
   public Object around(ProceedingJoinPoint point, Inner inner) {
      String header = request.getHeader(SecurityConstants.FROM);
      if (inner.value() && !StrUtil.equals(SecurityConstants.FROM_IN, header)) {
         log.warn("访问接口 {} 没有权限", point.getSignature().getName());
         throw new AccessDeniedException("Access is denied");
      }
      return point.proceed();
   }

   @Override
   public int getOrder() {
      return Ordered.HIGHEST_PRECEDENCE + 1;
   }
}
```

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Inner {

   /**
    * 是否AOP统一处理
    *
    * @return false, true
    */
   boolean value() default true;

   /**
    * 需要特殊判空的字段(预留)
    *
    * @return {}
    */
   String[] field() default {};
}
```

```java
@Inner
@GetMapping("/info/{username}")
public R info(@PathVariable String username) {
   SysUser user = userService.getOne(Wrappers.<SysUser>query()
      .lambda().eq(SysUser::getUsername, username));
   if (user == null) {
      return R.failed(String.format("用户信息为空 %s", username));
   }
   return R.ok(userService.getUserInfo(user));
}
```