[TOC]

## @Configuration 解读
		表示一个类声明了一个或多个 @Bean 方法，并且可以由 Spring 容器处理，以便在运行时为这些 bean 生成 bean 定义和
    服务请求，如:
```java
@Configuration
public class AppConfig {

 @Bean
 public MyBean myBean() {
     // instantiate, configure and return bean ...
 }
}
```
	Bootstrapping @Configuration classes:
    	Via AnnotationConfigApplicationContext:
        	@Configuration 类通常使用 AnnotationConfigApplicationContext 或其支持 Web 的变体
        AnnotationConfigWebApplicationContext 进行引导。前者的一个简单示例如下：
```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.register(AppConfig.class);
ctx.refresh();
MyBean myBean = ctx.getBean(MyBean.class);
// use myBean ...
```
			有关更多详细信息，请参阅 AnnotationConfigApplicationContext javadocs，有关 Servlet 容器中的Web配置
        说明，	请参阅 AnnotationConfigWebApplicationContext。

		Via Spring <beans> XML:
        	作为直接针对 AnnotationConfigApplicationContext 注册 @Configuration 类的替代方法，可以将 
        @Configuration 类声明为 Spring XML 文件中的普通 <bean> 定义：
```xml
<beans>
    <context:annotation-config/>
    <bean class="com.acme.AppConfig"/>
</beans>
```
			在上面的示例中，需要 <context：annotation-config /> 以启用 ConfigurationClassPostProcessor 和其他
        与 @Configuration 注解相关的后处理器便于处理类。

		Via component scanning:
        	@Configuration 使用 @Component 元注解进行声明，因此 @Configuration 类是组件扫描的候选者（通常使用
        Spring XML的 <context：component-scan /> 元素），因此也可以像任何常规 @Component 一样利用
    @Autowired/@Inject 零件。特别是，如果存在单个构造函数，则将为该构造函数透明地应用自动装配语义：
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
---
	Working with externalized values:
    	Using the Environment API:
			可以通过将Spring环境注入@Configuration类来查找外部化值 - 例如，使用@Autowired注释：
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
			通过Environment解析的属性驻留在一个或多个“属性源”对象中，@ Configuration配置类可以使用@PropertySource
        批注向Environment对象提供属性源：
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
			有关更多详细信息，请参阅Environment和@PropertySource javadocs。

		Using the @Value annotation:
			可以使用@Value注释将外化值注入@Configuration类：
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
```
			这种方法通常与Spring的PropertySourcesPlaceholderConfigurer一起使用，可以通过
        <context：property-placeholder />在XML配置中自动启用，或者通过专用的静态@Bean方法在@Configuration类中
        显式启用（请参阅“关于BeanFactoryPostProcessor返回的注释”）@Bean方法“@ Bean的javadocs详细信息）。但是，
        请注意，通常只有在需要自定义配置（如占位符语法等）时才需要通过静态@Bean方法显式注册
        PropertySourcesPlaceholderConfigurer。具体来说，如果没有bean后处理器（例如
        PropertySourcesPlaceholderConfigurer）已注册作为ApplicationContext的嵌入式值解析器，Spring将注册一个
        默认的嵌入式值解析器，该解析器根据在Environment中注册的属性源来解析占位符。请参阅下面有关使用@ImportResource
        使用Spring XML编写@Configuration类的部分;看@Value javadocs;有关使用BeanFactoryPostProcessor类型
        (如PropertySourcesPlaceholderConfigurer)的详细信息，请参阅@Bean javadocs。
---
    Composing @Configuration classes:
    	With the @Import annotation:
			@Configuration类可以使用@Import注释组成，类似于<import>在Spring XML中的工作方式。因为@Configuration
        对象在容器中作为Spring bean进行管理，所以可以注入导入的配置 - 例如，通过构造函数注入：
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
			现在，通过仅针对Spring上下文注册AppConfig，可以引导AppConfig和导入的DatabaseConfig：
```java
 new AnnotationConfigApplicationContext(AppConfig.class);
```
		With the @Profile annotation:
			@Configuration类可以使用@Profile注释进行标记，以指示只有在给定的配置文件或配置文件处于活动状态时才应处理
        它们：
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
			或者，您也可以在@Bean方法级别声明概要文件条件 - 例如，对于同一配置类中的备用bean变体：
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
			有关更多详细信息，请参阅@Profile和Environment javadocs。

		With Spring XML using the @ImportResource annotation:
			如上所述，@Consfiguration类可以在Spring XML文件中声明为常规的Spring <bean>定义。也可以使用
        @ImportResource 注释将Spring XML配置文件导入@Configuration类。可以注入从XML导入的Bean定义 - 例如，使用
    	@Inject批注：
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
		With nested @Configuration classes:
        	@Configuration类可以嵌套在一起，如下所示：
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
			在引导这样的安排时，只需要针对应用程序上下文注册AppConfig。由于是一个嵌套的@Configuration类，
        DatabaseConfig将自动注册。当AppConfig和DatabaseConfig之间的关系已经隐式清除时，这避免了使用@Import注释的
        需要。
        	另请注意，嵌套的@Configuration类可以通过@Profile注释使用，以便为封闭的@Configuration类提供相同bean的
        两个选项。

		Configuring lazy initialization:
        	默认情况下，@Bean方法将在容器引导时急切地实例化。为了避免这种情况，可以将@Configuration与@Lazy注释结合
        使用，以指示在类中声明的所有@Bean方法都是默认延迟初始化的。请注意，@ Lazy也可以用于各个@Bean方法。
---
    Testing support for @Configuration classes:
        spring-test模块中提供的Spring TestContext框架提供了@ContextConfiguration注释，它可以接受
    @Configuration类对象的数组：
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
			有关详细信息，请参阅TestContext框架参考文档。
---
	Enabling built-in Spring features using @Enable annotations:
		可以使用各自的“@Enable”注释从@Configuration类启用和配置Spring等功能，例如异步方法执行，计划任务执行，注释驱动
    事务管理，甚至Spring MVC。有关详细信息，请参阅 @EnableAsync，@EnableScheduling，
	@EnableTransactionManagement，@EnableAspectJAutoProxy 和 @EnableWebMvc。
---
	Constraints when authoring @Configuration classes:
    	1. 必须以类的形式提供配置类（即不是从工厂方法返回的实例），允许通过生成的子类进行运行时增强。
    	2. 配置类必须是 non-final。
    	3. 配置类必须是 non-local(如: 可能无法在方法中声明)。
    	4. 必须将任何嵌套配置类声明为static。
    	5. @Bean方法可能不会反过来创建更多的配置类（任何此类实例都将被视为常规bean，其配置注释仍未被检测到）。
---
	属性:
    	1. String value: ""
    		显式指定与@Configuration类关联的Spring bean定义的名称。如果未指定（常见情况），将自动生成bean名称。
            仅当通过组件扫描拾取@Configuration类或直接提供给AnnotationConfigApplicationContext时，自定义名称才
        适用。如果将@Configuration类注册为传统的XML bean定义，则bean元素的名称/ id将优先。
### ConfigurationClassPostProcessor 解读
		BeanFactoryPostProcessor用于@Configuration类的引导处理。
		使用<context：annotation-config />或<context：component-scan />时默认注册。否则，可以像任何其他
    BeanFactoryPostProcessor一样手动声明。
		此后处理器是按优先级排序的，因为在任何其他BeanFactoryPostProcessor执行之前，在@Configuration类中声明的任何
    Bean方法都必须注册其相应的bean定义。
## @Bean 解读
		表示方法生成由Spring容器管理的bean。
---
	Overview:
    	此批注的属性的名称和语义有意类似于Spring XML模式中的<bean />元素的名称和语义。例如：
```java
     @Bean
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
---
	Bean Names:
    	虽然name（）属性可用，但确定bean名称的默认策略是使用@Bean方法的名称。这很方便直观，但如果需要显式命名，可以使用
    name属性（或其别名值）。另请注意，name接受一个字符串数组，允许单个bean使用多个名称（即主bean名称加上一个或多个别名）。
