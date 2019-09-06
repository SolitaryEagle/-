
## @EnableTransactionManagement 解读
		启用Spring的注释驱动的事务管理功能，类似于Spring的<tx：*> XML命名空间中的支持。要在@Configuration类上使用
    如下：
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
		在上面的两个场景中，@EnableTransactionManagement和<tx：annotation-driven />负责注册为注释驱动的事务管理
    提供支持的必要Spring组件，例如TransactionInterceptor和基于代理或基于AspectJ的建议。调用JdbcFooRepository的
    @Transactional方法时，拦截器进入调用堆栈。
		两个示例之间的细微差别在于PlatformTransactionManager bean的命名：在@Bean情况下，名称为“txManager”
    （根据方法的名称）;在XML情况下，名称是“transactionManager”。<tx：annotation-driven />是硬连接的，默认情况下查找
    名为“transactionManager”的bean，但@EnableTransactionManagement更灵活;它将回退到容器中任何
    PlatformTransactionManager bean的类型查找。因此，名称可以是“txManager”，“transactionManager”或“tm”：
    它无关紧要。
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
		mode（）属性控制如何应用建议：如果模式为AdviceMode.PROXY（默认值），则其他属性控制代理的行为。请注意，代理模式
    仅允许通过代理拦截呼叫;同一类中的本地调用不能以这种方式截获。
		请注意，如果mode（）设置为AdviceMode.ASPECTJ，则将忽略proxyTargetClass（）属性的值。另请注意，在这种情况下，
    spring-aspects模块JAR必须存在于类路径中，编译时编织或加载时编织将方面应用于受影响的类。在这种情况下没有涉及代理人;
    本地电话也会被截获。
---
	属性:
    	1. AdviceMode mode: org.springframework.context.annotation.AdviceMode.PROXY
			说明应如何应用交易建议。
            默认值为AdviceMode.PROXY。请注意，代理模式仅允许通过代理拦截呼叫。同一类中的本地调用不能以这种方式截获;
        由于Spring的拦截器甚至没有为这样的运行时场景提供支持，因此将忽略本地调用中此类方法的事务性注释。对于更高级的拦截
        模式，请考虑将其切换为AdviceMode.ASPECTJ。

		2. int order: 2147483647
			指示在特定连接点应用多个建议时执行事务顾问的顺序。默认值为Ordered.LOWEST_PRECEDENCE。

		3. boolean proxyTargetClass: false
			指示是否要创建基于子类的（CGLIB）代理（true），而不是基于标准Java接口的代理（false）。默认值为false。
        仅当mode（）设置为AdviceMode.PROXY时才适用。
        	请注意，将此属性设置为true将影响所有需要代理的Spring托管bean，而不仅仅是那些标记为@Transactional的bean。
        例如，标有Spring的@Async注释的其他bean将同时升级为子类代理。这种方法在实践中没有负面影响，除非有人明确期望一种
        类型的代理与另一种代理相比，例如在测试中。
## @TransactionalEventListener 解读
		根据TransactionPhase调用的EventListener。
        如果事件未在托管事务的边界内发布，则除非显式设置fallbackExecution（）标志，否则将丢弃该事件。如果事务正在运行，
    则根据其TransactionPhase处理事件。
		将@Order添加到带注释的方法允许您在事务完成之前或之后运行的其他侦听器中对该侦听器进行优先级排序。
---
	属性:
    	1. Class<?>[] classes: {}
			此侦听器处理的事件类。如果使用单个值指定此属性，则带注释的方法可以选择接受单个参数。
        但是，如果使用多个值指定此属性，则带注释的方法不得声明任何参数。

		2. String condition: ""
			Spring Expression Language（SpEL）属性，用于使事件处理成为条件。默认值为“”，表示始终处理事件。

		3. boolean fallbackExecution: false
			如果没有正在运行的事务，是否应该处理事件。

		4. TransactionPhase phase: org.springframework.transaction.event.TransactionPhase.AFTER_COMMIT
			用于绑定事件处理的阶段。默认阶段是TransactionPhase.AFTER_COMMIT。如果没有正在进行的事务，
        则除非已显式启用fallbackExecution（），否则不会处理该事件。

		5. Class<?>[] value: {}
			classes() 的别名.