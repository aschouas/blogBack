---
title: SpringMvc rest
date: 2017-09-26 10:10:53
tags: java
toc: true
categories: 技术研究
---
<blockquote class="blockquote-center">一个人去看医生，说他得了抑郁症，生活如此的尖酸刻薄，他孤独而绝望。医生说：那个最有名的小丑在城里，去找他吧，他能让你开心起来。“但是啊医生，”他突然大哭，“我就是那个小丑啊。”</blockquote>

<!--more-->

## 什么是rest

REST本质上是使用URL来访问资源种方式。众所周知，URL就是我们平常使用的请求地址了，其中包括两部分：请求方式与请求路径，比较常见的请求方式是GET与POST，但在REST中又提出了几种其它类型的请求方式，汇总起来有六种：GET、POST、PUT、DELETE、HEAD、OPTIONS。尤其是前四种，正好与CRUD（Create-Retrieve-Update-Delete，增删改查）四种操作相对应，例如，GET（查）、POST（增）、PUT（改）、DELETE（删），这正是REST与CRUD的异曲同工之妙！需要强调的是，REST是“面向资源”的，这里提到的资源，实际上就是我们常说的领域对象，在系统设计过程中，我们经常通过领域对象来进行数据建模。
REST是一个“无状态”的架构模式，因为在任何时候都可以由客户端发出请求到服务端，最终返回自己想要的数据，当前请求不会受到上次请求的影响。也就是说，服务端将内部资源发布REST服务，客户端通过URL来访问这些资源，这不就是SOA所提倡的“面向服务”的思想吗？所以，REST也被人们看做是一种“轻量级”的SOA实现技术，因此在企业级应用与互联网应用中都得到了广泛应用。

| REST请求 | 描述 | 
| ------ | ------ | 
| GET:/users | 获取所有用户 | 
| GET:/users/1 | 获取ID为1的用户 | 
| PUT:/users/1 | 更新ID为1的用户 | 
| DELETE:/users/1 | 删除ID为1的用户 |
| POST:/users | 创建用户 |

可见，请求路径相同，但请求方式不同，所代表的业务操作也不同，例如，/advertiser/1这个请求，带有GET、PUT、DELETE三种不同的请求方式，对应三种不同的业务操作。
虽然REST看起来还是很简单的，实际上我们往往需要提供一个REST框架，让其实现前后端分离架构，让开发人员将精力集中在业务上，而并非那些具体的技术细节。下面我们将使用Java技术来实现这个REST框架，整体框架会基于Spring进行开发。


## 实现REST框架

- 统一响应架构

使用REST框架实现前后端分离架构，我们需要首先确定返回的JSON响应结构是统一的，也就是说，每个REST请求将返回相同结构的JSON响应结构。不妨定义一个相对通用的JSON响应结构，其中包含两部分：元数据与返回值，其中，元数据表示操作是否成功与返回值消息等，返回值对应服务端方法所返回的数据。该JSON响应结构如下：

```json
{
	"meta": {
		"success": true,
		"message": "ok"
	},
	"data": ...
}
```

为了在框架中映射以上JSON响应结构，我们需要编写一个Response类与其对应：

```java
public class Response {  
  
    private static final String OK = "ok";  
    private static final String ERROR = "error";  
  
    private Meta meta;  
    private Object data;  
  
    public Response success() {  
        this.meta = new Meta(true, OK);  
        return this;  
    }  
  
    public Response success(Object data) {  
        this.meta = new Meta(true, OK);  
        this.data = data;  
        return this;  
    }  
  
    public Response failure() {  
        this.meta = new Meta(false, ERROR);  
        return this;  
    }  
  
    public Response failure(String message) {  
        this.meta = new Meta(false, message);  
        return this;  
    }  
  
    public Meta getMeta() {  
        return meta;  
    }  
  
    public Object getData() {  
        return data;  
    }  
  
    public class Meta {  
  
        private boolean success;  
        private String message;  
  
        public Meta(boolean success) {  
            this.success = success;  
        }  
  
        public Meta(boolean success, String message) {  
            this.success = success;  
            this.message = message;  
        }  
  
        public boolean isSuccess() {  
            return success;  
        }  
  
        public String getMessage() {  
            return message;  
        }  
    }  
}  
```

