
## @Scheduled 解读
		标记要安排的方法的注释。必须指定cron（），fixedDelay（）或fixedRate（）属性中的一个。
        带注释的方法必须没有参数。它通常具有void返回类型;如果不是，则通过调度程序调用时将忽略返回的值。
        通过注册ScheduledAnnotationBeanPostProcessor来执行@Scheduled注释的处理。这可以通过
    <task：annotation-driven />元素或@EnableScheduling注释手动完成，也可以更方便地完成。
    	此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。
---
	属性:
    	1. String cron: ""
			一个类似cron的表达式，扩展了通常的UN * X定义，包括秒以及分钟，小时，星期几，月和星期几的触发器。
            例如。“0 * * * * MON-FRI”表示工作日每分钟一次（在分钟的顶部 - 第0秒）。
            特殊值“ - ”表示禁用的cron触发器，主要用于由${...}占位符解析的外部指定值。

		2. long fixedDelay: -1L
			在最后一次调用结束和下一次调用开始之间以固定周期（以毫秒为单位）执行带注释的方法。

		3. String fixedDelayString: ""
			在最后一次调用结束和下一次调用开始之间以固定周期（以毫秒为单位）执行带注释的方法。

		4. long fixedRate: -1L
			在调用之间以固定的周期（以毫秒为单位）执行带注释的方法。

		5. String fixedRateString: ""
			在调用之间以固定的周期（以毫秒为单位）执行带注释的方法。

		6. long initialDelay: -1L
			在第一次执行fixedRate（）或fixedDelay（）任务之前延迟的毫秒数。

        7. String initialDelayString: ""
			在第一次执行fixedRate（）或fixedDelay（）任务之前延迟的毫秒数。

		8. String zone: ""
			将解析cron表达式的时区。默认情况下，此属性为空字符串（即将使用服务器的本地时区）。
## @Async 解读
		将方法标记为异步执行候选的注释。也可以在类型级别使用，在这种情况下，所有类型的方法都被视为异步。
        就目标方法签名而言，支持任何参数类型。但是，返回类型被限制为void或Future。在后一种情况下，您可以声明更具体的
    ListenableFuture或CompletableFuture类型，这些类型允许与异步任务进行更丰富的交互，并通过进一步的处理步骤立即组合。
		从代理返回的Future句柄将是一个实际的异步Future，可用于跟踪异步方法执行的结果。但是，由于目标方法需要实现相同的
    签名，因此必须返回一个临时的Future句柄，该句柄只传递一个值：Spring的AsyncResult，EJB 3.1的AsyncResult或
    CompletableFuture.completedFuture（Object）。
---
	属性:
    	1. String value: ""
			指定异步操作的限定符值。
            可用于确定执行此方法时要使用的目标执行程序，匹配特定Executor或TaskExecutor bean定义的限定符值
        （或bean名称）。
        	在类级别@Async注释上指定时，表示给定的执行程序应该用于类中的所有方法。方法级别使用Async＃值始终覆盖在类级别
        设置的任何值。
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
    TaskScheduler bean;也将对ScheduledExecutorService bean执行相同的查找。如果两者都不可解析，则将在注册器中创建并
	使用本地单线程默认调度程序。
		当需要更多控制时，@Configuration类可以实现SchedulingConfigurer。这允许访问底层的ScheduledTaskRegistrar
    实例。例如，以下示例演示如何自定义用于执行计划任务的Executor：
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
		请注意上面的例子中使用@Bean（destroyMethod =“shutdown”）。这可确保在Spring应用程序上下文本身关闭时正确关闭
    任务执行程序。
		实现SchedulingConfigurer还允许通过ScheduledTaskRegistrar对任务注册进行细粒度控制。例如，以下配置根据自定义
    Trigger实现执行特定bean方法：
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
		这些示例是等效的，除了在XML中使用固定速率周期而不是自定义触发器实现;这是因为任务：命名空间调度不能轻易暴露这样的
    支持。这只是一个演示，即基于代码的方法如何通过直接访问实际组件来实现最大的可配置性。
		注意：@EnableScheduling仅适用于其本地应用程序上下文，允许在不同级别选择性地安排bean。请在每个单独的上下文中重新
    声明@EnableScheduling，例如公共根Web应用程序上下文和任何单独的DispatcherServlet应用程序上下文，如果您需要在多个
    级别应用其行为。
