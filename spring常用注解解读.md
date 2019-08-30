[TOC]
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
    	通过 AnnotationConfigApplicationContext:
        	@Configuration 类通常使用 AnnotationConfigApplicationContext 或其支持 Web 的变体
    	AnnotationConfigWebApplicationContext 进行引导。前者的一个简单示例如下：
```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.register(AppConfig.class);
ctx.refresh();
MyBean myBean = ctx.getBean(MyBean.class);
// use myBean ...
````
			有关更多详细信息，请参阅 AnnotationConfigApplicationContext javadocs，有关 Servlet 容器中的Web配置说明，
    	请参阅 AnnotationConfigWebApplicationContext。
        
        通过 Spring <beans> XML:
        	作为直接针对 AnnotationConfigApplicationContext 注册 @Configuration 类的替代方法，可以将 @Configuration
        类声明为 Spring XML 文件中的普通 <bean> 定义：
```xml
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
</beans>
```
			在上面的示例中，需要 <context：annotation-config /> 以启用 ConfigurationClassPostProcessor 和其他与注解
        相关的后处理器便于处理 @Configuration 类。
        
        通过 component scanning:
        	@Configuration 使用 @Component 元注解进行声明，因此 @Configuration 类是组件扫描的候选者（通常使用 Spring XML
        的 <context：component-scan /> 元素），因此也可以像任何常规 @Component 一样利用 @Autowired / @Inject 零件。
        特别是，如果存在单个构造函数，则将为该构造函数透明地应用自动装配语义：
```java
 @Configuration
 public class AppConfig {

     private final SomeBean someBean;

     public AppConfig(SomeBean someBean) {
         this.someBean = someBean;
     }

     // @Bean definition using "SomeBean"

 }
```
			@Configuration 类不仅可以使用组件扫描进行引导，还可以使用 @ComponentScan 注解自行配置组件扫描：
```java
 @Configuration
 @ComponentScan("com.acme.app.services")
 public class AppConfig {
     // various @Bean definitions ...
 }
```
			有关详细信息，请参阅 @ComponentScan javadocs。
	
    Working with externalized values
		使用 Environment API:
        	可以通过将 Spring Environment 注入 @Configuration 类来查找外化值 - 如, 使用 @Autowired 注解:
```java
 @Configuration
 public class AppConfig {

     @Autowired Environment env;

     @Bean
     public MyBean myBean() {
         MyBean myBean = new MyBean();
         myBean.setName(env.getProperty("bean.name"));
         return myBean;
     }
 }
```
			通过 Environment 解析的属性驻留在一个或多个“属性源”对象中，@Configuration 配置类可以使用 @PropertySource
        注解向 Environment 对象提供属性源：
```java
 @Configuration
 @PropertySource("classpath:/com/acme/app.properties")
 public class AppConfig {

     @Inject Environment env;

     @Bean
     public MyBean myBean() {
         return new MyBean(env.getProperty("bean.name"));
     }
 }
```
		使用 @Value 注解:
        	可以使用 @Value 注解将外化值注入 @Configuration 类：
```java
 @Configuration
 @PropertySource("classpath:/com/acme/app.properties")
 public class AppConfig {

     @Value("${bean.name}") String beanName;

     @Bean
     public MyBean myBean() {
         return new MyBean(beanName);
     }
 }
````
			这种方法通常与 Spring 的 PropertySourcesPlaceholderConfigurer 一起使用，可以通过
        <context：property-placeholder /> 在XML配置中自动启用，或者通过专用的静态 @Bean 方法在 @Configuration 类中
        显式启用（请参阅 @Bean 的 javadocs, “关于 BeanFactoryPostProcessor 返回的 @Bean 注解”）。但请注意，通常只有
        在需要自定义配置（如占位符语法等）时，才需要通过静态 @Bean 方法显式注册 PropertySourcesPlaceholderConfigurer。
        具体来说，如果没有为 ApplicationContext 注册嵌入值解析器的 bean 后处理器（例如
        PropertySourcesPlaceholderConfigurer），Spring 将注册一个默认的嵌入值解析器，它根据在 Environment 中注册的
        属性源来解析占位符。请参阅下面有关使用 @ImportResource 使用 Spring XM L编写 @Configuration 类的部分; 看 @Value
        javadocs; 有关使用 BeanFactoryPostProcessor 类型（如PropertySourcesPlaceholderConfigurer）的详细信息，
        请参阅 @Bean javadocs。
---
	Composing @Configuration classes
		使用 @Import 注解:
        	@Configuration 类可以使用 @Import 注解组成，类似于 <import> 在 Spring XML 中的工作方式。因为
        @Configuration 对象在容器中作为 Spring bean 进行管理，所以可以注入导入的配置 - 例如，通过构造函数注入：
```java
 @Configuration
 public class DatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return DataSource
     }
 }

 @Configuration
 @Import(DatabaseConfig.class)
 public class AppConfig {

     private final DatabaseConfig dataConfig;

     public AppConfig(DatabaseConfig dataConfig) {
         this.dataConfig = dataConfig;
     }

     @Bean
     public MyBean myBean() {
         // reference the dataSource() bean method
         return new MyBean(dataConfig.dataSource());
     }
 }
```
			现在，通过仅针对 Spring 上下文注册 AppConfig，可以引导 AppConfig 和导入的 DatabaseConfig：
```java
 new AnnotationConfigApplicationContext(AppConfig.class);
```
		使用 @Profile 注解:
        	@Configuration 类可以使用 @Profile 注解进行标记，以指示只有在给定的 profile 或 profile 处于活动状态时才
        应处理它们：
```java
 @Profile("development")
 @Configuration
 public class EmbeddedDatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return embedded DataSource
     }
 }

 @Profile("production")
 @Configuration
 public class ProductionDatabaseConfig {

     @Bean
     public DataSource dataSource() {
         // instantiate, configure and return production DataSource
     }
 }
```
			或者，您也可以在 @Bean 方法级别声明 @Profile 条件 - 例如，对于同一配置类中的备用 bean 变体：
```java
 @Configuration
 public class ProfileDatabaseConfig {

     @Bean("dataSource")
     @Profile("development")
     public DataSource embeddedDatabase() { ... }

     @Bean("dataSource")
     @Profile("production")
     public DataSource productionDatabase() { ... }
 }
```
			有关更多详细信息，请参阅 @Profile 和 Environment javadocs。
---
		在 Spring XML 中使用 @ImportResource 注解:
        	如上所述，@Consfiguration 类可以在 Spring XML 文件中声明为常规的 Spring <bean> 定义。也可以使用
        @ImportResource 注释将 Spring XML 配置文件导入 @Configuration 类。可以注入从 XML 导入的 Bean 定义. 如:
```java
 @Configuration
 @ImportResource("classpath:/com/acme/database-config.xml")
 public class AppConfig {

     @Inject DataSource dataSource; // from XML

     @Bean
     public MyBean myBean() {
         // inject the XML-defined dataSource bean
         return new MyBean(this.dataSource);
     }
 }
```
		使用嵌套的 @Configuration 类:
        	@Configuration 类可以嵌套在一起，如下所示：
```java
 @Configuration
 public class AppConfig {

     @Inject DataSource dataSource;

     @Bean
     public MyBean myBean() {
         return new MyBean(dataSource);
     }

     @Configuration
     static class DatabaseConfig {
         @Bean
         DataSource dataSource() {
             return new EmbeddedDatabaseBuilder().build();
         }
     }
 }
```
			在引导这样的安排时，只需要针对应用程序上下文注册 AppConfig。由于是一个嵌套的 @Configuration 类，DatabaseConfig
        将自动注册。当 AppConfig 和 DatabaseConfig 之间的关系已经隐式清除时，这避免了使用 @Import 注解的需要。
			另请注意，嵌套 的@Configuration 类可以通过 @Profile 注解使用，以便为封闭的 @Configuration 类提供相同 bean 的
        两个选项。
---
	Configuring lazy initialization
    	默认情况下，@Bean 方法将在容器引导时急切地实例化. 为了避免这种情况，可以将 @Configuration 与 @Lazy 注解结合使用，以
    指示在类中声明的所有 @Bean 方法都是默认延迟初始化的。请注意，@Lazy 也可以用于各个 @Bean 方法。
---
	Testing support for @Configuration classes
    	spring-test模块中提供的 Spring TestContext 框架提供了 @ContextConfiguration，它可以接受 @Configuration 类对象
    的数组：
```java
 @RunWith(SpringRunner.class)
 @ContextConfiguration(classes = {AppConfig.class, DatabaseConfig.class})
 public class MyTests {

     @Autowired MyBean myBean;

     @Autowired DataSource dataSource;

     @Test
     public void test() {
         // assertions against myBean ...
     }
 }
```
		有关详细信息，请参阅 TestContext 框架参考文档。
---
	Enabling built-in Spring features using @Enable annotations
    	可以使用各自的 “@Enable” 注解从 @Configuration 类启用和配置 Spring 等功能，例如异步方法执行，计划任务执行，
    注释驱动事务管理，甚至 Spring MVC。有关详细信息，请参阅 @EnableAsync, @EnableScheduling,
    @EnableTransactionManagement, @EnableAspectJAutoProxy, and @EnableWebMvc