以上Response类包括两类通用返回值消息：ok与error，还包括两个常用的操作方法：success( )与failure( )，通过一个内部类来展现元数据结构，我们在下文中多次会使用该Response类。
实现该REST框架需要考虑许多问题，首当其冲的就是对象序列化问题。

- 实现对象序列化

想要解释什么是对象序列化？不妨通过一些例子进行说明。比如，通过浏览器发送了一个普通的HTTP请求，该请求携带了一个JSON格式的参数，在服务端需要将该JSON参数转换为普通的Java对象，这个转换过程称为序列化。再比如，在服务端获取了数据，此时该数据是一个普通的Java对象，然后需要将这个Java对象转换为JSON字符串，并将其返回到浏览器中进行渲染，这个转换过程称为反序列化。不管是序列化还是反序列化，我们一般都称为序列化。
实际上，Spring MVC已经为我们提供了这类序列化特性，只需在Controller的方法参数中使用@RequestBody注解定义需要反序列化的参数即可，如以下代码片段：

```java
@Controller  
public class AdvertiserController {  
  
    @RequestMapping(value = "/advertiser", method = RequestMethod.POST)  
    public Response createAdvertiser(@RequestBody AdvertiserParam advertiserParam) {  
        ...  
    }  
}  
```

若需要对Controller的方法返回值进行序列化，则需要在该返回值上使用@ResponseBody注解来定义，如以下代码片段：

```java
@Controller  
public class AdvertiserController {  
  
    @RequestMapping(value = "/advertiser/{id}", method = RequestMethod.GET)  
    public @ResponseBody Response getAdvertiser(@PathVariable("id") String advertiserId) {  
        ...  
    }  
}
```

当然，@ResponseBody注解也可以定义在类上，这样所有的方法都继承了该特性。由于经常会使用到@ResponseBody注解，所以Spring提供了一个名为@RestController的注解来取代以上的@Controller注解，这样我们就可以省略返回值前面的@ResponseBody注解了，但参数前面的@RequestBody注解是无法省略的。实际上，看看Spring中对应@RestController注解的源码便可知晓：

```java
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Controller  
@ResponseBody  
public @interface RestController {  
  
    String value() default "";  
}  
```

可见，@RestController注解已经被@Controller与@ResponseBody注解定义过了，Spring框架会识别这类注解。需要注意的是，该特性在Spring 4.0中才引入。
因此，我们可将以上代码进行如下改写：

```java
@RestController  
public class AdvertiserController {  
  
    @RequestMapping(value = "/advertiser", method = RequestMethod.POST)  
    public Response createAdvertiser(@RequestBody AdvertiserParam advertiserParam) {  
        ...  
    }  
  
    @RequestMapping(value = "/advertiser/{id}", method = RequestMethod.GET)  
    public Response getAdvertiser(@PathVariable("id") String advertiserId) {  
        ...  
    }  
}  
```

除了使用注解来定义序列化行为以外，我们还需要使用Jackson来提供JSON的序列化操作，在Spring配置文件中只需添加以下配置即可：

```xml
<mvc:annotation-driven>  
    <mvc:message-converters>  
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>  
    </mvc:message-converters>  
</mvc:annotation-driven>  
```

若需要对Jackson的序列化行为进行定制，比如，排除值为空属性、进行缩进输出、将驼峰转为下划线、进行日期格式化等，这又如何实现呢？
首先，我们需要扩展Jackson提供的ObjectMapper类，代码如下：

```java
public class CustomObjectMapper extends ObjectMapper {  
  
    private boolean camelCaseToLowerCaseWithUnderscores = false;  
    private String dateFormatPattern;  
  
    public void setCamelCaseToLowerCaseWithUnderscores(boolean camelCaseToLowerCaseWithUnderscores) {  
        this.camelCaseToLowerCaseWithUnderscores = camelCaseToLowerCaseWithUnderscores;  
    }  
  
    public void setDateFormatPattern(String dateFormatPattern) {  
        this.dateFormatPattern = dateFormatPattern;  
    }  
  
    public void init() {  
        // 排除值为空属性  
        setSerializationInclusion(JsonInclude.Include.NON_NULL);  
        // 进行缩进输出  
        configure(SerializationFeature.INDENT_OUTPUT, true);  
        // 将驼峰转为下划线  
        if (camelCaseToLowerCaseWithUnderscores) {  
            setPropertyNamingStrategy(PropertyNamingStrategy.CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES);  
        }  
        // 进行日期格式化  
        if (StringUtil.isNotEmpty(dateFormatPattern)) {  
            DateFormat dateFormat = new SimpleDateFormat(dateFormatPattern);  
            setDateFormat(dateFormat);  
        }  
    }  
}  
```

