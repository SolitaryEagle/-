[TOC]
**AnnotationConfigApplicationContext 继承体系**

![AnnotationConfigApplicationContext 继承体系](image/AnnotationConfigApplicationContext继承体系.png)
## AnnotationConfigApplicationContext 解读
		独立应用程序上下文，接受注解类作为输入，特别是 @Configuration 类，也接受普通的 @Component 类和使用
    javax.inject 注解的JSR-330 兼容的类。允许使用 register(Class...) 逐个注册类，也允许使用 scan(String...)
    扫描 classpath。
    	在含有多个 @Configuration 类的情况下，定义在后面的类中的 @Bean 方法将覆盖在前面的类中定义的 Bean。这可以作为
    一个使用额外的 @Configuration 类覆盖某些 Bean 定义的方法。
		AnnotationConfigApplicationContext 中有两个属性：AnnotatedBeanDefinitionReader 和
    ClassPathBeanDefinitionScanner。而且 AnnotatedBeanDefinitionReader 是需要重点走查的，
    ClassPathBeanDefinitionScanner 不怎么使用。只有手动调用 scan(String...) 方法才会使用。
## AnnotationConfigRegistry 解读
		注解配置的应用程序上下文的通用接口, 定义了 register(java.lang.Class<?>...) 和 scan(java.lang.String...)
    方法.
## AnnotatedBeanDefinitionReader 解读
		方便的适配器，用于注释bean类的编程注册。这是ClassPathBeanDefinitionScanner的替代方法，应用相同的注释分辨率，
    但仅适用于显式注册的类。
## ClassPathBeanDefinitionScanner 解读
		一个bean定义扫描程序，它检测类路径上的bean候选者，使用给定的注册表（BeanFactory或ApplicationContext）
    注册相应的bean定义。
		通过可配置的类型过滤器检测候选类。默认过滤器包括使用Spring的@ Component，@ Repository，@ Service 或
    @Controller构造型注释的类。
    	还支持Java EE 6的ManagedBean和JSR-330的 Named 注释（如果可用）。
## GenericXmlApplicationContext 解读
		方便的应用程序上下文，内置XML支持。这是ClassPathXmlApplicationContext和FileSystemXmlApplicationContext
    的灵活替代方案，可通过setter配置，最终AbstractApplicationContext.refresh（）调用激活上下文。
    	如果有多个配置文件，以后文件中的bean定义将覆盖早期文件中定义的bean定义。这可以用于通过附加到列表的额外配置文件
    有意覆盖某些bean定义。
<!-- 属性 -->

<!-- 父类 -->
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读













