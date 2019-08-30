	AnnotationConfigApplicationContext 的继承体系
   ![AnnotationConfigApplicationContext 继承体系](image/AnnotationConfigApplicationContext继承体系.png)
## AnnotationConfigApplicationContext 解读
		独立应用程序上下文，接受注解类作为输入，特别是 @Configuration 类，也接受普通的 @Component 类和使用 javax.inject 注解的
    JSR-330 兼容的类。允许使用 register(Class...) 逐个注册类，也允许使用 scan(String...) 扫描 classpath。
    	在含有多个 @Configuration 类的情况下，定义在后面的类中的 @Bean 方法将覆盖在前面的类中定义的 Bean。这可以作为一个
    使用额外的 @Configuration 类覆盖某些 Bean 定义的方法。
		AnnotationConfigApplicationContext 中有两个属性：AnnotatedBeanDefinitionReader 和
    ClassPathBeanDefinitionScanner。而且 AnnotatedBeanDefinitionReader 是需要重点走查的，
    ClassPathBeanDefinitionScanner 不怎么使用。只有手动调用 scan(String...) 方法才会使用。
## AnnotationConfigRegistry 解读
	注解配置的应用程序上下文的通用接口, 定义了 register(java.lang.Class<?>...) 和 scan(java.lang.String...) 方法.
## GenericApplicationContext 解读
		泛型 ApplicationContext 实现, 它包含单个内部 DefaultListableBeanFactory 实例, 并且不假定特定的 bean 定义格式. 实现
    BeanDefinitionRegistry 接口以允许将任何 bean 定义读取器应用于它.
    	典型的用法是通过 BeanDefinitionRegistry 接口注册各种 bean 定义, 然后调用 AbstractApplicationContext.refresh()
    初始化那些具有应用程序上下文语义(处理 ApplicationContextAware, 自动检测 BeanFactoryPostProcessors 等)的 bean.
		用法示例:
```java
 GenericApplicationContext ctx = new GenericApplicationContext();
 XmlBeanDefinitionReader xmlReader = new XmlBeanDefinitionReader(ctx);
 xmlReader.loadBeanDefinitions(new ClassPathResource("applicationContext.xml"));
 PropertiesBeanDefinitionReader propReader = new PropertiesBeanDefinitionReader(ctx);
 propReader.loadBeanDefinitions(new ClassPathResource("otherBeans.properties"));
 ctx.refresh();

 MyBean myBean = (MyBean) ctx.getBean("myBean");
 ...
```
    	对于 XML bean 定义的典型情况, 只需要使用 ClassPathXmlApplicationContext 或 FileSystemXmlApplicationContext 更
    容易配置, 但不太灵活, 因为你只能使用 XML bean 定义的标准资源位置, 而不是混合任意 bean 定义格式. Web 环境中的等价物是
    XmlWebApplicationContext.
    	对于应该以可刷新的方法读取特殊 bean 定义格式的自定义应用程序上下文实现, 请考虑从 AbstractRefreshableApplicationContext
    基类派生.
## AliasRegistry 解读
	用于管理别名的的通用接口, 是 BeanDefinitionRegistry 接口的父接口.
## BeanDefinitionRegistry 解读
		持有 bean 定义的注册表接口, 如 RootBeanDefinition 和 ChildBeanDefinition 实例. 典型的实现是内部维持一个
    AbstractBeanDefinition 层次结构的 BeanFactories.
    	这是 Spring 工厂包中唯一封装 bean 定义注册的接口. 标准的 BeanFactory 接口仅涵盖对完全配置的工厂实例的访问.
        Spring 的 bean 定义读取器期望工作在这个接口的一个实现中. 已知的实现者是 DefaultListableBeanFactory 和
    GenericApplicationContext.
## AbstractApplicationContext 解读
		ApplicationContext 接口的抽象实现. 不要求用于配置的存储类型; 只需要简单实现常见的上下文功能, 使用 Template Method 
    设计模式, 需要具体的子类来实现抽象方法.
    	与此相反, 一个朴素的 BeanFactory, ApplicationContext 应该检测定义在它内部的 bean factory 中的特定的 bean: 因此,
    这个类自动注册 BeanFactoryPostProcessors, BeanPostProcessors, and ApplicationListeners 作为 bean 在 context 中.
		MessageSource 也可以作为 bean 在 context 中提供, 名称为 "messageSource"; 否则, 消息解析委托给 parent context.
    此外, 应用程序时间的多播器可以作为一个类型为 ApplicationEventMulticaster 的 "applicationEventMulticaster" bean 提供
    在 context 中; 否则, 一个类型为 SimpleApplicationEventMulticaster 的多播器将被默认使用.
    	通过继承 DefaultResourceLoader 实现资源加载. 因此, 将非URL资源路径视为类路径资源(支持包含路径的完整类路径资源名称, 如
    "mypackage / myresource.dat"), 除非 DefaultResourceLoader.getResourceByPath(java.lang.String) 方法被子类重写.
## ConfigurableApplicationContext 解读
		SPI 接口由大多数 application context 实现. 除了 ApplicationContext 接口中的 application context 客户端方法之外,
    还提供配置 application context 的工具.
    	这里封装了 Configuration and lifecycle 方法. 避免它们相对于 ApplicationContext 客户端代码更明显. 本方法只能由启动
    和关闭代码使用.