然后，将CustomObjectMapper注入到MappingJackson2HttpMessageConverter中，Spring配置如下：

```xml
<bean id="objectMapper" class="com.xxx.api.json.CustomObjectMapper" init-method="init">  
    <property name="camelCaseToLowerCaseWithUnderscores" value="true"/>  
    <property name="dateFormatPattern" value="yyyy-MM-dd HH:mm:ss"/>  
</bean>  
  
<mvc:annotation-driven>  
    <mvc:message-converters>  
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">  
            <property name="objectMapper" ref="objectMapper"/>  
        </bean>  
    </mvc:message-converters>  
</mvc:annotation-driven>  
```

通过以上过程，我们已经完成了一个基于Spring MVC的REST框架，只不过该框架还非常单薄，还缺乏很多关键性特性，尤其是异常处理。

- 处理异常行为

在Spring MVC中，我们可以使用AOP技术，编写一个全局的异常处理切面类，用它来统一处理所有的异常行为，在Spring 3.2中才开始提供。使用法很简单，只需定义一个类，并通过@ControllerAdvice注解将其标注即可，同时需要使用@ResponseBody注解表示返回值可序列化为JSON字符串。代码如下：

```java
@ControllerAdvice  
@ResponseBody  
public class ExceptionAdvice {  
  
    /** 
     * 400 - Bad Request 
     */  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(HttpMessageNotReadableException.class)  
    public Response handleHttpMessageNotReadableException(HttpMessageNotReadableException e) {  
        logger.error("参数解析失败", e);  
        return new Response().failure("could_not_read_json");  
    }  
  
    /** 
     * 405 - Method Not Allowed 
     */  
    @ResponseStatus(HttpStatus.METHOD_NOT_ALLOWED)  
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)  
    public Response handleHttpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {  
        logger.error("不支持当前请求方法", e);  
        return new Response().failure("request_method_not_supported");  
    }  
  
    /** 
     * 415 - Unsupported Media Type 
     */  
    @ResponseStatus(HttpStatus.UNSUPPORTED_MEDIA_TYPE)  
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)  
    public Response handleHttpMediaTypeNotSupportedException(Exception e) {  
        logger.error("不支持当前媒体类型", e);  
        return new Response().failure("content_type_not_supported");  
    }  
  
    /** 
     * 500 - Internal Server Error 
     */  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler(Exception.class)  
    public Response handleException(Exception e) {  
        logger.error("服务运行异常", e);  
        return new Response().failure(e.getMessage());  
    }  
}  
```

可见，在ExceptionAdvice类中包含一系列的异常处理方法，每个方法都通过@ResponseStatus注解定义了响应状态码，此外还通过@ExceptionHandler注解指定了具体需要拦截的异常类。以上过程只是包含了一部分的异常情况，若需处理其它异常，可添加方法具体的方法。需要注意的是，在运行时从上往下依次调用每个异常处理方法，匹配当前异常类型是否与@ExceptionHandler注解所定义的异常相匹配，若匹配，则执行该方法，同时忽略后续所有的异常处理方法，最终会返回经JSON序列化后的Response对象。

- 支持参数验证

我们回到上文所提到的示例，这里处理一个普通的POST请求，代码如下：

```java
@RestController  
public class AdvertiserController {  
  
    @RequestMapping(value = "/advertiser", method = RequestMethod.POST)  
    public Response createAdvertiser(@RequestBody AdvertiserParam advertiserParam) {  
        ...  
    }  
}  
```

其中，AdvertiserParam参数包含若干属性，通过以下类结构可见，它是一个传统的POJO：

```java
public class AdvertiserParam {  
  
    private String advertiserName;  
      
    private String description;  
  
    // 省略 getter/setter 方法  
}  
```

