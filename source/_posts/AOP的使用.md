---
title: 实用类
date: 2019-06-10 10:40:19
tags: aop的使用
---

## 1.RequestContextHolder

在spring mvc中，为了随时都能取到当前请求的request对象，可以通过RequestContextHolder的静态方法getRequestAttributes()获取Request相关的变量，如request, response等。 

```Java
//两个方法在没有使用JSF的项目中是没有区别的
RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
//RequestContextHolder.getRequestAttributes();
//从session里面获取对应的值
String str = (String) requestAttributes.getAttribute("name",RequestAttributes.SCOPE_SESSION);

HttpServletRequest request = ((ServletRequestAttributes)requestAttributes).getRequest();
HttpServletResponse response = ((ServletRequestAttributes)requestAttributes).getResponse();
```

## 2.注解的使用

- **@Documented**：注解表明制作javadoc时，是否将注解信息加入文档。如果注解在声明时使用了@Documented，则在制作javadoc时注解信息会加入javadoc。

- @Target

  ：用来说明该注解可以被声明在那些元素之前

  - @Target(ElementType.TYPE) //接口、类、枚举、注解
  - @Target(ElementType.FIELD) //字段、枚举的常量
  - @Target(ElementType.METHOD) //方法
  - @Target(ElementType.PARAMETER) //方法参数
  - @Target(ElementType.CONSTRUCTOR) //构造函数
  - @Target(ElementType.LOCAL_VARIABLE)//局部变量
  - @Target(ElementType.ANNOTATION_TYPE)//注解
  - @Target(ElementType.PACKAGE) ///包

- @Retention

  ：用来说明该注解类的生命周期。

  - @Retention(RetentionPolicy.SOURCE) —— 这种类型的Annotations只在源代码级别保留,编译时就会被忽略
  - @Retention(RetentionPolicy.CLASS) —— 这种类型的Annotations编译时被保留,在class文件中存在,但JVM将会忽略
  - @Retention(RetentionPolicy.RUNTIME) —— 这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用.

  ```java
  /**
   * 登陆用户请求单次限制
   */
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface SingleRequest {
      /**
       * 限定的时间(秒)
       */
      int limitTime() default 10;
  }
  ```

  #### springboot开启AOP

  1. 添加依赖

  ```
  <dependency>    
      <groupId>org.springframework.boot</groupId>    
      <artifactId>spring-boot-starter-aop</artifactId>
  </dependency>
  ```

  2. 在application.properties中加入配置

  ```
  spring.aop.auto=true
  ```

  3. 定义切面

  ```
  @Aspect
  @Component
  public class AopConfiguration {
  
  }
  ```

  4. 切面内定义切入点，就是执行的条件

  ```
      @Pointcut("execution(* com.test.service.*.*(..))")
      public void executeService()
      {
  
      }
  ```

  ```
  	 1）execution：用于匹配子表达式。
  
              //匹配com.cjm.model包及其子包中所有类中的所有方法，返回类型任意，方法参数任意
              @Pointcut("execution(* com.cjm.model..*.*(..))")
              public void before(){}
  
   
  
        2）within：用于匹配连接点所在的Java类或者包。
  
              //匹配Person类中的所有方法
              @Pointcut("within(com.cjm.model.Person)")
              public void before(){}
  
   
  
              //匹配com.cjm包及其子包中所有类中的所有方法
  
              @Pointcut("within(com.cjm..*)")
              public void before(){}
  
   
  
       3） this：用于向通知方法中传入代理对象的引用。
              @Before("before() && this(proxy)")
              public void beforeAdvide(JoinPoint point, Object proxy){
                    //处理逻辑
              }
  
   
  
        4）target：用于向通知方法中传入目标对象的引用。
              @Before("before() && target(target)
              public void beforeAdvide(JoinPoint point, Object proxy){
                    //处理逻辑
              }
  
   
  
        5）args：用于将参数传入到通知方法中,匹配的字段注入
              @Before("before() && args(age,username)")
              public void beforeAdvide(JoinPoint point, int age, String username){
                    //处理逻辑
              }
   
        6）@within ：用于匹配在类一级使用了参数确定的注解的类，其所有方法都将被匹配。 
  
              @Pointcut("@within(com.cjm.annotation.AdviceAnnotation)") － 所有被@AdviceAnnotation标注的类都将匹配
              public void before(){}
  
  　　
  
        7）@target ：和@within的功能类似，但必须要指定注解接口的保留策略为RUNTIME。
              @Pointcut("@target(com.cjm.annotation.AdviceAnnotation)")
              public void before(){}
  
   
  
        8）@args ：传入连接点的对象对应的Java类必须被@args指定的Annotation注解标注。
              @Before("@args(com.cjm.annotation.AdviceAnnotation)")
              public void beforeAdvide(JoinPoint point){
                    //处理逻辑
              }
  
  　　
  
        9）@annotation ：匹配连接点被它参数指定的Annotation注解的方法。也就是说，所有被指定注解标注的方法都将匹配。
              @Pointcut("@annotation(com.cjm.annotation.AdviceAnnotation)")
              public void before(){}
  
        10）bean：通过受管Bean的名字来限定连接点所在的Bean。该关键词是Spring2.5新增的。
              @Pointcut("bean(person)")
              public void before(){}
              
        11)@AfterReturning(value = POINT_CUT,returning = "keys")  
            public void doAfterReturningAdvice1(JoinPoint joinPoint,Object keys){  
            logger.info("第一个后置返回通知的返回值："+keys);  
           } 
       
  ```

  ##### JoinPoint 常用方法

  ```
     //获取目标方法的参数信息  
      Object[] obj = joinPoint.getArgs();  
      //AOP代理类的信息  
      joinPoint.getThis();  
      //代理的目标对象  
      joinPoint.getTarget();  
      //用的最多 通知的签名  
      Signature signature = joinPoint.getSignature();  
      //代理的是哪一个方法  
      logger.info("代理的是哪一个方法"+signature.getName());  
      //AOP代理类的名字  
      logger.info("AOP代理类的名字"+signature.getDeclaringTypeName());  
      //AOP代理类的类（class）信息  
      signature.getDeclaringType();  
  ```

  ProceedingJoinPoint常用方法

  ```
  //执行连接点的方法
  public Object proceed() throws Throwable;
  ```

  ##### 实例

  切点

  ```Java
  @Pointcut("@annotation(com.xb.merchant.aspect.CodeLimit)")
  public void sendCode() {
  }
  ```

  ```Java
  @Around(value = "sendCode()&&args(cellphone)&&@annotation(codeLimit)", argNames = "joinPoint,cellphone,codeLimit")
  public ResultVo codeLimit(ProceedingJoinPoint joinPoint,String cellphone,CodeLimit codeLimit) throws Throwable {
     //codeLimit获取注解的参数
     //RequestContextHolder获取request上的参数
  }
  ```

  