## @EnableAsync 解读
		启用Spring的异步方法执行功能，类似于Spring的<task：*> XML命名空间中的功能。
		要与@Configuration类一起使用，如下所示，为整个Spring应用程序上下文启用注释驱动的异步处理：
```java
 @Configuration
 @EnableAsync
 public class AppConfig {

 }
```
		MyAsyncBean是一个用户定义的类型，其中一个或多个方法使用Spring的@Async批注，EJB 3.1
    @javax.ejb.Asynchronous批注或通过annotation（）属性指定的任何自定义批注进行批注。透明地为任何已注册的bean添加
    方面，例如通过此配置：
```java
 @Configuration
 public class AnotherAppConfig {

     @Bean
     public MyAsyncBean asyncBean() {
         return new MyAsyncBean();
     }
 }
```
		默认情况下，Spring将搜索关联的线程池定义：上下文中的唯一TaskExecutor bean，或者另一个名为“taskExecutor”的
    Executor bean。如果两者都不可解析，则将使用SimpleAsyncTaskExecutor处理异步方法调用。此外，具有void返回类型的
    带注释的方法不能将任何异常发送回调用者。默认情况下，仅记录此类未捕获的异常。
		要自定义所有这些，请实现AsyncConfigurer并提供：
			1. 你自己的Executor通过getAsyncExecutor（）方法，
			2. 您通过getAsyncUncaughtExceptionHandler（）方法获得自己的AsyncUncaughtExceptionHandler。
		注意：AsyncConfigurer配置类在应用程序上下文引导程序的早期初始化。如果你需要对其他bean有任何依赖，请确保尽可能地
    声明它们“懒惰”，以便让它们通过其他后处理器。
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
		注意：在上面的示例中，ThreadPoolTaskExecutor不是完全托管的Spring bean。如果需要完全托管的bean，请将@Bean
    批注添加到getAsyncExecutor（）方法。在这种情况下，不再需要手动调用executor.initialize（）方法，因为这将在初始化
    bean时自动调用。
		作为参考，可以将上面的示例与以下Spring XML配置进行比较：
```java
 <beans>

     <task:annotation-driven executor="myExecutor" exception-handler="exceptionHandler"/>

     <task:executor id="myExecutor" pool-size="7-42" queue-capacity="11"/>

     <bean id="asyncBean" class="com.foo.MyAsyncBean"/>

     <bean id="exceptionHandler" class="com.foo.MyAsyncUncaughtExceptionHandler"/>

 </beans>
```
		除了设置Executor的线程名前缀外，上述基于XML和JavaConfig的示例是等效的。这是因为<task：executor>元素不公开这样
    的属性。这演示了基于JavaConfig的方法如何通过直接访问实际组件来实现最大的可配置性。
		mode（）属性控制如何应用建议：如果模式为AdviceMode.PROXY（默认值），则其他属性控制代理的行为。请注意，代理模式
    仅允许通过代理拦截呼叫;同一类中的本地调用不能以这种方式截获。
    	请注意，如果mode（）设置为AdviceMode.ASPECTJ，则将忽略proxyTargetClass（）属性的值。另请注意，在这种情况下，
    spring-aspects模块JAR必须存在于类路径中，编译时编织或加载时编织将方面应用于受影响的类。在这种情况下没有涉及代理人;
    本地电话也会被截获。
---
	属性:
    	1. Class<? extends Annotation> annotation: java.lang.annotation.Annotation.class
			指示要在类或方法级别检测的“异步”注释类型。
			默认情况下，将检测Spring的@Async注释和EJB 3.1 @javax.ejb.Asynchronous注释。
			存在此属性，以便开发人员可以提供自己的自定义注释类型，以指示应异步调用方法（或给定类的所有方法）。

		2. AdviceMode mode: org.springframework.context.annotation.AdviceMode.PROXY
			指出应如何应用异步通知。
            默认值为AdviceMode.PROXY。请注意，代理模式仅允许通过代理拦截呼叫。同一类中的本地调用不能以这种方式截获;
        由于Spring的拦截器甚至没有为这样的运行时场景启动，因此将忽略本地调用中此类方法的异步注释。对于更高级的拦截模式，
        请考虑将其切换为AdviceMode.ASPECTJ。

		3. int order: 2147483647
			指示应该应用AsyncAnnotationBeanPostProcessor的顺序。
            默认值为Ordered.LOWEST_PRECEDENCE，以便在所有其他后处理器之后运行，以便它可以向现有代理添加顾问程序而不是
        双代理。

		4. boolean proxyTargetClass: false
			指示是否要创建基于子类的（CGLIB）代理而不是基于标准Java接口的代理。
            仅当mode（）设置为AdviceMode.PROXY时才适用。
            默认值为false。
            请注意，将此属性设置为true将影响所有需要代理的Spring托管bean，而不仅仅是那些标记为@Async的bean。例如，
        标有Spring的@Transactional注释的其他bean将同时升级为子类代理。这种方法在实践中没有负面影响，除非有人明确期望
        一种代理与另一种代理 - 例如，在测试中。
