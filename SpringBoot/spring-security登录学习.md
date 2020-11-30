SpringSecurity采用的是责任链的设计模式，它有一条很长的过滤器链。

| 过滤器                                  | 作用                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| WebAsyncManagerIntegrationFilter        | 将Security上下文与SpringWeb中用于处理异步请求映射的WebAsyncManager进行集成。 |
| SecurityContextPersistenceFilter        | 在每次请求处理之前将该请求相关的安全上下文信息加载到 SecurityContextHolder 中，然后在该次请求处理完成之后，将 SecurityContextHolder 中关于这次请求的信息存储到一个“仓储”中，然后将 SecurityContextHolder 中的信息清除，例如在Session中维护一个用户的安全信息就是这个过滤器处理的。 |
| HeaderWriterFilter                      | 用于将头信息加入响应中。                                     |
| CsrfFilter                              | 用于处理跨站请求伪造。                                       |
| LogoutFilter                            | 用于处理退出登录。                                           |
| UsernamePasswordAuthenticationFilter    | 用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自`/login`的请求。从表单中获取用户名和密码时，默认使用的表单name值为`username` 和`password`，这两个值可以通过设置这个过滤器的`usernameParameter` 和`passwordParameter`两个参数的值进行修改。 |
| DefaultLoginPageGeneratingFilter        | 如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。 |
| BasicAuthenticationFilter               | 检测和处理http basic请求。                                   |
| RequestCacheAwareFilter                 | 用来处理请求的缓存。                                         |
| SecurityContextHolderAwareRequestFilter | 主要是包装请求对象request。                                  |
| AnonymousAuthenticationFilter           | 检测SecurityContextHolder中是否存在Authentication对象，如果不存在为其提供一个匿名Authentication。 |
| SessionManagerFilter                    | 管理session的过滤器。                                        |
| ExceptionTranslationFilter              | 处理AccessDeniedException和AuthenticationException异常。     |
| RememberMeAuthenticationFilter          | 当用户没有登录而直接访问资源时，从cookie里找出用户的信息，如果SpringSecurity能够识别出用户提供的remember me cookie，用户将不必填写用户名和密码，而是直接登录进入系统，该过滤器默认不开启。 |
| FilterSecurityInterceptor               | 可以看作过滤器链的出口。                                     |

流程图如下

![spring-security登录流程示意图](C:\Users\cqk\Desktop\notes\pictures\spring-security登录流程示意图.jpg)

AuthenticationEntryPoint：解决匿名用户访问无权限资源时的异常

AccessDeniedHandler：解决认证过的用户访问无权限资源时的异常