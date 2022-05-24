# Spring Security

Spring Security 是一个基于 Spring AOP 和 Servlet 过滤器的安全框架，它提供了安全性方面的解决方案，同时在Web请求和方法级处理身份确认和授权

## Spring Security 模块

**核心模块** - spring-security-core.jar：包含核心验证和访问控制类和接口，远程支持的基本配置API，是基本模块

**远程调用** - spring-security-remoting.jar：提供与 Spring Remoting 集成

**网页** - spring-security-web.jar：包括网站安全的模块，提供网站认证服务和基于URL访问控制

**配置** - spring-security-config.jar：包含安全命令空间解析代码，若使用XML进行配置则需要

**LDAP** - spring-security-ldap.jar：LDAP 验证和配置，若需要LDAP验证和管理LDAP用户实体

**ACL访问控制表** - spring-security-acl.jar：ACL专门领域对象的实现

**CAS** - spring-security-cas.jar：CAS客户端继承，若想用CAS的SSO服务器网页验证

**OpenID** - spring-security-openid.jar：OpenID网页验证支持

**Test** - spring-security-test.jar：支持Spring Security的测试

## 认证流程

用户使用用户名和密码登录

2.用户名密码被过滤器（默认为 UsernamePasswordAuthenticationFilter）获取到，封装成 Authentication

3.token（Authentication实现类）传递给 AuthenticationManager 进行认证

4.AuthenticationManager 认证成功后返回一个封装了用户权限信息的 Authentication 对象

5.通过调用 SecurityContextHolder.getContext().setAuthentication(...)  将 Authentication 对象赋给当前的 SecurityContext

### 默认执行顺序：

#### UsernamePasswordAuthenticationFilter ：

1. 用户通过url：/login 登录，该过滤器接收表单用户名密码
2. 判断用户名密码是否为空
3. 生成 UsernamePasswordAuthenticationToken
4. 将 Authentiction 传给 AuthenticationManager接口的 authenticate 方法进行认证处理
5. AuthenticationManager 默认是实现类为 ProviderManager ，ProviderManager 委托给 AuthenticationProvider 进行处理
6. UsernamePasswordAuthenticationFilter 继承了 AbstractAuthenticationProcessingFilter 抽象类，AbstractAuthenticationProcessingFilter 在 successfulAuthentication 方法中对登录成功进行了处理，通过 SecurityContextHolder.getContext().setAuthentication() 方法将 Authentication 认证信息对象绑定到 SecurityContext
7. 下次请求时，在过滤器链头的 SecurityContextPersistenceFilter 会从 Session 中取出用户信息并生成 Authentication（默认为 UsernamePasswordAuthenticationToken），并通过 SecurityContextHolder.getContext().setAuthentication() 方法将 Authentication 认证信息对象绑定到 SecurityContext
8. 需要权限才能访问的请求会从 SecurityContext 中获取用户的权限进行验证

#### DaoAuthenticationProvider （实现了 AuthenticationProvider）：

1. 通过 UserDetailsService 获取 UserDetails
2. 将 UserDetails 和 UsernamePasswordAuthentionToken 进行认证匹配用户名密码是否正确
3. 若正确则通过 UserDetails 中用户的权限、用户名等信息生成一个新的 Authentication 认证对象并返回



## 相关类

### WebSecurityConfigurerAdapter(方便提供配置)

为创建 WebSecurityConfigurer 实例提供方便的基类，重写它的 configure 方法来设置安全细节

- *configure(HttpSecurity http)*：重写该方法，通过 http 对象的 authorizeRequests()方法定义URL访问权限，默认会为 formLogin() 提供一个简单的测试HTML页面
- _configureGlobal(AuthenticationManagerBuilder auth) _：通过 auth 对象的方法添加身份验证

### SecurityContextHolder

SecurityContextHolder 用于存储安全上下文信息（如操作用户是谁、用户是否被认证、用户权限有哪些），它用 ThreadLocal 来保存 SecurityContext，者意味着 Spring Security 在用户登录时自动绑定到当前现场，用户退出时，自动清除当前线程认证信息，SecurityContext 中含有正在访问系统用户的详细信息