---
	Constraints when authoring @Configuration classes
    	1. 必须以类的形式提供配置类（即不是从工厂方法返回的实例），允许通过生成的子类进行运行时增强。
    	2. 配置类必须是 non-final。
    	3. 配置类必须是 non-local。(不懂是啥)
    	4. 嵌入式配置类必须是 static 的.
    	5. @Bean 方法可能不会反过来创建更多的配置类（任何此类实例都将被视为常规 bean，其 @Configuration 仍未被检测到）。
## @Bean 解读
	表示方法生成由 Spring 容器管理的 bean。
---
	Overview:
    	此批注的属性的名称和语义有意类似于 Spring XML 模式中的 <bean /> 元素的名称和语义。例如：
```java
     @Bean
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
	Bean Names:
    	虽然 name() 属性可用，但确定 bean 名称的默认策略是使用 @Bean 方法的名称。这很方便直观，但如果需要显式命名，
    可以使用name属性（或其别名值）。另请注意，name 接受一个字符串数组，允许单个bean使用多个名称（即主bean名称加上一个或多个别名）。
```java
     @Bean({"b1", "b2"}) // bean available as 'b1' and 'b2', but not 'myBean'
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
	Profile, Scope, Lazy, DependsOn, Primary, Order:
    	请注意，@Bean 注解提供profile，scope，lazy，depends-on 或 primary 的属性。相反，它应该与 @Scope，@ Lazy，
    @DependsOn 和 @Primary 结合使用来声明这些语义。如:
```java
     @Bean
     @Profile("production")
     @Scope("prototype")
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
		上述注解的语义与它们在组件类级别的使用相匹配： @Profile 允许选择性地包含某些 bean。@Scope 将 bean 的范围从
    singleton 更改为指定的范围。@Lazy 仅在默认单例范围的情况下具有实际效果。除了 bean 通过直接引用表达的任何依赖项外，
	@DependsOn 还会在创建此 bean 之前强制创建特定的其他 bean，这通常对单例启动很有帮助。@Primary 是一种在注入点级别
    解决歧义的机制，如果需要注入单个目标组件但几个 bean 按类型匹配。
    	此外，@Bean 方法还可以声明 qualifier 注解和 @Order 值，在注入点解析期间要考虑，就像对应的组件类上的相应注解一样，
    但可能是每个 bean 定义非常独立（如果有多个相同定义的 Bean）。qualifier 在初始类型匹配后缩小候选集; 在集合注入点的情况下，
    Order 值确定已解析元素的顺序（多个目标 bean 按类型和 qualifier 匹配）。
    	注意: @Order 值可能影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和 @DependsOn 声明确定的
    正交关注点，如上所述。此外，由于无法在方法上声明优先级，因此无法在此级别提供优先级; 它的语义可以通过 @Order 值和 @Primary
    在每个类型的单个 bean 上建模。
---
	@Bean Methods in @Configuration Classes:
    	通常，@Bean 方法在 @Configuration 类中声明。在这种情况下，bean 方法可以通过直接调用它们来引用同一个类中的其他
    @Bean 方法。这可确保 bean 之间的引用是强类型和可导航的。这种所谓的 “bean 间引用”保证尊重范围和AOP语义，就像getBean()
    查找一样。这些是从最初的 “Spring JavaConfig” 项目中已知的语义，它需要在运行时对每个这样的配置类进行 CGLIB 子类化。
    因此，在此模式下，不得将 @Configuration 类及其工厂方法标记为 final 或 private。如:
```java
 @Configuration
 public class AppConfig {

     @Bean
     public FooService fooService() {
         return new FooService(fooRepository());
     }

     @Bean
     public FooRepository fooRepository() {
         return new JdbcFooRepository(dataSource());
     }

     // ...
 }
```
	@Bean Lite Mode:
    	@Bean 方法也可以在未使用 @Configuration 的类中声明。例如，bean 方法可以在 @Component 类中声明，甚至可以在普通的
    旧类中声明。在这种情况下，@Bean 方法将在所谓的 “lite” 模式下处理。lite 模式中的 Bean 方法将被容器视为普通工厂方法
    (类似于XML中的工厂方法声明)，并正确应用范围和生命周期回调。在这种情况下，包含类保持不变，并且包含类或工厂方法没有异常约束。
    	与 @Configuration 类中 bean 方法的语义相反，在 lite 模式下不支持 “bean间引用”。相反，当一个 @Bean 方法在 lite
    模式下调用另一个 @Bean 方法时，调用是标准的 Java 方法调用; Spring 不会通过 CGLIB 代理拦截调用。这类似于
    inter-@Transactional 方法调用，在代理模式下，Spring 不拦截调用 -  Spring 仅在 AspectJ 模式中这样做。如:
```java
 @Component
 public class Calculator {
     public int sum(int a, int b) {
         return a+b;
     }

     @Bean
     public MyBean myBean() {
         return new MyBean();
     }
 }
```
	Bootstrapping:
    	有关更多详细信息，请参阅 @Configuration javadoc，包括如何使用 AnnotationConfigApplicationContext 和朋友引导容器。
---
	BeanFactoryPostProcessor-returning @Bean methods:
    	必须特别考虑返回 Spring BeanFactoryPostProcessor（BFPP）类型的 @Bean 方法。由于 BFPP 对象必须在容器生命周期的早期
    实例化，因此它们可能会干扰 @Configuration 类中 @Autowired，@Value 和 @PostConstruct 等注解的处理。要避免这些生命周期
    问题，请将 BFPP 返回 @Bean 方法标记为 static。如:
```java
     @Bean
     public static PropertySourcesPlaceholderConfigurer pspc() {
         // instantiate, configure and return pspc ...
     }