```java
     @Bean({"b1", "b2"}) // bean available as 'b1' and 'b2', but not 'myBean'
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
	Profile, Scope, Lazy, DependsOn, Primary, Order:
    	请注意，@ Node注释不提供profile，scope，lazy，depends-on或primary的属性。相反，它应该与@Scope，@Lazy，
    @DependsOn 和 @Primary注释结合使用来声明这些语义。例如：
```java
     @Bean
     @Profile("production")
     @Scope("prototype")
     public MyBean myBean() {
         // instantiate and configure MyBean obj
         return obj;
     }
```
		上述注释的语义与它们在组件类级别的使用相匹配：@Profile允许选择性地包含某些bean。@Scope将bean的范围从singleton
    更改为指定的范围。@Lazy仅在默认单例范围的情况下具有实际效果。除了bean通过直接引用表达的任何依赖项外，@DependsOn还会在
    创建此bean之前强制创建特定的其他bean，这通常对单例启动很有帮助。@Primary是一种在注入点级别解决歧义的机制，如果需要注入
    单个目标组件但几个bean按类型匹配。
		此外，@Bean方法还可以声明限定符注释和@Order值，在注入点解析期间要考虑，就像对应的组件类上的相应注释一样，但可能是
    每个bean定义非常独立（如果有多个定义，则相同）豆类）。限定符在初始类型匹配后缩小候选集;在集合注入点的情况下，订单值确定
    已解析元素的顺序（多个目标bean按类型和限定符匹配）。
        注意：@Order值可能会影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和@DependsOn声明确定的正
    交关注点，如上所述。此外，由于无法在方法上声明优先级，因此无法在此级别提供优先级;它的语义可以通过@Order值和@Primary在
    每个类型的单个bean上建模。
---
	@Bean Methods in @Configuration Classes:
    	通常，@Bean方法在@Configuration类中声明。在这种情况下，bean方法可以通过直接调用它们来引用同一个类中的其他
    @Bean方法。这可确保bean之间的引用是强类型和可导航的。这种所谓的“bean间引用”保证尊重范围和AOP语义，就像getBean（）
    查找一样。这些是从最初的“Spring JavaConfig”项目中已知的语义，它需要在运行时对每个这样的配置类进行CGLIB子类化。因此，
    在此模式下，不得将@Configuration类及其工厂方法标记为final或private。例如：
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
    	@Bean方法也可以在未使用@Configuration注释的类中声明。例如，bean方法可以在@Component类中声明，甚至可以在普通的
    旧类中声明。在这种情况下，@Bean方法将在所谓的“精简”模式下处理。
    	lite模式中的Bean方法将被容器视为普通工厂方法（类似于XML中的工厂方法声明），并正确应用范围和生命周期回调。在这种情
    况下，包含类保持不变，并且包含类或工厂方法没有异常约束。
    	与@Configuration类中bean方法的语义相反，在lite模式下不支持“bean间引用”。相反，当一个@Bean方法在lite模式下调
    用另一个@Bean方法时，调用是标准的Java方法调用;Spring不会通过CGLIB代理拦截调用。这类似于inter-@Transactional方法
    调用，在代理模式下，Spring不拦截调用 -  Spring仅在AspectJ模式中这样做。如:
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
    	有关更多详细信息，请参阅@Configuration javadoc，包括如何使用AnnotationConfigApplicationContext和朋友引导
    容器。

		BeanFactoryPostProcessor-returning @Bean methods:
			必须特别考虑返回Spring BeanFactoryPostProcessor（BFPP）类型的@Bean方法。由于BFPP对象必须在容器生命周期
        的早期实例化，因此它们可能会干扰@Configuration类中@Autowired，@Value和@PostConstruct等注释的处理。要避免这些
        生命周期问题，请将BFPP返回@Bean方法标记为静态。例如：
```java
     @Bean
     public static PropertySourcesPlaceholderConfigurer pspc() {
         // instantiate, configure and return pspc ...
     }