### AuthenticationManagerBuilder

- 用于构建认证 AuthenticationManager 认证，允许快速构建内存认证、LDAP身份认证、JDBC身份验证，添加 userDetailsService（获取认证信息数据） 和  AuthenticationProvider's（定义认证方式）
- 内存验证：



```
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
      auth.inMemoryAuthentication()
            .withUser("user").password("123").roles("USER").and()
            .withUser("admin").password("456").roles("USER","ADMIN");
}
```

- JDBC 验证：



```
@Autowired
private DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
            .dataSource(dataSource)
            .withDefaultSchema()
            .withUser("linyuan").password("123").roles("USER").and()
            .withUser("linyuan2").password("456").roles("ADMIN");
}
```

- *userDetailsService(T userDetailsService)*：传入自定义的 UserDetailsService 获取认证信息数据
- *authenticationProvider(AuthenticationProvider authenticationProvider)* ：传入自定义认证过程

### UserDetailsService

- 该接口仅有一个方法 loadUserByUsername，Spring Security 通过该方法获取

### UserDetails

- 我们可以实现该接口来定义自己认证用户的获取方式（如数据库中获取），认证成功后会将 UserDetails 赋给 Authentication 的 principal 属性，然后再把 Authentication 保存到 SecurityContext 中，之后需要实用用户信息时通过 SecurityContextHolder 获取存放在 SecurityContext 中的 Authentication 的 principal

### Authentication

- Authentication 是一个接口，用来表示用户认证信息，在用户登录认证之前相关信息（用户传过来的用户名密码）会封装为一个 Authentication 具体实现类对象，默认情况下为 UsernamePasswordAuthenticationToken，登录之后（通过AuthenticationManager认证）会生成一个更详细的、包含权限的对象，然后把它保存在权限线程本地的 SecurityContext 中，供后续权限鉴定用

### GrantedAuthority

- GrantedAuthority 是一个接口，它定义了一个 getAuthorities() 方法返回当前 Authentication 对象的拥有权限字符串，用户有权限是一个 GrantedAuthority 数组，每一个 GrantedAuthority 对象代表一种用户
- 通常搭配 SimpleGrantedAuthority 类使用

### AuthenticationManager

AuthenticationManager是用于定义Spring Security的过滤器如何执行身份验证的API。 然后由调用AuthenticationManager的控制器（即Spring Security的Filters）在SecurityContextHolder上设置返回的身份验证。

- AuthenticationManager 是用来处理认证请求的接口，它只有一个方法 authenticate()，该方法接收一个 Authentication 作为对象，如果认证成功则返回一个封装了当前用户权限信息的 Authentication 对象进行返回
- 它默认的实现是 ProviderManager，但它不处理认证请求，而是将委托给 AuthenticationProvider 列表，然后依次使用 AuthenticationProvider 进行认证，如果有一个 AuthenticationProvider 认证的结果不为null，则表示成功（否则失败，抛出 ProviderNotFoundException），之后不在进行其它 AuthenticationProvider 认证，并作为结果保存在 ProviderManager
- 认证校验时最常用的方式就是通过用户名加载 UserDetails，然后比较 UserDetails 密码与请求认证是否一致，一致则通过，Security 内部的 DaoAuthenticationProvider 就是实用这种方式
- 认证成功后加载 UserDetails 来封装要返回的 Authentication 对象，加载的 UserDetails 对象是包含用户权限等信息的。认证成功返回的 Authentication 对象将会保存在当前的 SecurityContext 中

### AuthenticationProvide

- AuthenticationProvider 是一个身份认证接口，实现该接口来定制自己的认证方式，可通过 UserDetailsSevice 对获取数据库中的数据
- 该接口中有两个需要实现的方法：
  - *Authentication authenticate(Authentication authentication)*：认证处理，返回一个 Authentication 的实现类则代表成功，返回 null 则为认证失败
  - *supports(Class<?> aClass)*：如果该 AuthenticationProvider 支持传入的 Authentication 认证对象，则返回 true ，如：return aClass.equals(UsernamePasswordAuthenticationToken.class);