```
		通过将此方法标记为 static，可以调用它而不会导致其声明 @Configuration 类的实例化，从而避免上述生命周期冲突。但请注意，
    如上所述，静态 @Bean 方法不会针对作用域和 AOP 语义进行增强。这在 BFPP 案例中有效，因为它们通常不被其他 @Bean 方法引用。
    作为提醒，将为任何具有可分配给 BeanFactoryPostProcessor 的返回类型的非静态 @Bean 方法发出 WARN 级别的日志消息。
---
	属性:
    	1. autowireCandidate:
    		这个bean是否可以自动连接到其他bean？ 默认为 true; 对于不打算妨碍其他地方相同类型的 bean 的内部委托，
        将此值设置为 false。
    	
    	2. destroyMethod:
    		关闭应用程序上下文时调用 bean 实例的方法的可选名称，例如 JDBC DataSource 实现上的 close() 方法或 Hibernate
        SessionFactory 对象。该方法必须没有参数，但可能抛出任何异常。
        	为方便用户，容器将尝试针对从@Bean方法返回的对象推断出destroy方法。例如，给定一个返回 Apache Commons DBCP
        BasicDataSource 的 @Bean 方法，容器将注意到该对象上可用的 close() 方法并自动将其注册为 destroyMethod。这种
        “破坏方法推理”目前仅限于检测名为 “close ”或 “shutdown” 的 public，no-arg 方法。该方法可以在继承层次结构的任何
        级别声明，并且无论 @Bean 方法的返回类型如何都将被检测到（即，在创建时反复地针对bean实例本身进行检测）。
    		要禁用特定 @Bean 的 destroy 方法推断，请指定一个空字符串作为值，例如@Bean（destroyMethod = “”）。请注意，
        仍会检测到 DisposableBean 回调接口并调用相应的 destroy 方法：换句话说，destroyMethod =“” 仅影响自定义
        close/shutdown 方法和 Closeable / AutoCloseable 声明的关闭方法。
        	注意: 仅在生命周期完全由工厂控制的 bean 上调用，对于 singletons 来说总是如此，但不保证任何其他范围。
            
    	3. initMethod:
    		初始化期间在bean实例上调用的方法的可选名称。不常用，因为可以在 Bean 注解方法的主体内直接以编程方式调用该方法。
            默认值为“”，表示不调用init方法。
            
        4. name:
        	此bean的名称，或多个名称，主 bean 名称加别名。
            如果未指定，则 bean 的名称是带注解的方法的名称。如果指定，则忽略方法名称。
            如果没有声明其他属性，也可以通过 value 属性配置 bean 名称和别名。
            
        5. value:
        	name 的别名.
            在不需要其他属性时使用，例如：@Bean（“customBeanName”）。
## @import 解读
		表示要导入的一个或多个 @Configuration 类。
    	提供与 Spring XML 中的 <import /> 元素等效的功能。允许导入 @Configuration 类，ImportSelector 和
	ImportBeanDefinitionRegistrar 实现，以及常规组件类（从4.2开始;类似于
    AnnotationConfigApplicationContext.register（java.lang.Class <？> ...））。
		应使用 @Autowired 注入来访问在导入的 @Configuration 类中声明的 @Bean 定义。bean 本身可以自动装配，或者声明 bean
    的配置类实例可以自动装配。后一种方法允许在 @Configuration 类方法之间进行显式的，IDE友好的导航。
		可以在类级别声明或作为元注释声明。
		如果需要导入 XML 或其他非 @Configuration bean定义资源，请改用 @ImportResource 注释。
---
	属性:
    	1. value: 要导入的 Configuration，ImportSelector，ImportBeanDefinitionRegistrar 或常规组件类。
## @Autowired 解读
		将构造函数，字段，setter 方法或配置方法标记为由 Spring 的依赖注入工具自动装配。这是 JSR-330 Inject 注释的替代方法，
    添加了必需与可选的语义。
    	任何给定 bean 类只有一个构造函数（最大值）可以声明这个注释，并将'required'参数设置为true，表示构造函数在用作
    Spring bean 时要自动装配。如果多个非必需构造函数声明了注释，则它们将被视为自动装配的候选者。将选择具有最大数量的依赖项的
    构造函数，这些构造函数可以通过匹配 Spring 容器中的 bean 来满足。如果不能满足任何候选者，则将使用主要/默认构造函数（如果存在）。
    如果一个类只声明一个构造函数开头，它将始终被使用，即使没有注释。带注释的构造函数不必是 public。
		在调用任何配置方法之前，在构造 bean 之后立即注入字段。这样的配置字段不必是 public。
		配置方法可以有任意名称和任意数量的参数; 每个参数都将使用 Spring 容器中的匹配 bean 进行自动装配。Bean 属性设置器方法
    实际上只是这种通用配置方法的特例。这种配置方法不必是 public。
    	对于多参数构造函数或方法，'required'参数适用于所有参数。单个参数可以声明为Java-8-style Optional，或者，从
    Spring Framework 5.0 开始，也可以在 Kotlin 中声明为 @Nullable 或非 null 参数类型，从而覆盖基本所需的语义。
    	在 Collection 或 Map 依赖类型的情况下，容器自动装配与声明的值类型匹配的所有 bean。为此目的，必须将映射键声明为 String
    类型，并将其解析为相应的 bean 名称。这样的容器提供的集合将被 order，考虑目标组件的 Ordered/Order 值，否则遵循它们在容器中的
    注册 order。或者，单个匹配的目标 bean 也可以是通常类型的 Collection 或 Map 本身，如此注入。
		请注意，实际注入是通过 BeanPostProcessor 执行的，而 BeanPostProcessor 又意味着您无法使用 @Autowired 将引用注入
    BeanPostProcessor 或 BeanFactoryPostProcessor 类型。请参考 javadoc 获取 AutowiredAnnotationBeanPostProcessor 类
    （默认情况下，它会检查是否存在此批注）。
---
	属性:
    	1. required: 声明是否需要带注释的依赖项。默认是 true.
## @Inject 解读
		标识可注入的构造函数，方法和字段。可能适用于静态成员和实例成员。可注射成员可具有任何访问修饰符(private, package-private,
    protected, public). 首先注入构造函数，然后是字段，然后是方法。超类中的字段和方法在子类中的字段和方法之前注入。未指定在字段之间
    以及同一类中的方法之间的注入顺序。
    	可注入的构造函数使用 @Inject 注释，并接受零个或多个依赖项作为参数。@Inject 可以应用于每个类最多一个构造函数。
```java
@Inject ConstructorModifiersopt SimpleTypeName(FormalParameterListopt) Throwsopt ConstructorBody
```
		当没有其他构造函数存在时，@Inject对于 public，no-argument 构造函数是可选的。这使注入器能够调用默认构造函数。
```java
@Injectopt Annotationsopt public SimpleTypeName() Throwsopt ConstructorBody
```
		可注射的字段：
        	1. 被 @Inject 注解的
        	2. non-final 的.
        	3. 可能有任何其他有效的名称。
```java
@Inject FieldModifiersopt Type VariableDeclarators;
```
		可注射的方法:
        	1. 被 @Inject 注解的
        	2. non-abstract 的
        	3. 不要声明自己的类型参数。
        	4. 可能返回一个结果.
        	5. 可能有任何其他有效的名称。
        	6. 接受零个或多个依赖项作为参数。
```java
@Inject MethodModifiersopt ResultType Identifier(FormalParameterListopt) Throwsopt MethodBody
```
		注入器忽略注入方法的结果，但允许非void返回类型支持在其他上下文中使用该方法（例如，构建器样式方法链接）。如:
```java
   public class Car {
     // Injectable constructor
     @Inject public Car(Engine engine) { ... }

     // Injectable field
     @Inject private Provider<Seat> seatProvider;

     // Injectable package-private method
     @Inject void install(Windshield windshield, Trunk trunk) { ... }
   }
```
		使用 @Inject 注释的方法将覆盖另一个使用 @Inject 注释的方法，每个实例的每个注入请求只会注入一次。不会注入没有 @Inject
    批注的方法，该方法将覆盖使用 @Inject 注释的方法。
    	注入使用@Inject注释的成员是必需的。虽然可注射成员可以使用任何可访问性修饰符（包括私有），但平台或注射器限制
    （如安全限制或缺乏反射支持）可能会阻止注入非公共成员。
---
	Qualifiers:
    	限定符可以注释可注入的字段或参数，并与该类型组合，标识要注入的实现。限定符是可选的，当与注入器无关的类中的 @Inject 一起
    使用时，不应该有多个限定符注释单个字段或参数。以下示例中的限定符为粗体：
```java
   public class Car {
     @Inject private @Leather Provider<Seat> seatProvider;

     @Inject void install(@Tinted Windshield windshield,
         @Big Trunk trunk) { ... }
   }
```
		如果一个可注入方法覆盖另一个，则覆盖方法的参数不会自动从重写方法的参数继承限定符。
---
	Injectable Values:
    	对于给定类型T和可选限定符，注入器必须能够注入用户指定的类：
        	1. 赋值与T兼容
        	2. 有一个可注射的构造函数。
		例如，用户可能使用外部配置来选择T的实现。除此之外，注入的值取决于注入器实现及其配置。
---
	Circular Dependencies:
    	检测和解决循环依赖关系留作注入器实现的练习。两个构造函数之间的循环依赖关系是一个明显的问题，但您也可以在可注入字段或方法之间
    存在循环依赖关系：
```java
   class A {
     @Inject B b;
   }
   class B {
     @Inject A a;
   }
```
		构造A的实例时，一个天真的注入器实现可能进入一个无限循环，构造一个B的实例设置在A上，一个B的第二个实例设置在B上，另一个B实例
    设置在A的第二个实例上， 等等。
    	保守的注入器可能在构建时检测到循环依赖并产生错误，此时程序员可以通过分别注入Provider<A>或Provider<B>而不是A或B来打破
    循环依赖。直接从它被注入的构造函数或方法调用提供者的get（）会破坏提供者分解循环依赖的能力。在方法或字段注入的情况下，确定其中
    一个依赖项（例如，使用单一范围）也可以启用有效的循环关系。
## @ComponentScan 解读
		配置组件扫描指令以与 @Configuration 类一起使用。提供与 Spring XML 的 <context：component-scan> 元素并行的支持。
        可以指定 basePackageClasses（）或 basePackages（）（或其别名 value（））来定义要扫描的特定包.如果未定义特定包，
    则将从声明此批注的类的包进行扫描。
    	请注意，<context：component-scan> 元素具有 annotation-config 属性; 但是，这个注释没有。这是因为在使用
    @ComponentScan 的几乎所有情况下，都假定默认注释配置处理（例如处理@Autowired和朋友）。此外，使用
    AnnotationConfigApplicationContext时，注释配置处理器始终处于注册状态，这意味着将忽略在 @ComponentScan 级别禁用它们的
    任何尝试。
    	有关用法示例，请参阅 @Configuration 的 Javadoc。
---
	属性:
    	1. Class<?>[] basePackageClasses:
    		basePackages（）的类型安全替代方法，用于指定要扫描带注释组件的包。将扫描指定的每个类的包。
            考虑在每个包中创建一个特殊的无操作标记类或接口，除了被该属性引用之外没有其它用途。
            
		2. String[] basePackages:
			用于扫描带注释组件的基础包。
            value（）是此属性的别名（并与之互斥）。
            使用 basePackageClasses（）作为基于 String 的包名称的类型安全替代方法。
            
		3. ComponentScan.Filter[] excludeFilters:
			指定哪些类型不符合组件扫描的条件。
            
 		4. ComponentScan.Filter[] includeFilters:
			指定哪些类型符合组件扫描的条件。
            进一步将候选组件集从basePackages（）中的所有内容缩小到与给定过滤器匹配的基本包中的所有内容。
            请注意，除了默认过滤器（如果已指定），还将应用这些过滤器。将包含与给定过滤器匹配的指定基础包下的任何类型，即使它与
        默认过滤器不匹配（即未使用@Component注释）。

		5. boolean lazyInit:
			指定是否应注册扫描的bean以进行延迟初始化。
            默认为false;需要时将其切换为true。

		6. Class<? extends BeanNameGenerator> nameGenerator:
			BeanNameGenerator类，用于命名Spring容器中检测到的组件。
            BeanNameGenerator 接口的默认值本身表示用于处理此 @ComponentScan 注释的扫描程序应使用其继承的 bean 名称生成器，
        例如，默认的 AnnotationBeanNameGenerator 或在引导时提供给应用程序上下文的任何自定义实例。

		7. String resourcePattern:
			控制符合组件检测条件的类文件。
            考虑使用includeFilters（）和excludeFilters（）来实现更灵活的方法。

		8. ScopedProxyMode scopedProxy:
			指示是否应为检测到的组件生成代理，这在以代理样式方式使用范围时可能是必需的。
            默认值是遵循用于执行实际扫描的组件扫描程序的默认行为。
            请注意，设置此属性会覆盖为scopeResolver（）设置的任何值。

		9. Class<? extends ScopeMetadataResolver> scopeResolver:
			ScopeMetadataResolver用于解析检测到的组件的范围。

		10. boolean useDefaultFilters:
			指示是否应启用使用 @Component @Repository，@Service 或 @Controller 注释的类的自动检测。

		11. String[] value:
			basePackages（）的别名。
            如果不需要其他属性，则允许更简洁的注释声明 - 例如，@ComponentScan（“org.my.pkg”）而不是
        @ComponentScan（basePackages =“org.my.pkg”）。
## @PropertySource 解读
		注释提供了一种方便的声明机制，用于将 PropertySource 添加到 Spring 的环境中。与@Configuration类一起使用。
---
	Example usage:
```java
 @Configuration
 @PropertySource("classpath:/com/myco/app.properties")
 public class AppConfig {

     @Autowired
     Environment env;

     @Bean
     public TestBean testBean() {
         TestBean testBean = new TestBean();
         testBean.setName(env.getProperty("testbean.name"));
         return testBean;
     }
 }
