# spring-security中核心概念（非oauth2)
* AuthenticationEntryPoint 用来解决匿名用户访问无权限资源时的异常
* AccessDeineHandler 用来解决认证过的用户访问无权限资源时的异常

# JsonDeserialize中的using和as
* 优先级：using > as
* @JsonDeserialize(as=ValueImpl.class)将类型反序列化成指定类型；
* @JsonDeserialize(using=CustomDateDeserialize.class)用于在反序列化时嵌入我们自定义的代码。
## jwt
### 签名验证
* JWT签名算法中，一般有两个选择，一个采用HS256,另外一个就是采用RS256。
* 签名实际上是一个加密的过程，生成一段标识（也是JWT的一部分）作为接收方验证信息是否被篡改的依据。
* HS256 (带有 SHA-256 的 HMAC 是一种对称算法, 双方之间仅共享一个 密钥。由于使用相同的密钥生成签名和验证签名, 因此必须注意确保密钥不被泄密。
* RS256 (采用SHA-256 的 RSA 签名) 是一种非对称算法, 它使用公共/私钥对: 标识提供方采用私钥生成签名, JWT 的使用方获取公钥以验证签名。由于公钥 (与私钥相比) 不需要保护, 因此大多数标识提供方使其易于使用方获取和使用 (通常通过一个元数据URL)。

### payload
jwt payload中包括的内容太多，对网络传输是需要考虑的，比如我们是否在JWT里直接包括权限信息，还是只包括角色信息，然后再通过角色信息去查询它的权限。
* iss(Issuser)：如果签发的时候这个claim的值是“a.com”，验证的时候如果这个claim的值不是“a.com”就属于验证失败
* sub(Subject)：如果签发的时候这个claim的值是“liuyunzhuge”，验证的时候如果这个claim的值不是“liuyunzhuge”就属于验证失败
* aud(Audience)：如果签发的时候这个claim的值是“['b.com','c.com']”，验证的时候这个claim的值至少要包含b.com，c.com的其中一个才能验证通过
* exp(Expiration time)：如果验证的时候超过了这个claim指定的时间，就属于验证失败；
* nbf(Not Before)：如果验证的时候小于这个claim指定的时间，就属于验证失败
* iat(Issued at)：它可以用来做一些maxAge之类的验证，假如验证时间与这个claim指定的时间相差的时间大于通过maxAge指定的一个值，就属于验证失败
* jti(JWT ID)：如果签发的时候这个claim的值是“1”，验证的时候如果这个claim的值不是“1”就属于验证失败
## 使用
### 登录
* url: http://localhost:8080/login
* method: POST
* Content-Type: application/json
* 请求体
```
{
    "username":"admin",
    "password":"abcd1234"
}
```
* 响应码：200
```
{
    "expiresAt": 1648100324000,
    "subject": "admin",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImV4cCI6MTY0ODEwMDMyNCwidXNlciI6IntcImlkXCI6XCIxXCIsXCJ1c2VybmFtZVwiOlwiYWRtaW5cIixcImVtYWlsXCI6XCJhZG1pbkBzaW5hLmNvbVwiLFwiYXV0aG9yaXRpZXNcIjpbe1wiaWRcIjpcIjFcIixcIm5hbWVcIjpcIueuoeeQhuWRmFwiLFwiYnV0dG9uR3JhbnRcIjowfV19IiwiaWF0IjoxNjQ4MDk4NTI0fQ.fgfPXLNRYE0kfrq__LmhM_UBy82ChHixjZ-Ve31QVVU"
}
```
### 登出
* url: http://localhost:8080/logout
* method: POST
* header: bearer ****token****
* 响应： 200
## 依赖引用
```
<dependency>
    <groupId>com.pkulaw</groupId>
    <artifactId>pkulaw-uaa-jwt-start</artifactId>
    <version>1.0.1</version>
</dependency>
```
## 登录
```
POST JSON /login
{
    "username":"Jack",
    "password":"123456"
}
```
![](./assets/readme-1642742553808.png)
## 登出
```
POST /logout
Header Bearer token
```
## 使用
1. 认证和授权的服务引用lind-uaa-jwt-start之后，需要实现ResourcePermissionService和UserDetailsService接口
2. 其它业务服务直接引用lind-uaa-jwt-start之后，通过SecurityUtil来获取当前登陆的用户信息
##  AuthenticationManager
用户认证的管理类，所有的认证请求（比如login）都会通过提交一个token给AuthenticationManager的authenticate()方法来实现。
当然事情肯定不是它来做，具体校验动作会由AuthenticationManager将请求转发给具体的实现类来做。根据实现反馈的结果再调用具体的Handler来给用户以反馈。
这个类基本等同于shiro的SecurityManager。
## AuthenticationProvider
认证的具体实现类，一个provider是一种认证方式的实现，比如提交的用户名密码我是通过和DB中查出的user记录做比对实现的，那就有一个DaoProvider；
如果我是通过CAS请求单点登录系统实现，那就有一个CASProvider。这个是不是和shiro的Realm的定义很像？基本上你可以帮他们当成同一个东西。
按照Spring一贯的作风，主流的认证方式它都已经提供了默认实现，比如DAO、LDAP、CAS、OAuth2等。前面讲了AuthenticationManager只是一个代理接口，
真正的认证就是由AuthenticationProvider来做的。一个AuthenticationManager可以包含多个Provider，每个provider通过实现一个support方法来
表示自己支持那种Token的认证。AuthenticationManager默认的实现类是ProviderManager。
## UserDetailService
用户认证通过Provider来做，所以Provider需要拿到系统已经保存的认证信息，获取用户信息的接口spring-security抽象成UserDetailService。
虽然叫Service,但是我更愿意把它认为是我们系统里经常有的UserDao。
## AuthenticationToken
所有提交给AuthenticationManager的认证请求都会被封装成一个Token的实现，比如最容易理解的UsernamePasswordAuthenticationToken。这个就不多讲了，
连名字都跟Shiro中一样。
## SecurityContext
当用户通过认证之后，就会为这个用户生成一个唯一的SecurityContext，里面包含用户的认证信息Authentication。通过SecurityContext我们可以获取到
用户的标识Principle和授权信息GrantedAuthrity。在系统的任何地方只要通过SecurityHolder.getSecruityContext()就可以获取到SecurityContext。
在Shiro中通过SecurityUtils.getSubject()到达同样的目的。
## 认证的流程
提交-》AuthenticationToken-》AuthenticationManager-》AuthenticationProvider-》成功-》SecurityContext.setAuthentication
## 登录认证流程
### Filter
对于用户登录行为，security通过定义一个Filter来拦截/login来实现的。spring security默认支持form方式登录，所以对于使用json发送登录信息的情况，我们自己定义一个Filter，
这个Filter直接从AbstractAuthenticationProcessingFilter继承，只需要实现两部分，一个是RequestMatcher，指名拦截的Request类型；另外就是从json body中提取出
username和password提交给AuthenticationManager。
### Provider
前面的流程图中讲到了，封装后的token最终是交给provider来处理的。对于登录的provider，spring security已经提供了一个默认实现DaoAuthenticationProvider
我们可以直接使用，这个类继承了AbstractUserDetailsAuthenticationProvider我们来看下关键部分的源代码是怎么做的。
```
public abstract class AbstractUserDetailsAuthenticationProvider implements
        AuthenticationProvider, InitializingBean, MessageSourceAware {
...
    //这个方法返回true，说明支持该类型的token
    public boolean supports(Class<?> authentication) {
        return (UsernamePasswordAuthenticationToken.class
                .isAssignableFrom(authentication));
   }
    public Authentication authenticate(Authentication authentication)
            throws AuthenticationException {
             ...

            try {
            // 获取系统中存储的用户信息
                user = retrieveUser(username,
                        (UsernamePasswordAuthenticationToken) authentication);
            }
            catch (UsernameNotFoundException notFound) {
                logger.debug("User '" + username + "' not found");

                if (hideUserNotFoundExceptions) {
                    throw new BadCredentialsException(messages.getMessage(
                            "AbstractUserDetailsAuthenticationProvider.badCredentials",
                            "Bad credentials"));
                }
                else {
                    throw notFound;
                }
            }

            
        }

        try {
           //检查user是否已过期或者已锁定
            preAuthenticationChecks.check(user);
           //将获取到的用户信息和登录信息做比对
            additionalAuthenticationChecks(user,
                    (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (AuthenticationException exception) {
            ...
            throw exception;            
        }
        ...
        //如果认证通过，则封装一个AuthenticationInfo, 放到SecurityContext中
        return createSuccessAuthentication(principalToReturn, authentication, user);
    }

...
}
```
上面的代码中，核心流程就是retrieveUser()获取系统中存储的用户信息，再对用户信息做了过期和锁定等校验后交给additionalAuthenticationChecks()和用户提交的信息做比对。
这两个方法我们看他的继承类DaoAuthenticationProvider是怎么实现的。

```$xslt
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {
    /**
     * 加密密码比对
     */
     protected void additionalAuthenticationChecks(UserDetails userDetails,
            UsernamePasswordAuthenticationToken authentication)
            throws AuthenticationException {
        if (authentication.getCredentials() == null) {
            logger.debug("Authentication failed: no credentials provided");

            throw new BadCredentialsException(messages.getMessage(
                    "AbstractUserDetailsAuthenticationProvider.badCredentials",
                    "Bad credentials"));
        }

        String presentedPassword = authentication.getCredentials().toString();

        if (!passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
            logger.debug("Authentication failed: password does not match stored value");

            throw new BadCredentialsException(messages.getMessage(
                    "AbstractUserDetailsAuthenticationProvider.badCredentials",
                    "Bad credentials"));
        }
    }
   /**
    * 系统用户获取
    */
    protected final UserDetails retrieveUser(String username,
            UsernamePasswordAuthenticationToken authentication)
            throws AuthenticationException {
        prepareTimingAttackProtection();
        try {
            UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
            if (loadedUser == null) {
                throw new InternalAuthenticationServiceException(
                        "UserDetailsService returned null, which is an interface contract violation");
            }
            return loadedUser;
        }
        catch (UsernameNotFoundException ex) {
            mitigateAgainstTimingAttack(authentication);
            throw ex;
        }
        catch (InternalAuthenticationServiceException ex) {
            throw ex;
        }
        catch (Exception ex) {
            throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
        }
    }
}
```
上面的方法实现中，用户获取是调用了UserDetailsService来完成的。这个是一个只有一个方法的接口，所以我们自己要做的，就是将自己的UserDetailsService实现类配置成一个Bean。下面是实例代码，真正的实现需要从数据库或者缓存中获取。
```$xslt
public class JwtUserService implements UserDetailsService{
    //真实系统需要从数据库或缓存中获取，这里对密码做了加密
     return User.builder().username("Jack").password(passwordEncoder.encode("jack-password")).roles("USER").build();
}

```
我们再来看另外一个密码比对的方法，也是委托给一个PasswordEncoder类来实现的。一般来说，存在数据库中的密码都是要经过加密处理的，这样万一数据库数据被拖走，也不会泄露密码。spring一如既往的提供了主流的加密方式，如MD5,SHA等。如果不显示指定的话，Spring会默认使用BCryptPasswordEncoder，这个是目前相对比较安全的加密方式。具体介绍可参考spring-security 的官方文档 - Password Endcoding
认证结果处理
filter将token交给provider做校验，校验的结果无非两种，成功或者失败。对于这两种结果，我们只需要实现两个Handler接口，set到Filter里面，Filter在收到Provider的处理结果后会回调这两个Handler的方法。
先来看成功的情况，针对jwt认证的业务场景，登录成功需要返回给客户端一个token。所以成功的handler的实现类中需要包含这个逻辑。

```$xslt
public class JsonLoginSuccessHandler implements AuthenticationSuccessHandler{
    
    private JwtUserService jwtUserService;
    
    public JsonLoginSuccessHandler(JwtUserService jwtUserService) {
        this.jwtUserService = jwtUserService;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {
                //生成token，并把token加密相关信息缓存，具体请看实现类
        String token = jwtUserService.saveUserLoginInfo((UserDetails)authentication.getPrincipal());
        response.setHeader("Authorization", token);
    }
    
}
```
再来看失败的情况，登录失败比较简单，只需要回复一个401的Response即可。
```$xslt
public class HttpStatusLoginFailureHandler implements AuthenticationFailureHandler{
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException exception) throws IOException, ServletException {
        response.setStatus(HttpStatus.UNAUTHORIZED.value());    
    }   
}
```
### JsonLoginConfigurer
以上整个登录的流程的组件就完整了，我们只需要把它们组合到一起就可以了。这里继承一个AbstractHttpConfigurer，对Filter做配置。
```$xslt
public class JsonLoginConfigurer<T extends JsonLoginConfigurer<T, B>, B extends HttpSecurityBuilder<B>> extends AbstractHttpConfigurer<T, B>  {

    private MyUsernamePasswordAuthenticationFilter authFilter;

    public JsonLoginConfigurer() {
        this.authFilter = new MyUsernamePasswordAuthenticationFilter();
    }
    
    @Override
    public void configure(B http) throws Exception {
        //设置Filter使用的AuthenticationManager,这里取公共的即可
        authFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));
        //设置失败的Handler
        authFilter.setAuthenticationFailureHandler(new HttpStatusLoginFailureHandler());
        //不将认证后的context放入session
        authFilter.setSessionAuthenticationStrategy(new NullAuthenticatedSessionStrategy());

        MyUsernamePasswordAuthenticationFilter filter = postProcess(authFilter);
        //指定Filter的位置
        http.addFilterAfter(filter, LogoutFilter.class);
    }
    //设置成功的Handler，这个handler定义成Bean，所以从外面set进来
    public JsonLoginConfigurer<T,B> loginSuccessHandler(AuthenticationSuccessHandler authSuccessHandler){
        authFilter.setAuthenticationSuccessHandler(authSuccessHandler);
        return this;
    }

}
```
这样Filter就完整的配置好了，当调用configure方法时，这个filter就会加入security FilterChain的指定位置。这个是在全局定义的地方，我们放在最后说。在全局配置的地方，也会将DaoAuthenticationProvider放到ProviderManager中，这样filter中提交的token就可以被处理了。

## 带Token请求校验流程
用户除登录之外的请求，都要求必须携带JWT Token。所以我们需要另外一个Filter对这些请求做一个拦截。这个拦截器主要是提取header中的token，跟登录一样，
提交给AuthenticationManager做检查。
### Filter
```$xslt
public class JwtAuthenticationFilter extends OncePerRequestFilter{
    ...
    public JwtAuthenticationFilter() {
        //拦截header中带Authorization的请求
        this.requiresAuthenticationRequestMatcher = new RequestHeaderRequestMatcher("Authorization");
    }
    
    protected String getJwtToken(HttpServletRequest request) {
        String authInfo = request.getHeader("Authorization");
        return StringUtils.removeStart(authInfo, "Bearer ");
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
       //header没带token的，直接放过，因为部分url匿名用户也可以访问
       //如果需要不支持匿名用户的请求没带token，这里放过也没问题，因为SecurityContext中没有认证信息，后面会被权限控制模块拦截
        if (!requiresAuthentication(request, response)) {
            filterChain.doFilter(request, response);
            return;
        }
        Authentication authResult = null;
        AuthenticationException failed = null;
        try {
            //从头中获取token并封装后提交给AuthenticationManager
            String token = getJwtToken(request);
            if(StringUtils.isNotBlank(token)) {
                JwtAuthenticationToken authToken = new JwtAuthenticationToken(JWT.decode(token));               
                authResult = this.getAuthenticationManager().authenticate(authToken);
            } else {  //如果token长度为0
                failed = new InsufficientAuthenticationException("JWT is Empty");
            }
        } catch(JWTDecodeException e) {
            logger.error("JWT format error", e);
            failed = new InsufficientAuthenticationException("JWT format error", failed);
        }catch (InternalAuthenticationServiceException e) {
            logger.error(
                    "An internal error occurred while trying to authenticate the user.",
                    failed);
            failed = e;
        }catch (AuthenticationException e) {
            // Authentication failed            
            failed = e;
        }
        if(authResult != null) {   //token认证成功
            successfulAuthentication(request, response, filterChain, authResult);
        } else if(!permissiveRequest(request)){   
            //token认证失败，并且这个request不在例外列表里，才会返回错误
            unsuccessfulAuthentication(request, response, failed);
            return;
        }
        filterChain.doFilter(request, response);
    }
    
    ...

    protected boolean requiresAuthentication(HttpServletRequest request,
            HttpServletResponse response) {
        return requiresAuthenticationRequestMatcher.matches(request);
    }
    
    protected boolean permissiveRequest(HttpServletRequest request) {
        if(permissiveRequestMatchers == null)
            return false;
        for(RequestMatcher permissiveMatcher : permissiveRequestMatchers) {
            if(permissiveMatcher.matches(request))
                return true;
        }       
        return false;
    }
}
```
这个Filter的实现跟登录的Filter有几点区别：

经过这个Filter的请求，会继续过FilterChain中的其它Filter。因为跟登录请求不一样，token只是为了识别用户。
如果header中没有认证信息或者认证失败，还会判断请求的url是否强制认证的（通过permissiveRequest方法判断）。如果请求不是强制认证，也会放过，这种情况比如博客类应用匿名用户访问查看页面；比如登出操作，如果未登录用户点击登出，我们一般是不会报错的。
其它逻辑跟登录一样，组装一个token提交给AuthenticationManager。


### JwtAuthenticationProvider
同样我们需要一个provider来接收jwt的token，在收到token请求后，会从数据库或者缓存中取出salt，对token做验证，代码如下：
```$xslt
public class JwtAuthenticationProvider implements AuthenticationProvider{
    
    private JwtUserService userService;
    
    public JwtAuthenticationProvider(JwtUserService userService) {
        this.userService = userService;
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        DecodedJWT jwt = ((JwtAuthenticationToken)authentication).getToken();
        if(jwt.getExpiresAt().before(Calendar.getInstance().getTime()))
            throw new NonceExpiredException("Token expires");
        String username = jwt.getSubject();
        UserDetails user = userService.getUserLoginInfo(username);
        if(user == null || user.getPassword()==null)
            throw new NonceExpiredException("Token expires");
        String encryptSalt = user.getPassword();
        try {
            Algorithm algorithm = Algorithm.HMAC256(encryptSalt);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withSubject(username)
                    .build();
            verifier.verify(jwt.getToken());
        } catch (Exception e) {
            throw new BadCredentialsException("JWT token verify fail", e);
        }
        //成功后返回认证信息，filter会将认证信息放入SecurityContext
        JwtAuthenticationToken token = new JwtAuthenticationToken(user, jwt, user.getAuthorities());
        return token;
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.isAssignableFrom(JwtAuthenticationToken.class);
    }

}
```
### 认证结果Handler
如果token认证失败，并且不在permissive列表中话，就会调用FailHandler，这个Handler和登录行为一致，所以都使用HttpStatusLoginFailureHandler 返回401错误。
token认证成功，在继续FilterChain中的其它Filter之前，我们先检查一下token是否需要刷新，刷新成功后会将新token放入header中。所以，新增一个JwtRefreshSuccessHandler来处理token认证成功的情况。
```
public class JwtRefreshSuccessHandler implements AuthenticationSuccessHandler{
    
    private static final int tokenRefreshInterval = 300;  //刷新间隔5分钟
    
    private JwtUserService jwtUserService;
    
    public JwtRefreshSuccessHandler(JwtUserService jwtUserService) {
        this.jwtUserService = jwtUserService;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {
        DecodedJWT jwt = ((JwtAuthenticationToken)authentication).getToken();
        boolean shouldRefresh = shouldTokenRefresh(jwt.getIssuedAt());
        if(shouldRefresh) {
            String newToken = jwtUserService.saveUserLoginInfo((UserDetails)authentication.getPrincipal());
            response.setHeader("Authorization", newToken);
        }   
    }
    
    protected boolean shouldTokenRefresh(Date issueAt){
        LocalDateTime issueTime = LocalDateTime.ofInstant(issueAt.toInstant(), ZoneId.systemDefault());
        return LocalDateTime.now().minusSeconds(tokenRefreshInterval).isAfter(issueTime);
    }

}
```
### JwtLoginConfigurer
跟登录逻辑一样，我们定义一个configurer，用来初始化和配置JWTFilter
```$xslt
public class JwtLoginConfigurer<T extends JwtLoginConfigurer<T, B>, B extends HttpSecurityBuilder<B>> extends AbstractHttpConfigurer<T, B> {
    
    private JwtAuthenticationFilter authFilter;
    
    public JwtLoginConfigurer() {
        this.authFilter = new JwtAuthenticationFilter();
    }
    
    @Override
    public void configure(B http) throws Exception {
        authFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));
        authFilter.setAuthenticationFailureHandler(new HttpStatusLoginFailureHandler());
        //将filter放到logoutFilter之前
        JwtAuthenticationFilter filter = postProcess(authFilter);
        http.addFilterBefore(filter, LogoutFilter.class);
    }
    //设置匿名用户可访问url
    public JwtLoginConfigurer<T, B> permissiveRequestUrls(String ... urls){
        authFilter.setPermissiveUrl(urls);
        return this;
    }
    
    public JwtLoginConfigurer<T, B> tokenValidSuccessHandler(AuthenticationSuccessHandler successHandler){
        authFilter.setAuthenticationSuccessHandler(successHandler);
        return this;
    }
    
}

```
### 配置集成

整个登录和无状态用户认证的流程都已经讲完了，现在我们需要吧spring security集成到我们的web项目中去。spring security和spring mvc做了很好的集成，一共只需要做两件事，给web配置类加上@EanbleWebSecurity，继承WebSecurityConfigurerAdapter定义个性化配置。
配置类WebSecurityConfig

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{

    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/image/**").permitAll() //静态资源访问无需认证
                .antMatchers("/admin/**").hasAnyRole("ADMIN") //admin开头的请求，需要admin权限
                .antMatchers("/article/**").hasRole("USER") //需登陆才能访问的url
                .anyRequest().authenticated()  //默认其它的请求都需要认证，这里一定要添加
                .and()
            .csrf().disable()  //CRSF禁用，因为不使用session
            .sessionManagement().disable()  //禁用session
            .formLogin().disable() //禁用form登录
            .cors()  //支持跨域
            .and()   //添加header设置，支持跨域和ajax请求
            .headers().addHeaderWriter(new StaticHeadersWriter(Arrays.asList(
                    new Header("Access-control-Allow-Origin","*"),
                    new Header("Access-Control-Expose-Headers","Authorization"))))
            .and() //拦截OPTIONS请求，直接返回header
            .addFilterAfter(new OptionRequestFilter(), CorsFilter.class)
            //添加登录filter
            .apply(new JsonLoginConfigurer<>()).loginSuccessHandler(jsonLoginSuccessHandler())
            .and()
           //添加token的filter
            .apply(new JwtLoginConfigurer<>()).tokenValidSuccessHandler(jwtRefreshSuccessHandler()).permissiveRequestUrls("/logout")
            .and()
            //使用默认的logoutFilter
            .logout()
//              .logoutUrl("/logout")   //默认就是"/logout"
                .addLogoutHandler(tokenClearLogoutHandler())  //logout时清除token
                .logoutSuccessHandler(new HttpStatusReturningLogoutSuccessHandler()) //logout成功后返回200
            .and()
            .sessionManagement().disable();
    }
    //配置provider
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(daoAuthenticationProvider()).authenticationProvider(jwtAuthenticationProvider());
    }
    
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    @Bean("jwtAuthenticationProvider")
    protected AuthenticationProvider jwtAuthenticationProvider() {
        return new JwtAuthenticationProvider(jwtUserService());
    }
    
    @Bean("daoAuthenticationProvider")
    protected AuthenticationProvider daoAuthenticationProvider() throws Exception{
        //这里会默认使用BCryptPasswordEncoder比对加密后的密码，注意要跟createUser时保持一致
        DaoAuthenticationProvider daoProvider = new DaoAuthenticationProvider();
        daoProvider.setUserDetailsService(userDetailsService());
        return daoProvider;
    }
    ...
}
```
以上的配置类主要关注一下几个点：

访问权限配置，使用url匹配是放过还是需要角色和认证
跨域支持，这个我们下面再讲
禁用csrf，csrf攻击是针对使用session的情况，这里是不需要的，关于CSRF可参考 Cross Site Request Forgery
禁用默认的form登录支持
logout支持，spring security已经默认支持logout filter，会拦截/logout请求，交给logoutHandler处理，同时在logout成功后调用LogoutSuccessHandler。
对于logout，我们需要清除保存的token salt信息，这样再拿logout之前的token访问就会失败。请参考TokenClearLogoutHandler：
```$xslt
public class TokenClearLogoutHandler implements LogoutHandler {
    
    private JwtUserService jwtUserService;
    
    public TokenClearLogoutHandler(JwtUserService jwtUserService) {
        this.jwtUserService = jwtUserService;
    }

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        clearToken(authentication);
    }
    
    protected void clearToken(Authentication authentication) {
        if(authentication == null)
            return;
        UserDetails user = (UserDetails)authentication.getPrincipal();
        if(user!=null && user.getUsername()!=null)
            jwtUserService.deleteUserLoginInfo(user.getUsername());
    }

}
```
## 角色配置

Spring Security对于访问权限的检查主要是通过AbstractSecurityIntercepter来实现，进入这个拦截器的基础一定是在context有有效的Authentication。
回顾下上面实现的UserDetailsService，在登录或token认证时返回的Authentication包含了GrantedAuthority的列表。
```
@Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //调用roles("USER")会将USER角色加入GrantedAuthority
        return User.builder().username("Jack").password(passwordEncoder.encode("jack-password")).roles("USER").build();  
    }
```
然后我们上面的配置类中有对url的role做了配置。比如下面的配置表示/admin开头的url支持有admin和manager权限的用户访问：
```$xslt
.antMatchers("/admin/**").hasAnyRole("ADMIN,MANAGER") 
```
对于Intecepter来说只需要吧配置中的信息和GrantedAuthority的信息一起提交给AccessDecisionManager来做比对。

> 参考:https://www.jianshu.com/p/d5ce890c67f7
