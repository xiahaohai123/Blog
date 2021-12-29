
## 参考资料
[官方文献——Servlet Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)

## 相关文章
[Spring Security笔记其一——核心架构](http://summersea.top:8090/archives/springsecurity%E7%AC%94%E8%AE%B0%E5%85%B6%E4%B8%80%E6%9E%B6%E6%9E%84)

## 本章概要

上一章简单介绍且剖析了`Spring Security`是如何工作的，本章会介绍`Spring Security`是如何认证的。

## 组件大纲


引用一下官方文档的组件大纲:

- SecurityContextHolder: 该类负责存储**通过认证**的认证信息。
- SecurityContext: 由`SecurityContextHolder`持有，内部包含了通过认证的认证信息。
- Authentication: 可以交给`AuthenticationManager`，让其对`Authentication`内的凭据进行认证。也可以在认证后通过`SecurityContext`中获取以获取当前线程下的用户信息。
- GrantedAuthority: 在`Authentication`中，对`Principal`授予的权限。
- AuthenticationManager: 一个接口，定义了`Spring Security`的`Filter`如何执行认证。
- ProviderManager: 是`AuthenticationManager`的所有实现类中，最常使用的实现类。
- AuthenticationProvider: 由`ProviderManager`调用，用来匹配对应的`Authentication`的实现类，当匹配上后，则使用该`Provider`进行校验。
- Request Credentials with `AuthenticationEntryPoint`(通过`AuthenticationEntryPoint`请求凭证): 用来向客户端请求凭证(比如重定向到登陆页面)。
- AbstractAuthenticationProcessingFilter: 一个基础的用以认证的`Filter`。这个`Filter`的代码给出了一个非常好的案例，**该案例诠释了高级身份认证流程，并解释了各个组件是如何组成一个系统进行运作的。**




## 让我们走一遍流程

上一章我们结束于`Security Filter`，那么我们这一章的流程也从这里开始，并只挑其中的认证用过滤器走一遍流程，并对流程中用到的各个组件进行介绍与解析。
如果对认证的组件不熟悉，流程可能会看得不太懂，可以先往下滑，看看各个组件的功能。

### 登陆型过滤器

鉴于大纲里写了`AbstractAthenticationProcessingFilter`给出了一个非常棒的案例，那么我们就以该`Filter`的入口为起点，一步一步向里面探寻。
![image.png](http://summersea.top:8090/upload/2021/12/image-8fe386b86312446aa6afd6ea2e03f4bb.png)




















首先，请求会在`SecurityFilterChain`里经过一个又一个`Filter`的过滤，经过前面的安全型`Filter`之后，进入了认证`Filter`。
根据`AbstractAuthenticationProcessingFilter`给出的模板方法案例，认证流程有以下步骤：

1. 检查该请求是否需要由此过滤器进行过滤，检查项可以包括但不限于：
	- 请求的路径是否需要由该过滤器进行认证
	- 该请求的访问凭据是否应该由该过滤器受理

如果发现请求不需要本过滤器过滤，则直接调用`chain.doFilter()`，将信息交给下一个过滤器过滤。

2. 当发现该请求在本过滤器的业务范围内时，本`Filter`就需要尝试对请求进行认证，认证方法在`AbstractAuthenticationProcessingFilter`中并未实现，这件事由子类负责实现。根据`Spring Security`默认提供的`UsernamePasswordAuthenticationFilter`中实现的方法来看，`Filter`需要实现的认证步骤为以下两点：
	1. 从请求体中抽取出**凭证信息**和**其他详细信息**，并将信息设置进一个新建的`Authentication`实例，此时，该`Authentication`的`isAuthenticated`应当返回`false`。
	2. 将构建的`Authentication`实例交给`AuthenticationManager`以进行具体的认证流程。

3. 经过认证后，可以根据不同的`sessionStrategy(会话处理策略)`对认证信息进行自定义操作。

4. 执行成功认证后要做的事：
	1. 将认证成功的`Authentication`实例交给`SecurityContext`，这样，在本`Filter`之后的业务都可以拿到`Authentication`，而后面的认证`Filter`也可以根据`SecurityContext`中有没有`Authentication`来判断该请求是否已经被认证过了。
	2. 执行`RememberMeService`的相关业务，让会话在线状态能被保持，这事也可以不做，全看用户自己怎么配置。
	3. 执行`AuthenticationSuccssHandler`的`onAuthenticationSuccess()`方法，常见的使用场景可以是发送日志消息等。

5. 当认证过程中抛出了异常，由`Filter`接住了异常，也有要做的事：
	1. 清除`SecurityContext`。
	2. 执行`RemeberMeService`的相关业务，可以对原本在线的会话进行下线操作。
	3. 执行`AuthenticationFailureHandler`的`onAuthenticationFailure()`方法，用户可以用这个记一下日志、确定重定向的`URL`以要求用户登录。



`AbstractAuthenticationProcessingFilter`的模板方法如下:

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
		throws IOException, ServletException {
	if (!requiresAuthentication(request, response)) {
		chain.doFilter(request, response);
		return;
	}
	try {
		Authentication authenticationResult = attemptAuthentication(request, response);
		if (authenticationResult == null) {
			// return immediately as subclass has indicated that it hasn't completed
			return;
		}
		this.sessionStrategy.onAuthentication(authenticationResult, request, response);
		// Authentication success
		if (this.continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}
		successfulAuthentication(request, response, chain, authenticationResult);
	}
	catch (InternalAuthenticationServiceException failed) {
		this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
		unsuccessfulAuthentication(request, response, failed);
	}
	catch (AuthenticationException ex) {
		// Authentication failed
		unsuccessfulAuthentication(request, response, ex);
	}
}
```

`AbstractAuthenticationFilter`是典型的**登录型过滤器**，所以该过滤器在认证后，无论结果成功与否，都会直接`return`或`throw`或直接提交`response`给客户端。

- **那么如何保证用户访问其他接口时不会因为没有认证信息而被拦截？**
- 在初始的`Spring Security`中，有一个`SecurityContextPersistenceFilter`，该过滤器是顺位第2，相较于`UsernamePasswordAuthenticationFilter`的顺位第6来说，它被调用的更早。在`SecurityContextPersistenceFilter`的`doFilter()`方法中，会在`finally`语句块中将`SecurityContext`缓存，当下一个请求进来时就可以取出`SecurityContext`并置入`SecurityContextHolder`中。


### 接口认证型过滤器
这种类型的过滤器与登陆型过滤器的思路基本一样，唯一的区别就是，**该过滤器在认证成功后会继续调用过滤器链以继续业务**，最终直到`Servlet`。
- 这种过滤器在认证外部系统的`token`时非常有用。

## 组件介绍

### 安全上下文组件组
这一组组件贯穿了整个认证流程。
#### SecurityContextHolder
`SecurityContextHolder`是认证模组的核心，其中包含着`SecurityContext`。`Spring Security`会将经过认证的用户信息存入这个组件，**也要求存入该组件的用户信息必须是经过认证的**，因为这个组件被使用时，`Spring Security`就会认为里面的用户信息就是经过认证的（**约定**）。
![image.png](http://summersea.top:8090/upload/2021/12/image-70818f653e5341abb799cafe479f8c50.png)

- 通常情况下，`SecurityContextHolder`使用`ThreadLocal`策略，因为后续业务和`Spring Security`处在同一个线程中，后续业务可以通过这个组件获取到正确的认证用户信息。
- 策略可以配置，其他配置有：全局策略、子线程策略。

#### SecurityContext
这个组件实例可以通过`SecurityContextHolder`获得，内部包含认证用户信息。

#### Authentication

这个组件在`Spring Security`领域里为两个目标服务：

- 作为`AuthenticationManager`的输入，用以完成具体的认证过程，认证之前`Authentication`的实例调用`isAuthenticated()`时会返回false。
- 代表当前已经经过认证的用户，这种实例可以通过`SecurityContext`获得。

一个`Authenticaion`实例包含了以下信息：

- `principal`: 代表一个用户。
- `credentials`: 凭证信息，常见情况是密码。
- `authorities`: 这个用户被授予的权限信息。



### 具体的身份认证组件
这里的组件负责执行具体的认证业务。


#### 身份认证管理器——AuthenticationManager
`Spring Security`的`Filter`在析取出`Authentication`实例后，通过调用该组件的`authentication()`方法来执行认证。
`AuthenticationManager`是一个接口标准，它有多种实现，而最常用的实现是`ProviderManager`。

#### 身份认证管理器的最常用实现——ProviderManager
`ProviderManager`维护了一个`AuthenticationProvider`集合，以责任链模式按顺序调用其的认证方法。
![image.png](http://summersea.top:8090/upload/2021/12/image-a80ce2b1694d414ab6ad1fb05e09ab7f.png)











- 每一个`AuthenticationProvider`都有机会认证传入的`Authentication`，当一个`AuthenticationProvider`受理了传入的`Authentication`，它就要负责表达认证是否成功，如果不受理，则该`Authencatition`会被交给下一个`AuthenticationProvider`。

- 如果没有一个`AuthenticationProvider`受理`Authentication`则会抛出`ProviderNotFoundException`，该异常继承自`AuthenticationException`。

`ProviderManager`可以配置父级的`AuthenticationManager`，当没有`AuthenticationProvider`受理认证信息的时候，`ProviderManager`会向上咨询。
![image.png](http://summersea.top:8090/upload/2021/12/image-2dba8894740245d39bfd761a1b89f2f5.png)











#### 身份认证器——AuthenticationProvider
`AuthenticationProvider`是真正执行了身份认证的组件，它可以对传入的`Authentication`表现出偏好：当`AuthenticationProvider:supports(Class<?>Authentication)`返回true时，即表示这个`AuthenticationProvider`支持受理此`Authentication`。
举例：
- `DaoAuthenticationProvider`支持受理`UsernamePasswordAuthenticationToken`
- `JwtAuthenticationProvider`支持受理`JwtAuthenticationToken`

`support`方法中，我们常用`Class:isAssignableFrom(Class)`方法判断是否支持。


### 身份认证入口——AuthenticationEntryPoint
在身份认证失败时，这个组件被用来向客户端请求凭证信息，比如将浏览器重定向到登录页。