```
			通过将此方法标记为静态，可以调用它而不会导致其声明@Configuration类的实例化，从而避免上述生命周期冲突。但请注
        意，如上所述，静态@Bean方法不会针对作用域和AOP语义进行增强。这在BFPP案例中有效，因为它们通常不被其他@Bean方法引
    	用。作为提醒，将为任何具有可分配给BeanFactoryPostProcessor的返回类型的非静态@Bean方法发出WARN级别的日志消息。
---
	属性:
    	1. boolean autowireCandidate: true
			这个bean是否可以自动连接到其他bean？
            默认为true;对于不打算妨碍其他地方相同类型的bean的内部委托，将此值设置为false。

        2. String destroyMethod: "(inferred)"
        	关闭应用程序上下文时调用bean实例的方法的可选名称，例如JDBC DataSource实现上的close（）方法或
        Hibernate SessionFactory对象。该方法必须没有参数，但可能抛出任何异常。
        	为方便用户，容器将尝试针对从@Bean方法返回的对象推断出destroy方法。例如，给定一个返回Apache Commons DBCP
        BasicDataSource的@Bean方法，容器将注意到该对象上可用的close（）方法并自动将其注册为destroyMethod。这种
    	“破坏方法推理”目前仅限于检测名为“close”或“shutdown”的公共，no-arg方法。该方法可以在继承层次结构的任何级别声明，
        并且无论@Bean方法的返回类型如何都将被检测到（即，在创建时反复地针对bean实例本身进行检测）。
        	要禁用特定@Bean的destroy方法推断，请指定一个空字符串作为值，例如@Bean（=了destroyMethod “”）。请注意，
        仍会检测到DisposableBean回调接口并调用相应的destroy方法：换句话说，destroyMethod =“”仅影响自定义关闭/关闭方法
        和Closeable / AutoCloseable声明的关闭方法。
        	注意：仅在生命周期完全由工厂控制的bean上调用，对于单身人员来说总是如此，但对于任何其他范围都不能保证。

		3. String initMethod: ""
            初始化期间在bean实例上调用的方法的可选名称。不常用，因为可以在Bean注释方法的主体内直接以编程方式调用该方法。
            默认值为“”，表示不调用init方法。

		4. String[] name: {}
			此bean的名称，或多个名称，主bean名称加别名。
            如果未指定，则bean的名称是带注释的方法的名称。如果指定，则忽略方法名称。
            如果没有声明其他属性，也可以通过value（）属性配置bean名称和别名。

        5. String[] value: {}
            name() 的别名.
            在不需要其他属性时使用，例如：@Bean（“customBeanName”）。
### BeanFactoryPostProcessor 解读
		允许自定义修改应用程序上下文的bean定义，调整上下文的基础bean工厂的bean属性值。
        应用程序上下文可以在其bean定义中自动检测BeanFactoryPostProcessor bean，并在创建任何其他bean之前应用它们。
        对于以系统管理员为目标的自定义配置文件非常有用，这些文件覆盖在应用程序上下文.
        请参阅PropertyResourceConfigurer及其针对此类配置需求的开箱即用解决方案的具体实现。
        BeanFactoryPostProcessor可以与bean定义交互并修改bean定义，但绝不能与bean实例交互。这样做可能会导致bean过早
    实例化，违反容器并导致意外的副作用。如果需要bean实例交互，请考虑实现BeanPostProcessor。
## @Import 解读
		表示要导入的一个或多个@Configuration类。
        提供与Spring XML中的<import />元素等效的功能。允许导入@Configuration类，ImportSelector和
    ImportBeanDefinitionRegistrar实现，以及常规组件类（从4.2开始;类似于
    AnnotationConfigApplicationContext.register（java.lang.Class <？> ...））。
    	应使用@Autowired注入来访问在导入的@Configuration类中声明的@Bean定义。bean本身可以自动装配，或者声明bean的
    配置类实例可以自动装配。后一种方法允许在@Configuration类方法之间进行显式的，IDE友好的导航。
    	可以在类级别声明或作为元注释声明。
        如果需要导入XML或其他非@Configuration bean定义资源，请改用@ImportResource注释。
---
	属性:
    	1. Class<?>[] value:  
			要导入的Configuration，ImportSelector，ImportBeanDefinitionRegistrar或常规组件类。
### ImportSelector 解读
		接口由类型实现，这些类型根据给定的选择标准（通常是一个或多个注释属性）确定应导入哪些@Configuration类。
        ImportSelector可以实现以下任何Aware接口，并且在
        selectImports（org.springframework.core.type.AnnotationMetadata）之前调用它们各自的方法：
        	· EnvironmentAware
			· BeanFactoryAware
			· BeanClassLoaderAware
			· ResourceLoaderAware
		ImportSelector实现通常以与常规@Import注释相同的方式处理，但是，也可以推迟选择导入，直到处理完所有
    @Configuration类（有关详细信息，请参阅DeferredImportSelector）。
## @DependsOn 解读
		当前bean所依赖的bean。指定的任何bean都保证在此bean之前由容器创建。在bean没有通过属性或构造函数参数显式依赖于
    另一个bean的情况下很少使用，而是依赖于另一个bean的初始化的副作用。
    	依赖声明可以指定初始化时间依赖性，并且仅在单例bean的情况下，指定相应的销毁时间依赖性。在给定的bean本身被销毁之前，
    首先销毁定义与给定bean的依赖关系的从属bean。因此，依赖声明也可以控制关闭顺序。
        可以在任何直接或间接使用Component注释的类或使用Bean注释的方法上使用。
        除非正在使用组件扫描，否则在类级别使用DependsOn无效。如果通过XML声明DependsOn-annotated类，则忽略DependsOn
    注释元数据，而忽略<bean depends-on =“...”/>。
---
	属性:
    	1. String[] value: {}
### DependencyDescriptor 解读
		即将注入的特定依赖项的描述符。包装构造函数参数，方法参数或字段，允许统一访问其元数据。
## @ConstructorProperties 解读
		构造函数上的注释，显示该构造函数的参数如何与构造对象的getter方法相对应。例如：
```java
   public class Point {
       @ConstructorProperties({"x", "y"})
       public Point(int x, int y) {
           this.x = x;
           this.y = y;
       }

       public int getX() {
           return x;
       }

       public int getY() {
           return y;
       }

       private final int x, y;
   }