### AuthorityUtils

- 是一个工具包，用于操作 GrantedAuthority 集合的实用方法：
  - *commaSeparatedStringToAuthorityList(String authorityString)*：从逗号分隔符中创建 GrantedAuthority 对象数组

### AbstractAuthenticationProcessingFilter

- 该抽象类继承了 GenericFilterBean，是处理 form 登录的过滤器，与 form 登录相关的所有操作都在该抽象类的子类中进行（UsernamePasswordAuthenticationFilter 为其子类），比如获取表单中的用户名、密码，然后进行认证等操作
- 该类在 doFilter 方法中通过 attemptAuthentication() 方法进行用户信息逻辑认证，认证成功会将用户信息设置到 Session 中

### UserDetails

- 代表了Spring Security的用户实体类，带有用户名、密码、权限特性等性质，可以自己实现该接口，供 Spring Security 安全认证使用，Spring Security 默认使用的是内置的 User 类
- 将从数据库获取的 User 对象传入实现该接口的类，并获取 User 对象的值来让类实现该接口的方法
- 通过 Authentication.getPrincipal() 的返回类型是 Object，但很多情况下其返回的其实是一个 UserDetails 的实例

### HttpSecurity

- 用于配置全局 Http 请求的权限控制规则，对哪些请求进行验证、不验证等

- 常用方法：

  - authorizeRequests()：返回一个配置对象用于配置请求的访问限制
  - formLogin()：返回表单配置对象，当什么都不指定时会提供一个默认的，如配置登录请求，还有登录成功页面
  - logout()：返回登出配置对象，可通过logoutUrl设置退出url
  - antMatchers：匹配请求路径或请求动作类型，如：.antMatchers("/admin/**")
  - addFilterBefore: 在某过滤器之前添加 filter
  - addFilterAfter：在某过滤器之后添加 filter
  - addFilterAt：在某过滤器相同位置添加 filter，不会覆盖相同位置的 filter
  - hasRole：结合 antMatchers 一起使用，设置请求允许访问的角色权限或IP，如：

  

  ```
  .antMatchers("/admin/**").hasAnyRole("ROLE_ADMIN","ROLE_USER") 
  ```

  |          方法名          |                    用途                     |
  | :----------------------: | :-----------------------------------------: |
  |      access(String)      |      SpringEL表达式结果为true时可访问       |
  |       anonymous()        |                 匿名可访问                  |
  |        denyAll()         |               用户不可以访问                |
  |   fullyAuthenticated()   | 用户完全认证访问（非remember me下自动登录） |
  | hasAnyAuthority(String…) |            参数中任意权限可访问             |
  |   hasAnyRole(String…)    |            参数中任意角色可访问             |
  |   hasAuthority(String)   |            某一权限的用户可访问             |
  |     hasRole(String)      |            某一角色的用户可访问             |
  |       permitAll()        |               所有用户可访问                |
  |       rememberMe()       |      允许通过remember me登录的用户访问      |
  |     authenticated()      |              用户登录后可访问               |
  |   hasIpAddress(String)   |          用户来自参数中的IP可访问           |

## 注解与Spring EL

- **@EnableWebSecurity**：开启 Spring Security 注解
- **@EnableGlobalMethodSecurity(prePostEnabled=true)**：开启security方法注解
- **@PreAuthorize**：在方法调用前，通过SpringEL表达式限制方法访问



```
@PreAuthorize("hasRole('ROLE_ADMIN')")
public void addUser(User user){
    //如果具有权限 ROLE_ADMIN 访问该方法
    ....
}
```

- @PostAuthorize：允许方法调用，但时如果表达式结果为false抛出异常



```
//returnObject可以获取返回对象user，判断user属性username是否和访问该方法的用户对象的用户名一样。不一样则抛出异常。
@PostAuthorize("returnObject.user.username==principal.username")
public User getUser(int userId){
   //允许进入
...
    return user;    
}
```

