---
title: springboot-Controller的全局控制
date: 2018-07-12 11:43:08
tags: 
categories: springboot
---
#### SpringMVC之Controller的全局控制



- @ControllerAdvice

  

  - @ExceptionHandler 捕捉Controller中发生的异常
  - @InitBinder 设置WebDataBinder，即设置请求数据中的统一操作，过滤，添加等等
  - @ModelAttribute 绑定键值对到Model中，在Spring中，所有的请求都会有一个Model
  - basePackageClasses = FooController.class为该controller相同包下的controller,不填则为所有的

```
@ControllerAdvice(basePackageClasses = FooController.class)
public class ExceptionHandlerAdvice {
    @ExceptionHandler(value = Exception.class)
    public ModelAndView exception(Exception exception, WebRequest request) {
        ModelAndView modelAndView =
                new ModelAndView("error");
        modelAndView.addObject("errorMessage", exception.getMessage());
        return modelAndView;
    }

    @InitBinder
    public void initBinder(WebDataBinder webDataBinder) {
        webDataBinder.setDisallowedFields("time");
    }

    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("time", new Date().getTime());
    }

}
```

#### Spring Boot图标

 favicon.ico在配置的静态内容位置和类路径的根目录（按此顺序）中查找 。如果存在这样的文件，它会自动用作应用程序的图标。

#### CORS支持

```
@Configuration
public class MyConfiguration {

​    @Bean
​    public WebMvcConfigurer corsConfigurer() {
​        return new WebMvcConfigurerAdapter() {
​            @Override
​            public void addCorsMappings(CorsRegistry registry) {
​                registry.addMapping("/api/**");
​            }
​        };
​    }
}  
```

#### 自定义参数转换器

正常情况下，前端传递来的参数都能直接被SpringMVC接收，但是也会遇到一些特殊情况，比如Date对象，当我的前端传来的一个日期时，就需要服务端自定义参数绑定，将前端的日期进行转换。自定义参数绑定也很简单，分两个步骤：


自定义参数转换器实现Converter接口，如下：

public class DateConverter implements Converter<String,Date> {
    private SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
    @Override
    public Date convert(String s) {
        if ("".equals(s) || s == null) {
            return null;
        }
        try {
            return simpleDateFormat.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }
}
convert方法接收一个字符串参数，这个参数就是前端传来的日期字符串，这个字符串满足yyyy-MM-dd格式，然后通过SimpleDateFormat将这个字符串转为一个Date对象返回即可。

配置转换器

自定义WebMvcConfig继承WebMvcConfigurer，在addFormatters方法中进行配置：

@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new DateConverter());
    }
}
OK，如上两步之后，我们就可以在服务端接收一个前端传来的字符串日期并将之转为Java中的Date对象了

#### 拦截器

生成拦截器，实现HandlerInterceptor接口，实现其中的方法。共有三个方法。如下：

```
public class MyIntecter implements HandlerInterceptor{
	/**
	 * controller执行前调用，为FALSE则controller不执行
	 */
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行之前调用");
		return true;
	}
	/**
	 * controller调用之后执行，且页面渲染之前调用
	 */
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("执行之后调用");
	}
	 /**
     * 页面渲染之后调用，一般用于资源清理操作
     */
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		
}

}
	/**
	 * controller执行前调用，为FALSE则controller不执行
	 */
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("执行之前调用");
		return true;
	}
	/**
	 * controller调用之后执行，且页面渲染之前调用
	 */
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("执行之后调用");
	}
	 /**
     * 页面渲染之后调用，一般用于资源清理操作
     */
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		
}

}
```

将拦截器加入到Spring容器中：

```
@Configuration
public class SpringMVCConfig extends WebMvcConfigurer{

@Override
public void addInterceptors(InterceptorRegistry registry) {
	registry.addInterceptor(new MyIntecter());
}

}
```



#### 配置静态资源

默认有/static,/public/resource，需要修改可以使用下面方法

```
@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        if(!registry.hasMappingForPattern("/static/**")){
            registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
        }

     }
```

