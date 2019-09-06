
## @RequestMapping 解读
		使用灵活方法签名将Web请求映射到请求处理类中的方法的注释。
        Spring MVC和Spring WebFlux都通过 RequestMappingHandlerMapping 和 RequestMappingHandlerAdapter 在它们
    各自的模块和包结构中支持这个注释。有关每个支持的处理程序方法参数和返回类型的确切列表，请使用下面的参考文档链接：
			1. Spring MVC Method Arguments and Return Values
			2. Spring WebFlux Method Arguments and Return Values
		注意：此批注可以在类和方法级别使用。在大多数情况下，在方法级别，应用程序将更喜欢使用HTTP方法特定变体之一
    @GetMapping，@PostMapping，@PutMapping，@DelaMapping或@PatchMapping。
		注意：使用控制器接口（例如，用于AOP代理）时，请确保始终将所有映射注释（例如@RequestMapping 和
    @SessionAttributes）放在控制器接口而不是实现类上。
---
	属性:
    	1. String[] consumes: {}
			映射请求的可消耗媒体类型，缩小主映射。
            格式是单个媒体类型或媒体类型序列，只有在Content-Type与这些媒体类型之一匹配时才会映射请求。例子：
```java
 consumes = "text/plain"
 consumes = {"text/plain", "application/*"}
```
			使用“！”可以取消表达式。运算符，如“！text / plain”，它匹配除“text / plain”以外的Content-Type的所有请求。
            在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都会覆盖此消耗限制。

		2. String[] headers: {}
			映射请求的标头，缩小主映射。
			任何环境的格式相同：一系列“My-Header = myValue”样式表达式，只有在发现每个此类标题具有给定值时才会映射请求。
        使用“！=”运算符可以取消表达式，如“My-Header！= myValue”。还支持“My-Header”样式表达式，这些标题必须存在于请求
    	中（允许具有任何值）。最后，“！My-Header”样式表达式表明指定的标头不应该出现在请求中。
			还支持媒体类型通配符（*），用于诸如Accept和Content-Type之类的标题。例如，