- **@PostFilter**：允许方法调用，但必须按表达式过滤方法结果



```
//将结果过滤，即选出性别为男的用户
@PostFilter("returnObject.user.sex=='男' ")
public List<User> getUserList(){
    //允许进入
    ...
    return user;    
}
```

- **@PreFilter**：允许方法调用，但必须在进入方法前过滤输入值

## Spring EL 表达式：

|             表达式             |                             描述                             |
| :----------------------------: | :----------------------------------------------------------: |
|        hasRole ([role])        |                   当前用户是否拥有指定角色                   |
|   hasAnyRole([role1,role2])    | 多个角色是一个以逗号进行分隔的字符串。如果当前用户拥有指定角色中的任意一个则返回true |
|     hasAuthority ([auth])      |                        等同于hasRole                         |
| hasAnyAuthority([auth1,auth2]) |                      等同于 hasAnyRole                       |
|           Principle            |                 代表当前用户的 principle对象                 |
|         authentication         |     直接从 Security context获取的当前 Authentication对象     |
|           permitAll            |               总是返回true,表示允许所有的访问                |
|            denyAll             |             总是返回false,表示拒绝所有的访问访问             |
|         isAnonymous()          |                  当前用户是否是一个匿名用户                  |
|          isRememberMe          |        表示当前用户是否是通过remember - me自动登录的         |
|       isAuthenticated()        |              表示当前用户是否已经登录认证成功了              |
|     isFullAuthenticated()      | 如果当前用户既不是一个匿名用户,同时又不是通过Remember-Me自动登录的,则返回true |

## 密码加密（PassWordEncoder）

- Spring 提供的一个用于对密码加密的接口，首选实现类为  BCryptPasswordEncoder
- 通过@Bean注解将它注入到IOC容器：