## Lifecycle 解读
		定义启动/停止生命周期控制方法的通用接口. 这种情况的典型用例是控制异步处理. 注意: 此接口不暗示特定的自动启动语义, 可以考虑
    实现 SmartLifecycle 接口达成此目的.
    	可以被 components (通常是 Spring context 中定义的 Spring bean) 和 containers (通常是 Spring ApplicationContext
    本身) 实现. containers 会将启动/停止信号传播到每个 container 中的所有 components. 例如, 在运行时停止/重启 scenario.
    	请注意, Lifecycle 接口仅支持"顶级单例 beans". 在任何其它 component 中, Lifecycle 接口将不会被检测到而忽略. 另外,
    继承 SmartLifecycle 提供与 application context 的启动和关闭阶段的复杂集成.
## AutoCloseable 解读
		在关闭之前可以保存资源 (如, 文件, 套接字句柄) 的对象. 当退出资源规范头中已声明对象的 try-with-resources 语句块时, 将
    自动调用 AutoCloseable 对象的 close() 方法. 这种结构可以确保快速释放, 避免资源耗尽异常和可能发生的错误.
    	API 注意: 对于基类来说, 实现 AutoCloseable 是可能的, 并且实际上是常见的, 即使并非所有子类或实例都将保留可释放的资源.
    对于必须完全通用运行的代码, 或者当已知 AutoCloseable 实例需要资源释放时, 建议使用 try-with-resources 结构. 但是, 当使用
    Stream 支持基于 I/O 或非 I/O 的表单工具时, 在使用非基于 I/O 的表单时, 通常不需要使用 try-with-resources 语句块.
## Closeable 解读
	Closeable 是可以关闭的数据的来源或目标. 调用 close 方法以释放对象所持有的资源(如, 打开的文件).
## ApplicationContext 解读
    	用于为 application 提供配置的中央接口. 这在 application 运行时是只读的, 但是如果实现支持, 则可以重新加载.
        ApplicationContext 提供:
        	· Bean 工厂方法, 用于访问 application components. 继承自 ListableBeanFactory.
            · 以通用方式加载文件资源的能力. 继承自 ResourceLoader.
            · 将时间发布到已注册的监听器的功能. 继承自 ApplicationEventPublisher.
            · 解析消息, 支持国际化的能力. 继承自 MessageSource.
            · 继承 parent context. 后代 context 中的定义始终优先. 这意味着, 整个 Web application 可以使用单个
        parent context, 而每个 servlet 都有自己的子 context, 该 context 独立于任何其它 servlet 的子 context.
        
        除了标准的 BeanFactory 生命周期的能力, ApplicationContext 实现检测和调用 ApplicationContextAware beans,
    ResourceLoaderAware, ApplicationEventPublisherAware 和 MessageSourceAware beans.
## MessageSource 解读
    用于解析消息的策略接口, 支持此类消息的参数化和国际化.
    Spring 为生产提供了两种开箱即用的实现:
        · ResourceBundleMessageSource, 建立在标准的 ResourceBundle 之上.
        · ReloadableResourceBundleMessageSource, 能够重新加载消息定义而无需重启 VM.
## ApplicationEventPublisher 解读
    封装时间发布功能的接口. 作为 ApplicationContext 的父接口.
## EnvironmentCapable 解读
		一个包含和暴露出 Environment 引用的接口.
    	所有 Spring application 都是 EnvironmentCapable, 并且该接口主要用于 instanceof 在接受 BeanFactory 实例的框架方法中
    执行检查, 这些实例实际上也可能不是 ApplicationContext 实例, 以便在 environment 可用时与 environment 进行交互.
    	如上所述, ApplicationContext 继承了 EnvironmentCapable, 从而暴露出一个 getEnvironment() 方法; 然而,
    ConfigurableApplicationContext 重新定义了 getEnvironment() 和缩小签名以返回 ConfigurableEnvironment. 结果是, 在从
    ConfigurableApplicationContext 访问 Environment 对象之前, 它是"只读"的, 此时也可配置.
## BeanFactory 解读
    	用于访问 Spring bean 容器的 root 接口. 这是 bean 容器的基本客户端视图; 进一步的接口, 如 ListableBeanFactory 和
    ConfigurableBeanFactory, 可用于特定的目的.
    	此接口由包含许多 bean 定义的对象实现, 每个 bean 定义由 String 名称唯一标识. 根据 bean 定义, 工厂将返回包含对象的独立实例
    (Prototype 设计模式) 或单个共享实例 (Singleton 设计模式的高级替代, 其中实例是单例工厂范围). 将返回哪种类型的实例取决与 bean
    工厂配置: API 是相同的. 从 Spring 2.0 开始, 根据具体的 application context (如, Web 环境中 "request" and "session"
    范围), 可以使用更多的范围.
    	这种方法的重点是 BeanFactory 是 application component 的中央注册表, 并集中 application component 的配置(如, 不再
    需要单个对象读取属性文件). 有关此方法的优点的讨论, 请参见 《Expert One-on-One J2EE Design and Development》第4章和第11章.
    	请注意, 通常最好依靠依赖注入 ("push" 配置) 来通过 setter 或构造函数来配置应用程序对象, 而不是像 BeanFactory 查找一样使用
    任何形式的 "pull" 配置. Spring 的依赖注入功能是使用这个 BeanFactory 接口及其子接口实现的.
    	通常, BeanFactory 将加载存储在配置源 (如,XML 文档) 中的 bean 定义, 并使用 org.springframework.beans 包来配置 bean.
    但是, 实现可以直接在 Java 代码中返回它创建的 Java 对象. 对如何存储定义没有限制: LDAP, RDBMS, XML, properties 文件等. 鼓励
    实现支持 bean 之间的引用 (依赖注入).
    	与 ListableBeanFactory 中的方法相反, 如果是 HierarchicalBeanFactory, 此接口中的所有操作也将检查父工厂. 如果在此工厂
    的实例中找不到 bean, 则会询问直接父工厂. 此工厂实例中的 bean 应该在任何父工厂中覆盖同名的 bean.
    	Bean 工厂实现应尽可能支持标准 bean 生命周期接口. 完整的初始化方法及其标准顺序是:
        	1. BeanNameAware#setBeanName
        	2. BeanClassLoaderAware#setBeanClassLoader
        	3. BeanFactoryAware#setBeanFactory
        	4. EnvironmentAware#setEnvironment
        	5. EmbeddedValueResolverAware#setEmbeddedValueResolver
        	6. ResourceLoaderAware#setResourceLoader (仅适用于在 application context 中运行时)
        	7. ApplicationEventPublisherAware#setApplicationEventPublisher (仅适用于在 application context 中运行时)
        	8. MessageSourceAware#setMessageSource (仅适用于在 application context 中运行时)
        	9. ApplicationContextAware#setApplicationContext (仅适用于在 application context 中运行时)
        	10. ServletContextAware#setServletContext (仅适用于在 application context 中运行时)
        	11. BeanPostProcessors 中的 postProcessBeforeInitialization 方法
        	12. InitializingBean#afterPropertiesSet
        	13. 自定义的  init-method 定义
        	14. BeanPostProcessors 中的 postProcessAfterInitialization 方法
    	
        在关闭 bean 工厂时, 以下生命周期方法将被应用:
        	1. DestructionAwareBeanPostProcessors 中的 postProcessBeforeDestruction 方法
        	2. DisposableBean#destroy
        	3. 自定义的 destroy-method 定义