如果业务上需要确保AdvertiserParam对象的advertiserName属性必填，如何实现呢？
若将这类参数验证的代码写死在Controller中，势必会与正常的业务逻辑搅在一起，导致责任不够单一，违背于“单一责任原则”。建议将其参数验证行为从Controller中剥离出来，放到另外的类中，这里仅提供一个@Valid注解来定义AdvertiserParam参数，并在AdvertiserParam类中通过@NotEmpty注解来定义advertiserName属性，就像下面这样：

```java
@RestController  
public class AdvertiserController {  
  
    @RequestMapping(value = "/advertiser", method = RequestMethod.POST)  
    public Response createAdvertiser(@RequestBody @Valid AdvertiserParam advertiserParam) {  
        ...  
    }  
}  
  
public class AdvertiserParam {  
  
    @NotEmpty  
    private String advertiserName;  
      
    private String description;  
  
    // 省略 getter/setter 方法  
}  
```

这里的@Valid注解实际上是Validation Bean规范提供的注解，该规范已由Hibernate Validator框架实现，因此需要添加以下Maven依赖到pom.xml文件中：

```xml
<dependency>  
    <groupId>org.hibernate</groupId>  
    <artifactId>hibernate-validator</artifactId>  
    <version>${hibernate-validator.version}</version>  
</dependency>  
```

需要注意的是，Hibernate Validator与Hibernate没有任何依赖关系，唯一有联系的只是都属于JBoss公司的开源项目而已。
要实现@NotEmpty注解的功能，我们需要做以下几件事情。

首先，定义一个@NotEmpty注解类，代码如下：

```java
@Documented  
@Target({ElementType.FIELD, ElementType.PARAMETER})  
@Retention(RetentionPolicy.RUNTIME)  
@Constraint(validatedBy = NotEmptyValidator.class)  
public @interface NotEmpty {  
  
    String message() default "not_empty";  
  
    Class<?>[] groups() default {};  
  
    Class<? extends Payload>[] payload() default {};  
}  
```

以上注解类必须包含message、groups、payload三个属性，因为这是规范所要求的，此外，需要通过@Constraint注解指定一个验证器类，这里对应的是NotEmptyValidator，其代码如下：

```java
public class NotEmptyValidator implements ConstraintValidator<NotEmpty, String> {  
  
    @Override  
    public void initialize(NotEmpty constraintAnnotation) {  
    }  
  
    @Override  
    public boolean isValid(String value, ConstraintValidatorContext context) {  
        return StringUtil.isNotEmpty(value);  
    }  
}  
```

以上验证器类实现了ConstraintValidator接口，并在该接口的isValid( )方法中完成了具体的参数验证逻辑。需要注意的是，实现接口时需要指定泛型，第一个参数表示验证注解类型（NotEmpty），第二个参数表示需要验证的参数类型（String）。

然后，我们需要在Spring配置文件中开启该特性，需添加如下配置：

```xml
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>  
```

最后，需要在全局异常处理类中添加参数验证处理方法，代码如下：

```java
@ControllerAdvice  
@ResponseBody  
public class ExceptionAdvice {  
  
    /** 
     * 400 - Bad Request 
     */  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(ValidationException.class)  
    public Response handleValidationException(ValidationException e) {  
        logger.error("参数验证失败", e);  
        return new Response().failure("validation_exception");  
    }  
}  
```

至此，REST框架已集成了Bean Validation特性，我们可以使用各种注解来完成所需的参数验证行为了。
看似该框架可以在本地成功跑起来，整个架构包含两个应用，前端应用提供纯静态的HTML页面，后端应用发布REST API，前端需要通过AJAX调用后端发布的REST API，然而AJAX是不支持跨域访问的，也就是说，前后端两个应用必须在同一个域名下才能访问。这是非常严重的技术障碍，一定需要找到解决方案。


- 解决跨域问题

[CORS全称为Cross Origin Resource Sharing（跨域资源共享），服务端只需添加相关响应头信息，即可实现客户端发出AJAX跨域请求。]
CORS技术非常简单，易于实现，目前绝大多数浏览器均已支持该技术（IE8浏览器也支持了），服务端可通过任何编程语言来实现，只要能将CORS响应头写入response对象中即可。
下面我们继续扩展REST框架，通过CORS技术实现AJAX跨域访问。
首先，我们需要编写一个Filter，用于过滤所有的HTTP请求，并将CORS响应头写入response对象中，代码如下：