```
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

## 过滤器链

### SecurityContextPersistenceFilter

- 过滤器链头，是从 SecurityContextRepository 中取出用户认证信息，默认实现为 HttpSessionSecurityContextRepository，它会从 Session 中取出已认证的用户信息，提高效率，避免每次请求都要查询用户认证信息
- 取出之后会放入 SecurityContextHolder 中，以便其它 filter 使用，SecurityContextHolder 使用 ThreadLocal 存储用户认证信息，保证线程之间信息隔离，最后再 finally 中清除该信息

### WebAsyncManagerIntegrationFilter

- 提供了对 SecurityContext 和 WebAsyncManager 的集成，会把 SecurityContext 设置到异步线程，使其也能获取到用户上下文认证信息

### HanderWriterFilter

- 会往请求的 Header 中添加相应的信息

### CsrfFilter

- 跨域请求伪造过滤器，通过客户端穿来的 token 与服务端存储的 token 进行对比来判断请求

### LogoutFilter

- 匹配URL，默认为 /logout，匹配成功后则会用户退出，清除认证信息，若有自己的退出逻辑，该过滤器可以关闭

### UsernamePasswordAuthenticationFilter

- 登录认证过滤器，默认是对 /login 的 POST 请求进行认证，首先该方法会调用 attemptAuthentication 尝试认证获取一个 Authentication 认证对象，然后通过 sessionStrategy.onAuthentication 执行持久化，也就是保存认证信息，然后转向下一个 Filter，最后调用 successfulAuthentication 执行认证后事件
- attemptAuthentication 该方法是认证的主要方法，认证基本流程为 UserDeatilService 根据用户名获取到用户信息，然后通过 UserDetailsChecker.check 对用户状态进行校验，最后通过 additionalAuthenticationChecks 方法对用户密码进行校验完后认证后，返回一个认证对象

### DefaultLoginPageGeneratingFilter

- 当请求为登录请求时，生成简单的登录页面，可以关闭

### BasicAuthenticationFilter

- Http Basci 认证的支持，该认证会把用户名密码使用 base64 编码后放入 header 中传输，认证成功后会把用户信息放入 SecurityContextHolder 中

### RequestCacheAwareFilter

- 恢复被打断时的请求

### SecurityContextHolderAwareRequestFilter

- 针对 Servlet api 不同版本做一些包装

### AnonymousAuthenticationFIlter

- SecurityContextHolder 中认证信息为空，则会创建一个匿名用户到 SecurityContextHolder 中

### SessionManagementFilter

- 与登录认证拦截时作用一样，持久化用户登录信息，可以保存到 Session 中，也可以保存到 cookie 或 redis 中

### ExceptionTranslationFilter

- 异常拦截，处于 Filter 链条后部，只能拦截其后面的节点并着重处理 AuthenticationException 与 AccessDeniedException 异常









## 认证Authentication

### 用户名密码

#### 读取用户名密码：

Spring Security提供了以下内置机制，用于从HttpServletRequest中读取用户名和密码：

- 表格登入（通过html表单提供的用户名和密码的支持）
- 基本认证
- 摘要式身份验证

#### 储存用户名密码：

用于读取用户名和密码的每种受支持的机制都可以利用任何受支持的存储机制：

- 通过内存中身份验证进行简单存储(如果再内存中配置了，则可以不走UserDetailsService)
- 具有JDBC身份验证的关系数据库
- 使用UserDetailsService的自定义数据存储**(难怪可以通过UserDetailsService，来实现mybatis或JPA读取)**
- 具有LDAP认证的LDAP存储



#### 表单模式登录验证流程：

提交用户名和密码后，UsernamePasswordAuthenticationFilter会对用户名和密码进行身份验证



1. 客户端请求受保护资源，被FilterSecurityInterceptor拦截，抛出异常AccessDeniedExecption，然后被捕获，通过LoginUrlAuthenticationEntryPoint返回到登录界面。
2. UsernamePasswordAuthenticationFilter通过从HttpServletRequest中提取用户名和密码来创建UsernamePasswordAuthenticationToken，这是一种身份验证类型。
3. **接下来将UsernamePasswordAuthenticationToken传递到AuthenticationManager进行身份验证。AuthenticationManager外观的详细信息取决于用户信息的存储方式。AuthenticationManager由 ProviderManager 实现，ProviderManager 被配置为使用 DaoAuthenticationProvider 类型的 AuthenticationProvider。**
4. DaoAuthenticationProvider 从 UserDetailsService 查找 UserDetails，然后，DaoAuthenticationProvider 使用 PasswordEncoder 验证前一步中返回的 UserDetails 上的密码。
5. 如果身份验证失败：则失败，清除SecurityContextHolder。
6. 如果成功：会向SessionAuthenticationStrategy通知新的登录  --》设置SecurityContextHolder  --》RememberMeServices调用记住我的功能  --》 ApplicationEventPublisher发布一个InteractiveAuthenticationSuccessEvent(可能是通知一个事件)。 --》 调用AuthenticationSuccessHandler通常，这是一个SimpleUrlAuthenticationSuccessHandler，当我们重定向到登录页面时 
7. 身份验证成功后，返回的身份验证类型为 UsernamePasswordAuthenticationToken，其主体为已配置的 UserDetailsService 返回的 UserDetails。最终，身份验证筛选器将在 SecurityContextHolder 上设置返回的 UsernamePasswordAuthenticationToken。

![image-20210226173815734](../../../Library/Application Support/typora-user-images/image-20210226173815734.png)

![image-20210226174216585](../../../Library/Application Support/typora-user-images/image-20210226174216585.png)

#### 默认登录页面模板

该表格应张贴到/ login

该表格将需要包含一个由Thymeleaf自动包含的CSRF令牌。

该表单应在名为username的参数中指定用户名

该表格应在名为password的参数中指定密码

如果找到HTTP参数错误，则表明用户未能提供有效的用户名/密码

如果找到HTTP参数注销，则表明用户已成功注销

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"> //自动包含CSRF令牌
    <head>
        <title>Please Log In</title>
    </head>
    <body>
        <h1>Please Log In</h1>
        <div th:if="${param.error}">
            Invalid username and password.</div>
        <div th:if="${param.logout}">
            You have been logged out.</div>
        <form th:action="@{/login}" method="post">
            <div>
            <input type="text" name="username" placeholder="Username"/>
            </div>
            <div>
            <input type="password" name="password" placeholder="Password"/>
            </div>
            <input type="submit" value="Log in" />
        </form>
    </body>
</html>
```