## @Cacheable 解读
		注释指示可以缓存调用方法（或类中的所有方法）的结果。
        每次调用一个建议的方法时，都会应用缓存行为，检查是否已经为给定的参数调用了该方法。合理的默认值只是使用方法参数来
    计算密钥，但可以通过key（）属性提供SpEL表达式，或者自定义KeyGenerator实现可以替换默认值（请参阅keyGenerator（））。
    	如果在计算键的高速缓存中未找到任何值，则将调用目标方法并将返回的值存储在关联的高速缓存中。请注意，Java8的Optional
    返回类型会自动处理，并且其内容存储在缓存中（如果存在）。
		此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。
---
	属性:
    	1. String cacheManager: ""
			如果没有设置，则用于创建默认CacheResolver的自定义CacheManager的bean名称。
            与cacheResolver（）属性互斥。

		2. String[] cacheNames: {}
			存储方法调用结果的高速缓存的名称。
            名称可用于确定目标高速缓存（或高速缓存），匹配特定bean定义的限定符值或bean名称。

		3. String cacheResolver: ""
			要使用的自定义CacheResolver的bean名称。

		4. String condition: ""
			Spring Expression Language（SpEL）表达式，用于使方法缓存条件。
            默认值为“”，表示方法结果始终缓存。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	2. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	3. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		5. String key: ""
			用于动态计算密钥的Spring Expression Language（SpEL）表达式。
            默认值为“”，表示除非已配置自定义keyGenerator（），否则所有方法参数都被视为键。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	2. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	3. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

        6. String keyGenerator: ""
			要使用的自定义KeyGenerator的bean名称。
            与key（）属性互斥。

		7. boolean sync: false
			如果多个线程正在尝试加载同一个键的值，则同步基础方法的调用。同步会导致一些限制：
				1. unless() 不受支持.
				2. 只能指定一个缓存
				3. 不能组合其他与缓存相关的操作
            这实际上是一个提示，您正在使用的实际缓存提供程序可能不会以同步方式支持它。有关实际语义的更多详细信息，请查看
        提供程序文档。

		8. String unless: ""
			用于否决方法缓存的Spring Expression Language（SpEL）表达式。
            与condition（）不同，此表达式在调用方法后进行计算，因此可以引用结果。
            默认为“”，表示缓存永远不会被否决。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #result用于引用方法调用的结果。对于受支持的包装器（如Optional），＃result引用实际对象，而不是包装器
            	2. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	3. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	4. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		9. String[] value: {}
			cacheNames()的别名.
## @CacheEvict 解读
		注释指示方法（或类上的所有方法）触发高速缓存逐出操作。
        此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。
---
	属性:
    	1. boolean allEntries: false
			是否删除了缓存中的所有条目。
            默认情况下，仅删除关联键下的值。
            请注意，将此参数设置为true并且不允许指定key（）。

		2. boolean beforeInvocation: false
			是否应该在调用方法之前进行驱逐。
            将此属性设置为true会导致驱逐发生而不管方法结果如何（即，是否抛出异常）。
            默认为false，这意味着在成功调用建议方法之后将发生缓存逐出操作（即，仅当调用未引发异常时）。

		3. String cacheManager: ""
			如果没有设置，则用于创建默认CacheResolver的自定义CacheManager的bean名称。
            与cacheResolver（）属性互斥。

		4. String[] cacheNames: {}
			用于高速缓存逐出操作的高速缓存的名称。
            名称可用于确定目标高速缓存（或高速缓存），匹配特定bean定义的限定符值或bean名称。

		5. String cacheResolver: ""
			要使用的自定义CacheResolver的bean名称。

		6. String condition: ""
			Spring Expression Language（SpEL）表达式，用于使缓存逐出操作成为条件。
            默认值为“”，表示始终执行缓存逐出。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	2. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	3. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		7. String key: ""
            用于动态计算密钥的Spring Expression Language（SpEL）表达式。
            默认值为“”，表示除非设置了自定义keyGenerator（），否则所有方法参数都被视为键。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #result用于引用方法调用的结果，只有在beforeInvocation（）为false时才能使用。对于受支持的包装器
            （如Optional），＃result引用实际对象，而不是包装器
            	2. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	3. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	4. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		8. String keyGenerator: ""
			要使用的自定义KeyGenerator的bean名称。
            与key（）属性互斥。

		9. String[] value: {}
			cacheNames() 的别名.
