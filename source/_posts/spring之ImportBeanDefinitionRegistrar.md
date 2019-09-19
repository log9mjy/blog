---
title: spring之ImportBeanDefinitionRegistrar
date: 2019-05-18 16:28:01
tags: 
categories: spring

---

## 

使用ImportBeanDefinitionRegistrar动态加载BeanDefinition

```java 
public class SecurityBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
   /**
    * 根据注解值动态注入资源服务器的相关属性
    *
    * @param metadata 注解信息
    * @param registry 注册器
    */
   @Override
   public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
      if (registry.isBeanNameInUse(SecurityConstants.RESOURCE_SERVER_CONFIGURER)) {
         log.warn("本地存在资源服务器配置，覆盖默认配置:" + SecurityConstants.RESOURCE_SERVER_CONFIGURER);
         return;
      }

      GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
      beanDefinition.setBeanClass(PigResourceServerConfigurerAdapter.class);
      registry.registerBeanDefinition(SecurityConstants.RESOURCE_SERVER_CONFIGURER, beanDefinition);

   }
}
```

在使用

```java
@Import({SecurityBeanDefinitionRegistrar.class})
```

就将该类注入