```
		请注意，Environment对象是@Autowired到配置类中，然后在填充TestBean对象时使用。鉴于上面的配置，对testBean.getName（）
    的调用将返回“myTestBean”。
---
	Resolving ${...} placeholders in <bean> and @Value annotations:
		为了使用 PropertySource 中的属性解析<bean>定义或@Value注释中的${...}占位符，必须确保在ApplicationContext使用的
    BeanFactory中注册了适当的嵌入值解析器。在XML中使用<context：property-placeholder>时会自动发生这种情况。使用
	@Configuration类时，可以通过静态@Bean方法显式注册PropertySourcesPlaceholderConfigurer来实现。但请注意，通常只有在
    需要自定义配置（如占位符语法等）时，才需要通过静态@Bean方法显式注册PropertySourcesPlaceholderConfigurer。有关详细信息
    和示例，请参阅@Cons的javadocs的“使用外部化值”部分和@Bean的javadocs的“关于BeanFactoryPostProcessor返回@Bean方法的
    注释”部分。
---
	Resolving ${...} placeholders within @PropertySource resource locations:
		存在于@PropertySource资源位置的任何${...}占位符将根据已针对环境注册的属性源集合进行解析。如:
```java
 @Configuration
 @PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
 public class AppConfig {

     @Autowired
     Environment env;

     @Bean
     public TestBean testBean() {
         TestBean testBean = new TestBean();
         testBean.setName(env.getProperty("testbean.name"));
         return testBean;
     }
 }
```
		假设“my.placeholder”存在于已经注册的一个属性源中，例如系统属性或环境变量，占位符将被解析为相应的值。如果没有，则
    “default / path”将用作默认值。表示默认值（由冒号“：”分隔）是可选的。如果未指定缺省值且无法解析属性，则将抛出
    IllegalArgumentException。
---
	A note on property overriding with @PropertySource:
		如果给定的属性键存在于多个.properties文件中，则处理的最后一个@PropertySource批注将“赢”并覆盖。
        例如，给定两个属性文件a.properties和b.properties，请考虑以下两个使用@PropertySource注释引用它们的配置类：
```java
 @Configuration
 @PropertySource("classpath:/com/myco/a.properties")
 public class ConfigA { }

 @Configuration
 @PropertySource("classpath:/com/myco/b.properties")
 public class ConfigB { }
```
		覆盖顺序取决于这些类在应用程序上下文中注册的顺序。
```java
 AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
 ctx.register(ConfigA.class);
 ctx.register(ConfigB.class);
 ctx.refresh();
```
		在上面的场景中，b.properties中的属性将覆盖a.properties中存在的任何重复项，因为ConfigB最后已注册。
        在某些情况下，使用@PropertySource注释时严格控制属性源顺序可能是不可能或不切实际的。例如，如果上面的@Configuration类
    是通过组件扫描注册的，则排序很难预测。在这种情况下 - 如果覆盖很重要 - 建议用户回退使用编程的 PropertySource API。有关详细信息，
    请参阅ConfigurableEnvironment和 MutablePropertySources javadocs。
		注意：根据Java 8约定，此注释是可重复的。但是，所有这些@PropertySource注释都需要在同一级别声明：直接在配置类上或作为同一
    自定义注释中的元注释。不建议混合直接注释和元注释，因为直接注释将有效地覆盖元注释。
---
	属性:
    	1. String[] value:
			指示要加载的属性文件的资源位置。
            支持传统和基于XML的属性文件格式, 如: "classpath:/com/myco/app.properties" or "file:/path/to/file.xml".
            不允许使用资源位置通配符（例如** / *。属性）;每个位置必须只评估一个.properties资源。
            ${...}占位符将针对已在环境中注册的任何/所有财产来源解决。见上面的例子。
            每个位置将作为其自己的属性源添加到封闭环境中，并按声明的顺序添加。

		2. String encoding:
			给定资源的特定字符编码，例如“UTF-8”。

		3. Class<? extends PropertySourceFactory> factory:
			指定自定义PropertySourceFactory（如果有）。
            默认情况下，将使用标准资源文件的默认工厂。

		4. boolean ignoreResourceNotFound:
			指示是否应忽略找不到属性资源的错误。
            如果属性文件是完全可选的，则true为合适。默认值为false。

		5. String name:
			指示此属性源的名称。如果省略，将根据底层资源的描述生成名称。
## @Value 解读
		字段或方法/构造函数参数级别的注释，指示受影响参数的默认值表达式。
        通常用于表达式驱动的依赖注入。还支持动态解析处理程序方法参数，例如在Spring MVC中。
        常见用例是使用＃{systemProperties.myProp}样式表达式分配默认字段值。
        请注意，@Value注释的实际处理是由BeanPostProcessor执行的，这反过来意味着您不能在BeanPostProcessor或
    BeanFactoryPostProcessor类型中使用@Value。请参考javadoc获取AutowiredAnnotationBeanPostProcessor类（默认情况下，
    它会检查是否存在此批注）。
---
	属性:
    	1. String value:
			实际值表达式：例如＃{systemProperties.myProp}。
## @ImportResource 解读
		指示包含要导入的bean定义的一个或多个资源。
        与@Import一样，此批注提供的功能类似于 Spring XML中的<import />元素。它通常在将 @Configuration 类设计为由
    AnnotationConfigApplicationContext引导时使用，但仍需要某些 XML 功能（如命名空间）。
		默认情况下，如果以“.groovy”结尾，将使用GroovyBeanDefinitionReader处理value（）属性的参数; 否则，将使用
    XmlBeanDefinitionReader来解析Spring <beans /> XML文件。可选地，可以声明reader（）属性，允许用户选择自定义
    BeanDefinitionReader实现。
---
	属性:
		1. String[] locations:
			要导入的资源位置。
            支持资源加载前缀，如classpath：，file：等。
            有关如何处理资源的详细信息，请咨询Javadoc for reader（）。

		2. Class<? extends BeanDefinitionReader> reader:
			BeanDefinitionReader实现在处理通过value（）属性指定的资源时使用。
            默认情况下，阅读器将适应指定的资源路径：“。groovy”文件将使用GroovyBeanDefinitionReader处理;而所有其他资源都
        将使用XmlBeanDefinitionReader进行处理。

		3. String[] value:
			locations() 的别名。
## @Lazy 解读
		指示是否要延迟初始化bean。
        可以在任何直接或间接使用@Component注释的类或使用@Bean注释的方法上使用。
        如果@Component或@Bean定义中不存在此批注，则会发生急切初始化。如果存在并设置为true，则@Bean或@Component将不会被初始化，
    直到被另一个bean引用或从封闭的BeanFactory中显式检索。如果存在并设置为false，那么bean将在启动时由bean工厂实例化，这些工厂执行
    单例的初始化。
		如果@Configuration类中存在Lazy，则表示该@Configuration中的所有@Bean方法都应该被懒惰地初始化。如果在@Lazy-annotated
    @Configuration类中的@Bean方法中存在@Lazy并且为false，则表示覆盖'default lazy'行为并且应该急切地初始化bean。
		除了它的组件初始化角色之外，此注释还可以放在标有Autowired或Inject的注入点上：在该上下文中，它导致为所有受影响的依赖项创建
    惰性解析代理，作为使用ObjectFactory或Provider的替代方法。
---
	属性:
    	1. boolean value:
			是否应该进行延迟初始化。
## @ContextConfiguration 解读
		@ContextConfiguration定义类级元数据，用于确定如何为集成测试加载和配置ApplicationContext。
---
	Supported Resource Types:
    	在Spring 3.1之前，仅支持基于路径的资源位置（通常是XML配置文件）。从Spring 3.1开始，上下文加载器可以选择支持基于路径的
    资源或基于类的资源。从Spring 4.0.4开始，上下文加载器可以选择同时支持基于路径和基于类的资源。因此，@ContextConfiguration可用
    于声明基于路径的资源位置（通过locations（）或value（）属性）或带注释的类（通过classes（）属性）。但请注意，
    SmartContextLoader的大多数实现仅支持单一资源类型。从Spring 4.1开始，基于路径的资源位置可以是XML配置文件或Groovy脚本
    （如果Groovy在类路径上）。当然，第三方框架可以选择支持其他类型的基于路径的资源。
---
	Annotated Classes:
    	术语注释类可以指以下任何一种:
        	1. 用@Configuration注释的类
        	2. 一个组件（即用@ Component，@ Service，@ Repository等注释的类）
        	3. 一个JSR-330兼容类，使用javax.inject注释进行批注
        	4. 包含@ Bean方法的任何其他类
		有关注释类的配置和语义的更多信息，请参阅Javadoc以获取@Configuration和@Bean。
        从Spring Framework 4.0开始，此批注可用作元注释来创建自定义组合注释。
---
	属性:
    	1. Class<?>[] classes:
			用于加载ApplicationContext的带注释的类。
            查看AnnotationConfigContextLoader.detectDefaultConfigurationClasses（）的javadoc，了解如果未指定带注释
        的类，将如何检测默认配置类的详细信息。有关默认加载器的更多详细信息，请参阅loader（）的文档。

		2. boolean inheritInitializers:
			是否应继承来自测试超类的上下文初始值设定项。
            默认值是true。这意味着带注释的类将继承由测试超类定义的应用程序上下文初始值设定项。具体而言，给定测试类的初始化程序将
        添加到由测试超类定义的初始化程序集中。因此，子类可以选择扩展初始化器集。
        	如果inheritInitializers设置为false，则带注释的类的初始值设定项将隐藏并有效替换超类定义的任何初始值设定项。
            在以下示例中，将使用BaseInitializer和ExtendedInitializer初始化ExtendedTest的ApplicationContext。但请注意，
        调用初始值设定项的顺序取决于它们是实现Ordered还是使用@Order注释。
```java
 @ContextConfiguration(initializers = BaseInitializer.class)
 public class BaseTest {
     // ...
 }

 @ContextConfiguration(initializers = ExtendedInitializer.class)
 public class ExtendedTest extends BaseTest {
     // ...
 }
