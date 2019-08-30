## @Profile 解读
		表示当一个或多个指定的 profiles 处于活动状态时，组件符合注册条件。
        profile 是一个命名的逻辑分组，可以通过 ConfigurableEnvironment.setActiveProfiles（java.lang.String ...）
    以编程方式激活，也可以通过将 spring.profiles.active 属性设置为 JVM 系统属性，环境变量或者声明为Web应用程序的web.xml中的
    Servlet上下文参数。也可以通过 @ActiveProfiles 注解在集成测试中以声明方式激活 profile。
    	@Profile 注解可以通过以下任何方式使用：
        	1. 作为直接或间接使用 @Component 注解的任何类, 包括 @Configuration 类, 的类型级注解.
        	2. 作为元注解，用于组成自定义构造型注解
        	3. 作为任何 @Bean 方法的方法级注解
        如果使用 Profile 标记 @Configuration, 则将绕过与该类关联的所有 @Bean 方法和 @Import 注解，除非一个或多个指定的
    profile 处于活动状态。 profile string 可能只包含一个简单的 profile name(如, "p1") 或者一个 profile 表达式. profile
    表达式允许表达更复杂的 profile 逻辑，例如“p1＆p2”。有关支持的格式的详细信息，请参阅Profiles.of（String ...）。
    	这类似于Spring XML中的行为：如果提供了 beans元素的 profile 属性，例如 <beans profile =“p1，p2”>，则不会解析 beans
    元素，除非至少激活了 profile 'p1'或'p2'。同样，如果 @Component 或 @Configuration 类标记为 @Profile（{“p1”，“p2”}），
    则除非至少激活了 profile “p1”或“p2”，否则不会注册或处理该类。
    	如果给定的 profile 以NOT运算符（！）作为前缀，则如果 profile 不活动，则将注册带注解的组件, 例如, 给定
    @Profile({"p1", "!p2"}), 如果 profile “p1”处于活动状态或 profile “p2”未激活，则会发生注册。
    	如果省略 @Profile 注解，则无论哪个（如果有） profile 处于活动状态，都将进行注册。
    	注意: 对于 @Bean 方法的 @Profile，可能会应用特殊方案：对于具有相同 Java 方法名称的重载 @Bean 方法（类似于构造函数重载），
    需要在所有重载方法上一致地声明 @Profile 条件。如果条件不一致，则只有重载方法中第一个声明的条件才重要。因此，@Profile 不能用于
    选择具有特定参数签名的重载方法而不是另一个; 同一个 bean 的所有工厂方法之间的分辨率遵循 Spring 的构造函数解析算法在创建时。如果
    要定义具有不同配置文件条件的备用 bean，请使用指向同一 bean 名称的不同 Java 方法名称;请参阅@Configuration的javadoc中的
    ProfileDatabaseConfig。
		通过 XML 定义 Spring bean 时，可以使用 <beans> 元素的 “profile” 属性。有关详细信息，请参阅spring-beans XSD
    （3.1或更高版本）中的文档。
    	属性:
        	1. value: 应注册带注解的组件的 profile 集。
## @Configuration 解读
		表示一个类声明了一个或多个 @Bean 方法，并且可以由 Spring 容器处理，以便在运行时为这些 bean 生成 bean 定义和服务请求，如:
```java
@Configuration
public class AppConfig {

 @Bean
 public MyBean myBean() {
     // instantiate, configure and return bean ...
 }
}
```
	引导 @Configuration 类:
    	通过 AnnotationConfigApplicationContext
        @Configuration 类通常使用 AnnotationConfigApplicationContext 或其支持 Web 的变体
    AnnotationConfigWebApplicationContext 进行引导。前者的一个简单示例如下：
```java
 AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
 ctx.register(AppConfig.class);
 ctx.refresh();
 MyBean myBean = ctx.getBean(MyBean.class);
 // use myBean ...
````
## @Bean 解读
## port 解读
        
        
        
        
        
        
        
        
        
        
        
   