```java
public class CorsFilter implements Filter  {

	private static final Logger log = Logger.getLogger(CorsFilter.class);
	
	private String allowOrigin;
	private String allowMethods;
	private String allowCredentials;
	private String allowHeaders;
	private String exposeHeaders;

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		allowOrigin = filterConfig.getInitParameter("allowOrigin");
		allowMethods = filterConfig.getInitParameter("allowMethods");
		allowCredentials = filterConfig.getInitParameter("allowCredentials");
		allowHeaders = filterConfig.getInitParameter("allowHeaders");
		exposeHeaders = filterConfig.getInitParameter("exposeHeaders");
	}

	  
	 
	@Override
	public void doFilter(ServletRequest req, ServletResponse res,
			FilterChain chain) throws IOException, ServletException {
		//System.out.println("心跳    inint dofileter commos");
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
	//	String currentOrigin = request.getHeader("Origin");
		
	
		
		//log.debug("currentOrigin : " + currentOrigin);
		if (StringUtils.isNotEmpty(allowOrigin)) {
			List<String> allowOriginList = Arrays .asList(allowOrigin.split(","));
			log.debug("allowOriginList : " + allowOrigin);
			if (CollectionUtils.isNotEmpty(allowOriginList)) {
			     //if (allowOriginList.contains(currentOrigin)) {
					response.setHeader("Access-Control-Allow-Origin","*");
				//}
			}
		}
		if (StringUtils.isNotEmpty(allowMethods)) {
			response.setHeader("Access-Control-Allow-Methods", allowMethods);
		}
		if (StringUtils.isNotEmpty(allowCredentials)) {
			response.setHeader("Access-Control-Allow-Credentials",
					allowCredentials);
		}
		if (StringUtils.isNotEmpty(allowHeaders)) {
			response.setHeader("Access-Control-Allow-Headers", allowHeaders);
		}
		if (StringUtils.isNotEmpty(exposeHeaders)) {
			response.setHeader("Access-Control-Expose-Headers", exposeHeaders);
		}
		chain.doFilter(req, res);
	}

	@Override
	public void destroy() {
	}

}
```

以上CorsFilter将从web.xml中读取相关Filter初始化参数，并将在处理HTTP请求时将这些参数写入对应的CORS响应头中，下面大致描述一下这些CORS响应头的意义：
- Access-Control-Allow-Origin：允许访问的客户端域名，例如：http://web.xxx.com，若为*，则表示从任意域都能访问，即不做任何限制。
- Access-Control-Allow-Methods：允许访问的方法名，多个方法名用逗号分割，例如：GET,POST,PUT,DELETE,OPTIONS。
- Access-Control-Allow-Credentials：是否允许请求带有验证信息，若要获取客户端域下的cookie时，需要将其设置为true。
- Access-Control-Allow-Headers：允许服务端访问的客户端请求头，多个请求头用逗号分割，例如：Content-Type。
- Access-Control-Expose-Headers：允许客户端访问的服务端响应头，多个响应头用逗号分割。
	
需要注意的是，CORS规范中定义Access-Control-Allow-Origin只允许两种取值，要么为*，要么为具体的域名，也就是说，不支持同时配置多个域名。为了解决跨多个域的问题，需要在代码中做一些处理，这里将Filter初始化参数作为一个域名的集合（用逗号分隔），只需从当前请求中获取Origin请求头，就知道是从哪个域中发出的请求，若该请求在以上允许的域名集合中，则将其放入Access-Control-Allow-Origin响应头，这样跨多个域的问题就轻松解决了。

以下是web.xml中配置CorsFilter的方法：

```xml
<filter>  
    <filter-name>corsFilter</filter-name>  
    <filter-class>com.xxx.api.cors.CorsFilter</filter-class>  
    <init-param>  
        <param-name>allowOrigin</param-name>  
        <param-value>http://web.xxx.com</param-value>  
    </init-param>  
    <init-param>  
        <param-name>allowMethods</param-name>  
        <param-value>GET,POST,PUT,DELETE,OPTIONS</param-value>  
    </init-param>  
    <init-param>  
        <param-name>allowCredentials</param-name>  
        <param-value>true</param-value>  
    </init-param>  
    <init-param>  
        <param-name>allowHeaders</param-name>  
        <param-value>Content-Type</param-value>  
    </init-param>  
</filter>  
<filter-mapping>  
    <filter-name>corsFilter</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>  
```