```
		3. boolean inheritLocations:
			是否应继承资源位置或来自测试超类的注释类。
            默认值是true。这意味着带注释的类将继承由测试超类定义的资源位置或带注释的类。具体而言，给定测试类的资源位置或带注释的类
        将附加到由测试超类定义的资源位置或带注释的类的列表中。因此，子类可以选择扩展资源位置列表或带注释的类。
        	如果inheritLocations设置为false，则带注释的类的资源位置或带注释的类将隐藏并有效地替换由超类定义的任何资源位置或带
        注释的类。
			在以下使用基于路径的资源位置的示例中，ExtendedTest的ApplicationContext将按此顺序从
        “base-context.xml”和“extended-context.xml”加载。因此，在“extended-context.xml”中定义的Bean可能会覆盖
        “base-context.xml”中定义的Bean。
```java
 @ContextConfiguration("base-context.xml")
 public class BaseTest {
     // ...
 }

 @ContextConfiguration("extended-context.xml")
 public class ExtendedTest extends BaseTest {
     // ...
 }
```
			类似地，在以下使用带注释的类的示例中，将按顺序从BaseConfig和ExtendedConfig配置类加载ExtendedTest的
        ApplicationContext。因此，ExtendedConfig中定义的Bean可能会覆盖BaseConfig中定义的Bean。
```java
 @ContextConfiguration(classes=BaseConfig.class)
 public class BaseTest {
     // ...
 }

 @ContextConfiguration(classes=ExtendedConfig.class)
 public class ExtendedTest extends BaseTest {
     // ...
 }
```
		4. Class<? extends ApplicationContextInitializer<?>>[] initializers:
			用于初始化ConfigurableApplicationContext的应用程序上下文初始化程序类。
            每个声明的初始化程序支持的具体ConfigurableApplicationContext类型必须与正在使用的SmartContextLoader创建的
        ApplicationContext类型兼容。
			SmartContextLoader实现通常检测是否已实现Spring的Ordered接口，或者是否存在@Order注释，并在调用它们之前相应地
        对实例进行排序。

		5. Class<? extends ContextLoader> loader:
			用于加载ApplicationContext的SmartContextLoader（或ContextLoader）的类型。
            如果未指定，则将从第一个使用@ContextConfiguration注释的超类继承加载器，并指定显式加载器。如果层次结构中没有类指定
        显式加载器，则将使用默认加载器。
			在运行时选择的默认具体实现将是DelegatingSmartContextLoader或WebDelegatingSmartContextLoader，具体取决于
        @WebAppConfiguration的缺失或存在。有关各种具体 SmartContextLoader 的默认行为的更多详细信息，请查看Javadoc for
        AbstractContextLoader，GenericXmlContextLoader，GenericGroovyXmlContextLoader，
        AnnotationConfigContextLoader，GenericXmlWebContextLoader，GenericGroovyXmlWebContextLoader和
        AnnotationConfigWebContextLoader。

		6. String[] locations:
			用于加载ApplicationContext的资源位置。
            查看用于AbstractContextLoader.modifyLocations（）的Javadoc，了解有关如何在运行时解释位置的详细信息，特别是
        在相对路径的情况下。另外，请查看有关AbstractContextLoader.generateDefaultLocations（）的文档，以获取有关在未指定
        任何内容时将使用的默认位置的详细信息。
        	请注意，上述默认规则仅适用于标准的AbstractContextLoader子类，例如GenericXmlContextLoader或
        GenericGroovyXmlContextLoader，它们是在运行时使用的有效默认实现（如果已配置位置）。有关默认加载器的更多详细信息，
        请参阅loader（）的文档。
        	此属性不能与value（）一起使用，但可以使用它来代替value（）。

		7. String name:
			此配置表示的上下文层次结构级别的名称。
            如果未指定，将根据层次结构中所有已声明的上下文中的数字级别推断名称。
            此属性仅在使用@ContextHierarchy配置的测试类层次结构中使用时才适用，在这种情况下，该名称可用于在超类中定义的层次结构
        级别中使用相同名称的配置来合并或覆盖此配置。有关详细信息，请参阅Javadoc以获取@ContextHierarchy。

		8. String[] value:
			locations() 别名.
            此属性不能与locations（）一起使用，但可以使用它来代替locations（）。
## @EnableAsync 解读
		启用Spring的异步方法执行功能，类似于Spring的<task：*> XML命名空间中的功能。
        要与@Configuration类一起使用，如下所示，为整个Spring应用程序上下文启用注释驱动的异步处理：
```java
 @Configuration
 @EnableAsync
 public class AppConfig {

 }
```
		MyAsyncBean是一个用户定义的类型，其中一个或多个方法使用Spring的@Async批注，EJB 3.1 @ javax.ejb.Asynchronous批注
    或通过annotation（）属性指定的任何自定义批注进行批注。透明地为任何已注册的bean添加方面，例如通过此配置：
```java
 @Configuration
 public class AnotherAppConfig {

     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }
 }
```
		默认情况下，Spring将搜索关联的线程池定义：要么是上下文中的唯一TaskExecutor bean，要么是名为“taskExecutor”的
    Executor bean。如果两者都不可解析，则将使用SimpleAsyncTaskExecutor处理异步方法调用。此外，具有void返回类型的带注释的方法
    不能将任何异常发送回调用者。默认情况下，仅记录此类未捕获的异常。
		要自定义所有这些，请实现AsyncConfigurer并提供：
        	1. 你自己的Executor通过getAsyncExecutor（）方法，
        	2. 您通过getAsyncUncaughtExceptionHandler（）方法获得自己的AsyncUncaughtExceptionHandler。
		注意：AsyncConfigurer配置类在应用程序上下文引导程序的早期初始化。如果你需要对其他bean有任何依赖，请确保尽可能地声明它们
    “lazy”，以便让它们通过其他后处理器。
```java
 @Configuration
 @EnableAsync
 public class AppConfig implements AsyncConfigurer {

     @Override
     public Executor getAsyncExecutor() {
         ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
         executor.setCorePoolSize(7);
         executor.setMaxPoolSize(42);
         executor.setQueueCapacity(11);
         executor.setThreadNamePrefix("MyExecutor-");
         executor.initialize();
         return executor;
     }

     @Override
     public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
         return new MyAsyncUncaughtExceptionHandler();
     }
 }