#### UserDetailsService（用户的获取）

到了这一步，其实就很明显了，既然你实现 他的接口，如果我想实现从数据进行匹配的话，我实现你的接口不就完事了吗，那么我们如果要实现自定义的认证流程也只需要实现UserDetailsService接口重写loadUserByUsername就可以了。如果设置了InMemoryUserDetailsManager，可以发现他实现了UserDetailsService，他的loadUserByUsername方法其实就是查询开始存放在内存的用户。(说白了loadUserByUsername方法就是根据用户名查出一个用户实体，具体实现是查数据库还是内存，根据自己的实现，仅仅只是拿到用户，这也说明用户名唯一)

那怎么注入我们的UserDetailsService呢？

```
  @Autowired
   private UserService userService;
   
  @Resource
  private BCryptPasswordEncoder bCryptPasswordEncoder;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userService).passwordEncoder(bCryptPasswordEncoder);
        }
        
  public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }

```

#### UserDetails:

UserDetails由UserDetailsService返回。 DaoAuthenticationProvider验证UserDetails，然后返回身份验证，该身份验证的主体是已配置的UserDetailsService返回的UserDetails。DaoAuthenticationProvider使用UserDetailsService检索用户名，密码和其他用于使用用户名和密码进行身份验证的属性。你可以通过将自定义UserDetailsService公开为bean来定义自定义身份验证

#### DaoAuthenticationProvider：

DaoAuthenticationProvider是AuthenticationProvider实现，它利用UserDetailsService和PasswordEncoder来验证用户名和密码。

DaoAuthenticationProvider处理了Web表单的认证逻辑，认证成功后得到一个Authencation(UsernamePasswordAuthenticationToken实现)，

 DaoAuthenticationProvider中包含了一个UserDetailsService实例，它负责根据用户名提取用户信息UserDetailService实例，它负责根据用户名提取用户信息UserDetails(含密码)。 DaoAuthenticationProvider会去对比UserDetailsService提取的用户密码与用户提交的密码是否匹配作为认证成功的关键依据，因此可以通过自定义UserDetailsService的实现类来实现自定义的身份认证。

### 名称

#### SecurityContextHolder

SecurityContextHolder 是 Spring Security 存储被验证者的详细信息的地方，包含SecurityContextHolder

<img src="../../../Library/Application Support/typora-user-images/image-20210228205441689.png" alt="image-20210228205441689" style="zoom:33%;" />

设置SecurityContextHolder

```
SecurityContext context = SecurityContextHolder.createEmptyContext(); 
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER"); 
context.setAuthentication(authentication);

SecurityContextHolder.setContext(context); 
```

使用SecurityContextHolder，默认情况下 SecurityContextHolder 使用 ThreadLocal 来存储这些细节，这意味着 SecurityContext 总是可用于同一线程中的方法。

```
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```



### SecurityContext

SecurityContext 从 SecurityContextHolder 获得，SecurityContext 包含一个身份验证对象。



### Authentication

- Principal-标识用户。当使用用户名/密码进行身份验证时，这通常是 UserDetails 的一个实例。
- credentials-通常是密码。在许多情况下，在用户经过身份验证以确保其不泄漏之后，这些信息将被清除。
- Authorities —— GrantedAuthoritys 是授予用户的高级权限。

getAuthorities()	权限信息列表，默认是GrantedAuthority接口的一些，通常是代表权限信息的一系列字符串
getCredentials()	凭证信息，用户输入的密码字符串，在认证过后通常会被移除，以保障安全
getDetails()	细节信息，web应用中的实现接口通常为WebAuthenticationDetails，它记录了访问者的ip地址
getPrincipal()	身份信息，大部分情况下返回的是UserDetails接口的实现类，UserDetails代表用户的详细信息，从这个接口可以获取登录用户信息