## ListableBeanFactory 解读
		BeanFactory 接口的扩展由 bean 工厂实现, 可以枚举所有的 bean 实例, 而不是按客户端的请求逐个尝试按名称查找 bean. 预加载所有
    bean 定义 (如, 基于 XML 的工厂) 的 BeanFactory 实现可以实现此接口.
    	如果是 HierarchicalBeanFactory, 则不会考虑加载任何 BeanFactory 层次结构中的 bean, 只是注册涉及当前工厂的 bean. 使用
    辅助类 BeanFactoryUtils 来考虑祖先工厂中的 bean.
    	此接口中的方法将仅考虑此工厂的 bean 定义. 它们会忽略通过其它手段 (如,  ConfigurableBeanFactory#registerSingleton 方法)
    注册的单例 beans, 除了手动注册的单例 beans 的 getBeanNamesOfType() 和 getBeansOfType() 之外. 当然, BeanFactory#getBean
    也允许同名访问这些特殊的 bean. 但是, 在典型的场景中, 所有 bean 定义将被定义在外部, 因此大多数应用程序不需要担心这种区别.
		注意: 除了 getBeanDefinitionCount and containsBeanDefinition 之外, 这个接口中的方法不是为了频繁调用而设计的, 执行可能
    很慢.
## HierarchicalBeanFactory 解读
		bean 工厂的子接口, 可以是层次结构的一部分.
    	bean 工厂中的 setParentBeanFactory 方法与 ConfigurableBeanFactory#setParentBeanFactory 相对应, 允许以可配置方式
    设置父级的 bean 工厂
## ResourceLoader 解读
		用于加载资源的策略接口（例如，类路径或文件系统资源）。ApplicationContext 需要提供这样的功能, 再加上扩展
    ResourcePatternResolver 支持.
    	DefaultResourceLoader 是一个独立的实现, 可以在 ApplicationContext 之外使用, 也可以使用 ResourceEditor.
    	使用特定 context 的资源加载策略, 在 ApplicationContext 中运行时, Resource 和 Resource 数组类型的 bean 属性可以从
    Strings 填充.
## ResourcePatternResolver 解读
		用于将 location pattern (如, Ant-style 路径模式) 解析为 Resource 对象的策略接口.
		这是 ResourceLoader 接口的扩展. 传入的 ResourceLoader (如, 在 context 中运行时 ApplicationContext 传入的
    ResourceLoaderAware) 可以检查它是否也实现了这个扩展接口
		PathMatchingResourcePatternResolver 是一个独立的实现，可以在 ApplicationContext 外部使用，也可以用于
    ResourceArrayPropertyEditor 填充 Resource 数组 bean 属性。
    	可以与任何类型的未知模式一起使用 (如, "/WEB-INF/*-context.xml"): 输入模式必须与策略实现模式相匹配. 此接口仅指定转换方法
    而不是特定的模式格式.
    	此接口还为类路径中的所有匹配资源建议新的资源前缀 "classpath*:". 请注意, 在这种情况下, 资源位置应该是没有占位符的路径 (如,
    "/beans.xml"); Jar 文件或类目录可以包含多个同名文件.
## ContextClosedEvent 解读
	ApplicationContext关闭时引发的事件。
## ApplicationContextEvent 解读
	ApplicationContext 引发事件的基类
## ApplicationEvent 解读
	所有应用程序事件都要扩展的类。摘要因为直接发布通用事件没有意义。
## EventObject 解读
	从中派生所有事件状态对象的根类。
    所有事件都是通过对对象的引用构建的，“source”在逻辑上被认为是最初发生事件的对象.
## DefaultResourceLoader 解读
		ResourceLoader 接口的默认实现。使用于 ResourceEditor 和 作为  AbstractApplicationContext 的基类. 也可以单独使用.
		如果 location 值是一个 URL, 将返回 UrlResource; 如果是非 URL 路径或 "classpath:" 伪 URL 路径, 将返回
    ClassPathResource.
## PathMatchingResourcePatternResolver 解读
    	一个 ResourcePatternResolver 的实现, 能够将指定资源的 location path 解析为一个或多个 Resources. 源路径可以是与目标
    Resource 具有一对一映射的简单路径, 或者可以包含特殊的 "classpath*:" 前缀 and/or Ant-style 正则表达式 (使用 Spring 的
    AntPathMatcher 工具匹配). 后者都是有效的通配符.
    
    	没有通配符: 在简单的情况下, 如果指定的位置的路径不以 "classpath*:" 前缀开头, 并且不包含 PathMatcher 模式, 则此解析器将仅
    通过调用底层的 ResourceLoader#$getResource() 返回一个单独的资源. 如, 真实的 URLs(如, "file:C:/context.xml"), 伪 URLs
    (如, "classpath:/context.xml"), 简单的无前缀路径(如, "/WEB-INF/context.xml"). 后者将以特定于底层 ResourceLoader 的
    方式解析(如, ServletContextResource 用于 WebApplicationContext).
    
    	Ant-Style 模式: 当位置路径中包含 Ant-Style 模式时, 如:
```
 /WEB-INF/*-context.xml
 com/mycompany/**/applicationContext.xml
 file:C:/some/path/*-context.xml
 classpath:com/mycompany/**/applicationContext.xml
```
    	解析器将遵循一个更复杂但定义好的过程来尝试解析通配符. 它生成一个 Resource 直到最后一个非通配符段的路径并从中获取 URL. 如果
    URL 不是一个 "jar:" URL 或者容器专用的变种(如, WebLogic 的 "zip:", WebSphere 的 "wsjar" 等), 则从中获得一个
    java.io.File.并通过遍历文件系统来解析通配符. 如果是一个 jar URL, 解析器从中获取一个 java.net.JarURLConnection, 或者手动
    解析这个 jar URL,然后遍历 jar 文件内容, 以解析通配符.
    
    	可移植性启示:
        	如果指定的路径已经是文件 URL(显示或隐式, 因为 base ResourceLoader 是文件系统), 那么通配符保证以完全可移植的方式工作.
    		如果指定的路径是类路径位置, 则解析器必须通过调用 Classloader.getResource() 来获得最后一段非通配符路径 URL. 由于这
        只是路径的一个节点 (不是最后的文件), 因此在这种情况下, 实际上未定义 (在 ClassLoader 中) 究竟返回了什么类型的 URL. 
        实际上, 它通常是一个代表目录的 java.io.File (意味着 classpath 资源解析为文件系统位置), 或者某种类型的 jar URL
        (意味着 classpath 资源解析为文件 jar 位置). 尽管如此, 这个操作仍然存在可移植性问题.
    
    	"classpath*:" 前缀:
    		特别支持通过 "classpath*:" 前缀检索具有相同名称的多个类路径资源. 如, "classpath*:META-INF/beans.xml" 将在
        类路径中找到所有的 "bean.xml" 文件, 无论是在 "classes" 目录还是 jar 文件中. 这对于在每个 jar 文件中的相同位置自动
        检测同名的配置文件特备有用. 在内部, 这通过调用 Classloader.getResources() 发生, 并且完全可移植.
        	"classpath*:" 前缀也可以与位置路径的其余部分中的 PathMatcher 模式组合. 如, "classpath*:META-INF/*-bean.xml".
        在这种情况下, 解析策略非常简单: 在最后一个非通配符路径段上 调用 Classloader.getResources() 来获取类加载器层次结构中的
        所有匹配资源. 然后关闭每个资源, 使用与上述相同的解析策略的 PathMatcher 解析通配符子路径.
        
        其它说明:
        	警告: 请注意, "classpath*:" 与 Ant-style 模式结合使用时, 只能在模式启动前与至少一个根目录可靠地工作, 除非实际目标
        文件驻留在文件系统中. 这意味着 "classpath*:*.xml" 这样的模式不会从 jar 文件的根目录中检索文件, 而只能从扩展目录的根目录
        中检索文件. 这源于 JDK Classloader.getResources() 方法的限制, 该方法仅在传入空字符串时返回文件系统位置(指示搜索的潜在
        根).ResourcePatternResolver 实现尝试通过 URLClassloader 自我检查和 "java.class.path" 清单评估来缓解 jar 根查找
        限制; 但是,没有可移植性保证.
        	警告: 如果要搜索的根包在多个类路径中可用, 则不保证具有 "classpath:" 资源的 Ant-Style 模式可以找到匹配的资源.
        这是因为资源如 "com/mycompany/package1/service-context.xml", 可能只在一个位置, 但是当一个路径如
        "classpath:com/mycompany/**/service-context.xml", 被用于尝试解析时, 解析器将工作在
        getResource("com/mycompany")返回的 (第一个) URL 上; 如果基本包节点不存在于多个类加载器位置中, 则实际的最终资源
        可能不再下面. 因此, 在这样的情况下, 使用的 "classpath*:" 最好与 Ant-style 模式一致, 这将搜索包含根路径的所有类路径位置.
## PathMatcher 解读
		基于 String 路径匹配的策略接口.
        使用于 PathMatchingResourcePatternResolver, AbstractUrlHandlerMapping, and WebContentInterceptor.
        默认实现是 AntPathMatcher, 支持 Ant-style 模式语法.
## AntPathMatcher 解读
		Ant-Style 路径模式的 PathMathcer 实现.
        部分映射代码从 Apache Ant 中借用.
        映射使用以下规则匹配 URL:
        	· ? 匹配一个字符
            · * 匹配一个或多个字符
            · ** 匹配路径上的一个或多个目录
            · {spring: [a-z]+} 匹配正则表达式 [a-z]+ 作为一个变量名为 spring 的路径.
    	例子:
        	· com/t?st.jsp  	--->   	com/test.jsp | com/tast.jsp | com/txst.jsp
            · com/*.jsp     	--->   	com 目录中的所有 jsp 文件
            · com/**/test.jsp	--->	com 路径下的所有 test.jsp 文件
            · org/springframework/**/*.jsp	--->	org/springframework 路径下的所有 jsp 文件
            · org/**/servlet/bla.jsp		--->	org/springframework/servlet/bla.jsp | org/servlet/bla.jsp |
        org/springframework/testing/servlet/bla.jsp
        	· com/{filename:\\w+}.jsp		--->	匹配 com/test.jsp 并且将变量 filename 赋值为 "test"
    
    	注意: 模式和路径必须都是绝对的或者相对的, 以便两者匹配. 因此, 建议此实现的用户清理模式, 以便在它们使用的 context 中
    使用 "/" 作为前缀.
	DefaultListableBeanFactory 继承体系
![DefaultListableBeanFactory 继承体系](image/DefaultListableBeanFactory继承体系.png)
## DefaultListableBeanFactory 解读
		Spring 的 ConfigurableListableBeanFactory and BeanDefinitionRegistry 接口的默认实现: 一个基于 bean 定义
    元数据的完整 bean 工厂, 可以通过 后置处理器 扩展.
    	典型的用法是在访问 bean 之前首先注册所有 bean 定义(可能从 bean 定义文件中读取). 因此, 按名称进行 bean 查找是本地 bean
    定义表中的廉价操作, 对预解析的 bean 定义元数据对象进行操作.
    	请注意, 特定于 bean 定义格式的 reader 通常是单独实现的, 而不是作为 bean 工厂子类实现的: 如,
     PropertiesBeanDefinitionReader and XmlBeanDefinitionReader.
     	有关 ListableBeanFactory 接口的替代实现, 请查看 StaticListableBeanFactory, 它管理现有的 bean 实例, 而不是基于
    bean 定义创建新的实例.
## ConfigurableListableBeanFactory 解读
		配置接口由大多数可列出的 bean 工厂实现。除了 ConfigurableBeanFactory 之外，它还提供了分析和修改 bean 定义以及
    预先实例化单例的工具。
    	BeanFactory 的这个子接口并不适用于普通的应用程序代码：针对典型用例，坚持使用 BeanFactory 或 ListableBeanFactory。
    这个接口只是为了允许框架内部的即插即用，即使需要访问bean工厂配置方法。
## AutowireCapableBeanFactory 解读
		BeanFactory 接口的扩展由能够自动装配的 bean 工厂实现，前提是它们希望为现有 bean 实例公开此功能。
    	BeanFactory 的这个子接口并不适用于普通的应用程序代码：针对典型用例，坚持使用 BeanFactory 或 ListableBeanFactory。
        其他框架的集成代码可以利用此接口来连接和填充 Spring 无法控制其生命周期的现有 Bean 实例。例如，这对 WebWork Actions
    和 Tapestry Page 对象特别有用。
    	请注意，ApplicationContext 外观并未实现此接口，因为应用程序代码几乎不使用它。也就是说，它也可以从应用程序上下文中获得，
    可以通过 ApplicationContext的ApplicationContext.getAutowireCapableBeanFactory() 方法访问。
    	您还可以实现 BeanFactoryAware 接口，该接口即使在 ApplicationContext 中运行时也会公开内部 BeanFactory，以访问
    AutowireCapableBeanFactory：简单地将传入的 BeanFactory 转换为 AutowireCapableBeanFactory。
## SingletonBeanRegistry 解读
		为共享 bean 实例定义注册表的接口。可以通过 BeanFactory 实现来实现，以便以统一的方式公开其单例管理工具。
        ConfigurableBeanFactory 接口扩展了此接口。
## ConfigurableBeanFactory 解读
		配置接口由大多数 bean 工厂实现。除了 BeanFactory 接口中的 bean 工厂客户机方法之外，还提供配置 Bean 工厂的工具。
    	BeanFactory 的这个子接口并不适用于普通的应用程序代码：针对典型用例，坚持使用 BeanFactory 或 ListableBeanFactory。
    这个接口只是为了允许框架内部的即插即用，即使需要访问bean工厂配置方法。
#### 
    AutowireCandidateResolver 的继承体系
![AutowireCandidateResolver 继承体系](image/AutowireCandidateResolver继承体系.png)
## AutowireCandidateResolver 解读
    用于确定特定 bean 定义是否有资格作为特定依赖关系的自动线候选的策略接口。
## SimpleAutowireCandidateResolver 解读
	AutowireCandidateResolver 没有注释支持时使用的实现。此实现仅检查 bean 定义。
## AbstractAutowireCapableBeanFactory 解读
		Abstract bean 工厂的超类，它实现了默认的 bean 创建，具有 RootBeanDefinition 类指定的全部功能。除了
    AbstractBeanFactory#createBean(java.lang.Class <T>) 方法之外，还实现了 AutowireCapableBeanFactory 接口。
        提供 bean 创建（具有构造函数解析），属性填充，装配（包括自动装配）和初始化。处理运行时 bean 引用，解析托管集合，
    调用初始化方法等。支持自动装配构造函数，按名称的属性和按类型的装配属性。
    	子类实现的主要模板方法是 AutowireCapableBeanFactory.resolveDependency(DependencyDescriptor，String，Set，
    TypeConverter)，用于按类型自动装配. 在工厂能够搜索其 bean 定义的情况下，匹配 bean 通常将通过这种搜索来实现。对于其他工厂
    样式，可以实现简化的匹配算法。
    	请注意，此类不承担或实现 bean 定义注册表功能。有关 ListableBeanFactory 和 BeanDefinitionRegistry 接口的实现，
    请参阅 DefaultListableBeanFactory，它们分别代表此类工厂的 API 和 SPI 视图。
#### 
    CglibSubclassingInstantiationStrategy 继承体系
![CglibSubclassingInstantiationStrategy 继承体系](image/CglibSubclassingInstantiationStrategy继承体系.png)
## CglibSubclassingInstantiationStrategy 解读
    在 BeanFactories 中使用的对象实例化的默认策略.
    如果容器需要覆盖方法以实现方法注入, 则使用 CGLIB 动态生成子类.
## SimpleInstantiationStrategy 解读
    在 BeanFactories 中使用的对象实例化的简单策略.
    不支持方法注入, 尽管它提供了子类的 hooks 来覆盖以添加方法注入支持, 如通过重写方法.
## InstantiationStrategy 解读
	负责创建与 root bean 定义相对应的实例接口.
    随着各种方法的实现, 这被拉入战略, 包括使用 CGLIB 动态创建子类支持方法注入.
#### 
    DefaultParameterNameDiscoverer 继承体系
![DefaultParameterNameDiscoverer 继承体系](image/DefaultParameterNameDiscoverer继承体系.png)
## DefaultParameterNameDiscoverer 解读
    	ParameterNameDiscoverer 策略接口的默认实现, 使用 Java 8 标准反射机制 (如果可用), 并回退到基于 ASM 的
    LocalVariableTableParameterNameDiscoverer, 以检查类文件中的调试信息.
    	如果存在 Kotlin 反射实现, 则首先在列表中添加 KotlinReflectionParameterNameDiscoverer, 并用于 Kotlin 类和接口.
    当编译和运行一个 Graal 本地图片时, 没有 ParameterNameDiscoverer 被使用.
    	可以通过 PrioritizedParameterNameDiscoverer.addDiscoverer(ParameterNameDiscoverer) 添加更多的发现者.
## PrioritizedParameterNameDiscoverer 解读
    	ParameterNameDiscoverer 实现连续尝试几个发现者委托. 首先在 addDiscoverer 方法中添加的那些具有最高优先级.
    如果一个返回 null, 则将尝试下一个.
    	如果没有 discoverer 匹配, 默认返回 null.
## ParameterNameDiscoverer 解读
    	用于发现方法和构造器的参数名称的接口.
        参数名称发现并不总是可行，但可以尝试各种策略，例如查找可能在编译时发出的调试信息，以及查找可选的 AspectJ 注解方法的 argname
    注解值。
## AbstractBeanFactory 解读
		BeanFactory 实现的抽象基类，提供 ConfigurableBeanFactory SPI 的全部功能。不假设可列出的bean工厂：因此，也可以用作
    bean 工厂实现的基类，它从一些后端资源获取 bean 定义（其中 bean 定义访问是一项昂贵的操作）。
    	该类提供单例缓存(通过其基类 DefaultSingletonBeanRegistry，singleton/prototype 确定)，FactoryBean 处理，别名，
    子 bean 定义的 bean 定义合并，bean销毁（DisposableBean 接口，自定义销毁方法）。此外，它可以通过实现
    HierarchicalBeanFactory 接口来管理 bean 工厂层次结构（在未知 bean 的情况下委托给父级）。
    	子类实现的主要模板方法是 getBeanDefinition(java.lang.String) 和 createBean(java.lang.String，
    org.springframework.beans.factory.support.RootBeanDefinition，java.lang.Object [])，分别检索一个给定 bean 名称的
    bean 定义，为给定的 bean 定义创建 bean 实例。可以在 DefaultListableBeanFactory 和
    AbstractAutowireCapableBeanFactory 中找到这些操作的默认实现。
## FactoryBeanRegistrySupport 解读
    支持需要处理 FactoryBean 实例的单例注册表的基类，与 DefaultSingletonBeanRegistry 的单例管理集成.
    用作 AbstractBeanFactory 的基类。
## DefaultSingletonBeanRegistry 解读
    	共享 bean 实例的泛型注册表，实现 SingletonBeanRegistry。允许通过 bean 名称注册应该为注册表的所有调用者共享的单例实例。
    	还支持在关闭注册表时销毁 DisposableBean实 例（可能对应于已注册的单例，也可能不对应于已注册的单例）。可以注册 bean 之间
    的依赖关系以强制执行适当的关闭顺序。
    	该类主要用作 BeanFactory 实现的基类，分解了单例 bean 实例的通用管理。请注意，ConfigurableBeanFactory 接口扩展了
    SingletonBeanRegistry 接口。
    	请注意，与 AbstractBeanFactory 和 DefaultListableBeanFactory（继承自它）相比，此类既不假定 bean 定义概念也不假定
    bean 实例的特定创建过程。或者也可以用作委托的嵌套助手。
## SimpleAliasRegistry 解读
		简单实现 AliasRegistry 接口。用作 BeanDefinitionRegistry 实现的基类。
## AnnotatedBeanDefinitionReader 解读
		方便的适配器，用于注解 bean 类的编程注册。这是 ClassPathBeanDefinitionScanner的替代方法，应用相同的注解分辨率，
    但仅适用于显式注册的类。
## AnnotationBeanNameGenerator 解读
		BeanNameGenerator 实例用于生成 bean classes 的 beanName, beans classes 上有如下注解声明:
        	1. @Component 注解
        	2. 以 @Component 作为元注解的注解
    	如, Spring 的 stereotype 注解(如, @Repository), 它本身有 @Component 注解声明.
        还支持 JavaEE 6 的 ManagedBean 和 JSR-330 的 Named 注解(如果可用). 请注意, Spring component 注解总是覆盖这些标准
    注解.
    	如果注解的 value 没有设置一个 beanName, 则根据类的短名称 (第一个字母为小写) 构建适当的 beanName. 如:
```
com.xyz.FooServiceImpl -> fooServiceImpl
```
## BeanNameGenerator 解读
    用于为 bean 定义生成 beanName 的策略接口.
## AnnotationScopeMetadataResolver 解读
    ScopeMetadataResolver 实现, 默认情况下检查 bean 类上是否存在 Spring 的 @Scope 注解.
    检查的确切的注解类型可通过 setScopeAnnotationType(Class) 配置.
## ScopeMetadataResolver 解读
	用于解析 bean 定义范围的策略接口.
## ScopedProxyMode 解读
	枚举各种范围代理选项.
    有关 scoped proxy 的确切内容的更全面的讨论, 请参阅 Spring 参考文档的 'Scoped beans as dependencies' 部分.
## ScopeMetadata 解读
    描述 Spring 管理的 bean 的 scope 特征, 包括范围名称和范围代理的行为.
    默认的 Scope 是 "singleton", 默认情况下不创建 scoped-proxy.
## StandardEnvironment 解读
		适用于“标准”（即非Web）应用程序的环境实现。
        除了可配置环境的常用功能（如属性解析和与配置文件相关的操作）之外，此实现还配置两个默认属性源，按以下顺序搜索：
        	1. system properties
        	2. system environment variables
        也就是说，如果 key “xyz”既存在于 JVM 系统属性中，也存在于当前进程的环境变量集中，则调用
    environment.getProperty("xyz")时, 将返回系统属性中的 key “xyz” 的值. 默认情况下会选择此排序，因为系统属性是 per-JVM，
    而环境变量在给定系统上的许多 JVM 上可能是相同的。赋予系统属性优先权允许基于每个 JVM 覆盖环境变量。
    	可以删除，重新排序或替换这些默认属性源;可以使用 MutablePropertySources实例中提供的
    AbstractEnvironment.getPropertySources()添加其他属性源。用法请看 ConfigurableEnvironment javadoc 文档.
    	有关在　shell　环境（例如Bash）中特殊处理属性名称的详细信息，请参阅　SystemEnvironmentPropertySource javadoc，
    该环境禁止使用变量名称中的句点字符。
##　AbstractEnvironment 解读
		Environment 实现的抽象基类。支持保留的默认配置文件名称的概念，并允许通过 ACTIVE_PROFILES_PROPERTY_NAME 和
    DEFAULT_PROFILES_PROPERTY_NAME 属性指定活动和默认配置文件。
    	具体的子类主要区别在于它们默认添加的 PropertySource 对象。AbstractEnvironment 不添加. 子类应通过 protected 的
    customizePropertySources(MutablePropertySources) hook 提供属性源，而客户端应使用自定义的
    ConfigurableEnvironment.getPropertySources() 并对 MutablePropertySources API 进行操作。用法请看
    ConfigurableEnvironment javadoc.
## ConfigurableEnvironment 解读
		配置接口由大多数 Environment 类型实现。提供用于设置 active 和 default profiles 文件以及操作基础属性源的工具。
    允许客户端通过 ConfigurablePropertyResolver 超级接口设置和验证所需的属性，自定义转换服务等。
    	操纵属性资源:
        	属性源可能被删除，重新排序或替换;可以使用从 getPropertySources() 返回的 MutablePropertySources 实例添加
        其他属性源。以下示例针对 ConfigurableEnvironment 的 StandardEnvironment 实现，但通常适用于任何实现，但特定的
        默认属性源可能不同。
```java
    例如: 使用最好的搜索优先级添加一个属性源
    ConfigurableEnvironment environment = new StandardEnvironment();
    MutablePropertySources propertySources = environment.getPropertySources();
    Map<String, String> myMap = new HashMap<>();
    myMap.put("xyz", "myValue");
    propertySources.addFirst(new MapPropertySource("MY_MAP", myMap));
```
```java
	例如: 删除默认系统属性的属性源
    MutablePropertySources propertySources = environment.getPropertySources();
	propertySources.remove(StandardEnvironment.SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME)
```
```java
	例如: 模拟系统环境以进行测试
    MutablePropertySources propertySources = environment.getPropertySources();
	MockPropertySource mockEnvVars = new MockPropertySource().withProperty("xyz", "myValue");
	propertySources.replace(StandardEnvironment.SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, mockEnvVars);
```
		当 ApplicationContext 正在使用 Environment 时，在调用 context 的 refresh() 方法之前执行任何此类
    PropertySource 操作都很重要。这可确保在容器引导过程中所有属性源都可用，包括属性占位符配置器使用。
## Environment 解读
		表示当前应用程序正在运行的环境的接口。模拟应用程序环境的两个关键方面：profiles and properties. 与属性访问相关的方法
    通过 PropertyResolver 超接口公开。
    	profiles 是仅在给定 profiles 处于活动状态时才向容器注册的 Bean 定义的命名逻辑组。可以将 Bean 分配给 profiles，
    无论是以 XML 还是通过注解定义; 查看 Spring-beans 3.1 schema 或者 @Profile 注解了解详细语法. 与 profiles 相关的
    Environment 对象的作用是确定哪些 profiles（如果有）当前处于活动状态，以及默认情况下哪些 profiles（如果有）应处于活动状态。
    	Properties 在几乎所有应用程序中都发挥着重要作用，并且可能源自各种来源：properties files, JVM system properties,
    system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects, Maps 等. 与属性相关
    的 Environment 对象的作用是为用户提供方便的服务接口，用于配置属性源和从中解析属性。
    	在 ApplicationContext 中管理的 Bean 可以注册为 EnvironmentAware 或 @Inject the Environment，以便直接查询
    profile 状态或解析属性。
    	但是，在大多数情况下，应用程序级的 bean 不需要直接与 Environment 交互，而是可能必须将 ${...} 属性值替换为属性占位符
    配置器，例如 PropertySourcesPlaceholderConfigurer，它本身就是 EnvironmentAware，并且是使用
    <context:property-placeholder/>时，默认情况下会注册 Spring 3.1。
    	必须通过 ConfigurableEnvironment 接口完成环境对象的配置，该接口从所有 AbstractApplicationContext 子类的
    getEnvironment() 方法返回。有关在 application context refresh() 之前演示属性源操作的用法示例，请参阅
    ConfigurableEnvironment Javadoc。
## ConfigurablePropertyResolver 解读
		配置接口由大多数 PropertyResolver 类型实现。提供用于访问和自定义将属性值从一种类型转换为另一种类型时使用的
    ConversionService 的工具。
## PropertyResolver 解读
		用于解析任何基础源的属性的接口。
## MutablePropertySources　解读
		PropertySources 接口的默认实现. 允许操作包含的属性源，并提供用于复制现有 PropertySources 实例的构造函数。
        在诸如 addFirst(org.springframework.core.env.PropertySource<?>) and
    addLast(org.springframework.core.env.PropertySource<?>) 等方法中提到优先级的情况下，这是关于属性的顺序使用
    PropertyResolver 解析给定属性时将搜索源。
## PropertySourcesPropertyResolver 解读
	PropertyResolver 的实现，它针对一组基础的 PropertySources 解析属性值。
## PropertiesPropertySource 解读
		从 Properties 对象中提取属性的 PropertySource 实现。
        请注意，因为 Properties 对象在技术上是<Object，Object> Hashtable，所以可以包含非String键或值。但是，此实现仅限于
    访问基于 String 的键和值，方法与 Properties.getProperty（java.lang.String）和
    Properties.setProperty（java.lang.String，java.lang.String）相同。
## SystemEnvironmentPropertySource 解读
		MapPropertySource 的专业化设计用于系统环境变量。补偿 Bash 和其他 shell 中的约束，这些 shell 不允许包含句点字符和/或
    连字符的变量; 还允许对属性名称进行大写变体，以便更加惯用 shell 使用。
    	例如, 调用 getProperty("foo.bar") 将尝试从源属性中寻找一个值,或任何'等量'属性, 返回第一个发现:
        	1. foo.bar - 原始名称
        	2. foo_bar - 带有下划线的（如果有的话）
        	3. FOO.BAR - 原始值的大写
        	4. FOO_BAR - 原始值大写的下划线连接
        上述的任何连字符变体都可以使用，甚至可以混合使用点/连字符变体。
        这同样适用于对 containsProperty（String）的调用，如果存在任何上述属性，则返回 true，否则返回 false。
        将活动或默认 profiles 指定为环境变量时，此功能特别有用。在 Bash 下不允许以下内容：
```java
spring.profiles.active=p1 java -classpath ... MyApp
```
		但是，允许使用以下语法，这也是更常规的：
```java
SPRING_PROFILES_ACTIVE=p1 java -classpath ... MyApp
```
		为此类（或包）启用调试或跟踪级别日志记录，以获取解释何时发生这些“属性名称解析”的消息。
        默认情况下，此属性源包含在 StandardEnvironment 及其所有子类中。
## PropertySourcesPlaceholderConfigurer 解读
		PlaceholderConfigurerSupport 的专业化，它针对当前 Spring 环境及其 PropertySources 集解析 bean 定义属性值和
    @Value 注解中的 ${...} 占位符。
    	此类被设计为 PropertyPlaceholderConfigurer 的一般替代品。默认情况下，它用于支持 property-placeholder 元素处理
    spring-context-3.1 或更高版本的 XSD; spring-context versions <= 3.0 默认为 PropertyPlaceholderConfigurer 以
    确保向后兼容性。有关完整的详细信息，请参阅 spring-context XSD 文档。
    	任何本地属性（例如通过PropertiesLoaderSupport.setProperties（java.util.Properties）添加的属性，
    PropertiesLoaderSupport.setLocations（org.springframework.core.io.Resource ...）等）都将添加为PropertySource。
    本地属性的搜索优先级基于 localOverride 属性的值，默认情况下为 false，表示在所有环境属性源之后最后搜索本地属性。
    	有关操作环境属性源的详细信息，请参阅 ConfigurableEnvironment 和相关的javadoc。
## ConversionService 解读
## PropertyPlaceholderConfigurer 解读
## AnnotationConfigWebApplicationContext 解读
## ConfigurationClassPostProcessor 解读
## BeanFactoryPostProcessor 解读
## ImportSelector 解读
## ImportBeanDefinitionRegistrar 解读
## BeanPostProcessor 解读
## AutowiredAnnotationBeanPostProcessor 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读
## 解读

## AnnotationConfigUtils 解读