```
		如果只需要自定义一个项目，则可以返回null以保留默认设置。考虑尽可能从AsyncConfigurerSupport扩展。
		注意：在上面的示例中，ThreadPoolTaskExecutor不是完全托管的Spring bean。如果需要完全托管的bean，请将@Bean批注添加到
    getAsyncExecutor（）方法。在这种情况下，不再需要手动调用executor.initialize（）方法，因为这将在初始化bean时自动调用。
		作为参考，可以将上面的示例与以下Spring XML配置进行比较：
```xml
 <beans>

     <task:annotation-driven executor="myExecutor" exception-handler="exceptionHandler"/>

     <task:executor id="myExecutor" pool-size="7-42" queue-capacity="11"/>

     <bean id="asyncBean" class="com.foo.MyAsyncBean"/>

     <bean id="exceptionHandler" class="com.foo.MyAsyncUncaughtExceptionHandler"/>

 </beans>
```
		除了设置Executor的线程名前缀外，上述基于XML和JavaConfig的示例是等效的。这是因为<task：executor>元素不公开这样的属性。
    这演示了基于JavaConfig的方法如何通过直接访问实际组件来实现最大的可配置性。
		mode（）属性控制如何应用建议：如果模式为AdviceMode.PROXY（默认值），则其他属性控制代理的行为。请注意，代理模式仅允许通过
    代理拦截呼叫;同一类中的本地调用不能以这种方式截获。
    	请注意，如果mode（）设置为AdviceMode.ASPECTJ，则将忽略proxyTargetClass（）属性的值。另请注意，在这种情况下，
    spring-aspects模块JAR必须存在于类路径中，编译时编织或加载时编织将方面应用于受影响的类。在这种情况下没有涉及代理人;本地电话也会
	被截获。
---
	属性:
    	1. Class<? extends Annotation> annotation:
			指示要在类或方法级别检测的“异步”注释类型。
            默认情况下，将检测Spring的@Async 注释和EJB 3.1 @ javax.ejb.Asynchronous注释。
			存在此属性，以便开发人员可以提供自己的自定义注释类型，以指示应异步调用方法（或给定类的所有方法）。

		2. AdviceMode mode:
			指出应如何应用异步通知。
            默认值为AdviceMode.PROXY。请注意，代理模式仅允许通过代理拦截呼叫。同一类中的本地调用不能以这种方式截获;由于Spring
        的拦截器甚至没有为这样的运行时场景启动，因此将忽略本地调用中此类方法的异步注释。对于更高级的拦截模式，请考虑将其切换为
        AdviceMode.ASPECTJ。

		3. int order:
			指示应该应用 AsyncAnnotationBeanPostProcessor 的顺序。
            默认值为Ordered.LOWEST_PRECEDENCE，以便在所有其他后处理器之后运行，以便它可以向现有代理添加顾问程序而不是双代理。

		4. boolean proxyTargetClass:
			指示是否要创建基于子类的（CGLIB）代理而不是基于标准Java接口的代理。
            仅当mode（）设置为AdviceMode.PROXY时才适用。
            默认值为false。
            请注意，将此属性设置为true将影响所有需要代理的Spring托管bean，而不仅仅是那些标记为@Async的bean。例如，标有Spring
        的@Transactional 注释的其他bean将同时升级为子类代理。除非有人明确期待一种代理与另一种代理，否则这种方法在实践中没有负面
        影响.
## @EnableScheduling 解读
		启用Spring的计划任务执行功能，类似于Spring的<task：*> XML名称空间中的功能。要在@Configuration类上使用如下：
```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     // various @Bean definitions
 }
```
		这使得能够在容器中的任何Spring管理的bean上检测@Scheduled注释。例如，给定一个MyTask类
```java
 package com.myco.tasks;

 public class MyTask {

     @Scheduled(fixedRate=1000)
     public void work() {
         // task execution logic
     }
 }
```
		以下配置将确保每1000毫秒调用一次MyTask.work（）：
```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     @Bean
     public MyTask task() {
         return new MyTask();
     }
 }
```
		或者，如果MyTask使用@Component注释，则以下配置将确保以所需的时间间隔调用其@Scheduled方法：
```java
 @Configuration
 @EnableScheduling
 @ComponentScan(basePackages="com.myco.tasks")
 public class AppConfig {
 }
```
		使用@Scheduled注释的方法甚至可以直接在@Configuration类中声明：
```java
 @Configuration
 @EnableScheduling
 public class AppConfig {

     @Scheduled(fixedRate=1000)
     public void work() {
         // task execution logic
     }
 }
```
		默认情况下，将搜索关联的调度程序定义：上下文中的唯一TaskScheduler bean，或者另一个名为“taskScheduler”的
    TaskScheduler bean;也将对ScheduledExecutorService bean执行相同的查找。如果两者都不可解析，则将在注册器中创建并使用本地
    单线程默认调度程序。
		当需要更多控制时，@ Configuration类可以实现SchedulingConfigurer。这允许访问底层的ScheduledTaskRegistrar实例。
    例如，以下示例演示如何自定义用于执行计划任务的Executor：
```java
 @Configuration
 @EnableScheduling
 public class AppConfig implements SchedulingConfigurer {

     @Override
     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
         taskRegistrar.setScheduler(taskExecutor());
     }

     @Bean(destroyMethod="shutdown")
     public Executor taskExecutor() {
         return Executors.newScheduledThreadPool(100);
     }
 }
```
		请注意上面的例子中使用@Bean（destroyMethod =“shutdown”）。这可确保在Spring应用程序上下文本身关闭时正确关闭任务执行
    程序。
		实现SchedulingConfigurer还允许通过ScheduledTaskRegistrar对任务注册进行细粒度控制。例如，以下配置根据自定义Trigger
    实现执行特定bean方法：
```java
 @Configuration
 @EnableScheduling
 public class AppConfig implements SchedulingConfigurer {

     @Override
     public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
         taskRegistrar.setScheduler(taskScheduler());
         taskRegistrar.addTriggerTask(
             new Runnable() {
                 public void run() {
                     myTask().work();
                 }
             },
             new CustomTrigger()
         );
     }

     @Bean(destroyMethod="shutdown")
     public Executor taskScheduler() {
         return Executors.newScheduledThreadPool(42);
     }

     @Bean
     public MyTask myTask() {
         return new MyTask();
     }
 }
```
		作为参考，可以将上面的示例与以下Spring XML配置进行比较：
```xml
 <beans>

     <task:annotation-driven scheduler="taskScheduler"/>

     <task:scheduler id="taskScheduler" pool-size="42"/>

     <task:scheduled-tasks scheduler="taskScheduler">
         <task:scheduled ref="myTask" method="work" fixed-rate="1000"/>
     </task:scheduled-tasks>

     <bean id="myTask" class="com.foo.MyTask"/>

 </beans>
```
		这些示例是等效的，除了在XML中使用固定速率周期而不是自定义触发器实现;这是因为任务：命名空间调度不能轻易暴露这样的支持。这只是
    一个演示，即基于代码的方法如何通过直接访问实际组件来实现最大的可配置性。
		注意：@EnableScheduling仅适用于其本地应用程序上下文，允许在不同级别选择性地安排bean。请在每个单独的上下文中重新声明
    @EnableScheduling，例如公共根Web应用程序上下文和任何单独的DispatcherServlet应用程序上下文，如果您需要在多个级别应用其行为。
## @EnableTransactionManagement 解读
		启用Spring的注释驱动的事务管理功能，类似于Spring的<tx：*> XML命名空间中的支持。要在@Configuration类上使用如下：
```java
 @Configuration
 @EnableTransactionManagement
 public class AppConfig {

     @Bean
     public FooRepository fooRepository() {
         // configure and return a class having @Transactional methods
         return new JdbcFooRepository(dataSource());
     }

     @Bean
     public DataSource dataSource() {
         // configure and return the necessary JDBC DataSource
     }

     @Bean
     public PlatformTransactionManager txManager() {
         return new DataSourceTransactionManager(dataSource());
     }
 }
```
		作为参考，可以将上面的示例与以下Spring XML配置进行比较：
```xml
 <beans>

     <tx:annotation-driven/>

     <bean id="fooRepository" class="com.foo.JdbcFooRepository">
         <constructor-arg ref="dataSource"/>
     </bean>

     <bean id="dataSource" class="com.vendor.VendorDataSource"/>

     <bean id="transactionManager" class="org.sfwk...DataSourceTransactionManager">
         <constructor-arg ref="dataSource"/>
     </bean>

 </beans>