### GrantedAuthority

GrantedAuthoritys 是授予用户的高级权限。可以从 Authentication.getAuthorities ()方法获得 GrantedAuthoritys。此方法提供了 GrantedAuthority 对象的集合。当使用基于用户名/密码的身份验证时，GrantedAuthoritys 通常由 UserDetailsService 加载。



### AuthenticationManager

AuthenticationManager 是定义 Spring Security 的筛选器如何执行身份验证的 API。然后由调用 AuthenticationManager 的控制器(即 Spring Security 的 Filterss)在 SecurityContextHolder 上设置返回的身份验证。虽然 AuthenticationManager 的实现可以是任何东西，但最常见的实现是 ProviderManager。



### ProviderManager

ProviderManager 是 AuthenticationManager 最常用的实现。ProviderManager 代理  authenticationprovider 列表。每个身份验证提供者都有机会指示身份验证应该成功、失败或指示它不能做出决定，并允许下游身份验证提供者做出决定。如果配置的 authenticationprovider 没有一个能够进行身份验证，那么身份验证将失败。

实际上，每个身份验证提供者都知道如何执行特定类型的身份验证。例如，一个 AuthenticationProvider 可能能够验证用户名/密码，而另一个可能能够验证 SAML 断言。这允许每个 AuthenticationProvider 执行特定类型的身份验证，同时支持多种类型的身份验证，并且只公开一个 AuthenticationManager bean。

(ProviderManager相当于代理，AuthenticationProvider是个列表，提供多种身份验证)

<img src="../../../Library/Application Support/typora-user-images/image-20210228211009891.png" alt="image-20210228211009891" style="zoom:33%;" />



### AuthenticationProvider

将多个身份验证提供者注入到 ProviderManager 中。每个 AuthenticationProvider 执行特定类型的身份验证。例如，DaoAuthenticationProvider 支持基于用户名/密码的身份验证，而 jwtauthinctionprovider 支持对 JWT 令牌进行身份验证。

public interface AuthenticationProvider {
    Authentication authenticate(Authentication var1) throws AuthenticationException;
    boolean supports(Class<?> var1);
}
`authenticate()`方法定义了认证的实现过程，参数是一个`Authentication`，里面包含了用户提交的用户、密码等。而返回值也是一个`Authentication`，这个`Authentication`则是在认证成功后，将用户的权限及其他信息重新组装后生成。

 不同的认证方式使用不同的实现类的AuthenticationProvider，而每个AuthenticationProvider需要实现support()方法来表明自己支持的认证方式。如我们使用表单方式认证，在提交请求时SpringSecurity会生成UsernamePasswordAutheticationToken,它是一个Authentication。


### 主要Filter

SecurityContextPersistenceFilter	这个Filter是整个拦截过程的入口和出口，会在请求开始时从配置好的 SecurityContextRepository 中获取 SecurityContext，然后把它设置给 SecurityContextHolder。在请求完成后将 SecurityContextHolder 持有的 SecurityContext 再保存到配置好 的 SecurityContextRepository，同时清除 securityContextHolder 所持有的 SecurityContext；

UsernamePasswordAuthenticationFilter	用于处理来自表单提交的认证。该表单必须提供对应的用户名和密 码，其内部还有登录成功或失败后进行处理的 AuthenticationSuccessHandler 和 AuthenticationFailureHandler，这些都可以根据需求做相关改

FilterSecurityInterceptor	是用于保护web资源的，使用AccessDecisionManager对当前用户进行授权访问，前 面已经详细介绍过了

ExceptionTranslationFilter	能够捕获来自 FilterChain 所有的异常，并进行处理。但是它只会处理两类异常AuthenticationException 和 AccessDeniedException，其它的异常它会继续抛出。



### 认证流程图

![image-20210301104516970](../../../Library/Application Support/typora-user-images/image-20210301104516970.png)



# 授权Authorization

#### Authorities

GrantedAuthority 是一个只有一个方法的接口: