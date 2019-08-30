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
## @PropertySource 解读
## @Value 解读
## @ImportResource 解读
## @Lazy 解读
## @ContextConfiguration 解读
## @EnableAsync 解读
## @EnableScheduling 解读
## @EnableTransactionManagement 解读
## @EnableAspectJAutoProxy 解读
## @EnableWebMvc 解读
## @Scope 解读
## @EnableScheduling 解读
## @Order 解读
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
## @解读
## @解读
## @解读
## @解读
## @解读
## @解读
        
        
        
        
        
        
        
        
        
        
        
   