```
		在上面的两个场景中，@EnableTransactionManagement和<tx：annotation-driven />负责注册为注释驱动的事务管理提供支持的
    必要Spring组件，例如 TransactionInterceptor 和基于代理或基于AspectJ的建议。调用JdbcFooRepository的@Transactional
	方法时，拦截器进入调用堆栈。
		两个示例之间的细微差别在于PlatformTransactionManager bean的命名：在@Bean情况下，名称为“txManager”(根据方法的名称);
    在XML情况下，名称是“transactionManager”。<tx：annotation-driven />是硬连接的，默认情况下查找名为
    “transactionManager”的bean，但@EnableTransactionManagement更灵活;它将回退到容器中任何 PlatformTransactionManager
    bean的类型查找。因此，名称可以是“txManager”，“transactionManager”或“tm”：它无关紧要。
		对于那些希望在@EnableTransactionManagement和要使用的确切事务管理器bean之间建立更直接关系的人，可以实现
    TransactionManagementConfigurer回调接口 - 注意下面的implements子句和@Override-annotated方法：
```java
 @Configuration
 @EnableTransactionManagement
 public class AppConfig implements TransactionManagementConfigurer {

     @Bean
     public FooRepository fooRepository() {
         // configure and return a class having @Transactional methods
         return new JdbcFooRepository(dataSource());
     }

     @Bean
     public DataSource dataSource() {
         // configure and return the necessary JDBC DataSource
     }

     @Bean
     public PlatformTransactionManager txManager() {
         return new DataSourceTransactionManager(dataSource());
     }

     @Override
     public PlatformTransactionManager annotationDrivenTransactionManager() {
         return txManager();
     }
 }
```
		这种方法可能只是因为它更明确，或者为了区分同一容器中存在的两个PlatformTransactionManager bean而可能是必要的。
    顾名思义，annotationDrivenTransactionManager（）将用于处理@Transactional方法。有关更多详细信息，请参阅
    TransactionManagementConfigurer Javadoc。
		mode（）属性控制如何应用建议：如果模式为AdviceMode.PROXY（默认值），则其他属性控制代理的行为。请注意，代理模式仅允许
    通过代理拦截呼叫;同一类中的本地调用不能以这种方式截获。
		请注意，如果mode（）设置为AdviceMode.ASPECTJ，则将忽略proxyTargetClass（）属性的值。另请注意，在这种情况下，
    spring-aspects模块JAR必须存在于类路径中，编译时编织或加载时编织将方面应用于受影响的类。在这种情况下没有涉及代理人;
    本地电话也会被截获。
---
	属性:
    	1. AdviceMode mode:
			说明应如何应用交易建议。
            默认值为AdviceMode.PROXY。请注意，代理模式仅允许通过代理拦截呼叫。同一类中的本地调用不能以这种方式截获;由于Spring
        的拦截器甚至没有为这样的运行时场景提供支持，因此将忽略本地调用中此类方法的事务性注释。对于更高级的拦截模式，请考虑将其切换
        为AdviceMode.ASPECTJ。

		2. int order:
			指示在特定连接点应用多个建议时执行事务顾问的顺序。
            默认值为Ordered.LOWEST_PRECEDENCE。

        3. boolean proxyTargetClass:
			指示是否要创建基于子类的（CGLIB）代理（true），而不是基于标准Java接口的代理（false）。默认值为false。仅当mode（）
        设置为AdviceMode.PROXY时才适用。
        	请注意，将此属性设置为true将影响所有需要代理的Spring托管bean，而不仅仅是那些标记为@Transactional的bean。例如，
        标有Spring的@Async注释的其他bean将同时升级为子类代理。这种方法在实践中没有负面影响，除非有人明确期望一种类型的代理与
        另一种代理相比，例如在测试中。
## @EnableAspectJAutoProxy 解读
		支持处理使用AspectJ的@Aspect注释标记的组件，类似于Spring的<aop：aspectj-autoproxy> XML元素中的功能。要在
    @Configuration类上使用如下：
```java
 @Configuration
 @EnableAspectJAutoProxy
 public class AppConfig {

     @Bean
     public FooService fooService() {
         return new FooService();
     }

     @Bean
     public MyAspect myAspect() {
         return new MyAspect();
     }
 }
```
		FooService是典型的POJO组件，而MyAspect是@ Aspect风格的方面：
```java
 public class FooService {

     // various methods
 }

 @Aspect
 public class MyAspect {

     @Before("execution(* FooService+.*(..))")
     public void advice() {
         // advise FooService methods as appropriate
     }
 }
```
		在上面的场景中，@EnableAspectJAutoProxy确保正确处理MyAspect，并将FooService代理混合在它提供的建议中。
		用户可以使用proxyTargetClass（）属性控制为FooService创建的代理类型。以下是启用CGLIB样式的“子类”代理，而不是基于默认
    接口的JDK代理方法。
```java
 @Configuration
 @EnableAspectJAutoProxy(proxyTargetClass=true)
 public class AppConfig {
     // ...
 }
```
		请注意，@Aspect bean可以像任何其他bean一样进行组件扫描。只需使用@Aspect和@Component标记方面：
```java
 package com.foo;

 @Component
 public class FooService { ... }

 @Aspect
 @Component
 public class MyAspect { ... }
```
		然后使用@ComponentScan注释来同时选择：
```java
 @Configuration
 @ComponentScan("com.foo")
 @EnableAspectJAutoProxy
 public class AppConfig {

     // no explicit @Bean definitions required
 }
```
		注意：@EnableAspectJAutoProxy仅适用于其本地应用程序上下文，允许在不同级别选择性地代理bean。请在每个单独的上下文中重新
    声明@EnableAspectJAutoProxy，例如公共根Web应用程序上下文和任何单独的DispatcherServlet应用程序上下文，如果您需要在多个级
    别应用其行为。
    	此功能需要在类路径上存在aspectjweaver。虽然这种依赖关系对于spring-aop来说是可选的，但它是@EnableAspectJAutoProxy及
    其底层设施所必需的。
---
	属性:
    	1. boolean exposeProxy:
			指示代理应由AOP框架作为ThreadLocal公开，以便通过AopContext类进行检索。默认情况下关闭，即不保证AopContext访问将
        起作用。

		2. boolean proxyTargetClass:
			指示是否要创建基于子类的（CGLIB）代理而不是基于标准Java接口的代理。默认值为false。
## @EnableWebMvc 解读
		将此批注添加到@Configuration类可从WebMvcConfigurationSupport导入Spring MVC配置，例如：
```java
 @Configuration
 @EnableWebMvc
 @ComponentScan(basePackageClasses = MyConfiguration.class)
 public class MyConfiguration {

 }
```
		要自定义导入的配置，请实现接口WebMvcConfigurer并覆盖单个方法，例如：
```java
 @Configuration
 @EnableWebMvc
 @ComponentScan(basePackageClasses = MyConfiguration.class)
 public class MyConfiguration implements WebMvcConfigurer {

           @Override
           public void addFormatters(FormatterRegistry formatterRegistry) {
				formatterRegistry.addConverter(new MyConverter());
           }

           @Override
           public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
         		converters.add(new MyHttpMessageConverter());
           }

 }
```
		注意：只有一个@Configuration类可以使用@EnableWebMvc注释来导入Spring Web MVC配置。但是，可以有多个@Configuration
    类实现WebMvcConfigurer以自定义提供的配置。
    	如果WebMvcConfigurer没有公开需要配置的更高级设置，请考虑删除@EnableWebMvc注释并直接从WebMvcConfigurationSupport或
    DelegatingWebMvcConfiguration扩展，例如：
```java
 @Configuration
 @ComponentScan(basePackageClasses = { MyConfiguration.class })
 public class MyConfiguration extends WebMvcConfigurationSupport {

           @Override
           public void addFormatters(FormatterRegistry formatterRegistry) {
         		formatterRegistry.addConverter(new MyConverter());
           }

           @Bean
           public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
         		// Create or delegate to "super" to create and
         		// customize properties of RequestMappingHandlerAdapter
           }
 }