```
		注释显示构造函数的第一个参数可以使用getX（）方法检索，第二个参数可以使用getY（）方法检索。由于参数名称在运行时
    通常不可用，因此没有注释就无法知道参数是对应于getX（）和getY（）还是反过来。
---
	属性:
    	1. String[] value: 
    		getter的名字.
## @Component 解读
		表示带注释的类是“组件”。当使用基于注释的配置和类路径扫描时，这些类被视为自动检测的候选者。
        其他类级注释也可以被认为是识别组件，通常是特殊类型的组件：例如，@Repository注释或AspectJ的@Aspect注释。
---
	属性:
    	1. String value:
    		该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
## ComponentDefinition 解读
		描述一些配置上下文中显示的一组BeanDefinitions和BeanReferences的逻辑视图的接口。
        通过引入可插入的自定义XML标记，现在可以为单个逻辑配置实体（在本例中为XML标记）创建多个BeanDefinitions和
    RuntimeBeanReferences，以便为最终用户提供更简洁的配置和更大的便利。因此，不再假设每个配置实体（例如XML标签）映射到
    一个BeanDefinition。对于希望提供可视化或支持配置Spring应用程序的工具供应商和其他用户，重要的是有一些机制可以将
    BeanFactory中的BeanDefinitions以对最终用户具有特定含义的方式绑定回配置数据。因此，NamespaceHandler实现能够以
    ComponentDefinition的形式为正在配置的每个逻辑实体发布事件。然后，第三方可以订阅这些事件，从而允许以bean为单位的以
    用户为中心的视图。
		每个ComponentDefinition都有一个特定于配置的源对象。在基于XML的配置的情况下，这通常是包含用户提供的配置信息的
    节点。除此之外，ComponentDefinition中包含的每个BeanDefinition都有自己的源对象，可以指向不同的，更具体的配置数据集。
    除此之外，诸如PropertyValues之类的bean元数据也可能具有源对象，从而提供更高级别的细节。源对象提取通过
    SourceExtractor处理，可以根据需要进行自定义。
		虽然通过getBeanReferences（）提供对重要BeanReferences的直接访问，但工具可能希望检查所有BeanDefinition以
    收集完整的BeanReferences集。需要实现来提供验证整个逻辑实体的配置所需的所有BeanReferences，以及提供配置的完整用户
    可视化所需的那些。期望某些BeanReferences对于验证或配置的用户视图不重要，因此可以省略这些。工具可能希望显示通过提供的
    BeanDefinitions获取的任何其他BeanReferences，但这不被视为典型情况。
    	工具可以通过检查角色标识符来确定包含的BeanDefinitions的重要性。该角色本质上是对工具提示配置提供程序认为
    BeanDefinition对最终用户的重要性。预计工具不会显示给定ComponentDefinition的所有BeanDefinition，而是选择基于
    角色进行过滤。工具可以选择使此过滤用户可配置。应特别注意INFRASTRUCTURE角色标识符。使用此角色分类的BeanDefinition
    对最终用户完全不重要，仅出于内部实现原因而需要。
## @Repository 解读
		表示带注释的类是“存储库”，最初由域驱动设计（Evans，2003）定义为“用于封装模拟对象集合的存储，检索和搜索行为的
    机制”。
    	实现传统Java EE模式（如“数据访问对象”）的团队也可以将此构造型应用于DAO类，但在此之前应注意理解数据访问对象和DDD
    样式存储库之间的区别。这个注释是一个通用的刻板印象，个别团队可能会缩小其语义并在适当时使用。
    	当与PersistenceExceptionTranslationPostProcessor一起使用时，这样注释的类有资格进行Spring
    DataAccessException转换。为了工具，方面等目的，注释类还阐明了它在整个应用程序体系结构中的作用。
    	从Spring 2.5开始，这个注释也可以作为@Component的特化，允许通过类路径扫描自动检测实现类。
---
	属性:
    	1. String value: ""
    		该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
## @Service 解读
		表示带注释的类是“服务”，最初由域驱动设计（Evans，2003）定义为“作为模型中独立的接口提供的操作，没有封装状态”。
        也可能表明一个类是“业务服务门面”（在核心J2EE模式意义上）或类似的东西。这个注释是一个通用的刻板印象，个别团队可能
    会缩小其语义并在适当时使用。
    	此注释用作@Component的特化，允许通过类路径扫描自动检测实现类。
---
	属性:
    	1. String value: ""
    		该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
## @Controller 解读
		表示带注释的类是“控制器”（例如Web控制器）。
        此注释用作@Component的特化，允许通过类路径扫描自动检测实现类。它通常与基于RequestMapping批注的带注释的处理程序
    方法结合使用。
---
	属性:
    	1. String value: ""
    		该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
### Controller 解读
		基本控制器接口，表示接收HttpServletRequest和HttpServletResponse实例的组件，就像HttpServlet一样，但能够参与
    MVC工作流。控制器与Struts Action的概念相当。
    	Controller接口的任何实现都应该是可重用的，线程安全的类，能够在应用程序的整个生命周期中处理多个HTTP请求。为了能够
    轻松配置Controller，鼓励Controller实现（通常是）JavaBeans。
---
	Workflow:
    	在DispatcherServlet收到请求并完成其工作以解析区域设置，主题等之后，它会尝试使用HandlerMapping解析
    Controller。当找到Controller来处理请求时，将调用所定位的Controller的handleRequest方法;然后，定位的Controller
    负责处理实际请求，并且 - 如果适用 - 返回适当的ModelAndView。实际上，这个方法是DispatcherServlet的主要入口点，它将
    请求委托给控制器。
    	所以基本上任何Controller接口的直接实现都只处理HttpServletRequests，并且应该返回一个ModelAndView，由
    DispatcherServlet进一步解释。任何其他功能，如可选验证，表单处理等，都应该通过扩展AbstractController或其子类之一
    来获得。
---
	Notes on design and testing:
    	Controller接口明确设计为在HttpServletRequest和HttpServletResponse对象上运行，就像HttpServlet一样。与
    Weblet，JSF或Tapestry相比，它的目的不是将自己与Servlet API分离。相反，Servlet API的全部功能可用，允许控制器具有
    通用性：Controller不仅能够处理Web用户界面请求，还能处理远程协议或按需生成报告。
    	通过将HttpServletRequest和HttpServletResponse对象的模拟对象作为参数传递给handleRequest方法，可以轻松地测试
    控制器。为方便起见，Spring附带了一组Servlet API模拟，适用于测试任何类型的Web组件，但特别适合测试Spring Web控制器。
    与Struts Action相比，不需要模拟ActionServlet或任何其他基础结构;模拟HttpServletRequest和HttpServletResponse
    就足够了。
    	如果控制器需要知道特定的环境引用，他们可以选择实现特定的感知接口，就像Spring（web）应用程序上下文中的任何其他bean
    一样，例如：
			1. org.springframework.context.ApplicationContextAware
			2. org.springframework.context.ResourceLoaderAware
			3. org.springframework.web.context.ServletContextAware
		通过相应感知界面中定义的相应设置器，可以在测试环境中容易地传递这样的环境引用。通常，建议尽可能减少依赖关系：例如，
    如果您只需要资源加载，请仅实现ResourceLoaderAware。或者，派生自WebApplicationObjectSupport基类，它通过方便的
    访问器为您提供所有这些引用，但在初始化时需要ApplicationContext引用。
		控制器可以选择实现LastModified接口。
## @Required 解读
		将方法（通常是JavaBean setter方法）标记为“required”：即，必须将setter方法配置为使用值依赖注入。
        请参考javadoc获取RequiredAnnotationBeanPostProcessor类（默认情况下，它会检查是否存在此批注）。
### RequiredAnnotationBeanPostProcessor 解读
    Deprecated. 
    	从5.1开始，支持使用构造函数注入所需的设置（或自定义的InitializingBean实现）
## @Autowired 解读
		将构造函数，字段，setter方法或配置方法标记为由Spring的依赖注入工具自动装配。这是JSR-330 Inject注释的替代方法，
    添加了必需与可选的语义。
    	任何给定bean类只有一个构造函数（最大值）可以声明这个注释，并将'required'参数设置为true，表示构造函数在用作
    Spring bean时要自动装配。如果多个非必需构造函数声明了注释，则它们将被视为自动装配的候选者。将选择具有最大数量的依赖项
    的构造函数，这些构造函数可以通过匹配Spring容器中的bean来满足。如果不能满足任何候选者，则将使用主要/默认构造函数
    （如果存在）。如果一个类只声明一个构造函数开头，它将始终被使用，即使没有注释。带注释的构造函数不必是公共的。
    	在调用任何配置方法之前，在构造bean之后立即注入字段。这样的配置字段不必是公共的。
        配置方法可以有任意名称和任意数量的参数;每个参数都将使用Spring容器中的匹配bean进行自动装配。Bean属性设置器方法
    实际上只是这种通用配置方法的特例。这种配置方法不必是公开的。
    	对于多参数构造函数或方法，'required'参数适用于所有参数。单个参数可以声明为Java-8-style Optional，或者，从
    Spring Framework 5.0开始，也可以在Kotlin中声明为@Nullable或非null参数类型，从而覆盖基本所需的语义。
    	在Collection或Map依赖类型的情况下，容器自动装配与声明的值类型匹配的所有bean。为此目的，必须将映射键声明为
    String类型，并将其解析为相应的bean名称。这样的容器提供的集合将被订购，考虑目标组件的订购/订单值，否则遵循它们在容器中
    的注册订单。或者，单个匹配的目标bean也可以是通常类型的Collection或Map本身，如此注入。
    	请注意，实际注入是通过BeanPostProcessor执行的，而BeanPostProcessor又意味着您无法使用@Autowired将引用注入
    BeanPostProcessor或BeanFactoryPostProcessor类型。请参考javadoc获取AutowiredAnnotationBeanPostProcessor
    类（默认情况下，它会检查是否存在此批注）。
---
	属性:
    	1. boolean required: true
    		声明是否需要带注释的依赖项。
### AutowiredAnnotationBeanPostProcessor 解读
		BeanPostProcessor实现，自动装配带注释的字段，setter方法和任意配置方法。要注入的这些成员是通过Java 5注释检测的：
    默认情况下，Spring的@Autowired和@Value注释。
    	还支持JSR-330的@Inject注释（如果可用），作为Spring自己的@Autowired的直接替代品。
        任何给定bean类只有一个构造函数（最大值）可以声明这个注释，并将'required'参数设置为true，表示构造函数在用作
    Spring bean时要自动装配。如果多个非必需构造函数声明了注释，则它们将被视为自动装配的候选者。将选择具有最大数量的依赖项
    的构造函数，这些构造函数可以通过匹配Spring容器中的bean来满足。如果不能满足任何候选者，则将使用主要/默认构造函数
    （如果存在）。如果一个类只声明一个构造函数开头，它将始终被使用，即使没有注释。带注释的构造函数不必是公共的。
		在调用任何配置方法之前，在构造bean之后立即注入字段。这样的配置字段不必是公共的。
        配置方法可以有任意名称和任意数量的参数;每个参数都将使用Spring容器中的匹配bean进行自动装配。Bean属性设置器方法
    实际上只是这种通用配置方法的特例。配置方法不必是公开的。
    	注意：默认的AutowiredAnnotationBeanPostProcessor将由“context：annotation-config”和“context：
    component-scan”XML标记注册。如果要指定自定义AutowiredAnnotationBeanPostProcessor bean定义，请删除或关闭默认
    注释配置。
    	注意：注释注入将在XML注入之前执行;因此后一种配置将覆盖通过两种方法连接的属性的前者。
		除了上面讨论的常规注入点之外，这个后处理器还处理Spring的@Lookup注释，该注释标识了在运行时由容器替换的查找方法。
    这本质上是getBean（Class，args）和getBean（String，args）的类型安全版本，有关详细信息，请参阅@Lookup的javadoc。
## @Lookup 解读
		一个注释，指示“查找”方法，由容器重写以将它们重定向回BeanFactory以进行getBean调用。这本质上是XML查找方法属性的
    基于注释的版本，从而产生相同的运行时安排。
    	目标bean的解析可以基于返回类型（getBean（Class）），也可以基于建议的bean名称（getBean（String）），在两种情况
    下都将方法的参数传递给getBean调用，以将它们作为目标工厂应用方法参数或构造函数参数。
    	这样的查找方法可以具有默认（存根）实现，它们将简单地被容器替换，或者它们可以被声明为抽象 - 以便容器在运行时填充
    它们。在这两种情况下，容器将通过CGLIB生成方法包含类的运行时子类，这就是为什么这样的查找方法只能在容器通过常规构造函数
    实例化的bean上工作：即查找方法无法替换从工厂方法返回的bean我们无法为它们动态提供子类。
    	典型Spring配置场景中的具体限制：当与组件扫描或任何其他过滤掉抽象bean的机制一起使用时，提供查找方法的存根实现，
    以便能够将它们声明为具体类。请记住，查找方法不适用于从配置类中的@Bean方法返回的bean;您将不得不求助于@Inject Provider
    <TargetBean>等。
---
	属性:
    	1. String value: ""
			此批注属性可以建议要查找的目标bean名称。如果未指定，将根据带注释的方法的返回类型声明来解析目标bean。
### LookupOverride 解读
		表示在同一IoC上下文中查找对象的方法的覆盖。
        符合查询覆盖条件的方法必须没有参数。
## @RequestScope 解读
		@RequestScope是@Scope的一个特殊化，用于生命周期绑定到当前Web请求的组件。
        具体来说，@RequestScope是一个组合注释，它充当@Scope（“request”）的快捷方式，默认的proxyMode（）设置为
    TARGET_CLASS。
    	@RequestScope可以用作元注释来创建自定义组合注释。
---
	属性:
    	1. ScopedProxyMode proxyMode: org.springframework.context.annotation.ScopedProxyMode.TARGET_CLASS
			Scope.proxyMode（）的别名。
### RequestScope 解读
		请求支持的Scope实现。
        依赖于线程绑定的RequestAttributes实例，该实例可以通过RequestContextListener，RequestContextFilter或
    DispatcherServlet导出。
## @SessionScope 解读
		@SessionScope是@Scope的专业化，用于组件的生命周期绑定到当前Web会话。
        具体来说，@SessionConcope是一个组合注释，它充当@Scope（“session”）的快捷方式，默认的proxyMode（）设置为
    TARGET_CLASS。
    	@SessionScope可以用作元注释来创建自定义组合注释。
---
	属性:
    	1. ScopedProxyMode proxyMode: org.springframework.context.annotation.ScopedProxyMode.TARGET_CLASS
    		Scope.proxyMode（）的别名。
### SessionScope 解读
		会话支持的Scope实现。
        依赖于线程绑定的RequestAttributes实例，该实例可以通过RequestContextListener，RequestContextFilter 或
    DispatcherServlet导出。
## @ApplicationScope 解读
		@ApplicationScope是@Scope的专业化，用于组件的生命周期绑定到当前Web应用程序。
        具体来说，@ ApplicationScope是一个组合注释，它充当@Scope（“application”）的快捷方式，默认的proxyMode（）
    设置为TARGET_CLASS。
    	@ApplicationScope可以用作元注释来创建自定义组合注释。
---
	属性:
    	1. ScopedProxyMode proxyMode: org.springframework.context.annotation.ScopedProxyMode.TARGET_CLASS
    		Scope.proxyMode（）的别名。
## @RestController 解读
		一个便利注释，它本身用@Controller和@ResponseBody注释。
        带有此批注的类型被视为控制器，其中@RequestMapping方法默认采用@ResponseBody语义。
        注意：如果配置了适当的HandlerMapping-HandlerAdapter对，则处理@RestController，例如
    RequestMappingHandlerMapping-RequestMappingHandlerAdapter对，它们是MVC Java配置和MVC命名空间中的缺省值。
---
	属性:
    	1. String value: ""
			该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
## @Primary 解读
		指示当多个候选者有资格自动装配单值依赖项时，应优先考虑bean。如果候选者中只存在一个“主”bean，则它将是自动装配的值。
        此批注在语义上等同于Spring XML中的<bean>元素的主要属性。
        可以在任何直接或间接使用@Component注释的类或使用@Bean注释的方法上使用。
---
	Example:
```java
 @Component
 public class FooService {

     private FooRepository fooRepository;

     @Autowired
     public FooService(FooRepository fooRepository) {
         this.fooRepository = fooRepository;
     }
 }

 @Component
 public class JdbcFooRepository extends FooRepository {

     public JdbcFooRepository(DataSource dataSource) {
         // ...
     }
 }

 @Primary
 @Component
 public class HibernateFooRepository extends FooRepository {

     public HibernateFooRepository(SessionFactory sessionFactory) {
         // ...
     }
 }
