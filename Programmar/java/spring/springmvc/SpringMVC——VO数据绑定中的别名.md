
# 前言


VO类是我们在Spring Web编程中常用的类，但是当Json数据中有key带下划线时，我们需要将其与小驼峰变量进行绑定，此时我们就需要别名功能。
一般情况下，大家熟悉的是绑定请求体内的数据，但是`GET`等一些数据不在请求体内的数据该如何绑定，这篇文档记录了解决方案。


# VO数据绑定


## 理想状况

当外部传入参数时，参数名与我们的变量一一对应，这是最理想的状况——**参数会被自动绑定**。举个例子：

- http接口
```java
@GetMapping("user")
public ResponseEntity<?> getSessionList(UserVO userVO) {
    System.out.println(userVO);
    return ResponseEntity.ok(userVO);
}
```

- 定义的VO类。
```java
public class UserVO {
    private String userId;
    private String username;
    // ignore setter and getter
}
```
- 传入的Json参数
```json
{
    "userId": "0a2f7bk8cc000f5d1f98c00ccb990ac0"
    "username": "summersea"
}
```
- 传入的Get参数
```
localhost:8080/user?userId=0a2f7bk8cc000f5d1f98c00ccb990ac0&username=summersea
```
这种情况下，传入什么参数，就会返回什么参数，一切正常。


## 约定冲突

按照刚刚的例子继续举例，有时候我们会遇到这样的要求：

- 传入的Json参数
```json
{
    "user_id": "0a2f7bk8cc000f5d1f98c00ccb990ac0"
    "user_name": "summersea"
}
```
- 传入的Get参数
```
localhost:8080/user?user_id=0a2f7bk8cc000f5d1f98c00ccb990ac0&user_name=summersea
```

通常这种情况下，说明传入参数的标准不是我们能控制的，而`Java`通用编码规范里已经明确说明了：**变量名应当使用小驼峰格式命名，不得使用下划线**。

- 两种约定的冲突导致数据绑定出现了问题。

## 冲突的解决方案

### 请求体入参

请求体入参不受控是比较好解决的，我们只需要在`VO类`的变量声明上增加一个`@JsonProperty`注解，并注明它的别名即可。

- 适配后的VO类

```java
public class UserVO {
    @JsonProperty("user_id")
    private String userId;
    @JsonProperty("user_name")
    private String username;
    // ignore setter and getter
}
```
这样做之后，当传入参数包含别名参数时，也会被自动绑定到对应的变量上。

- 可以发现，`Spring MVC`这块是依赖于`jacksonJson`的。

### url参数入参

当参数不是通过请求体传入，而是通过**url参数**传入时，我们会发现上面的方法不管用了。需要使用新的方法——自定义数据绑定器。

首先定义两个注解:

- 别名注解，写在变量名上，用来标识出该变量还有什么别的别名。
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AliasProperty {

    /** 别名列表 */
    String[] alias();
}
```

- 启动别名数据绑定的注解，写在`controller`请求方法上，表示该方法需要启用别名数据绑定。
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AliasProcessor {
}
```

然后编写对应数据绑定的方法

- 数据绑定器
```java
public class AliasDataBinder extends ExtendedServletRequestDataBinder {

    /** 日志 */
    private final Log log = LogFactory.getLog(this.getClass());

    public AliasDataBinder(Object target, String objectName) {
        super(target, objectName);
    }

    @Override
    protected void addBindValues(MutablePropertyValues mpvs, ServletRequest request) {
        super.addBindValues(mpvs, request);
        Object target = getTarget();
        if (target == null) {
            return;
        }
        Class<?> targetClass = target.getClass();
        Field[] declaredFields = targetClass.getDeclaredFields();
        for (Field declaredField : declaredFields) {
            AliasProperty annotation = declaredField.getAnnotation(AliasProperty.class);
            if (mpvs.contains(declaredField.getName()) || annotation == null) {
                continue;
            }
            for (String alias : annotation.alias()) {
                if (mpvs.contains(alias)) {
                    PropertyValue propertyValue = mpvs.getPropertyValue(alias);
                    if (propertyValue == null) {
                        log.error("Got null from mpvs. It can't happen. Shut the fuck up, my IDE.");
                    } else {
                        mpvs.add(declaredField.getName(), propertyValue.getValue());
                    }
                    break;
                }
            }
        }
    }
}
```

- 方法级处理器，用于调用数据绑定器对数据进行绑定。
```java
public class AliasModelAttributeMethodProcessor extends ServletModelAttributeMethodProcessor {

    /** 日志 */
    private final Log log = LogFactory.getLog(this.getClass());

    /** Spring应用程序上下文 */
    private final ApplicationContext applicationContext;

    public AliasModelAttributeMethodProcessor(ApplicationContext applicationContext) {
        super(true);
        this.applicationContext = applicationContext;
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getMethodAnnotation(AliasProcessor.class) != null;
    }

    @Override
    protected void bindRequestParameters(WebDataBinder binder, NativeWebRequest request) {
        AliasDataBinder aliasDataBinder = new AliasDataBinder(binder.getTarget(), binder.getObjectName());
        RequestMappingHandlerAdapter requestMappingHandlerAdapter =
                this.applicationContext.getBean(RequestMappingHandlerAdapter.class);
        WebBindingInitializer webBindingInitializer = requestMappingHandlerAdapter.getWebBindingInitializer();
        if (webBindingInitializer == null) {
            log.error("WebBindingInitializer is null.");
            return;
        }
        webBindingInitializer.initBinder(aliasDataBinder);
        ServletRequest nativeRequest = request.getNativeRequest(ServletRequest.class);
        if (nativeRequest == null) {
            log.error("Got null when execute: NativeWebRequest.getNativeRequest(ServletRequest.class)");
            return;
        }
        aliasDataBinder.bind(nativeRequest);
    }
}
```

- WebConfig，将自己写的方法级处理器添加到处理器链条中。
```java
@Configuration
public class WebMvcConfiguration implements ApplicationContextAware {

    /** 日志 */
    private final Log log = LogFactory.getLog(this.getClass());

    private RequestMappingHandlerAdapter adapter;

    private ApplicationContext applicationContext;

    @Autowired
    public void setAdapter(RequestMappingHandlerAdapter adapter) {
        this.adapter = adapter;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @PostConstruct
    protected void injectSelfMethodArgumentResolver() {
        List<HandlerMethodArgumentResolver> argumentResolverList = new ArrayList<>();
        argumentResolverList.add(new AliasModelAttributeMethodProcessor(applicationContext));
        List<HandlerMethodArgumentResolver> argumentResolvers = adapter.getArgumentResolvers();
        if (argumentResolvers != null) {
            argumentResolverList.addAll(argumentResolvers);
        } else {
            log.warn("Got null resolver from RequestMappingHandlerAdapter.");
        }
        adapter.setArgumentResolvers(argumentResolverList);
    }
}
```

之后，VO类进行如下改造以适配url入参后即可顺利绑定数据。
```java
public class UserVO {
    @AliasProperty(alias = "user_id")
    private String userId;
    @AliasProperty(alias = "user_name")
    private String username;
    // ignore setter and getter
}
```