```
## @Scope 解读
		当与@Component一起用作类型级别注释时，@Scope指示用于注释类型实例的范围的名称。
        当与@Bean一起用作方法级注释时，@Scope指示用于从方法返回的实例的范围的名称。
        注意：@Scope注释仅在具体bean类（对于带注释的组件）或工厂方法（对于@Bean方法）上进行内省。与XML bean定义相比，没有bean
    定义继承的概念，类级别的继承层次结构与元数据目的无关。
    	在此上下文中，范围表示实例的生命周期，例如单例，原型等。可以使用ConfigurableBeanFactory和WebApplicationContext接口
    中提供的SCOPE_ *常量来引用Spring中提供的开箱即用的范围。
    	要注册其他自定义作用域，请参阅CustomScopeConfigurer。
---
	属性:
    	1. ScopedProxyMode proxyMode:
			指定是否应将组件配置为作用域代理，如果是，则指定代理应基于接口还是基于子类。
            默认为ScopedProxyMode.DEFAULT，它通常表示除非在组件扫描指令级别配置了不同的默认值，否则不应创建范围代理。
            类似于Spring XML中的<aop：scoped-proxy />支持。

		2. String scopeName:
			指定要用于带注释的组件/ bean的作用域的名称。

		3. String value:
			scopeName()的别名.
## @Order 解读
		@Order定义带注释的组件的排序顺序。
        value（）是可选的，表示Ordered接口中定义的订单值。较低的值具有较高的优先级默认值为Ordered.LOWEST_PRECEDENCE，表示
    最低优先级（丢失到任何其他指定的订单值）。
    	注意：自Spring 4.0以来，Spring中的多种组件都支持基于注释的排序，即使对于目标组件的订单值（来自其目标类或来自其@Bean方法）
    的集合注入也是如此。虽然此类订单值可能影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和@DependsOn声明
    （影响运行时确定的依赖关系图）确定的正交关注点。
		从Spring 4.1开始，标准优先级注释可用作订购方案中此注释的替代品。请注意，当必须选择单个元素时，@ Priority可能具有其他语义
    （请参阅AnnotationAwareOrderComparator.getPriority（java.lang.Object））。
    	或者，也可以通过Ordered接口在每个实例的基础上确定订单值，允许配置确定的实例值而不是附加到特定类的硬编码值。
        有关非有序对象的排序语义的详细信息，请参阅OrderComparator的javadoc。
---
	属性:
    	1. int value:
			order 值.
            默认值为Ordered.LOWEST_PRECEDENCE。
## @Qualifier 解读
		在自动装配时，此注释可以在字段或参数上用作候选bean的限定符。它还可以用于注释其他自定义注释，然后可以将其用作限定符。
---
	属性:
    	1. String value:
			需要装配的 beanName;默认值为 "".
## @ComponentScan.Filter 解读
		声明要用作包含过滤器或排除过滤器的类型过滤器。
---
	属性:
    	1. Class<?>[] classes:
			要用作过滤器的类。
            下表说明了如何根据type（）属性的配置值解释类。
| FilterType |Class Interpreted As|
|------------|--------------------|
| ANNOTATION |     注解本身        |
| ASSIGNABLE_TYPE | 检测到组件的类型应该可以分配给|
| CUSTOM | TypeFilter 的实现 |
			指定多个类时，将应用OR逻辑 - 例如，“包含使用@Foo OR @Bar注释的类型”。
            Custom TypeFilters可以选择实现以下任何Aware接口，并在匹配之前调用它们各自的方法：
            	1. EnvironmentAware
            	2. BeanFactoryAware
            	3. BeanClassLoaderAware
            	4. ResourceLoaderAware
			允许指定零类，但不会影响组件扫描。

		2. String[] pattern:
			用于过滤器的模式（或模式），作为指定Class值（）的替代方法。
            如果type（）设置为ASPECTJ，则这是一个AspectJ类型模式表达式。如果type（）设置为REGEX，则这是要匹配的完全限定类名
        的正则表达式模式。

		3. FilterType type:
			要使用的过滤器类型。
            默认是 FilterType.ANNOTATION.

		4. Class<?>[] value:
			classes() 的别名.
## @WebAppConfiguration 解读
		@WebAppConfiguration是一个类级别注释，用于声明为集成测试加载的ApplicationContext应该是WebApplicationContext。
        测试类上存在@WebAppConfiguration指示应使用Web应用程序根路径的默认值为测试加载WebApplicationContext。要覆盖默认值，
    请通过value（）属性指定显式资源路径。
    	请注意，@WebAppConfiguration必须与@ContextConfiguration结合使用，可以在单个测试类中，也可以在测试类层次结构中使用。
        从Spring Framework 4.0开始，此批注可用作元注释来创建自定义组合注释。
---
	属性:
    	1. String value:
			Web应用程序根目录的资源路径。
            不包含Spring资源前缀的路径（例如，classpath：，file：等）将被解释为文件系统资源，并且路径不应以斜杠结尾。
            默认为“src / main / webapp”作为文件系统资源。请注意，这是项目中Web应用程序根目录的标准目录，该目录遵循WAR的
        标准Maven项目布局。
## @Async 解读
		将方法标记为异步执行候选的注释。也可以在类型级别使用，在这种情况下，所有类型的方法都被视为异步。
        就目标方法签名而言，支持任何参数类型。但是，返回类型被限制为void或Future。在后一种情况下，您可以声明更具体的
    ListenableFuture或CompletableFuture类型，这些类型允许与异步任务进行更丰富的交互，并通过进一步的处理步骤立即组合。
    	从代理返回的Future句柄将是一个实际的异步Future，可用于跟踪异步方法执行的结果。但是，由于目标方法需要实现相同的签名，
    因此必须返回一个临时的Future句柄，该句柄只传递一个值：Spring的AsyncResult，EJB 3.1的AsyncResult或
    CompletableFuture.completedFuture（Object）。
---
	属性:
    	1. String value:
			指定异步操作的限定符值。
            可用于确定执行此方法时要使用的目标执行程序，匹配特定Executor或TaskExecutor bean定义的限定符值（或bean名称）。
            在类级别@Async注释上指定时，表示给定的执行程序应该用于类中的所有方法。方法级别使用Async＃值始终覆盖在类级别设置的
        任何值。
## @Transactional 解读
		描述单个方法或类上的事务属性。
        在类级别，此批注作为默认应用于声明类及其子类的所有方法。请注意，它不适用于类层次结构中的祖先类;方法需要在本地重新声明才能
    参与子类级别的注释。
		这个注释类型通常可以直接与Spring的RuleBasedTransactionAttribute类相比，实际上
    AnnotationTransactionAttributeSource将直接将数据转换为后一个类，因此Spring的事务支持代码不必知道注释。如果没有规则与
    异常相关，则将其视为DefaultTransactionAttribute（在RuntimeException和Error上回滚但不在已检查的异常上回滚）。
		有关此批注的属性的语义的特定信息，请参阅TransactionDefinition和TransactionAttribute javadocs。
---
	属性:
    	1. Isolation isolation:
			事务隔离级别。
            默认为Isolation.DEFAULT。
            专门设计用于Propagation.REQUIRED或Propagation.REQUIRES_NEW，因为它仅适用于新启动的事务。如果您希望隔离级别声明
        在参与具有不同隔离级别的现有事务时被拒绝，请考虑在事务管理器上将“validateExistingTransactions”标志切换为“true”。

		2. Class<? extends Throwable>[] noRollbackFor:
			定义零（0）或更多异常类，它们必须是Throwable的子类，指示哪些异常类型不得导致事务回滚。
            这是构造回滚规则的首选方法（与noRollbackForClassName（）相反），匹配异常类及其子类。
            与NoRollbackRuleAttribute.NoRollbackRuleAttribute（Class clazz）类似。

		3. String[] noRollbackForClassName:
			定义零（0）或更多异常名称（对于必须是Throwable的子类的异常），指示哪些异常类型不得导致事务回滚。
            有关如何处理指定名称的详细信息，请参阅rollbackForClassName（）的说明。
            与NoRollbackRuleAttribute.NoRollbackRuleAttribute（String exceptionName）类似。

		4. Propagation propagation:
			事务传播类型。
            默认为Propagation.REQUIRED。

		5. boolean readOnly:
			如果事务是有效只读的，则可以设置为true的布尔标志，允许在运行时进行相应的优化。
            默认为false。
            这仅仅是实际事务子系统的提示;它不一定会导致写访问尝试失败。无法解释只读提示的事务管理器在被要求进行只读事务时不会抛出
        异常，而是默默地忽略提示。

		6. Class<? extends Throwable>[] rollbackFor:
			定义零（0）或更多异常类，它们必须是Throwable的子类，指示哪些异常类型必须导致事务回滚。
            默认情况下，事务将在RuntimeException和Error上回滚，但不会在已检查的异常（业务异常）上回滚。有关详细说明，请参见
        DefaultTransactionAttribute.rollbackOn（Throwable）。
        	这是构造回滚规则（与rollbackForClassName（）相反）的首选方法，匹配异常类及其子类。
            与RollbackRuleAttribute.RollbackRuleAttribute（Class clazz）类似。

		7. String[] rollbackForClassName:
			定义零（0）或更多异常名称（对于必须是Throwable的子类的异常），指示哪些异常类型必须导致事务回滚。
            这可以是完全限定类名的子字符串，目前没有通配符支持。例如，值“ServletException”将匹配
        javax.servlet.ServletException及其子类。
        	注意：仔细考虑模式的具体程度以及是否包含包信息（这不是强制性的）。例如，“异常”几乎可以匹配任何内容，并且可能隐藏其他
        规则。如果“异常”用于为所有已检查的异常定义规则，则“java.lang.Exception”将是正确的。使用更多不寻常的异常名称，
        例如“BaseBusinessException”，不需要使用FQN。
        	与RollbackRuleAttribute.RollbackRuleAttribute（String exceptionName）类似。

		8. int timeout:
			此事务的超时（以秒为单位）。
            默认为基础事务系统的默认超时。
            专门设计用于Propagation.REQUIRED或Propagation.REQUIRES_NEW，因为它仅适用于新启动的事务。

		9. String transactionManager:
			指定事务的限定符值。
            可用于确定目标事务管理器，匹配特定PlatformTransactionManager bean定义的限定符值（或bean名称）。

		10. String value:
			transactionManager() 的别名.


## @Scheduled 解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
        
        
        
        
        
        
        
        
        
        
        
   