```
		因为HibernateFooRepository用@Primary标记，所以它将优先注入基于jdbc的变体，假设它们在同一个Spring应用程序
    上下文中作为bean存在，这通常是组件扫描被大量应用的情况。
    	请注意，除非正在使用组件扫描，否则在类级别使用@Primary无效。如果通过XML声明@ Primary-annotated类，则忽略
    @Primary注释元数据，而忽略<bean primary =“true | false”/>。
## @Qualifier 解读
		在自动装配时，此注释可以在字段或参数上用作候选bean的限定符。它还可以用于注释其他自定义注释，然后可以将其用作限定符。
---
	属性:
    	1. String value: ""
### QualifierAnnotationAutowireCandidateResolver 解读
		AutowireCandidateResolver实现，它将bean定义限定符与要自动装配的字段或参数上的限定符注释相匹配。还通过值注释
    支持建议的表达式值。
		如果可用，还支持JSR-330的Qualifier注释。
## @ResponseBody 解读
		指示方法返回值的注释应绑定到Web响应主体。支持带注释的处理程序方法。
        从版本4.0开始，此注释也可以添加到类型级别，在这种情况下，它是继承的，不需要在方法级别添加。
## @ComponentScan 解读
		配置组件扫描指令以与@Configuration类一起使用。提供与Spring XML <context：component-scan>元素并行的支持。
        可以指定basePackageClasses（）或basePackages（）（或其别名value（））来定义要扫描的特定包。如果未定义特定包，
    则将从声明此批注的类的包进行扫描。
    	请注意，<context：component-scan>元素具有annotation-config属性;但是，这个注释没有。这是因为在使用
    @ComponentScan的几乎所有情况下，都假定默认注释配置处理（例如处理@Autowired和朋友）。此外，使用
	AnnotationConfigApplicationContext时，注释配置处理器始终处于注册状态，这意味着将忽略在@ComponentScan级别禁用
    它们的任何尝试。
		有关用法示例，请参阅@ Configuration的Javadoc。
---
	属性:
		1. Class<?>[] basePackageClasses: {}
			basePackages（）的类型安全替代方法，用于指定要扫描带注释组件的包。将扫描指定的每个类的包。
            考虑在每个包中创建一个特殊的无操作标记类或接口，除了被该属性引用之外没有其它用途。

        2. String[] basePackages: {}
			用于扫描带注释组件的基础包。
            value（）是此属性的别名（并与之互斥）。
            使用basePackageClasses（）作为基于String的包名称的类型安全替代方法。

		3. ComponentScan.Filter[] excludeFilters: {}
			指定哪些类型不符合组件扫描的条件。

		4. ComponentScan.Filter[] includeFilters: {}
			指定哪些类型符合组件扫描的条件。
            进一步将候选组件集从basePackages（）中的所有内容缩小到与给定过滤器匹配的基本包中的所有内容。
            请注意，除了默认过滤器（如果已指定），还将应用这些过滤器。将包含与给定过滤器匹配的指定基础包下的任何类型，即使
        它与默认过滤器不匹配（即未使用@Component注释）。

		5. boolean lazyInit: false
			指定是否应注册扫描的bean以进行延迟初始化。
            默认为false;需要时将其切换为true。

		6. Class<? extends BeanNameGenerator> nameGenerator: 
    			org.springframework.beans.factory.support.BeanNameGenerator.class
			BeanNameGenerator类，用于命名Spring容器中检测到的组件。
            BeanNameGenerator接口的默认值本身表示用于处理此@ComponentScan注释的扫描程序应使用其继承的bean名称生成器，
        例如，默认的AnnotationBeanNameGenerator或在引导时提供给应用程序上下文的任何自定义实例。

		7. String resourcePattern: "**/*.class"
			控制符合组件检测条件的类文件。
            考虑使用includeFilters（）和excludeFilters（）来实现更灵活的方法。

		8. ScopedProxyMode scopedProxy: org.springframework.context.annotation.ScopedProxyMode.DEFAULT
			指示是否应为检测到的组件生成代理，这在以代理样式方式使用范围时可能是必需的。
            默认值是遵循用于执行实际扫描的组件扫描程序的默认行为。
            请注意，设置此属性会覆盖为scopeResolver（）设置的任何值。

		9. Class<? extends ScopeMetadataResolver> scopeResolver: 
				org.springframework.context.annotation.AnnotationScopeMetadataResolver.class
            ScopeMetadataResolver用于解析检测到的组件的范围。

		10. boolean useDefaultFilters: true
			指示是否应启用使用@Component @Repository，@Service或@Controller注释的类的自动检测。

		11. String[] value: {}
			basePackages（）的别名。
            如果不需要其他属性，则允许更简洁的注释声明 - 例如，@ComponentScan（“org.my.pkg”）而不是
        @ComponentScan（basePackages =“org.my.pkg”）。
### ComponentScanBeanDefinitionParser 解读
		<context：component-scan />元素的解析器。
## @ComponentScans 解读
		用于聚合多个ComponentScan注释的容器注释。
        可以原生使用，声明几个嵌套的ComponentScan注释。也可以与Java 8对可重复注释的支持结合使用，其中ComponentScan
    可以在同一方法上简单地多次声明，隐式生成此容器注释。
---
	属性:
    	1. ComponentScan[] value
## @ComponentScan.Filter 解读
		声明要用作包含过滤器或排除过滤器的类型过滤器。
---
	属性:
    	1. Class<?>[] classes: {}
			要用作过滤器的类。
            下表说明了如何根据type（）属性的配置值解释类。
            	ANNOTATION  -->  注释本身
                ASSIGNABLE_TYPE -->  检测到组件的类型应该可以分配给
                CUSTOM  -->  TypeFilter的实现
            指定多个类时，将应用OR逻辑 - 例如，“包含使用@Foo OR @Bar注释的类型”。
            Custom TypeFilters可以选择实现以下任何Aware接口，并在匹配之前调用它们各自的方法：
            	EnvironmentAware
				BeanFactoryAware
				BeanClassLoaderAware
				ResourceLoaderAware
			允许指定零类，但不会影响组件扫描。

		2. String[] pattern: {}
			用于过滤器的模式（或模式），作为指定Class值（）的替代方法。
            如果type（）设置为ASPECTJ，则这是一个AspectJ类型模式表达式。如果type（）设置为REGEX，则这是要匹配的完全
        限定类名的正则表达式模式。

		3. FilterType type: org.springframework.context.annotation.FilterType.ANNOTATION
			要使用的过滤器类型。
            默认为FilterType.ANNOTATION。

		4. Class<?>[] value: {}
			classes() 的别名。
## @Lazy 解读
		指示是否要延迟初始化bean。
        可以在任何直接或间接使用@Component注释的类或使用@Bean注释的方法上使用。
        如果@Component或@Bean定义中不存在此批注，则会发生急切初始化。如果存在并设置为true，则@Bean或@Component将不会
    被初始化，直到被另一个bean引用或从封闭的BeanFactory中显式检索。如果存在并设置为false，那么bean将在启动时由bean工厂
    实例化，这些工厂执行单例的初始化。
    	如果@Configuration类中存在Lazy，则表示该@Configuration中的所有@Bean方法都应该被懒惰地初始化。如果在
    @Lazy-annotated @Configuration类中的@Bean方法中存在@Lazy并且为false，则表示覆盖'default lazy'行为并且应该急
    切地初始化bean。
    	除了它的组件初始化角色之外，这个注释还可以放在标有Autowired或Inject的注入点上：在该上下文中，它导致为所有受影响
    的依赖项创建一个惰性解析代理，作为使用ObjectFactory的替代方法或提供者。
---
	属性:
    	1. boolean value: true
			是否应该进行延迟初始化。
## @Value 解读
		字段或方法/构造函数参数级别的注释，指示受影响参数的默认值表达式。
        通常用于表达式驱动的依赖注入。还支持动态解析处理程序方法参数，例如在Spring MVC中。
        常见用例是使用＃{systemProperties.myProp}样式表达式分配默认字段值。
        请注意，@Value注释的实际处理是由BeanPostProcessor执行的，这反过来意味着您不能在BeanPostProcessor或
    BeanFactoryPostProcessor类型中使用@Value。请参考javadoc获取AutowiredAnnotationBeanPostProcessor类
    （默认情况下，它会检查是否存在此批注）。
---
	属性:
    	1. String value: 
			实际值表达式：例如＃{systemProperties.myProp}。
## @Description 解读
		向从Component或Bean派生的bean定义添加文本描述。
---
	属性:
    	1. String value: 
			与bean定义关联的文本描述。
## @Profile 解读
		表示当一个或多个指定的配置文件处于活动状态时，组件符合注册条件。
        概要文件是一个命名的逻辑分组，可以通过ConfigurableEnvironment.setActiveProfiles（java.lang.String ...）
    以编程方式激活，也可以通过将spring.profiles.active属性设置为JVM系统属性，环境变量或者声明为Web应用程序的web.xml中
    的Servlet上下文参数。也可以通过@ActiveProfiles注释在集成测试中以声明方式激活配置文件。
		@Profile注释可以通过以下任何方式使用：
        	1. 作为直接或间接使用@Component注释的任何类的类型级注释，包括@Configuration类
        	2. 作为元注释，用于组成自定义构造型注释
        	3. 作为任何@Bean方法的方法级注释
		如果使用@Profile标记@Configuration类，则将绕过与该类关联的所有@Bean方法和@Import注释，除非一个或多个指定的配置
    文件处于活动状态。配置文件字符串可以包含简单的配置文件名称（例如“p1”）或配置文件表达式。轮廓表达式允许表达更复杂的轮廓
    逻辑，例如“p1＆p2”。有关支持的格式的详细信息，请参阅Profiles.of（String ...）。
    	这类似于Spring XML中的行为：如果提供了beans元素的profile属性，例如<beans profile =“p1，p2”>，则不会解析
    beans元素，除非至少有成员'p1'或'p2' 已被激活。同样，如果@Component或@Configuration类标记为
	@Profile（{“p1”，“p2”}），则除非至少激活了配置文件“p1”或“p2”，否则不会注册或处理该类。
    	如果给定的配置文件以NOT运算符（！）作为前缀，则如果配置文件未处于活动状态，则将注册带注释的组件 - 例如，给定
    @Profile（{“p1”，“！p2”}），如果配置文件'p1'处于活动状态或配置文件'p2'未激活。
    	如果省略@Profile注释，则无论哪个（如果有）配置文件处于活动状态，都将进行注册。
		注意：对于@Bean方法使用@Profile，可能会应用特殊方案：对于相同Java方法名称的重载@Bean方法（类似于构造函数重载），
    需要在所有重载方法上一致地声明@Profile条件。如果条件不一致，则只有重载方法中第一个声明的条件才重要。因此，@Version不
    能用于选择具有特定参数签名的重载方法而不是另一个;同一个bean的所有工厂方法之间的分辨率遵循Spring的构造函数解析算法在创
    建时。如果要定义具有不同配置文件条件的备用bean，请使用指向同一bean名称的不同Java方法名称;请参阅@Configuration的
    javadoc中的ProfileDatabaseConfig。
    	通过XML定义Spring bean时，可以使用<beans>元素的“profile”属性。有关详细信息，请参阅spring-beans XSD
    （3.1或更高版本）中的文档。
---
	属性:
		1. String[] value: 
			应注册带注释的组件的配置文件集。
## @Conditional 解读
		表示只有在所有指定条件匹配时，组件才有资格进行注册。
        条件是可以在bean定义到期之前以编程方式确定的任何状态（有关详细信息，请参阅条件）。
        @Conditional批注可以通过以下任何方式使用：
        	1. 作为直接或间接使用@Component注释的任何类的类型级注释，包括@Configuration类
            2. 作为元注释，用于组成自定义构造型注释
            3. 作为任何@Bean方法的方法级注释
		如果使用@Conditional标记@Configuration类，则与该类关联的所有@Bean方法，@ Import注释和@ComponentScan注释
    都将受条件限制。
		注意：不支持继承@Conditional注释;来自超类或来自重写方法的任何条件都不会被考虑。为了强制执行这些语义，
    @Conditional本身不会声明为@Inherited;此外，任何使用@Conditional元注释的自定义组合注释都不能声明为@Inherited。
---
	属性:
    	1. Class<? extends Condition>[] value: 
			必须匹配的所有条件才能注册组件。
## @Inherited 解读
		表示自动继承注释类型。如果注释类型声明中存在Inherited元注释，并且用户在类声明上查询注释类型，并且类声明没有此类型的
    注释，则将自动查询类的超类以获取注释类型。将重复此过程，直到找到此类型的注释，或者到达类层次结构（对象）的顶部。如果没有
    超类具有此类型的注释，则查询将指示相关类没有此类注释。
		请注意，如果使用带注释的类型来注释除类之外的任何内容，则此元注释类型不起作用。另请注意，此元注释仅导致注释从超类
    继承;已实现接口上的注释无效。
## @ImportResource 解读
		指示包含要导入的bean定义的一个或多个资源。
        与@Import一样，此批注提供的功能类似于Spring XML中的<import />元素。它通常在将@Configuration类设计为
    由AnnotationConfigApplicationContext引导时使用，但仍需要某些XML功能（如命名空间）。
    	默认情况下，如果以“.groovy”结尾，将使用GroovyBeanDefinitionReader处理value（）属性的参数;否则，将使用
    XmlBeanDefinitionReader来解析Spring <beans /> XML文件。可选地，可以声明reader（）属性，允许用户选择自定义
    BeanDefinitionReader实现。
---
	属性:
    	1. String[] locations: {}
			要导入的资源位置。
            支持资源加载前缀，如classpath：，file：等。
            有关如何处理资源的详细信息，请咨询Javadoc for reader（）。

		2. Class<? extends BeanDefinitionReader> reader: 
				org.springframework.beans.factory.support.BeanDefinitionReader.class
			BeanDefinitionReader实现在处理通过value（）属性指定的资源时使用。
            默认情况下，阅读器将适应指定的资源路径：“。groovy”文件将使用GroovyBeanDefinitionReader处理;而所有其他
        资源都将使用XmlBeanDefinitionReader进行处理。

		3. String[] value: {}
            locations() 的别名。









## @RequestMapping 解读
		使用灵活方法签名将Web请求映射到请求处理类中的方法的注释。
        Spring MVC和Spring WebFlux都通过RequestMappingHandlerMapping和RequestMappingHandlerAdapter在它们
    各自的模块和包结构中支持这个注释。有关每个支持的处理程序方法参数和返回类型的确切列表，请使用下面的参考文档链接：
    		1. Spring MVC Method Arguments and Return Values
    		2. Spring WebFlux Method Arguments and Return Values
		注意：此批注可以在类和方法级别使用。在大多数情况下，在方法级别，应用程序将更喜欢使用HTTP方法特定变体之一
    @GetMapping，@PostMapping，@PutMapping，@DelaMapping或@PatchMapping。