完成以上过程即可实现AJAX跨域功能了，但似乎还存在另外一个问题，由于REST是无状态的，后端应用发布的REST API可在用户未登录的情况下被任意调用，这显然是不安全的，如何解决这个问题呢？我们需要为REST请求提供安全机制。


- 提供安全机制

解决REST安全调用问题，可以做得很复杂，也可以做得特简单，可按照以下过程提供REST安全机制：

- 当用户登录成功后，在服务端生成一个token，并将其放入内存中（可放入JVM或Redis中），同时将该token返回到客户端。
- 在客户端中将返回的token写入cookie中，并且每次请求时都将token随请求头一起发送到服务端。
- 提供一个AOP切面，用于拦截所有的Controller方法，在切面中判断token的有效性。
- 当登出时，只需清理掉cookie中的token即可，服务端token可设置过期时间，使其自行移除。
	
首先，我们需要定义一个用于管理token的接口，包括创建token与检查token有效性的功能。代码如下：


```java
public interface TokenManager {  
  
    String createToken(String username);  
  
    boolean checkToken(String token);  
}  
```

然后，我们可提供一个简单的TokenManager实现类，将token存储到JVM内存中。代码如下：

```java
public class DefaultTokenManager implements TokenManager {  
  
    private static Map<String, String> tokenMap = new ConcurrentHashMap<>();  
  
    @Override  
    public String createToken(String username) {  
        String token = CodecUtil.createUUID();  
        tokenMap.put(token, username);  
        return token;  
    }  
  
    @Override  
    public boolean checkToken(String token) {  
        return !StringUtil.isEmpty(token) && tokenMap.containsKey(token);  
    }  
} 
```

需要注意的是，如果需要做到分布式集群，建议基于Redis提供一个实现类，将token存储到Redis中，并利用Redis与生俱来的特性，做到token的分布式一致性。
然后，我们可以基于Spring AOP写一个切面类，用于拦截Controller类的方法，并从请求头中获取token，最后对token有效性进行判断。代码如下：

```java
public class SecurityAspect {  
  
    private static final String DEFAULT_TOKEN_NAME = "X-Token";  
  
    private TokenManager tokenManager;  
    private String tokenName;  
  
    public void setTokenManager(TokenManager tokenManager) {  
        this.tokenManager = tokenManager;  
    }  
  
    public void setTokenName(String tokenName) {  
        if (StringUtil.isEmpty(tokenName)) {  
            tokenName = DEFAULT_TOKEN_NAME;  
        }  
        this.tokenName = tokenName;  
    }  
  
    public Object execute(ProceedingJoinPoint pjp) throws Throwable {  
        // 从切点上获取目标方法  
        MethodSignature methodSignature = (MethodSignature) pjp.getSignature();  
        Method method = methodSignature.getMethod();  
        // 若目标方法忽略了安全性检查，则直接调用目标方法  
        if (method.isAnnotationPresent(IgnoreSecurity.class)) {  
            return pjp.proceed();  
        }  
        // 从 request header 中获取当前 token  
        String token = WebContext.getRequest().getHeader(tokenName);  
        // 检查 token 有效性  
        if (!tokenManager.checkToken(token)) {  
            String message = String.format("token [%s] is invalid", token);  
            throw new TokenException(message);  
        }  
        // 调用目标方法  
        return pjp.proceed();  
    }  
}  
```

若要使SecurityAspect生效，则需要添加如下Spring 配置：

```xml
<bean id="securityAspect" class="com.xxx.api.security.SecurityAspect">  
    <property name="tokenManager" ref="tokenManager"/>  
    <property name="tokenName" value="X-Token"/>  
</bean>  
  
<aop:config>  
    <aop:aspect ref="securityAspect">  
        <aop:around method="execute" pointcut="@annotation(org.springframework.web.bind.annotation.RequestMapping)"/>  
    </aop:aspect>  
</aop:config>  
```

最后，别忘了在web.xml中添加允许的X-Token响应头，配置如下：

```xml
<init-param>  
    <param-name>allowHeaders</param-name>  
    <param-value>Content-Type,X-Token</param-value>  
</init-param>  
```

**路漫漫**










