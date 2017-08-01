[Bob Martin, The Open-Closed Principle (PDF)](https://www.cs.duke.edu/courses/fall07/cps108/papers/ocp.pdf)

[Why you cannot add advice to final Methods](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#aop-understanding-aop-proxies)

---

[Spring Web MVC 4.3.8 文档链接](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/spring-web.html)

---

# 1. Web MVC框架

Spring view解析是非常灵活。Controller是负责准备一个Map结构的数据对象，和选择一个view名称。同时也支持直接将response数据流直接写给request方。View名称解析是可以通过文件扩展名和Accept头信息协商，Bean名称，属性文件或定制的ViewResolver实现类来配置。Model是一个Map接口，实现了对View层技术的完全抽象。可以直接集成基于模板语言的展示层，如JSP，Velocity和Freemarker，或者直接产生XML，JSON，Atom或其它类型的内容。只需要将Map对象(model)转换成适当格式，如JSP请求属性或Velocity模板模型(model)。

Spring web模块所支持的特性：

* 更清楚的角色划分。controller, validator, command object, form object, model object, DispatcherServlet, handler mapping, view resolver
* 强大且简单的框架和Java Bean配置
* 可插拔，无侵入和灵活，仅需要添加相应注解到相应的controller方法，如，@RequestParam, @RequestHeader和@PathVariable
* 可重用的业务代码。使用存在业务对象作为command和form对象，而不需要继承特定的框架基类
* 可定制的binding和validation
* 可定制的handler映射和view解析
* 灵活的model转换
* 可定制的locale，time zone和theme解析
* spring tag library
* spring-form jsp tag library
* Bean生命周期支持http request或session
    
    * request scope
    * session scope
    * global session scope
    * application scope
    * web scoket scope

## 1.1 DispatcherServlet

Spring的web mvc框架和别的mvc框架一样，请求驱动(request-driven)，由一个统一的servlet来分发请求到controller并提供一些其它有助于web应用开发的功能。但Spring的DispatcherServlet提供了许多额外的功能。它是完全与Spring的IOC容器集成，允许用户使用其它Spring拥有的特性。

![Alt ](images/request-processing-workflow-in-spring-web-mvc.png)

根据[Section 7.15, "Additional Capabilities of the ApplicationContext"](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/beans.html#context-introduction)中介绍Spring中的ApplicationContext是有生命周期的。在web mvc框架中，每个DispatcherServlet有它自己的WebApplicationContext，这个WebApplicationContext继承所有定义在root WebApplicationContext中的bean对象。Root WebApplicationContext应该包含所有的基础bean，这些bean可以在别的context和servlet实例中被共享。这些被继承的bean可以在servlet-specific scope中重载，可以在指定的servlet实例中定义本地新的scope-specific bean。

![Alt ](images/typical-context-hierarchy-in-spring-web-mvc.png)

DispatcherServlet初始化时，Spring mvc会查找在WEB-INF目录中以[servlet-name]-servlet.xml格式命名的文件，创建servlet-specifics scope bean对象，并重载和global scope中相同bean的bean对象。

__Signle root context in Spring Web MVC__


![Alt ](images/single-root-context-in-spring-web-mvc.png)

如果想初始化只有root context的情况，需要在servlet中把contextConfigLocation指定为空。

    <web-app>
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/root-context.xml</param-value>
        </context-param>
        <listener>
            <listener-class>
                org.springframework.web.context.ContextLoaderListener
            </listener-class>
        </listener>

        <servlet>
            <servlet-name>api</servlet-name>
            <servlet-class>
                org.springframework.web.servlet.DispatcherServlet
            </servlet-name>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value></param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>api</servlet-name>
            <url-pattern>/api/*</url-pattern>
        </servlet-mapping>
    </web-app>

WebApplicationContext是ApplicationContext的一个扩展，并添加一些web应用需要的特性。与ApplicationContext不同的是，它可以解析themes，能够获知与它绑定的servlet(有一个指向ServletContext的引用)。WebApplicationContext是被限定在了ServletContext内，在需要访问它时，可以通过使用RequestContextUtils类中的静态方法查找到对应的WebApplicationContext。

### 1.1.1 WebApplicationContext中特殊bean类型

<table>
    <tr>
        <th>Bean类型</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>HandlerMapping</td>
        <td>映射用户请求到相应的handler，和一组pre-及post-processors对象上。</td>
    </tr>
    <tr>
        <td>HandlerAdapter</td>
        <td>辅助DispatcherServlet调用映射到当前请求上的handler，但不处理handler被调用的具体细节。如，在调用一个被注解的controller，要求解析各种不同的注解(annotations)。HandlerAdapter的主要目的是保护DispatcherServlet避开这些具体的处理细节。</td>
    </tr>
    <tr>
        <td>HandlerExceptionResolver</td>
        <td>映射异常到相应的view，同时也允许一些复杂的异常处理逻辑。</td>
    </tr>
    <tr>
        <td>ViewResolver</td>
        <td>解析基于String的view逻辑名到实际的view类型上。</td>
    </tr>
    <tr>
        <td>LocaleResolver & LocaleContextResolver</td>
        <td>为了提供国际化的view，解析客户端使用的locale和time zone</td>
    </tr>
    <tr>
        <td>ThemeResolver</td>
        <td>解析应用使用的theme，如提供个性化的部局</td>
    </tr>
    <tr>
        <td>MultipartResolver</td>
        <td>为了支持文件上传，解析multi-part请求</td>
    </tr>
    <tr>
        <td>FlashMapResolver</td>
        <td>为了处理请求间或redirect场景中的请求属性传递，存储或获取FalshMap中的输入或输出</td>
    </tr>
<table>

### 1.1.2 DispatcherServlet默认配置

默认配置位于org.springframework.web.servlet中的DispactherServlet.properties文件中。

### 1.1.3 DispatcherServlet处理顺序

Request请求进入特定DispactherServlet后的处理流程：

* 查找并把WebApplicationContext作为属性，绑定到request上。在处理过程中controller和别的元素都可以使用WebApplicationContext对象。默认的绑定Key为DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE。
* locale解析器被绑定到request上，使处理中的元素可以获取到locale。
* theme解析器被绑定到request上，帮助view层来决定使用那种theme。
* 如果指定了multipart file解析器，请求将被multiparts解析器处理。如果multiparts被发现，请求将会被封装在MultipartHttpServletRequest中，供处理过程中其它元素处理。
* 匹配合适的handler。如果handler被找到，和执行链联合的handler(preprocessors, postprocessors和controllers)将会被执行，为了准备model和处理页面渲染逻辑。
* 如果model被返回，view将会被渲染。如果没有返回model(由于安全问题，请求可能被某个preprocessor或postprocessor拦截住了)，将没有view会被渲染，因为请求可能已经被处理完成。

被声明在WebApplicationContext中的exception resolvers handler，将会获取请求处理中抛出的异常。使用这些配置的exception resolvers允许用户以自定义的方式来处理这些异常。

Spring通过查找实现LastModified接口的handler，来处理是否返回给用户last-modification-date。

__DispatcherServlet初始化参数__
<table>
    <tr>
        <th>参数名</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>contextClass</td>
        <td>WebApplicationContext的实现类，被这个servlet用来初始化context环境。默认是，XmlWebApplicationContext。</td>
    </tr>
    <tr>
        <td>contextConfigLocation</td>
        <td>用来指定servlet配置文件的位置。如果有多个文件，可以逗号分隔。</td>
    </tr>
    <tr>
        <td>namespace</td>
        <td>WebApplicationContext命名空间，默认是[servlet-name]-servlet</td>
    </tr>
</table>

## 1.2 实现Controller

    @Controller
    public class ProgrammerController {
    
        @RequestMapping("/programmers")
        public String get(Model model) {
            
            model.addAttribute("username", "liuyf");
            return "programmers";
        }
    }

类级别的@RequestMapping是可选的，如果不存在，则method上的指定的路径是绝对的；否则，method上的路径是相对的。

__Spring 4.3引入了以下method级的annotations__

* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping

> Note:
> 
> 如果controller必须实现一个非Spring context callback接口，用户需要明确的配置采用基于类的代理。如，在类上添加@Transactional注释后，需要使用<tx:annotation-driven proxy-target-class="true" />替换<tx:annotation-driven />。

> 为了支持添加@RequestMapping注解的方法，Spring 3.1引入了新的支持类，分别是RequestMappingHandlerMapping和RequestMappingHandlerAdapter。

Spring 3.1之前，针对type-和method-level @RequestMapping是在两个独立的阶段进行检查的，DefaultAnnotationHandlerMapping先挑选controller, 实际的方法调用被放在了AnnotationMethodHanlderAdapter中。

对于Spring 3.1中新的支持类，由RequestMappingHandlerMapping来决定哪个方法处理请求。把controller methods看作一个包含唯一并带映射关系的endpoints集合，这个集合是根据type和method-level上@RequestMapping分析出来的。

这也创造了一些新的可能性。对于原来的HandlerInterceptor或HandlerExceptionResolver，现在可以期望基于对象的handler是一个HandlerMethod，HandlerMethod允许它们检测明确的方法及方法参数，也包括方法上的annotations。对URL的处理，不再需要被划分在多个不同的Controller中了。

在3.1之后，有一些情况将不会存在了：

* 先通过SimpleUrlHandlerMapping或BeanNameUrlHandlerMapping选择一个controller，然后再基于方法上的@RequestMapping来匹配具体的方法。
* 依赖方法名来消除两个@RequestMapping方法产生的歧义，这两个方法除了没有明确的path mapping URL，其它都一样，如HTTP方法。在新的支持类中，@RequestMapping方法的映射关系必须唯一。
* 在只有一个默认方法的情况下，如果没有别的controller方法更匹配，请求将由这个方法处理(没有明确的路径映射)。在新的支持类中，如果没有匹配的method被找到，将抛出404错误。

__以上特性在之前的support类中仍然被支持。如果想要使用Spring MVC 3.1的新特性，需要使用新的support类。__

### 1.2.1 使用@RequestMapping映射request

#### 1.2.1.1 URI模板模式

    @GetMapping("/mvc/{id}")

#### 1.2.1.2 支持正则表达式的URI模板模式

    @RequestMapping("/mvc/{varName:regex}")

#### 1.2.1.3 路径模式(path patterns)

    @RequestMapping("/mvc/*.do")

#### 1.2.1.4 路径模式比较

/hotels/{hotel}/\*与/hotels/{hotel}/\*\*相比，/hotels/{hotel}/\*更优先被匹配。

如果两个模式(patterns)拥有相同的数量，则更长的将优先被匹配。/foo/bar\*与/foo/\*相比，/foo/bar\*将被匹配。

当两个模式有相同的变量个数，并且长度相等。拥有最少通配符的将被优先匹配。/hotels/{hotel}与/hotels/\*相比，/hotels/{hotel}更优先匹配。


__特殊规则：__

* 默认的映射模式/**，优先级要低于别的模式。如，/api/{a}/{b}/{c}
* 前缀模式，如/public/**优先级要低于任何未包含两个通配符的柜式。如，/public/path/{a}/{b}/{c}

[Path Matching](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-config-path-matching)

#### 1.2.1.5 带占位符的路径模式

${envPath}，运行时Spring MVC会去本地配置文件、系统属性和环境变量中去搜索envPath。

#### 1.2.1.6 后缀模式匹配

默认情况下Spring MVC执行".*"后缀模式匹配，因此映射到/person的controller也意味着是被映射到/person.\*。这使通过URL path请求不同的资源变得简单，如/person.pdf，/person.xml。

后缀模式匹配可以被关闭，或被限制到为了内容协商而明确注册到系统中的后缀集合。

__推荐最小化使用common request mappings，如/person/{id}，在这个场景中逗号不代表一种文件后缀。如/person/one@one.com或/person/one@one.json。__

#### 1.2.1.7 后缀模式匹配和RFD

RFD攻击: reflected file download attack

RFD攻击与XSS攻击类似，依赖于用户输入内容(query parameter或URI变量)未经处理直接放在返回给用户的响应中。与把Javascript插入到HTML中不同的是，RFD攻击依赖浏览器执行下载或把响应处理成一个执行脚本(如，.bat或.cmd)。

Spring MVC中添加@ResponseBody或@ResponseEntity的方法将会遭遇该问题，因为根据用户请求URL中的路径扩展(URL path extensions)，他们可以渲染不同的content type。

__屏蔽后缀模式匹配或内容协商机制，可以有效阻止RFD攻击。__

为了防范RFD攻击，在渲染Response body前，Spring MVC添加`Content-Disposition:inline;filename=f.txt`头，建议使用一个固定的且安全的文件名。这只有URL中文件扩展名在白名单中或明确配置为了内容协商时，才会被执行。对于用户直接在浏览器中输入URL的情况，这种方式不能有效阻止RFD攻击。

许多常用的扩展名默认已经被加入白名单了。

> Note:
> 
> Spring做这部分工作，最初是为了解决[CVE-2015-5211](https://pivotal.io/security/cve-2015-5211)问题。下面是这个报告中的一些推荐：
> 
> * 不要使用json字符escape操作，而是对其重新编码(encode)
> * 关闭后缀匹配或限定只能是已注册过的后缀
> * 使用useJaf属性，并将ignoreUnknownPathExtensions设置成false，来配置内容协商，对于不能识别的扩展名，返回406。但这不适用于期望URL最后有个dot的情况。
> * Spring Security 4默认会添加`X-Content-Type-Options: nosniff`响应头。

#### 1.2.1.8 矩阵变量(Matrix Variables)

矩阵变量可以出现在path的任何部分，每个matrix变量由";"分隔。如，"/cars;color=red;year=2012"。如果变量有多个值可以使用逗号分隔"color=red,green,blue"；也可以声明多个同名变量"color=red;color=green;color=blue"。

    @GetMapping("/matrix/variables/{vId}")
    public void matrix(@PathVariable String vId, @MatrixVariable int matrixVariable) {}

    @GetMapping("/matrix/variables/{matrixVariable}")
    public void anotherMatrix(
        @MatrixVariable(name="matrixVariable", pathVar="matrixVariable1") int matrixVariable1,
        @MatrixVariable(name="matrixVariable", pathVar="matrixVariable2") int matrixVariable2) {
    }

__如果期望使用matrix variables，必须把RequestMappingHandlerMapping的removeSemicolonContent属性设为false，默认为true。__

> Note:
> 
> 启用matrix variables功能。
> 
> `<mvc:annotation-driven enable-matrix-variables="true"/>`

#### 1.2.1.9 可接收(consumable)的media types

    @PostMapping(path = "/mvc", consumes = "application/json")

#### 1.2.1.10 可返回(producible)的media types

    // 如果请求头中Accept值与produces的值，有一个相匹配，这个方法才会被用来处理这个请求。

    @GetMapping(path = "/mvc", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)

#### 1.2.1.11 根据请求参数或请求头的值匹配

    @GetMapping(path = "/mvc", params = "type=MAN")

#### 1.2.1.12 HTTP HEAD和HTTP OPTIONS

映射到"GET"上的@RequestMapping方法，隐含的也会被映射到"HEAD"。
不需要有明确的HEAD声明。

@RequestMapping方法内置了对HTTP OPTIONS的支持。默认情况下，HTTP OPTIONS会被设置了Allow响应头的并且与URL相匹配的@RequestMapping处理。如果这儿没有方法明确声明"GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS"值到"Allow"头上，理想情况下，应该使用@RequestMapping声明一个方法来处理它。或者通过使用@RequestMapping variants来声明。

__虽然不需要明确的为HEAD和OPTIONS方法专门设置对应添加@RequestMapping的方法，推荐明确设置。__

### 1.2.2 定义@RequestMapping handler方法

#### 1.2.2.1 支持的方法参数类型

* Request和Response对象
* Session对象

> Note:
> 在Servlet环境中，访问session是非线程安全的。如果允许多请求并发访问同一个Session，可以打开RequestMappingHandlerAdapter上的开关"synchronizeOnSession"。

* org.springframework.web.context.request.WebRequest或org.springframework.web.context.request.NativeWebRequest，支持泛型请求参数。
* java.util.Locale，获取当前locale。启用，需要在MVC环境中配置LocaleResolver和LocaleContextResolver。
* java.util.TimeZone和java.time.ZoneId
* java.io.InputStream/java.io.Reader，访问request内容
* java.io.OutputStream/java.io.Writer，产生response内容
* org.springframework.http.HttpMethod
* java.security.Principal，获取当前的授权用户
* @PathVariable，获取路径中的参数变量
* @MatrixVariable，获取matrix变量
* @RequestParam，获取参数
* @RequestHeader，获取某个特定的请求头
* @RequestBody，映射request body到bean对象。
* @RequestPart，访问一个"multipart/form-data" request part的内容。
* @SessionAttribute，获取session属性
* @RequestAttribute，获取request属性
* HttpEntity<?>，访问servlet request的请求头和内容。使用HttpMessageConverter将请求流被转换成entity body。
* java.util.Map/org.springframework.ui.Model/org.springframework.ui.ModelMap，被用来传递model对象到web view中。
* org.springframework.web.servlet.mvc.support.RedirectAttributes，在处理redirect请求时，在上下文中传递属性。
* form对象，通过@InitBinder实现
* org.springframework.validation.Errors/org.springframework.validation.BindingResult
* org.springframework.web.bind.support.SessionStatus
* org.springframework.web.util.UriComponentsBuilder

> Note:
> 
> BindingResult和@ModelAttribute顺序无效
> 
> @PostMapping
> 
> public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) {}
> 
> 使以上代码工作，需要调整参数顺序：
> 
> @PostMapping
> 
> public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) {}
> 
> __对于拥有required属性的注解，同样可以使用JDK 1.8中的java.util.Optional来标注参数可选。java.util.Optional等同于required=false。__

#### 1.2.2.2 支持的方法返回类型
* ModelAndView对象
* Model对象
* Map对象
* View对象
* String，逻辑view名。
* void
* 添加有@ResponseBody的方法，序列化bean对象到response body。
* HttpEntity<?>或ResponseEntity<?>
* HttpHeaders，只返回header，不返回response body
* Callable<?>，异步产生返回值
* DeferredResult<?>，返回值由线程产生
* ListenableFuture<?>或CompletableFuture<?>/CompletionStage<?>，由提交到线程池中的task产生返回值
* ResponseBodyEmitter，异步写多个对象到response中
* SseEmitter，异步写Server-Sent Event事件到响应体中
* StreamingResponseBody，异步向response的OuputStream写数据。

#### 1.2.2.3 使用@RequestBody映射request body

RequestMappingHandlerAdapter支持用以下默认HttpMessageConverters来解析reqestBody到@RequestBody标注对象：

* ByteArrayHttpMessageConverter，转换字节数组
* StringHttpMessageConverter，转换字符串
* FormHttpMessageConverter，在data和MultiValueMap<String, String>之前转换数据。
* SourceHttpMessageConverter，转换或序列化java.xml.transform.Source。

[Message Converters](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/remoting.html#rest-message-conversion)

@RequestBody方法的参数可以添加@Valid注解，它被匹配的Validator实例验证。当使用MVC namespace或MVC Java config，如果classpath中有JSR-303实现类，那么JSR-303 validator将被自动配置，

#### 1.2.2.4 使用@RestController创建REST Controller

@RestController是一具构造型(stereotype)注解，由@ResponseBody和@Controller组成。

#### 1.2.2.5 在方法上应用@ModelAttribute

在方法上的@ModelAttribute表示，这个方法是的目的是添加一个或多个model属性。这类方法与@RequestMapping所支持的参数类型相同，但不能直接被映射到相应的请求上。在Controller中标注@ModelAttribute的方法是在同一Controller中标注@RequestMapping方法前被调用。

一个Controller可以添加多个@ModelAttribute方法。在同一个Controller中，所有标注为@ModelAttribute的方法，将会在@RequestMapping方法前被调用。

@ModelAttribute方法也可以被定义在@ControllerAdvice类中，这样的方法将会被应用到所有的controller上。

@ModelAttribute也可以被加在标注了@RequestMapping的方法上。在这种情况中，@RequestMapping方法的返回值会被解析成一个model属性，而不是view名。替代的View名将会基于view名规则被推出来。

#### 1.2.2.6 在方法参数上应用@ModelAttribute

在方法参数上标注@ModelAttribute注解，表示应该从model上取这个参数。如果model上没有，参数会先被初始化，然后添加到model中。一旦model包含了这个参数，参数的内容会被同名的request参数填充。

    public String processSubmit(@ModelAttribute MVC mvc) { }

上例中MVC可能的取值有：

* 在使用@SessionAttriutes情况下，model中可能已包含该对象
* 在controller中包含一个@ModelAttribute方法的情况下，model中可能已包含该对象
* 可以从URI模板变量或类型转换中取得
* 通过默认构造函数初始化

在一些某些情况下，也可以在@ModelAttribute方法上通过配置URI模板变更和类型转换器，从URI提取并转换成对应的bean对象：

    @PutMapping("/mvc/{user}")
    public String save(@ModelAttribute("user") User user) {
    }

在上面的例子中，Model属性的名字与URL模板变量的名字相匹配。如果你注册了Converter<String, User>，一个将String user值转换成User实例的转换器，在没有@ModelAttribute方法的情况下，上面的例子也会正确运行。

接下来是数据绑定。WebDataBinder类，会将请求参数，查询参数以及form表单中的属性值，根据字段名匹配到对应的model attribute属性中。

在必要的类型转换后，相匹配的属性将被正确赋与相应的属性值。

__扩展参考：__

[Chapter 9, Validation, Data Binding and Type Conversion](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/validation.html)

[Customizing WebDataBinder Initialization](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)

数据绑定过程中，可能会遇到必填属性未赋值或类型转换错误。为了检测到这些错误情况，可以在@ModelAttribute参数后面，直接添加BindingResult参数。

    @PostMapping("/mvc")
    public String processSubmit(@Valid @ModelAttribute("user") User user, BindingResult result) {
    }

__注，在User对象中使用javax.validation.constraints.*中的注解来标注对字段的限定约束。__

#### 1.2.2.7 使用@SessionAttributes在多个请求之间存储model attributes

在类上使用@SessionAttributes注解，声明session attributes被特定的Handler使用。

    @SessionAttribute("User")
    public class EditUserForm {
    }

#### 1.2.2.8 使用@SessionAttribute访问已存在的global session attributes

访问全局的已存在的session attributes，session在controller之外被管理，例如通过filter管理。

    @RequestMapping("/")
    public String handle(@SessionAttribute User) {
    }

#### 1.2.2.9 使用@RequestAttribute访问request attributes

与@SessionAttribute相似，可以通过@RequestAttribute来访问由filter或interceptor创建的request attributes。

    @RequestMapping("/")
    public String handle(@RequestAttribute Address address) {
    }

#### 1.2.2.10 处理application/x-www-form-urlencoded数据

之前的章节中介绍了，通过浏览器客户端提交表单给服务器，通过@ModelAttribute接收的场景。同样的注解也支持非浏览器客户端提交的请求。需要了解的是在处理put请求时，会有一些不同。浏览器可以提交表单数据，经由get或post方法。非浏览器客户端也可以通过put方法提交表单数据。这样就产生了一个矛盾，因为Servlet规范要求ServletRequest.getParameter*()这一类的方法，只支持访问通过post提交的表单内容，不支持put方法。

为了支持put和patch请求，spring-web模块提供了一个过滤器HttpPutFormContentFilter。

    <filter>
        <filter-name>httpPutFormFilter</filter-name>
        <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name></filter-name>
        <servlet-name>dispatcherServlet</servlet-name>
    </filter-mapping>

    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>

上面的filter拦截content-type为application/x-www-form-urlencoded的put和patch请求，从request body中读取form数据，包装成一个新的ServletReqeust对象中，以便可以通过servletRequest.getParameter*()等方法访问form数据。

> Note:
> 
> 由于HttpPutFormContentFilter会处理request body，不应该再为put和patch URLs配置别的application/x-www-form-urlencoded转换器。这包括@RequestBody MultiValueMap<String, String>和HttpEntity<MultiValueMap<String, String>>。

#### 1.2.2.11 使用@CookieValue映射cookie值

@CookieValue允许一个将cookie值绑定到一个方法的参数上。

    @RequestMapping("/mvc")
    public void displayCookie(@CookieValue("JSESSIONID") String cookie) {
    }

#### 1.2.2.12 使用@RequestHeader映射request header attributes

    @RequestMapping("/mvc")
    public void displayHeaders(@RequestHeader("Accept-Encoding") String encoding, @RequestHeader("Keep-Alive") long keepAlive) {
    }

> Note:
> 
> 当在Map<String, String>，MultiValueMap<String, String>和HttpHeaders类型的参数上使用@RequestHeader时，将会把所有的header值存储到Map对象中。

> Spring内置了将逗号分隔的字符串，转换成一个String数据或集合，或类型转换系统已知的别的数据类型。例如，@RequestHeader("Accept")，可能是String类型，也可能是String[]或List<String>。

#### 1.2.2.13 方法参数或类型转换

除了String外，乌有之乡所有简单类型如，int， long，Date等类型，Spring将自己转换成相应的类型。用户也可以通过WebDataBinder或使用FormattingConversionService注册Formatters，定义转换过程。

[Customizing WebDataBinder initialization](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-webdatabinder)

[Spring Field Formatting](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/validation.html#format)

#### 1.2.2.14 定制WebDataBinder初始化

通过Spring WebDataBinder，使用PropertyEditors来定制请求参数绑定。同样，用户也可以在controller中定义带@InitBinder的方法，在@ControllerAdvice中的@InitBinder方法，或提供定制的WebBindingInitializer来定制请求参数绑定。

[Advising controllers with @ControllerAdvice and @RestControllerAdvice](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-controller-advice)

##### 1.2.2.14.1 使用@InitBinder定制数据绑定

标注@InitBinder的方法，允许用户直接在controller中配置Web数据绑定。

Init-binder方法不能有返回值。因此，通常它们被声明成void。

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(flase);
        binder.registerCustomEditor(Data.class, new CustomDateEditor(dateFormat, false));
    }

从Spring 4.2开始，考虑使用addCustomFormatter替代PropertyEditor
实例来指定Formatter实现。

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

##### 1.2.2.14.2 配置定制的WebBindingInitializer

为了重载默认配置，用户可以提供一个WebBindingInitializer实现类，将该类的实例注入到AnnotationMethodHandlerAdapter中。

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="cacheSeconds" value="0" />
        <property name="webBindingInitializer">
            <bean class="com.liuyf.demos.spring.web.mvc.controllers.custom.CustomBindingInitializer" />
        </property>
    </bean>

#### 1.2.2.15 配置@ControllerAdvice和@RestControllerAdvice

在使用MVC命名空间或MVC java config时，@ControllerAdvice会被自动启用。

使用@ControllerAdvice标注的类，也可以包含标注@ExceptionHandler，@InitBinder和@ModelAttribute的方法，这些方法将被应用到所有匹配的Controller中@RequestMapping方法上。

@RestControllerAdvice与@ControllerAdvice的区别是，前者会假设@ExceptionHandler方法默认添加@ResponseBody注解。

    @ControllerAdvice(annotations = RestController.class)
    public class LiuyfControllerAdvice { }

#### 1.2.2.16 支持Jackson序列化view

[Jackson's Serialization Views](http://wiki.fasterxml.com/JacksonJsonViews)

@JsonView控制输入与输出的JSON串。

__Controller__

    @RequestMapping(value = "/user/json/view")
    @ResponseBody
    @JsonView(com.liuyf.demos.spring.web.mvc.model.User.userJsonView.class)
    public User userJsonView() {
        
        User user = new User();
        user.setName("liuyf");
        user.setPhone("13800138000");
        return user;
    }

__添加了JsonView的model对象__

    public class User {

        public interface userJsonView { }
        
        @NotNull
        private String name;
        
        @NotNull
        private String phone;
    
        @JsonView(userJsonView.class)
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        @JsonView(userJsonView.class)
        public String getPhone() {
            return phone;
        }
    
        public void setPhone(String phone) {
            this.phone = phone;
        }
    
    }

#### 1.2.2.17 支持Jackson JSONP

为了支持JSON，需要添加一个继承了AbstractJsonpResponseBodyAdvice的@ControllerAdvice对象。向下面展示的一样，需要在构造参数中指定JSON查询参数名。

    @ControllerAdvice
    public class JsonpAdvice extends AbstractJsonpResponseBodyAdvice {

        public JsonpAdvice() {
            super("callback");
        }
    }

对于依赖view解析的，如果请求参数中包括jsonp或callback时，JSONP将自动被启用。这些参数名可以通过jsonpParameterNames属性来指定。

### 1.2.3 异步Request处理
Spring MVC 3.2引入了基于Servlet 3的异步请求处理。不像之前请求返回一个值，通常情况下，controller方法现在返回一个java.util.concurrent.Callable或从Spring MVC管理的线程中返回一个值。与此同时，主Servlet容器线程退出并被释放，允许去处理别的请求。Spring MVC在一个由TaskExecutor实现的分离线程中调用Callable。当Callable返回，请求被发送回servlet容器，使用Callable返回的值，继续下面的处理。

    @RequestMapping("/mvc")
    public Callable<String> processUpload(final MultipartFile file) {

        return new Callable<String>() {
            
            public String call() throws Exception {
                
                return "mvc-view";
            }
        };
    }

对于controller方法，还有一种返回值可选DeferredResult。在这种情况，返回值也是由线程产生。如，线程不是由Spring MVC管理。例如，响应中的结果可以由外部事件产生，如JMS消息，调度task等。

    @RequestMapping("/mvc")
    @ResponseBody
    public DeferredResult<String> processMvc() {

        DeferredResult<String> deferredResult = new DeferredResult<String>();

        return deferredResult;
    }

__在另外的线程中__

    defferedResult.setResult(data);

如果对Servlet 3.0中异步请求处理不了解，理解上面的内容是有一些困难的。这是一些关于低层原理的一些基本情况：

* 通过调用request.startAsync()，ServletRequest可以被放进一个异步模型中。对Servlet的影响是，像Filter一样，servlet退出时，response将保持打开状态，允许处理在servlet退出之后完成。

* 调用request.startAsync()，将返回一个AsyncContext。AsyncContext可以被用来进一步控制异步处理。如，它提供了一个dispatch方法，这个方法与Servlet API的forward类似，不同的是它允许应用在Servlet容器线程中重新开始request处理。

* ServletRequest提供了对当前DispatcherType的访问，这可以被用来区分不同的处理，如初始化request，异步dispatch，forward和其它的dispatcher类型。

有了上面的介绍，下面是一个callable异步请求的事件序列：

* controller返回一个Callable
* Spring开启异步处理并提交Callable到一个TaskExecutor
* DispatcherServlet和所有的Filter退出Servlet容器线程，但响应保持打开状态
* Callable返回一个结果，Spring MVC发送request给Servlet容器，重新处理
* DispatcherServlet再次被调用，被唤醒重新处理由Callable返回结果

DeferredResult处理序列与上面的类似，不同的是应用处理的结果由其它线程返回：

* controller返回DeferredResult，保存可访问的内存队列或列表的
* Spring MVC开始异步处理
* DispatcherServlet和所有配置的Filter退出请求处理线程，但响应保持打开状态
* 应用收集DeferredResult从线程，Spring MVC发送请求给Servlet容器
* DispatcherServlet再次被调用，重新处理由异步生成的结果。

[When or why use the async request processing](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support)

#### 1.2.3.1 异步请求的异常处理

与普通请求是采用同样的异常处理机制。对于DeferredResult，你可以选择使用Exception实例，调用setResult或setErrorResult。
#### 1.2.3.2 拦截异步请求

创建实现AsyncHandlerInterceptor接口的HandlerInterceptor实例，实现afterConcurrentHandlingStarted回调函数，当异步处理开始时，该方法将会被调用。

注册HandlerInterceptor时，也可以注册CallableProcessingInterceptor或DeferredResultProcessingInterceptor。

DeferredResult也提供了onTimeout(Runnable)和onCompletion(Runnable)两个方法。

当使用Callable，你可以使用WebAsyncTask对它进行包装，WebAsyncTask提供了timeout和completion的注册方法。
#### 1.2.3.3 Http流

controller方法可以使用DeferredResult和Callable来异步生成它的返回值。这可以被用来实现长轮询，服务器可以尽可能快的推送事件到客户端。

如果你想在单个http response中推送多个事件？这是一个与长轮询相关的技巧，被称为http streaming。Spring通过ResponseBodyEmitter返回值实现了发送多个对象。从而替换由@ResponseBody实现的返回单个值，ResponseBodyEmitter发送的每个对象，都是通过HttpMessageConverter写入到Response中。

    @RequestMapping("/events")
    public ResponseBodyEmitter handle() {
        
        ResponseBodyEmitter emitter = new ResponseBodyEmitter();
        
        return emitter;
    }

__在别的线程中__

    emitter.send("Hello once");
    emitter.send("Hello again");
    emitter.complete();

__ResponseBodyEmitter也可以被用在ResponseEntity的情况，为了定制response的状态和头信息。__
#### 1.2.3.4 带Server-Sent事件的Http流

SseEmitter是ResponseBodyEmitter的子类，提供对Server-Sent事件的支持。Server-sent事件是http streaming技巧的一个变种，不同的是被推送的事件，是根据W3C Server-Sent事件规范对事件进行了格式化。

> Note:
> Internet Explorer不支持Server-Sent事件。对于高级web应用消息场景，如在线游戏，协作，金融应用或其它，最好考虑Spring WebSocket支持。Spring WebSocket支持包括支持大多数浏览器(包括Internet Explorer)的SockJS风格的WebSocket，支持在以消息为中心，通过发布订阅的模式与客户端交互的高级消息模式。

[WebSocket architecture in Spring 4.0](http://blog.pivotal.io/pivotal/products/websocket-architecture-in-spring-4-0)
#### 1.2.3.5 Http流转换成OutputStream

ResponseBodyEmitter允许通过HttpMessageConverter直接发送事件对象到Response中。在大多数情况下这是可行的，如写JSON数据。但有时，它是非常有用的，绕过消息转换，直接写到OutputStream响应中，例如文件下载。这可以通过StreamingResponseBody来实现。

    @RequestMapping("/download")
    public StreamingResponseBody handle() {

        return new StreamingResponseBody() {
        
            @Override
            public void writeTo(OutputStream outputStream) throws IOException {
                
            }
        }
    }

__StreamingResponseBody也可以被用在ResponseEntity的情况，为了定制response的状态和请求头。__
#### 1.2.3.6 配置异步请求处理
##### 1.2.3.6.1 Servlet容器配置

确保web.xml是被升级到3.0或以上版本。

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://xmlns.jcp.org/xml/ns/javaee/"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
        version="3.1">
    </web-app>

如果要启用异步支持，需要在web.xml文件中，声明DispatcherServlet时，指定<async-supported>true</async-supported>。另外，参与异步处理的filter必须配置支持ASYNC dispatcher type。它应该是安全启用ASYNC dispatcher type对于所有Spring框架提供的filter，因为它们通常继承自OncePerRequestFilter，同时在运行期也会检查是否该filter会参与异步处理。

__启用异步__

    <servlet>
        <servlet-name>api</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>dispatchOptionsRequest</param-name>
            <param-value>true</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

如果正在使用Servlet 3，通过基于Java的WebApplicationInitializer，你也设置asyncSupported标志，为filter设置ASYNC dispatcher type，就像在web.xml中设置的一样。为了简化所有的配置，考虑继承AbstractDispatcherServletInitializer。更好的情况是，继承AbstractAnnotationConfigDispatcherServletInitializer，它将自动的设置这些选项，使注册filter实例更容易。

__异步请求方法__

    @RequestMapping(value = "/callable")
    @ResponseBody
    public Callable<User> callable() {
        
        return new Callable<User>() {

            @Override
            public User call() throws Exception {
                
                Thread.sleep(5000L);
                
                return getUser();
            }
        };
    }

##### 1.2.3.6.2 Spring MVC配置

MVC Java config和MVC namespace都提供了配置异步请求处理的选项。在WebMvcConfigurer中有configureAsyncSupport方法，在<mvc:annotation-driven>中有<async-support>子项。

这允许用户为异步请求配置默认的超时时间，如果没有设置，则依赖于低层Servlet容器(Tomcat中为10秒)。也可以配置AsyncTaskExecutor，为了执行controller方法返回的Callable实例。推荐配置AsyncTaskExecutor，因为默认Spring MVC使用SimpleAsyncTaskExecutor。MVC Java config和MVC namespace也允许用户注册CallableProcessingInterceptor和DeferredResultProcessingInterceptor实例。

如果你想重载特定DeferredResult中默认的超时时间，可以使用适当的类构造函数来实现。同样的，对于Callable，也可以把它包装成WebAsyncTask，使用适当的类构造函数来定制超时时间。WebAsyncTask类构造函数也允许提供一个AsyncTaskExecutor。

### 1.2.4 测试Controller

[Spring MVC Test Framework](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/integration-testing.html#spring-mvc-test-framework)

## 1.3. Handler mappings
在Spring的以前的版本，为了把接收的web request映射到正确的handler，要求用户在web application context中定义多个HandlerMapping bean对象。随注解controller被引入，因为RequestMappingHandlerMapping会在所有标注@Controller注解的controller中查找@RequestMapping注解。请牢记，所有继承了AbstractHandlerMapping的HandlerMapping类，可能通过以下属性来定制它们的行为：

* interceptors，配置所有使用的interceptor。
* defaultHandler，当handler mapping没有匹配的handler，触发这个默认的handler。
* order，基于order属性的值，Spring将排序所有上下文中的handler mapping，应用最优匹配的handler。
* alwaysUseFullPath，值如果是true，Spring使用在当前servlet上下文中的全路径来查找匹配的handler。如果是false(默认)，在当前上下文中的路径将被使用。举例，如果servlet是被映射到/testing/*，alwaysUseFullPath值为true，全路径是/testing/viewPage.html。如果alwaysUseFullPath值为false，则路径/viewPage.html将会被使用。
* urlDecode，从Spring 2.5，urlDecode默认为true。如果你宁愿比较encoded paths，可以把这个值设置为false。否则，HttpServletRequest总是使用decoded form来暴露servlet path。需要了解的是，在与encoded paths比较时，servlet path将不会被匹配。

接下来例子，展示了怎么样配置一个interceptor：

    <beans>
        <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping">
            <property name="interceptors">
                <bean class="com.liuyf.demos.spring.web.mvc.AuthenticationIntercetpor" />
            </property>
        </bean>
    </beans>

### 1.3.1 使用HandlerInterceptor拦截request

Spring handler mapping机制，包含handler interceptor。当用户想要应用特定功能到特定请求时，比如验证用户，需要使用到这个功能。

在handler mapping中的interceptor必须实现org.springframework.web.servlet包中HandlerInterceptor。这个接口定义了三个方法：

* preHandle()，在actual handler执行前被调用
* postHandle()，在handler执行后被调用
* afterCompletion，在请求完成被调用

这三个方法提供了足够的灵活性，可以应对不同的preprocessing和postprocessing情况。

preHandle方法返回一个布尔值。你可以使用这个方法来决定中止或继续执行链处理(the processing of the execution chain)。这个方法返回true，handler执行链将继续；当返回false，DispatcherServlet假设拦截器它自己处理请求，不继续执行别的拦截器和执行链中实际的handler。

拦截器可能通过interceptors属性来配置，在所有继承了AbstractHandlerMapping的HandlerMapping类上都有该属性。

> Note:
> 
> 在使用RequestMappingHandlerMapping时，实际的handler是handlerMethod的一个实例，表示特定的controller方法将被调用。

正如你所知道的那样，Spring的HandlerInterceptorAdapter类，使实现HandlerInterceptor接口更容易。

__需要注意的是__使用HandlerInterceptor的postHandle方法拦截标注了@ResponseBody或返回值为ResponseEntity的方法，总是不合适。在这些情况中，HttpMessageConverter在postHandle执行前，写或委派response，这样postHandle不可能改变response，如添加response header。替代的方案是，应用实现ResponseBodyAdvice，声明它作为一个@ControllAdvice bean，并且把它配置在RequestMappingHandlerAdapter。

## 1.4 解析view

Web应用中所有的MVC框架，都提供了处理view方法。Spring提供了view解析，使你既可以在浏览器中展示model，又不需要绑定与特定的view层技术绑定。Spring使你可以使用JSPs，Velocity模板和XSLT views。

[Chapter 23, View technologies](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/view.html)

对于Spring处理view，核心的两个接口是ViewResolver和View。ViewResolver提供了view名称与实际view的映射关系。View接口解决request准备，传递请求到相应的view层。

### 1.4.1 使用ViewResolver接口解析view

就像在[Section 22.3, "Implementing Controllers"](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-controller)中讨论的一样，在Spring web mvc controller中所有的handler method方法必须返回一个逻辑的view名，不管是明确指定(返回String, View或ModelAndView)还是隐含指定(基于约定)。在Spring中的view是通过逻辑名来指定的，并且由view resolver解析的。Spring提供了很多veiw resolvers。下面的表格列出常用的view resolver：

<table>
    <tr>
        <th>ViewResolver</th>
        <th>描述</th>
    </tr>
    <tr>
        <td>AbstractCachingViewResolver</td>
        <td>一个抽象的view resolver，这个resolver缓存了view。通常情况下，view在使用前需要被预处理；扩展这个view resolver后，同时提供缓存view特性。</td>
    </tr>
    <tr>
        <td>XmlViewResolver</td>
        <td>这个ViewResolver接收一个与Spring xml bean factory使用相同DTD的XML格式的配置文件。默认的配置文件是/WEB-INF/views.xml。</td>
    </tr>
    <tr>
        <td>ResourceBundleViewResolver</td>
        <td>这个View使用ResourceBundle中的bean定义，ResourceBundle由bundle的base name指定。通常情况下，在属性文件中定义bundle，该文件位于classpath中。默认的文件名为views.properties。</td>
    </tr>
    <tr>
        <td>UrlBasedViewResolver</td>
        <td>最简单的ViewResolver实现形式，直接把逻辑view名解析到url上，不需要明确的映射定义。这适合，你的逻辑名与view资源名称一一对应的情况。</td>
    </tr>
    <tr>
        <td>InternalResourceViewResolver</td>
        <td>UrlBasedViewResolver的便利子类，支持InternalResourceView以及子类，如JstlView和TilesView。可以通过setViewClass()来指定由这个view resolver产生view所对应的view class。</td>
    </tr>
    <tr>
        <td>VelocityViewResolver/
FreeMarkerViewResolver</td>
        <td>UrlBasedViewResolver的便利子类，支持VelocityView和FreeMarkerView。</td>
    </tr>
    <tr>
        <td>ContentNegotiatingViewResolver</td>
        <td>基于请求的文件名或Accept头来解析view。</td>
    </tr>
</table>

如果view层想使用JSP，你可以使用UrlBasedViewResolver。这个view resolver可以将view name转换成一个URL，将请求传递给RequestDispatcher来展示view。

    <bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>

当返回user-veiw作为一个逻辑view名时，这个view resolver将转发请求到RequestDispatcher，RequestDispatcher将请求发送到/WEB-INF/jsp/user-view.jsp。

在一个web应用中，如果希望组合不同的veiw层技术，可以使用ResourceBundleViewResolver来解决：

    <bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
        <property name="basename" value="views" />
        <property name="defaultParentView" value="parentView" />
    </bean>

ResourceBundleViewResolver通过basename和它所期望解析的view来识别出ResourceBundle，它使用属性[viewname].(class)的值作为view class，使用属性[viewname].url作为view url。像你看到的一样，你可以指定一个parent view，在配置文件中的所有view都继承自这个view。例如，你可以指定一个默认的view class。

> Note:
> 
> AbstractCachingViewResolver的子类缓存由它们解析的view实例。缓存改善了特定view层技术的性能。可以将cache属性设置成false，来关闭缓存。因此，如果在运行期你必须刷新特定的view(如，当Velocity template被修改时)，可以使用removeFromCache(String viewName, Locale locale)方法。

### 1.4.2 ViewResolvers链

Spring支持多view resolver。因此你可以将resolver串联起来，或在特定场景中重载指定的view。如果要使用view resolver链，可以在应用上下文中添加多个resolver。同时可以通过order属性来指定顺序。请牢记，order值越大，view resolver在链中的位置越靠后。

在下面的例子中，view resolver链由两个resolver组成，一个是InternalResourceViewResolver，这个resovler自动被放置在resovler链的最后；另一个是XmlViewResolver，为了指定excel view。InternalResourceViewResolver不支持Excel view。

    <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp" />
        <property name="suffix" value=".jsp" />
    </bean>

    <bean id="excelViewResolver" class="org.springframework.web.servlet.view.XmlViewResolver">
        <property name="order" value="1" />
        <property name="location" value="/WEB-INF/views.xml" />
    </bean>

__views.xml__

    <beans>
        <bean name="report" class="com.liuyf.demos.spring.web.mvc.views.ReportExcelView" />
    </beans>

如果特定的view resolver不会产生一个view，Spring将检查上下文中别的view resolver。如果附加的view resolver存在，Spring继续审查它们，只到找到一个能解析view的resolver。如果没有resolver返回view，Spring将返回一个ServletException。

view resolver规范指定view resolver可以返回null，表示view不能被找到。但不是所有的view resolver都实现这个功能，因为在一些情况下，resolver不能够简单的判断view是否存在。例如，InternalResourceViewResolver内部使用RequestDispatcher，如果JSP存在，分发(dispatching)是唯一方式来判断JSP是否存在，但是这个判断操作只会被执行一次。对于VelocityViewResolver和别的resolver，采用了同样的实现方式。查看特定view resolver的API文档，了解对应的resolver会不会报告不存在的view。因此，如果不把InternalResourceViewResolver放在resolver链最后，不能保证这个resolver链全部被执行。因为InternalResourceViewResolver总是会返回一个view。

### 1.4.3 重定向到view

#### 1.4.3.1 RedirectView

强制将redirect作为controller response的一种方法是为controller创建并返回一个Spring的RedirectView。在这种情况中，DispatcherServlet不使用正常的view解析机制。因为它已经给出了一个redirect view，DispatcherServlet仅仅指导view做它自己的工作。RedirectView然后调用HttpServletResponse.sendRedirect()，发送一个Http redirect到浏览器。

如果你使用RedirectView，并且view是由controller自已创建，推荐你通过注入方式配置redirect URL，以便它不与controller耦合，而是和view name一块被配置在上下文件中。

[The redirect: prefix](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-redirecting-redirect-prefix)使解耦变的很容易。

#### 1.4.3.2 传递数据到Redirect目标

默认情况下，model属性会被暴露作为redirect URL的URI模板变量。剩余简单类型的属性或简单类型集合/数组形式的属性自动作为query parameter被追加到URI后面。

可以通过在@RequestMapping方法上声明RedirectAttributes类型的参数来指定明确的属性对RedirectView可见。如果方法做redirect，RedirectAttributes将会被使用，否则model的内容将会被使用。

RequestMappingHandlerAdapter提供了一个标志"ignoreDefaultModelOnRedirect"，用来指定在controller方法返回redirect时，默认的model内容对于redirect target不可用。为了维护向下兼容，MVC namespace和MVC Java config设置这个标志默认值为false。对于新的应用，推荐将它设置为true。

__注意：__当前请求中的模板变量对于将要的被redirect URL是自动可用的，不需要被明确的通过Model或RedirectAttributes添加。

    @PostMapping("/mvc/{attr}")
    public String mvc() {

        return "redirect:mvc/new/{path}";
    }

另一种传递数据到redirect target的方法是通过Flash Attributes。不像别的redirect attributes，flash attributes是被保存在http session中。

[Using flash attributes](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-flash-attributes)

#### 1.4.3.3 redirect:前缀

可以使controller不耦合redirect view，甚至controller都不需要关心响应怎么样被处理。

可以通过在view名前添加redirect:前缀实现解耦。如果一个带redirect:前缀的view名被返回，UrlBasedViewResolver将识别这个，认为这是一个redirect请求。View的剩余部分将被作为redirect URL。

__注意：__对于添加了@ResponseStatus的方法，注解的值是优先于RedirectView设置的值的。

#### 1.4.3.4 forward:前缀

另一个可以加在view名前面的是forward:前缀，这个前缀由UrlBasedViewResolver和它的子类解析。它使用剩余的view名创建了一个InternalResourceView(最后调用RequestDispatcher.forward())，view名被当作一个URL。这个前缀和InternalResourceViewResolver和InternalResourceView一块使用，不会起什么作用。但在你主要使用一种view层技术时，它可以帮助强制转发请求到由Servlet/JSP引擎负责处理的其它资源。

### 1.4.4 ContentNegotiatingViewResolver

ContentNegotiatingViewResolver自己不解析views，而是委托给别的view resolver，选择客户端所期望的view。客户端从服务器请求一个展示结果，有两种策略可被应用：

* 对每个资源都使用唯一的URI，通常在URI后面添加不同的文件扩展来实现

    如，请求PDF，可以访问http://localhost:8080/mvc/req.pdf；请求XML，访问http://localhost:8080/mvc/req.xml

* 客户端请求资源时，使用相同的URI，同时设置Accept头来列出可以识别的media types

    如，访问PDF，访问http://localhost:8080/mvc/req同时设置Accept为application/pdf；访问XML，访问http://localhost:8080/mvc/req同时设置Accept头为text/xml。

    该策略也被称为内容协商(content negotiation)。

> Note:
> 
> 使用Accept头有一个问题，不能在浏览器中通过HTML来设置它。例如，在Firefox中，它必须是：Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8。基于这个原因，在开发基于浏览器的web应用时，通常采用每个资源有一个唯一的URI。

为了支持一个资源多种展示形式，Spring提供了ContentNegotiationViewResolver根据文件扩展名或http请求中的Accept来解析相应的view。ContentNegotiatingViewResolver自己不做view的解析工作，而是委托给由ViewResolvers指定的一系列view resolver。

    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
        <property name="viewResolvers">
            <list>
                <bean class="org.springframework.web.servlet.view.BeanNameViewResolver" />
                <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="prefix" value="/WEB-INF/jsp/" />
                    <property name="suffix" value=".jsp" />
                </bean>
            </list>
        </property>
        <property name="defaultViews">
            <list>
                <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView" />
            </list>
        </property>
    </bean>

    <bean id="content" class="com.liuyf.demos.web.mvc.SampleContentAtomView" />

在上面的配置中，如果产生的请求带有.html扩展，view resolver根据text/html来搜索匹配的view。InternalResourceViewResolver为text/html提供匹配的view。如果产生的请求带有.atom扩展，view resolver根据application/atom+xml来搜索匹配的view。这个view由BeanNameViewResolver提供，如果返回的view名为content，它将会被映射到SampleContentAtomView。如果产生的请求带有.json扩展，不考虑view名，将由defaultViews指定MappingJackson2JsonView来处理。同样，客户端如果采用Accept头，也会产生同样的效果。

> Note:
> 
> 如果ContentNegotiatingViewResolver的ViewResolvers没有明确被设置，它自动使用定义在上下文中ViewResolvers里的其中一个。


## 1.5 使用flash attributes

Flash attributes提供了一种方式，使一个请求存储的attributes有意为了在另一个请求中使用。当redircting发生时，就会有这种需求，例如，Post/Redirect/Get模式。在redirect发生前，Flash attributes是被临时保存的(如，临时保存在session中)，在redirect后这些属性可以被请求使用，请求完成时，将立即被销毁(从session移出)。

Spring MVC为了支持falsh attributes有两个主要的抽象。FlashMap是被用来持有flash attribtues，而FlashMapManager被用来存储，取回，和管理FlashMap实例。

对Flash attributes的支持始终是"on"，不需要明确的启用它。如果不是使用它，它决不会产生http session。在每个request上，都有一个从前一个请求传递过来的持有属性的"input" FlashMap和一个为让后续请求来使用的持有属性的"output" FlashMap。从Spring MVC中通过RequestContextUtils提供的静态方法可以随时访问这两个FlashMap实例。

Annotated controllers通过不直接跟FlashMap一块工作。替代的方案是@RequestMapping可以接收一个RedirectAttributes类型的参数，为redirect场景添加flash attributes。由RedirectAttributes添加的Flash attributes是自动被传播到"output" FlashMap。类似的，在redirect之后，"input" FlashMap中的属性是自动被添加到controller的Model中，服务于target URL。

> Note:
> 
> __request与flash attributes搭配使用__
> 
> 在许多别的Web框架都存在有flash attributes概念，已经被证明有时会暴露一些并发问题。从flash attributes定义上来说，flash attributes会被保存，为了下一个请求而使用。有一种可能，下一个请求可能不是预期的接收者，而是另一个异步请求(轮询或资源请求)，在这种情况中，flash attributes会被过早的移出。
> 
> 为了减少这个问题，RedirectView使用redirect目标URL的路径和请求参数来标记FlashMap实例。接下来，在查找"input" FlashMap时，默认的FlashMapManager将匹配进入的请求信息。
> 
> 这不能完全消除并发可能，但根据redirect URL中传递的这些信息可以大大减少这些问题。因此flash attributes是主要推荐在redirect场景中使用。


__\*\*\* 实践中发现，RequestContextUtils.getOutputFlashMap(request)返回为空对象，做个标记，待看完以下内容后，再查清这个问题。__
## 1.6 建造URIs

Spring MVC提供了一种机制，可以使用UriComponentsBuilder和UriComponents来构造和编码一个URI。

你可以扩展和编码一个URI模板字符串：

    UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("http://localhost:8080/spring-web-mvc-demos/person/{person}/bookings/{booking}")
        .build();
    URI uri = uriComponents.expand("42", "21").encode().toUri();

__NOTE:__UriComponents是不可变的，expand()和encode将会返回新的对象实例。

在Servlet环境中，ServletUriComponentsBuilder子类提供了从Servlet requset中复制URL信息的静态工厂方法。

    ServletUriComponentsBuilder builder = ServletUriComponentsBuilder
        .fromRequest(request)
        .replaceQueryParam("param1", "{id}")
        .build()
        .expand("123")
        .encode();

### 1.6.1 构造访问某个controller或handler method的URI

    UriComponents uriComponents = MvcUriComponentsBuilder
        .fromMethodName(BookingController.class, "getBooking", 21)
        .buildAndExpand(42);

    URI uri = uriComponets.encode().toUri();
        
### 1.6.2 在view中构造访问某个controller或handler method的URI

也可以在view中构造到相应annotated controller的URI。view层技术可以是JSP，Thymeleaf和FreeMarker。这可以通过MvcUriComponentsBuilder(通过名称连接到对应的mapping)的fromMappingName方法来实现。

每个@RequestMapping分配一个基于类首母大写并加上方法名形成的一个默认名称。如FooController中的getFoo，被分配的名称是"FC#getFoo"。可以通过创建一个HandlerMethodMappingNamingStrategy实例，并将该实例注入到RequestMappingHandlerMapping中来替换或定制该策略。默认的策略实现也会查看@RequestMapping中的name属性，如果name有值，则会实例该值。也就是说，如果按默认的mapping名进行分配时和别的mapping名发生冲突，可能通过@RequestMapping中的name属性来明确指定。

> Note:
> 
> 在初始化时，被分配的请求mapping名会被输出到TRACE级的日志中。

    <%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
    <a href="${s:mvcUrl('PAC#getAddress').arg(0, 'US').buildAndExpand('123')}">Get Address</a>

## 1.7 使用locales

Spring架构的大部分内容都支持国际化，就像Spring web MVC框架这样。DispatcherServlet可能通过客户端的locale自动解析成对应的响应信息。这是通过LocaleResolver实现的。

在接收到用户的请求时后，DispatcherServlet查找locale解析器，如果它发现locale解析器，尝试使用解析出来的结果来设置locale。使用RequestContext.getLocale()方法，可以提取由locale解析器解析出的locale对象。

除了自动解析，用户也可以添加拦截handler mapping的拦截器，在特定情况下更改对应的locale。例如，基于请求中的某个参数(parameter)。

Locale解析器和拦截器被定义在org.springframework.web.servlet.i18n包中，可以被配置在application context中。Spring提供了多个locale解析器。

### 1.7.1 获取TimeZone信息

LocaleContextResolver接口对LocaleResolver进行扩展，允许解析器提供一个更丰富的LocaleContext，即能够保含timezone信息。

用户的timezone信息可以通过RequestContext.getTimeZone()方法获取到。time zone信息将自动被在Spring ConversionService中注册的Date/Time converter和Formatter使用。

### 1.7.2 AcceptHeaderLocaleResolver

这个locale解析器，负责检查请求头中的accept-language头信息。

__Note:__这个resolver不支持time zone信息。

### 1.7.3 CookieLocaleResolver

这个locale解析器，负责存储cookie信息到客户端。判断是否Locale或TimeZone是被指定的，如果被指定，它将使用指定的信息。通过这个locale解析器的属性，可以指定cookie的名称和最大存储时间。

    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">
        <property name="cookieName" value="clientlanguage" />
        <!-- in seconds -->
        <property name="cookieMaxAge" value="100000" />
    </bean>

<table>
    <tr>
        <th>属性</th>
        <th>默认值</th>
        <th>描述</th>
    </tr>
    <tr>
        <td>cookieName</td>
        <td>类名+LOCALE</td>
        <td>cookie名字</td>
    </tr>
    <tr>
        <td>cookieMaxAge</td>
        <td>servlet容器默认</td>
        <td>cookie在客户端持久化多长时间。如果设置成-1，cookie将不持久化，客户端关闭浏览器则消失。</td>
    </tr>
    <tr>
        <td>cookiePath</td>
        <td>/</td>
        <td>限制cookie只对网站的某一部分可见。当cookiePath被指定时，</td>
    </tr>
</table>

### 1.7.4 SessionLocaleResolver

SessionLocaleResolver允许用户从session中获取Locale和TimeZone，这些取值与用户请求相关联。与之相反的是CookieLocaleResolver，从Servlet容器的HttpSession中获取locale settings存储到客户端。因此，这些信息对于每个session都是临时的，当session结束后，会丢失。

__Note:__这与扩展的session管理机制没有直接的关系，例如Spring session项目。SessionLocaleResolver仅仅计算和修改与HttpServletRequest关联的HttpSession属性。
### 1.7.5 LocaleChangeInterceptor

    <bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
        
        <property name="paramName" value="siteLanguage" />
    </bean>

    <!-- interceptors -->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <ref bean="localeChangeInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>
    
    <bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>
    
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">
        <property name="defaultLocale" value="en" />
        <property name="defaultTimeZone" value="America/Los_Angeles" />
        <property name="cookieName" value="my-cookie-name" />
        <!-- in seconds -->
        <property name="cookieMaxAge" value="100000" />
        <property name="cookiePath" value="/spring-framework-web-mvc-demos/" />
    </bean>

## 1.8 使用themes

### 1.8.1 themes概述

你可以利用Spring Web MVC的tehmes设置整个应用的look-and-feel，从而提升用户的体验。theme是一个静态资源集合，典型的是样式表和图片，这些内容通常影响着应用的可视化样式。

### 1.8.2 定义themes

为了在应用中使用themes，必须配置一个org.springframework.ui.context.ThemeSource接口的实现类。WebApplicationContext接口继承了ThemeSource接口，并委托提供的服务到特定的实现上。默认的委托实现是org.springframework.ui.context.support.ResourceBundleThemeSource实现类，从classpath根目录加载配置文件。

在使用ResourceBundleThemeSource，一个theme是被定义在一个简单的属性文件中。属性文件列出了构造这个theme的资源信息。如，

    styleSheet=/themes/cool/style.css
    background=/themes/cool/img/coolBg.jpg

在相应的view代码中，可以根据属性文件中相应的key，来引用对应的themed元素。例如，下面在JSP中引用的示例：

    <%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
    <html>
        <head>
            <link rel="stylesheet" href="<spring:theme code='styleSheet'" type="text/css" />
        </head>
        <body style="background=<spring:theme code='background' />">
            Spring Web MVC
        </body>
    </html>

默认情况下，ResourceBundleThemeSource使用空的base name前缀。结果则是，属性文件会从classpath根目录被加载。因此，你可以把theme定义的文件cool.properties放在classpath根目录，例如，/WEB-INF/classes。ResourceBundleThemeSource使用用标准的Java属性绑定加载机制，支持整个themes实现国际化。我们可以存放一个支持不同语言或背景的属性文件，如/WEB-INF/classes/cool_en.properties。

### 1.8.3 Theme解析器

在定义theme后，就像之前描述的那样，如何来决定使用那个theme。DispatcherServlet查找以themeResolver命名的bean对象，分析出使用的是那个ThemeResolver。

<table>
    <tr>
        <th>实现类</th>
        <th>描述</th>
    </tr>
    <tr>
        <td>FixedThemeResolver</td>
        <td>选择固定的theme，由defaultThemeName属性指定。</td>
    </tr>
    <tr>
        <td>SessionThemeResolver</td>
        <td>theme是被维护在用户的session中。对于每个session，它仅需要被设置一次，在session之间是不被持久化的。</td>
    </tr>
    <tr>
        <td>CookieThemeResolver</td>
        <td>被选择的theme被存储在客户端的cookie中</td>
    </tr>
</table>

## 1.9 Spring multipart 支持

### 1.9.1 介绍

Spring内置对multipart支持，在web应用中处理文件上传。可能通过实例化MultipartResolver对象来启用对multipart的支持，这个类定义在org.springframework.web.multipart包中。为了支持Commons FileUpload和Servlet 3.0 multipart请求解析的使用，Spring提供了一个实现类MultipartResolver。

__默认情况下：__Spring不处理multipart，因为一些开发者他们自己想要处理。通过在web application context上下文中声明multipart resolver来启用对multipart的处理。每个请求都被检测是否包含multipart。如果不包含，则请求继续按期望的操作被处理。如果request中包含multipart，在上下文中声明的MultipartResolver将被使用。接下来，request中的multipart属性将像别的属性一样被处理。

### 1.9.2 与Commons FileUpload一块儿使用MultipartResolver

    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- the maximum file size in bytes -->
        <property name="maxUploadSize" value="100000" />
    </bean>

__需要将相应的jar包加入到classpath中，本例中为commons-fileupload.jar。__

当Spring DispatcherServlet检测到一个multi-part请求时，它启动在上下文中的解析器来接管request。解析器将当前的HttpServletRequest包装成支持multipart文件上传的MultipartHttpServletRequest对象。通过MultipartHttpServletRequest，可以获取包含在这个请求中的multipart信息，同时也可以在controller中访问multipart文件。

### 1.9.3 与Servlet 3.0一块儿使用MultipartResolver

为了使用基于Servlet 3.0的multipart解析功能，需要在DispatcherServlet中添加multipart-config；或在编程环境中使用javax.servlet.MultipartConfigElement；或通过在Servlet类中添加javax.servlet.annotation.MultipartConfig来实现。

像maximum size和存储位置这些参数需要在Servlet注册时设置，因为Servlet 3.0不允许MultipartResolver来设置这些参数。

在Servlet 3.0 multipart解析启用后，可能使用上面采用的方式，添加StandardServletMultipartResolver到Spring上下文。

    <bean id="multipartResolver"
            class="org.springframework.web.multipart.support.StandardServletMultipartResolver">
    </bean>

### 1.9.4 处理通过form表单进行文件上传

在MultipartResolver完成它的工作后，请求将像别的请求一样被处理。首先创建一个带file input的form表单，允许用户进行文件上传。指定enctype="multipart/form-data"，让浏览器知道怎么样把form表单编码成multipart请求：

    <form method="post" action="/form" enctype="multipart/form-data">
        <input type="text" name="name"/>
        <input type="file" name="file"/>
        <input type="submit"/>
    </form>

接下为创建一个controller来处理文件上传。这个controller除了在方法中添加MultipartHttpServletRequest或MultipartFile参数外，与其它controller一样。

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
        @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }

        return "redirect:uploadFailure";
    }


在采用Servlet 3.0 multipart解析时，你也可以添加javax.servlet.http.Part作为方法参数：


    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
        @RequestParam("file") Part file) {

        InputStream inputStream = file.getInputStream();
        // store bytes from uploaded file somewhere

        return "redirect:uploadSuccess";
    }

### 1.9.5 处理来自可编程客户端的文件上传请求

在RESTful服务场景中，multipart请求也可以通过非浏览器客户端进行提交。不像浏览器，通常提交文件或简单的form属性，可编程客户端可以发送特定content type的更复杂的数据，如包含一个文件和一段json格式的数据。

    POST /mvc/fileupload
    Content-Type: multipart/mixed
    
    --edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
    Content-Disposition: form-data; name="meta-data"
    Content-Type: application/json; charset=UTF-8
    Content-Transfer-Encoding: 8bit
    
    {
    	"name": "value"
    }
    --edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
    Content-Disposition: form-data; name="file-data"; filename="file.properties"
    Content-Type: text/xml
    Content-Transfer-Encoding: 8bit
    ... File Data ...

可以通过@RequestParam("meta-data") String metadata方法参数来访问"meta-data"部分。

也可以通过@RequestPart注解替代@RequestParam注解，来接收由request part中json数据生成的强类型对象。它允许传递通过HttpMessageConverter解析的特定的multipart内容：

    @PostMapping("/mvc/")
    public String onSubmit(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    }

## 1.10 处理异常

### 1.10.1 HandlerExceptionResolver

在controller执行期间发生不期望的异常时，可以使用Spring HandlerExceptionResolver实现类来处理。HandlerExceptionResolver提供了一种与在web.xml中定义异常映射(exception mappings)类似的功能，然而它提供了更灵活的实现方式。例如，当异常抛出时，它提供了正在执行的controller信息。因此，在请求被转发到另一个URL前，这种机制给你更多的选项，返回恰当的响应结果。

除了实现HandlerExceptionResolver接口，更具体的是仅仅实现resolveException(Exception, Handler)方法并返回一个ModeAndView；你也可以使用Spring提供的SimpleMappingExceptionResolver或创建@ExceptionHandler方法。如果仅在controller中使用@ExceptionHandler方法，则该方法可以被定义在@Controller内部；如果想让更多的@Controller类使用，则可以定义在@ControllerAdvice类中。

### 1.10.2 @ExceptionHandler

HandlerExceptionResolver接口和SimpleMappingExceptionResolver实现类允许你以声明的方式将异常映射到特定的view上，在转发到这些view前，可以添加一些可选的Java逻辑。在其它情况下，尤其是不依赖view解析而是依赖于@ResponseBody方法，更简便的是直接设置response状态，同时了可以选择将错误内容写到response body中。

    @ExceptionHandler(IOException.class)
    public ResponseEntity<String> handleIOException(IOException ex) {
        return new ResponseEntity<String>(HttpStatus.INTERNAL_SERVER_ERROR);
    }

__@ExceptionHandler的value属性可以设置一个异常类型集合。如果value属性没有设置，方法参数中的异常类型将被使用。__

@ExceptionHanlder方法的返回值可以是String(被解析成view名)，ModelAndView对象，ResponseEntity或在方法上添加@ResponseBody(方法的返回值通过message转换器转换后，写入response流中)。

### 1.10.3 处理标准的Spring MVC异常

默认的DefaultHandlerExceptionResolver将Spring MVC异常转换成特定的错误状态码。默认情况下，它将被MVC namespace，MVC Java Config和DispatcherServlet注册到容器中。

[该类可以处理的异常列表](http://docs.spring.io/spring/docs/4.3.8.RELEASE/spring-framework-reference/html/mvc.html#mvc-ann-rest-spring-mvc-exceptions)

### 1.10.4 为业务异常添加@ResponseStatus注解

业务异常也可以添加@ResponseStatus注解。当异常发生时，ResponseStatusExceptionResolver设置相应的response状态码。默认情况下，DispatcherServlet会注册ResponseStatusExceptionResolver到容器中。

### 1.10.5 定制默认的Servlet容器错误页面

当response status设置成一个错误状态码，并且response body是空的情况下，Servlet容器通常会返回一个HTML格式的错误页面。为了定制容器中默认的错误页面，可以在web.xml中声明一个<error-page>元素。在Servlet 3之前，元素必须被映射到一个特定的状态码或异常类型上。从Servlet 3开始，不是必须要映射到一个错误页面，实际上可以通过指定location来定制默认的Servlet容器错误页面。

    <error-page>
        <location>/error</location>
    </error-page>

错误页面中指定的location可以是JSP页面或是在包含在容器中的别的URL(例如，@Controller中的方法)。

    @RequestMapping(path = "/error", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    @ResponseBody
    public Map<String, Object> handle(HttpServletRequest request) {

        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));

        return map;
    }

__JSP__

    <%@ page contentType="application/json" pageEncoding="UTF-8"%>
    {
        status:<%=request.getAttribute("javax.servlet.error.status_code") %>,
        reason:<%=request.getAttribute("javax.servlet.error.message") %>
    }

## 1.11 Web安全(Web Security)

Spring security项目提供很多特性来保护web应用免遭攻击。可以参考以下内容，[CSRF protection](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf)，[Security Response Headers](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#headers)和[Spring MVC Integration](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#mvc)。如果使用Spring security来保护你的应用，不要求你使用所有的特性。例如，可能通过CsrfFiltert和CsrfRequestDataValueProcessor添加到配置文件中，从而给应用添加CSRF保护。

其它的选择就是使用一个专门的Web security框架。HDIV就是一个这样的框架，并且可以和Spring MVC实现集成。

## 1.12 对约定大于配置(convention over configuration)的支持

Spring对Convention-over-configuration的支持，涵盖了MVC的三个核心组件：model，view和controller。

### 1.12.1 针对controller提供了ControllerClassNameHandlerMapping

ControllerClassNameHandlerMapping类是HandlerMapping的一个实现类，使用约定来确定request URL与Controller实例之前的映射关系。

    public class ViewShoppingCartController implements Controller {

        public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {

            return null;
        }
    }

__配置信息__

    <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"/>

    <bean id="viewShoppingCart" class="x.y.z.ViewShoppingCartController">
    </bean>

ControllerClassNameHandlerMapping查找所有定义在应用上下文中的handler(或称为Controller) bean对象，把名称中Controller去除后剩余的名称部分定义成handler mapping。因此，ViewShoppingCartController会被映射到/viewshoppingcart*请求路径。

__对于以骆驼格式命名的Controller，产生的URL也都是小写的URL。__

更多示例如下：

* WelcomeController被映射到/welcome*
* HomeController被映射到/home*

对于MultiActionController处理类，生成的mapping是更复杂一些。在接下的示例中，假设这些Controller都是MultiActionController实现类。

* AdminController被映射到/admin/*
* CatalogController被映射到/catalog/*

__由于ControllerClassNameHandlerMapping继承自AbstractHandlerMapping基类，因此你也可以定义HandlerInterceptor实例。__


### 1.12.2 针对Model提供了ModelMap(ModelAndView)

ModelMap类实际上是一个美化的Map对象，可以使添加到内部的变量，根据约定在View中显示。

例如，被添加到ModelAndView中的对象没有指定任何名称：

    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {

        List cartItems = new ArrayList();
        User user = new User();

        ModelAndView mav = new ModelAndView("displayShoppingCart"); <-- the logical view name

        mav.addObject(cartItems); <-- look ma, no name, just the object
        mav.addObject(user); <-- and again ma!

        return mav;
    }

ModelAndView类内部使用了一个ModelMap类，当对象被加入到ModelMap中时，自动为对象添加一个key。为添加的对象产生名字的策略是：在处理按包名划分对象时，如User，使用对象类的短类名。

下面的例子中，将按照上面提到的策略，在对象被添加到ModelMap实例中时产生相应的名字：

* x.y.User实例，被产生的名字是user
* x.y.Registration实例，被产生的名字是registration
* java.util.HashMap实例，被产生的名字是hashMap。
* 如果将null添加到ModelMap对象中，将引发一个IllegalArgumentException。如果，你添加的对象可能是null，你可以明确为该对象指定名称。

> Note:
> 
> 什么，没有自动的多元化？
> Spring web MVC约定大于配置(convention-over-configuration support)不支持自动多元化。也就是说，你不能把List<Person>加入到ModelAndView，并以person命名。
> 
> 在一些争论后，作出这样的决定，最后胜出者是"最小意外原则/最小惊讶原则(Principle of Least Surprise)"。



# XXXXXX. 参考资源

[JavaEE: XML Schemas for Java EE Deployment Descriptors](http://www.oracle.com/webfolder/technetwork/jsc/xml/ns/javaee/index.html)