## @CachePut 解读
		注释指示方法（或类上的所有方法）触发缓存放置操作。
        与@Cacheable注释相反，此注释不会导致跳过建议的方法。相反，它总是导致调用方法并将其结果存储在关联的缓存中。请注意，
    Java8的Optional返回类型会自动处理，并且其内容存储在缓存中（如果存在）。
		此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。
---
	属性:
    	1. String cacheManager: ""
			如果没有设置，则用于创建默认CacheResolver的自定义CacheManager的bean名称。
            与cacheResolver（）属性互斥。

		2. String[] cacheNames: {}
			用于缓存放置操作的缓存的名称。
            名称可用于确定目标高速缓存（或高速缓存），匹配特定bean定义的限定符值或bean名称。

		3. String cacheResolver: ""
			要使用的自定义CacheResolver的bean名称。

		4. String condition: ""
			Spring Expression Language（SpEL）表达式，用于使缓存放置操作有条件。
            默认值为“”，表示方法结果始终缓存。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	2. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	3. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		5. String key: ""
			用于动态计算密钥的Spring Expression Language（SpEL）表达式。
            默认值为“”，表示除非设置了自定义keyGenerator（），否则所有方法参数都被视为键。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
				1. #result用于引用方法调用的结果。对于受支持的包装器（如Optional），＃result引用实际对象，而不是包装器
				2. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
				3. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
				4. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		6. String keyGenerator: ""
			要使用的自定义KeyGenerator的bean名称。
            与key（）属性互斥。

		7. String unless: ""
			Spring Expression Language（SpEL）表达式用于否决缓存放置操作。
            与condition（）不同，此表达式在调用方法后进行计算，因此可以引用结果。
            默认为“”，表示缓存永远不会被否决。
            SpEL表达式针对提供以下元数据的专用上下文进行评估：
            	1. #result用于引用方法调用的结果。对于受支持的包装器（如Optional），＃result引用实际对象，而不是包装器
            	2. #root.method，＃root.target和#root.caches分别用于引用方法，目标对象和受影响的缓存。
            	3. 方法名称（＃root.methodName）和目标类（#root.targetClass）的快捷方式也可用。
            	4. 方法参数可以通过索引访问。例如，可以通过#root.args [1]，＃p1或＃a1访问第二个参数。如果该信息可用，
            也可以通过名称访问参数。

		8. String[] value: {}
			cacheNames() 的别名.
## @Caching 解读
		多个缓存注释（不同或相同类型）的组注释。
        此注释可用作元注释，以创建具有属性覆盖的自定义组合注释。
---
	属性:
    	1. Cacheable[] cacheable: {}
    	2. CacheEvict[] evict: {}
    	3. CachePut[] put: {}
## @CacheConfig 解读
		@CacheConfig提供了一种在类级别共享与公共缓存相关的设置的机制。
        当此注释出现在给定类上时，它为该类中定义的任何高速缓存操作提供一组默认设置。
---
	属性:
    	1. String cacheManager: ""
			如果没有设置，则用于创建默认CacheResolver的自定义CacheManager的bean名称。
            如果在操作级别没有设置解析器和缓存管理器，并且没有通过cacheResolver（）设置缓存解析器，则使用此解析器而不是
        默认值。

		2. String[] cacheNames: {}
			要在注释类中定义的缓存操作要考虑的默认缓存的名称。
            如果没有在操作级别设置，则使用这些而不是默认值。
            可用于确定目标高速缓存（或高速缓存），匹配特定bean定义的限定符值或bean名称。

		3. String cacheResolver: ""
			要使用的自定义CacheResolver的bean名称。
            如果在操作级别没有设置解析器和缓存管理器，则使用此解析器而不是默认值。

		4. String keyGenerator: ""
			用于类的默认KeyGenerator的bean名称。
            如果没有在操作级别设置，则使用此一个而不是默认值。
            密钥生成器与使用自定义密钥互斥。为操作定义此键时，将忽略此键生成器的值。



