```java
 @RequestMapping(value = "/something", headers = "content-type=text/*")
```
			将匹配请求的内容类型为“text / html”，“text / plain”等。
			在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都继承此标头限制（即，甚至在解析处理程序方法之前
        检查类型级别限制）。

		3. RequestMethod[] method: {}
			映射到的HTTP请求方法，缩小主映射：GET，POST，HEAD，OPTIONS，PUT，PATCH，DELETE，TRACE。
            在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都继承此HTTP方法限制（即，甚至在解析处理程序方法
        之前检查类型级别限制）。

		4. String name: ""
			为此映射指定名称。
            在类型级别和方法级别支持！在两个级别上使用时，组合名称通过串联以“＃”作为分隔符派生。

		5. String[] params: {}
			映射请求的参数，缩小主映射。
            任何环境的格式相同：一系列“myParam = myValue”样式表达式，只有在发现每个此类参数具有给定值时才会映射请求。
        使用“！=”运算符可以取消表达式，如“myParam！= myValue”。还支持“myParam”样式表达式，这些参数必须存在于请求中
        （允许具有任何值）。最后，“！myParam”样式表达式表明指定的参数不应该出现在请求中。
			在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都继承此参数限制（即，甚至在解析处理程序方法之前
        检查类型级别限制）。

		6. String[] path: {}
			路径映射URI（例如“/myPath.do”）。还支持Ant样式的路径模式（例如“/myPath/*.do”）。在方法级别，在类型级别
        表示的主映射内支持相对路径（例如“edit.do”）。路径映射URI可以包含占位符（例如“/ $ {connect}”）。
            在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都会继承此主映射，从而缩小特定处理程序方法的范围。

		7. String[] produces: {}
			映射请求的可生成媒体类型，缩小主映射。
            格式是单个媒体类型或媒体类型序列，只有在Accept与这些媒体类型之一匹配时才会映射请求。例子：
```java
 produces = "text/plain"
 produces = {"text/plain", "application/*"}
 produces = MediaType.APPLICATION_JSON_UTF8_VALUE
```
			它会影响写入的实际内容类型，例如，使用UTF-8编码生成JSON响应，应使用
        MediaType.APPLICATION_JSON_UTF8_VALUE。
        	使用“！”可以取消表达式。运算符，如“！text / plain”，它使用“text / plain”以外的Accept匹配所有请求。
            在类型级别和方法级别支持！在类型级别使用时，所有方法级别的映射都会覆盖此限制。

		8. String[] value: {}
			此注释表示的主要映射。
            这是path（）的别名。例如，@ RequestMapping（“/ foo”）等同于@RequestMapping（path =“/ foo”）。
            在类型级别和方法级别支持！在类型级别使用时，所有方法级别映射都会继承此主映射，从而缩小特定处理程序方法的范围。
### RequestMappingHandlerAdapter 解读
		支持@RequestMapping带注释的HandlerMethods的AbstractHandlerMethodAdapter的扩展。
        可以通过setCustomArgumentResolvers（
    java.util.List <org.springframework.web.method.support.HandlerMethodArgumentResolver>）和
    setCustomReturnValueHandlers（java.util.List <org.springframework.web.method）添加对自定义参数和返回值类型
    的支持。support.HandlerMethodReturnValueHandler>），或者，重新配置所有参数和返回值类型，使用
    setArgumentResolvers
    （java.util.List <org.springframework.web.method.support.HandlerMethodArgumentResolver>）和
    setReturnValueHandlers
    （java.util.List<org.springframework.web.method.support.HandlerMethodReturnValueHandler>）。
### RequestMappingHandlerMapping 解读
		从@Controller类中的类型和方法级@RequestMapping注释创建RequestMappingInfo实例。
## @ExceptionHandler 解读
		用于处理特定处理程序类和/或处理程序方法中的异常的注释。
        使用此注释注释的处理程序方法允许具有非常灵活的签名。它们可以按任意顺序具有以下类型的参数：
        	1. An exception argument: 声明为一般异常或更具体的例外。如果注释本身不通过其value（）缩小异常类型，这也
        可以作为映射提示。
        	2. Request and/or response objects (typically from the Servlet API). 您可以选择任何特定的请求/响应
        类型，例如ServletRequest / HttpServletRequest。
        	3. Session object: typically HttpSession. 此类型的参数将强制存在相应的会话。因此，这样的论证永远不会是
        空的。请注意，会话访问可能不是线程安全的，特别是在Servlet环境中：如果允许多个请求同时访问会话，请考虑将
        “synchronizeOnSession”标志切换为“true”。
        	4. WebRequest or NativeWebRequest.允许通用请求参数访问以及请求/会话属性访问，而不与本机Servlet API
        绑定。
        	5. Locale for the current request locale: 由可用的最具体的语言环境解析器确定，即Servlet环境中配置的
        LocaleResolver.
        	6. InputStream / Reader for access to the request's content. 这将是Servlet API公开的原始
        InputStream / Reader。
			7. OutputStream / Writer for generating the response's content. 这将是Servlet API公开的原始
        OutputStream / Writer。
        	8. Model: 作为从处理程序方法返回模型映射的替代方法。请注意，提供的模型不预先填充常规模型属性，因此始终为空，
        以便为特定于异常的视图准备模型。

		处理程序方法支持以下返回类型：
        	1. A ModelAndView object (from Servlet MVC).
        	2. Model object: 通过RequestToViewNameTranslator隐式确定视图名称。
        	3. Map object: 用于公开模型的Map对象，其中视图名称通过RequestToViewNameTranslator隐式确定。
        	4. View object.
        	5. String: 一个String值，它被解释为视图名称。
        	6. @ResponseBody注释方法（仅限Servlet）设置响应内容。返回值将使用消息转换器转换为响应流。
        	7. HttpEntity<?> or ResponseEntity<?> object: 用于设置响应头和内容的HttpEntity <？> 或
        ResponseEntity <？>对象（仅限Servlet）。ResponseEntity正文将被转换并使用消息转换器写入响应流。
			8. void: 如果方法处理响应本身（通过直接编写响应内容，为此目的声明类型为ServletResponse /
        HttpServletResponse的参数）或者是否应该通过RequestToViewNameTranslator隐式确定视图名称（不在处理程序中
        声明响应参数）方法签名）。

		您可以将ExceptionHandler注释与@ResponseStatus结合使用以获取特定的HTTP错误状态。
---
	属性:
    	1. Class<? extends Throwable>[] value: {}
    		注释方法处理的异常。如果为空，则默认为方法参数列表中列出的任何例外。
### ExceptionHandlerExceptionResolver 解读
		AbstractHandlerMethodExceptionResolver，它通过@ExceptionHandler方法解析异常。
		可以通过setCustomArgumentResolvers
    （java.util.List <org.springframework.web.method.support.HandlerMethodArgumentResolver>）和
    setCustomReturnValueHandlers（java.util.List <org.springframework.web.method）添加对自定义参数和返回值类型
    的支持。support.HandlerMethodReturnValueHandler>）。或者，要重新配置所有参数和返回值类型，请使用
    setArgumentResolvers
    （java.util.List <org.springframework.web.method.support.HandlerMethodArgumentResolver>）和
    setReturnValueHandlers（List）。
### ExceptionHandlerMethodResolver 解读
		发现给定类中的@ExceptionHandler方法，包括其所有超类，并帮助解决给定方法支持的异常类型的给定Exception。
## @ResponseStatus 解读
		使用应返回的状态代码（）和reason（）标记方法或异常类。
        调用处理程序方法时，状态代码将应用于HTTP响应，并覆盖通过其他方式设置的状态信息，如ResponseEntity
    或“redirect：”。
		警告：在异常类上使用此批注时，或者在设置此批注的reason属性时，将使用HttpServletResponse.sendError方法。
        使用HttpServletResponse.sendError，响应被认为是完整的，不应再写入。此外，Servlet容器通常会编写HTML错误页面，
    因此会使用不适合REST API的原因。对于这种情况，最好使用ResponseEntity作为返回类型，并完全避免使用@ResponseStatus。
		请注意，控制器类也可以使用@ResponseStatus进行注释，然后由所有@RequestMapping方法继承。
## @ResponseBody 解读
		指示方法返回值的注释应绑定到Web响应主体。支持带注释的处理程序方法。
        从版本4.0开始，此注释也可以添加到类型级别，在这种情况下，它是继承的，不需要在方法级别添加。
## @ControllerAdvice 解读
		@Component的专门化，用于声明要在多个@Controller类之间共享的@ExceptionHandler，@InitBinder 或
    @ModelAttribute方法的类。
		具有@ControllerAdvice的类可以显式声明为Spring bean，也可以通过类路径扫描自动检测。所有这些bean都通过
    AnnotationAwareOrderComparator进行排序，即基于@Order和Ordered，并在运行时以该顺序应用。对于处理异常，将使用匹配
    的异常处理程序方法在第一个通知上选择@ExceptionHandler。对于模型属性和InitBinder初始化，@ModelAttribute 和
    @InitBinder方法也将遵循@ControllerAdvice顺序。
		注意：对于@ExceptionHandler方法，在特定通知bean的处理程序方法中，根目录异常匹配将优先于匹配当前异常的原因。
    但是，优先级较高的建议的原因匹配仍然优先于较低优先级的通知bean上的任何匹配（无论是根目录还是原因级别）。因此，请在具有
    相应顺序的优先级通知bean上声明主根异常映射！
		默认情况下，@ControllerAdvice中的方法全局应用于所有控制器。使用selectors annotations（），
    basePackageClasses（）和basePackages（）（或其别名value（））来定义更窄的目标控制器子集。如果声明了多个选择器，
    则应用OR逻辑，这意味着所选的控制器应匹配至少一个选择器。请注意，选择器检查是在运行时执行的，因此添加许多选择器可能会对
    性能产生负面影响并增加复杂性。
---
	属性:
    	1. Class<? extends Annotation>[] annotations: {}
			注释数组。
            使用这个/其中一个注释注释的控制器将由@ControllerAdvice注释类辅助。
            考虑创建一个特殊的注释或使用预定义的注释，如@RestController。

		2. Class<?>[] assignableTypes: {}
            类的数组。
            可分配给至少一种给定类型的控制器将由@ControllerAdvice注释类辅助。

		3. Class<?>[] basePackageClasses: {}
			value（）的类型安全替代，用于指定包以选择由@ControllerAdvice注释类辅助的控制器。
            考虑在每个包中创建一个特殊的无操作标记类或接口，除了被该属性引用之外没有其它用途。

		4. String[] basePackages: {}
			基础包的数组。
            将包括属于那些基础包或其子包的控制器，例如：@ControllerAdvice（basePackages =“org.my.pkg”）或
        @ControllerAdvice（basePackages = {“org.my.pkg”，“org.my.other.pkg“}）。
        	value（）是此属性的别名，只是允许更简洁地使用注释。
            还要考虑使用basePackageClasses（）作为基于String的包名称的类型安全替代方法。

		5. String[] value: {}
			basePackages（）属性的别名。
            允许更简洁的注释声明，例如：@ControllerAdvice（“org.my.pkg”）等同于
        @ControllerAdvice（basePackages =“org.my.pkg”）。
## @InitBinder 解读
		标注，用于标识初始化WebDataBinder的方法，该方法将用于填充带注释的处理程序方法的命令和表单对象参数。
        这样的init-binder方法支持RequestMapping支持的所有参数，命令/表单对象和相应的验证结果对象除外。Init-binder
    方法不能有返回值;它们通常被宣布为无效。
    	典型的参数是WebDataBinder与WebRequest或Locale的组合，允许注册特定于上下文的编辑器。
---
	属性:
    	1. String[] value: {}
    		应该将此init-binder方法应用于的命令/表单属性和/或请求参数的名称。
            默认是应用于所有命令/表单属性以及由带注释的处理程序类处理的所有请求参数。这里指定模型属性名称或请求参数名称将
        init-binder方法限制为那些特定属性/参数，使用不同的init-binder方法通常应用于不同的属性或参数组。
## @ModelAttribute 解读
		将方法参数或方法返回值绑定到命名模型属性的注释，公开给Web视图。支持使用@RequestMapping方法的控制器类。
        可以使用特定的属性名称，通过注释@RequestMapping方法的相应参数，将命令对象公开给Web视图。
        也可以通过使用@RequestMapping方法在控制器类中注释访问器方法，将引用数据公开给Web视图。允许此类访问器方法具有
    @RequestMapping方法支持的任何参数，将模型属性值返回到公开。
    	但请注意，当请求处理导致异常时，Web视图无法使用引用数据和所有其他模型内容，因为可能在任何时候引发异常，从而使模型
    的内容不可靠。因此，@ExceptionHandler方法不提供对Model参数的访问。
---
	属性:
    	1. boolean binding: true
            允许直接在@ModelAttribute方法参数或从@ModelAttribute方法返回的属性上声明数据绑定，这两种方法都会阻止该
        属性的数据绑定。
        	默认情况下，此参数设置为true，在这种情况下应用数据绑定。将此设置为false可禁用数据绑定。

        2. String name: ""
        	要绑定的model属性的名称。
            默认模型属性名称是根据非限定类名称从声明的属性类型（即方法参数类型或方法返回类型）推断出来的：
        类“mypackage.OrderAddress”的“orderAddress”，或“List <mypackage.OrderAddress>”的“orderAddressList”。

	    3. String value: ""
			name（）的别名。
## @RestController 解读
		一个便利注释，它本身用@Controller和@ResponseBody注释。
        带有此批注的类型被视为控制器，其中@RequestMapping方法默认采用@ResponseBody语义。
        注意：如果配置了适当的HandlerMapping-HandlerAdapter对，则处理@RestController，例如
    RequestMappingHandlerMapping-RequestMappingHandlerAdapter对，它们是MVC Java配置和MVC命名空间中的缺省值。
---
	属性:
    	1. String value: ""
			该值可以指示对逻辑组件名称的建议，在自动检测的组件的情况下将其转换为Spring bean。
## @SessionAttributes 解读
		注释，指示特定处理程序使用的会话属性。
        这通常会列出应该透明地存储在会话中的模型属性的名称或某些会话存储，作为表单支持bean。在类型级别声明，应用于带注释的
    处理程序类操作的模型属性。
    	注意：使用此注释指示的会话属性对应于特定处理程序的模型属性，透明地存储在会话会话中。一旦处理程序指示其会话会话完成，
    将删除这些属性。因此，将此工具用于此类会话属性，这些属性应在特定处理程序的对话过程中临时存储在会话中。
		对于永久会话属性，例如用户身份验证对象，请改用传统的session.setAttribute方法。或者，考虑使用通用WebRequest接口
    的属性管理功能。
    	注意：使用控制器接口（例如，用于AOP代理）时，请确保始终将所有映射注释（例如@RequestMapping和
    @SessionAttributes）放在控制器接口而不是实现类上。
---
	属性:
    	1. String[] names: {}
			模型中应存储在会话或某些会话存储中的会话属性的名称。
            注意：这表示模型属性名称。会话属性名称可能与模型属性名称匹配，也可能不匹配。因此，应用程序不应依赖会话属性名称，
        而应仅对模型进行操作。

		2. Class<?>[] types: {}
			模型中应存储在会话或某些会话存储中的会话属性类型。
            无论属性名称如何，这些类型的所有模型属性都将存储在会话中。

		3. String[] value: {}
			names() 的别名。
## @Transactional 解读
		描述单个方法或类上的事务属性。
        在类级别，此批注作为默认应用于声明类及其子类的所有方法。请注意，它不适用于类层次结构中的祖先类;方法需要在本地重新
    声明才能参与子类级别的注释。
    	这个注释类型通常可以直接与Spring的RuleBasedTransactionAttribute类相比，实际上
    AnnotationTransactionAttributeSource将直接将数据转换为后一个类，因此Spring的事务支持代码不必知道注释。如果没有
    规则与异常相关，则将其视为DefaultTransactionAttribute（在RuntimeException和Error上回滚但不在已检查的异常上
    回滚）。
    	有关此批注的属性的语义的特定信息，请参阅TransactionDefinition和TransactionAttribute javadocs。
---
	属性:
    	1. Isolation isolation: org.springframework.transaction.annotation.Isolation.DEFAULT
			事务隔离级别。
            默认为Isolation.DEFAULT。
            专门设计用于Propagation.REQUIRED或Propagation.REQUIRES_NEW，因为它仅适用于新启动的事务。如果您希望
        隔离级别声明在参与具有不同隔离级别的现有事务时被拒绝，请考虑在事务管理器上将“validateExistingTransactions”
        标志切换为“true”。

		2. Class<? extends Throwable>[] noRollbackFor: {}
			定义零（0）或更多异常类，它们必须是Throwable的子类，指示哪些异常类型不得导致事务回滚。
            这是构造回滚规则的首选方法（与noRollbackForClassName（）相反），匹配异常类及其子类。
            与NoRollbackRuleAttribute.NoRollbackRuleAttribute（Class clazz）类似。

		3. String[] noRollbackForClassName: {}
			定义零（0）或更多异常名称（对于必须是Throwable的子类的异常），指示哪些异常类型不得导致事务回滚。
			有关如何处理指定名称的详细信息，请参阅rollbackForClassName（）的说明。
            与NoRollbackRuleAttribute.NoRollbackRuleAttribute（String exceptionName）类似。

		4. Propagation propagation: org.springframework.transaction.annotation.Propagation.REQUIRED
			事务传播类型。
            默认为Propagation.REQUIRED。

		5. boolean readOnly: false
			如果事务是有效只读的，则可以设置为true的布尔标志，允许在运行时进行相应的优化。
            默认为false。
            这仅仅是实际事务子系统的提示;它不一定会导致写访问尝试失败。无法解释只读提示的事务管理器在被要求进行只读事务时
        不会抛出异常，而是默默地忽略提示。

		6. Class<? extends Throwable>[] rollbackFor: {}
			定义零（0）或更多异常类，它们必须是Throwable的子类，指示哪些异常类型必须导致事务回滚。
            默认情况下，事务将在RuntimeException和Error上回滚，但不会在已检查的异常（业务异常）上回滚。有关详细说明，
        请参见DefaultTransactionAttribute.rollbackOn（Throwable）。
        	这是构造回滚规则（与rollbackForClassName（）相反）的首选方法，匹配异常类及其子类。
            与RollbackRuleAttribute.RollbackRuleAttribute（Class clazz）类似。

		7. String[] rollbackForClassName: {}
			定义零（0）或更多异常名称（对于必须是Throwable的子类的异常），指示哪些异常类型必须导致事务回滚。
            这可以是完全限定类名的子字符串，目前没有通配符支持。例如，值“ServletException”将匹配
        javax.servlet.ServletException及其子类。
        	注意：仔细考虑模式的具体程度以及是否包含包信息（这不是强制性的）。例如，“异常”几乎可以匹配任何内容，并且可能
        隐藏其他规则。如果“异常”用于为所有已检查的异常定义规则，则“java.lang.Exception”将是正确的。使用更多不寻常的异
        常名称，例如“BaseBusinessException”，不需要使用FQN。

		8. int timeout: -1
			此事务的超时（以秒为单位）。
            默认为基础事务系统的默认超时。
            专门设计用于Propagation.REQUIRED或Propagation.REQUIRES_NEW，因为它仅适用于新启动的事务。

		9. String transactionManager: ""
			指定事务的限定符值。
            可用于确定目标事务管理器，匹配特定PlatformTransactionManager bean定义的限定符值（或bean名称）。

		10. String value: ""
			transactionManager() 的别名.
## @PathVariable 解读
		注释，指示方法参数应绑定到URI模板变量。支持RequestMapping带注释的处理程序方法。
        如果方法参数是Map <String，String>，则使用所有路径变量名称和值填充映射。
---
	属性:
    	1. String name: ""
			要绑定的路径变量的名称。

   		2. boolean required: true
			是否需要路径变量。
            默认为true，如果传入请求中缺少路径变量，则会导致抛出异常。如果您在这种情况下更喜欢null或Java 8
        java.util.Optional，请将其切换为false。例如在ModelAttribute方法上，该方法用于不同的请求。

		3. String value: ""
			name() 的别名.
### PathVariableMapMethodArgumentResolver 解读
		解析使用@PathVariable注释的Map方法参数，其中注释未指定路径变量名称。创建的Map包含所有URI模板名称/值对。
### PathVariableMethodArgumentResolver 解读
        解析用@PathVariable注释的方法参数。
        @PathVariable是从URI模板变量解析的命名值。它始终是必需的，并且没有默认值可供使用。有关如何处理命名值的更多信息，
    请参阅基类AbstractNamedValueMethodArgumentResolver。
		如果方法参数类型为Map，则注释中指定的名称用于解析URI变量String值。然后，假设已注册了合适的Converter或
    PropertyEditor，则通过类型转换将该值转换为Map。
		调用WebDataBinder以将类型转换应用于尚未与方法参数类型匹配的已解析路径变量值。
## @RequestBody 解读
		指示方法参数的注释应绑定到Web请求的主体。请求的主体通过HttpMessageConverter传递，以根据请求的内容类型解析方法
    参数。可选地，可以通过使用@Valid注释参数来应用自动验证。
		支持带注释的处理程序方法。
---
	属性:
    	1. boolean required: true
			是否需要身体内容。
            默认值为true，导致在没有正文内容的情况下抛出异常。如果您希望在body内容为null时传递null，请将其切换为false。
## @RequestParam 解读
		注释，指示应将方法参数绑定到Web请求参数。
        支持Spring MVC和Spring WebFlux中带注释的处理程序方法，如下所示：
        	1. 在Spring MVC中，“请求参数”映射到多部分请求中的查询参数，表单数据和部件。这是因为Servlet API将查询参数和
        表单数据组合到一个名为“parameters”的映射中，其中包括自动解析请求体。
        	2. 在Spring WebFlux中，“请求参数”仅映射到查询参数。要使用所有3，查询，表单数据和多部分数据，可以使用数据绑定
        到使用ModelAttribute注释的命令对象。

		如果方法参数类型是Map并且指定了请求参数名称，则假定适当的转换策略可用，请求参数值将转换为Map。
        如果方法参数为Map <String，String>或MultiValueMap <String，String>且未指定参数名称，则使用所有请求参数名称
    和值填充map参数。
---
	属性:
    	1. String defaultValue: "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n"
			未提供请求参数或具有空值时用作回退的默认值。
            默认提供默认值会将required（）设置为false。

		2. String name: ""
			要绑定的请求参数的名称。

		3. boolean required: true
			是否需要参数。
            默认为true，如果请求中缺少参数，则会导致抛出异常。如果您在请求中不存在参数时更喜欢空值，请将其切换为false。
            或者，提供defaultValue（），它将此标志隐式设置为false。

		4. String value: ""
			name() 的别名.
### RequestParamMapMethodArgumentResolver 解读
		解析使用@RequestParam注释的Map方法参数，其中注释未指定请求参数名称。
        如果使用MultipartFile作为值类型专门声明，则创建的Map包含所有请求参数名称/值对，或给定参数名称的所有多部分文件。
    如果方法参数类型是MultiValueMap，则创建的映射包含请求参数具有多个值（或同一个多个多部分文件）的情况下的所有请求参数及
    其所有值。
### RequestParamMethodArgumentResolver 解读
		解析使用@RequestParam注释的方法参数，类型为MultipartFile的参数以及Spring的MultipartResolver抽象，以及类型
    为javax.servlet.http.Part的参数以及Servlet 3.0多部分请求。此解析器也可以在默认解析模式下创建，其中未使用
	@RequestParam注释的简单类型（int，long等）也被视为请求参数，参数名称从参数名称派生。
    	如果方法参数类型为Map，则注释中指定的名称用于解析请求参数String值。然后，假设已注册了合适的Converter或
    PropertyEditor，则通过类型转换将该值转换为Map。或者，如果未指定请求参数名称，则使用
    RequestParamMapMethodArgumentResolver来提供对地图形式的所有请求参数的访问。
    	调用WebDataBinder以将类型转换应用于尚未与方法参数类型匹配的已解析请求标头值。
## @RequestHeader 解读
		注释，指示应将方法参数绑定到Web请求标头。
        支持Spring MVC和Spring WebFlux中带注释的处理程序方法。
        如果方法参数是Map <String，String>，MultiValueMap <String，String>或HttpHeaders，则会使用所有标题名称和
    值填充映射。
---
	属性:
    	1. String defaultValue: "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n"
			用作后备的默认值。
            默认提供默认值会将required（）设置为false。

		2. String name: ""
			要绑定的请求标头的名称。

		3. boolean required: true
			是否需要标头。
            默认为true，如果请求中缺少标头，则会导致抛出异常。如果您在请求中不存在标头时更喜欢空值，请将此项切换为false。	
            或者，提供defaultValue（），它将此标志隐式设置为false。

		4. String value: ""
			name() 的别名.
## @MatrixVariable 解读
		注释，指示方法参数应绑定到路径段中的名称 - 值对。支持RequestMapping带注释的处理程序方法。
		如果方法参数类型是Map并且指定了矩阵变量名称，则矩阵变量值将转换为Map，假设有适当的转换策略。
		如果方法参数为Map <String，String>或MultiValueMap <String，String>且未指定变量名称，则使用所有矩阵变量名称
    和值填充映射。
---
	属性:
    	1. String defaultValue: "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n"
			用作后备的默认值。默认提供默认值会将required（）设置为false。

		2. String name: ""
			矩阵变量的名称。

		3. String pathVar: "\n\t\t\n\t\t\n\ue000\ue001\ue002\n\t\t\t\t\n"
			矩阵变量所在的URI路径变量的名称，如果需要消除歧义（例如，在多个路径段中存在同名的矩阵变量）。

		4. boolean required: true
			是否需要矩阵变量。
            默认值为true，导致在请求中缺少变量时抛出异常。如果您缺少变量，则将此项切换为false。
            或者，提供defaultValue（），它将此标志隐式设置为false。

		5. String value: ""
			name() 别名.
## @CookieValue 解读
## @RequestPart 解读








