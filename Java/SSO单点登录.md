#### 一、了解什么是跨域？

浏览器使用同源策略限制，同源策略会阻止javascript脚本在不同域上交互，而ip、端口、域名、主机名不同都会造成跨域。

#### 二、Session与Token的区别

##### Session：

1. 用户发送登录请求
2. 服务端创建session
3. 浏览器拿到返回的资源并且创建一个cookie存放**jsessionid**
4. 浏览器再次请求携带cookie
5. 服务端根据cookie中的jsessionid查表，如果存在响应返回数据，如果不存在跳转登录

##### Token：

1. 用户发送登录请求
2. 服务端校验用户名和密码，返回token
3. 浏览器将token存放到cookie、localstorage中
4. 浏览器再次请求携带token
5. 服务端校验token真实性并解析token校验用户名和密码是否正确，正确响应返回数据，错误返回错误信息。

##### Session与Token比较：

```html
采用cookie容易导致跨站请求伪造（攻击者通过伪造跳转信息，由于用户已在浏览器认证过，所以间接操作用户账号。），欺骗浏览器，以用户的名义操作。
```



1. session存放于服务器，占用一定资源session过多将会造成服务器压力，token属于无状态，存放于浏览器。
2. cookie容易导致跨站请求伪造。
3. 分布式环境下，session只会存在一台服务器，导致session共享问题，而token则无这一问题。
4. token比session扩展性更高

#### 三、JWT是什么？

JWT全称JSON Web Token，以JSON对象的形式在各方之间安全地传输信息。由于该信息是数字签名的，因此可以对其进行验证和信任。jwt可以使用秘密(使用HMAC算法)签名，也可以使用RSA或ECDSA公钥/私钥对签名。

虽然jwt也可以加密，以提供各方之间的保密，但我们将重点关注签名令牌。已签名的令牌可以验证其中包含的声明的完整性，而加密的令牌则对其他方隐藏这些声明。当使用公钥/私钥对对令牌进行签名时，签名还证明只有持有私钥的一方是签名方。

- Header 头部

  ```json
  {
      "typ": "JWT",	//固定JWT
      "alg": "HS256"	//加密的算法
  }
  ```

  

- Payload 载荷

  ```json
  {
      "exp": 1627113081,
      "username": "a"
  }
  iss: jwt签发者
  sub: 面向的用户(jwt所面向的用户)
  aud: 接收jwt的一方
  exp: 过期时间戳(jwt的过期时间，这个过期时间必须要大于签发时间)
  nbf: 定义在什么时间之前，该jwt都是不可用的.
  iat: jwt的签发时间
  jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
  
  ```

  

- Signature 令牌

  ```json
  包含三部分：
  header (base64后的)+
  payload (base64后的)+
  secret 盐
  (header+payload+secret) --->加密
  这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密
  ```

  

#### 四、SpirngBoot集成Shiro+JWT

由于Shiro采用session存储，jwt属于无状态的，二者结合可以做单点登录的认证和授权。

##### 1、引入依赖

```java
<!--Shiro-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-starter</artifactId>
    <version>1.5.3</version>
</dependency>
<!--JWT-->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.4.0</version>
</dependency>
```



引入依赖后，接下来就是需要进行配置了，Shiro和JWT的整合所需要的配置和工具类有很多，主要分为下面两部分：

- 和JWT相关：JwtUtil、JwtToken

- 和shiro相关：ShiroConfig、JWTFilter、MyRealm

  

##### 2、编写JWTUtils工具类

```java
import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTDecodeException;
import com.auth0.jwt.interfaces.DecodedJWT;
import lombok.extern.slf4j.Slf4j;
import java.util.Date;
/**
 * jwt工具，用来生成、校验token以及提取token中的信息
 */
@Slf4j
public class JwtUtil {
    //指定一个token过期时间（毫秒）
    private static final long EXPIRE_TIME = 7 * 24 * 60 * 60 * 1000;  //7天

    /**
     * 生成token
     */
    //注意这里的sercet不是密码，而是进行三件套（salt+MD5+1024Hash）处理密码后得到的凭证
    //这里为什么要这么做，在controller中进行说明
    public static String getJwtToken(String username, String secret) {
        Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
        Algorithm algorithm = Algorithm.HMAC256(secret);    //使用密钥进行哈希
        // 附带username信息的token
        return JWT.create()
                .withClaim("username", username)
                .withExpiresAt(date)  //过期时间
                .sign(algorithm);     //签名算法
    }

    /**
     * 校验token是否正确
     */
    public static boolean verifyToken(String token, String username, String secret) {
        try {
            //根据密钥生成JWT效验器
            Algorithm algorithm = Algorithm.HMAC256(secret);
            JWTVerifier verifier = JWT.require(algorithm)
                    .withClaim("username", username)
                    .build();
            //效验TOKEN（其实也就是比较两个token是否相同）
            DecodedJWT jwt = verifier.verify(token);
            return true;
        } catch (Exception exception) {
            return false;
        }
    }

    /**
     * 在token中获取到username信息
     */
    public static String getUsername(String token) {
        try {
            DecodedJWT jwt = JWT.decode(token);
            return jwt.getClaim("username").asString();
        } catch (JWTDecodeException e) {
            return null;
        }
    }

    /**
     * 判断是否过期
     */
    public static boolean isExpire(String token){
        DecodedJWT jwt = JWT.decode(token);
        return jwt.getExpiresAt().getTime() < System.currentTimeMillis() ;
    }
}
```



##### 3、编写JwtToken

```java
import org.apache.shiro.authc.AuthenticationToken;

/**
 * 自定义的shiro接口token，可以通过这个类将string的token转型成AuthenticationToken，可供shiro使用
 * 注意：需要重写getPrincipal和getCredentials方法，因为是进行三件套处理的，没有特殊配置shiro无法通过这*两个方法获取到用户名和密码，需要直接返回token，之后交给JwtUtil去解析获取。（当然了，可以对realm进行配 置  *HashedCredentialsMatcher，这里就不这么处理了）
 */
public class JwtToken implements AuthenticationToken {
    private String token;

    public JwtToken(String token) {
        this.token = token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }
    @Override
    public Object getCredentials() {
        return token;
    }
}
```



##### 4、自定义MyRealm

```java
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * 自定义Realm
 */
public class MyRealm extends AuthorizingRealm {
    @Autowired
    private UserService userService;

    /**
     * 限定这个realm只能处理JwtToken（不加的话会报错）
     */
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    /**
     * 授权(授权部分这里就省略了，先把重心放在认证上)
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        //获取到用户名，查询用户权限
        return null;
    }

    /**
     * 认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) {
        String token = (String) auth.getCredentials();  //JwtToken中重写了这个方法了
        String username = JwtUtil.getUsername(token);   // 获得username
        
        //用户不存在（这个在登录时不会进入，只有在token校验时才有可能进入）
        if(username == null)
            throw new UnknownAccountException();

        //根据用户名，查询数据库获取到正确的用户信息
        User user = userService.getUserInfoByName(username);

        //用户不存在（这个在登录时不会进入，只有在token校验时才有可能进入）
        if(user == null)
            throw new UnknownAccountException();

        //密码错误(这里获取到password，就是3件套处理后的保存到数据库中的凭证，作为密钥)
        if (! JwtUtil.verifyToken(token, username, user.getPassword())) { 
            throw new IncorrectCredentialsException();
        }
        //toke过期
        if(JwtUtil.isExpire(token)){
            throw new ExpiredCredentialsException();
        }

        return new SimpleAuthenticationInfo(user, token, getName());
    }
}

```



##### 5、编写JWTFilter进行token拦截

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.klane.smartwatersystem.common.jwt.JwtToken;
import com.klane.smartwatersystem.common.ResultTemplate;
import com.klane.smartwatersystem.common.StatusCode;
import com.klane.smartwatersystem.common.jwt.JwtUtil;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.ShiroException;
import org.apache.shiro.authc.*;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter;
import org.apache.shiro.web.util.WebUtils;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.RequestMethod;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author: lhy
 *  jwt过滤器，作为shiro的过滤器，对请求进行拦截并处理
    跨域配置不在这里配了，我在另外的配置类进行配置了，这里把重心放在验证上
 */
@Slf4j
@Component
public class JwtFilter extends BasicHttpAuthenticationFilter{

    /**
     * 进行token的验证
     */
    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        //在请求头中获取token
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        String token = httpServletRequest.getHeader("Authorization"); //前端命名Authorization
        //token不存在
        if(token == null || "".equals(token)){
            ResultTemplate<Object> res = new ResultTemplate<>();
            res.setCode(StatusCode.UNLOGIN.getCode()).setMessage("无token，无权访问，请先登录");
            out(response, res);
            return false;
        }

        //token存在，进行验证
        JwtToken jwtToken = new JwtToken(token);
        try {
            SecurityUtils.getSubject().login(jwtToken);  //通过subject，提交给myRealm进行登录验证
            return true;    
        } catch (ExpiredCredentialsException e){
            ResultTemplate<Object> res = new ResultTemplate<>();
            res.setCode(StatusCode.TOKENEXPIRED.getCode()).setMessage("token过期，请重新登录");
            out(response,res);
            e.printStackTrace();
            return false;
        } catch (ShiroException e){  
            // 其他情况抛出的异常统一处理，由于先前是登录进去的了，所以都可以看成是token被伪造造成的
            ResultTemplate<Object> res = new ResultTemplate<>();
            res.etCode(StatusCode.FAKETOKEN.getCode()).setMessage("token被伪造，无效token");
            out(response,res);
            e.printStackTrace();
            return false;
        }
    }

    /**
     * json形式返回结果token验证失败信息，无需转发
     */
    private void out(ServletResponse response, ResultTemplate<Object> res) throws IOException {
        HttpServletResponse httpServletResponse = WebUtils.toHttp(response);
        ObjectMapper mapper = new ObjectMapper();
        String jsonRes = mapper.writeValueAsString(res);
        httpServletResponse.setCharacterEncoding("UTF-8");
        httpServletResponse.setContentType("application/json; charset=utf-8");
        httpServletResponse.getOutputStream().write(jsonRes.getBytes());
    }

    /**
     * 过滤器拦截请求的入口方法
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        try {
            return executeLogin(request, response);  //token验证
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * isAccessAllowed()方法返回false，即认证不通过时进入onAccessDenied方法
     */
    @Override
   protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
       return false;
    }

    /**
     * token认证executeLogin成功后，进入此方法，可以进行token更新过期时间
     */
//    @Override
//    protected boolean onLoginSuccess(AuthenticationToken token, Subject subject, ServletRequest request, ServletResponse response) throws Exception {
			
//    }
}
```

##### 6、ShiroConfig配置

```java
import com.klane.smartwatersystem.common.shiro.JwtFilter;
import com.klane.smartwatersystem.common.shiro.MyRealm;
import lombok.extern.slf4j.Slf4j;
import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
import org.apache.shiro.mgt.DefaultSubjectDAO;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.DependsOn;

import javax.servlet.Filter;
import java.util.HashMap;
import java.util.Map;

/**
 * shiro的配置类
 */
@Configuration
@Slf4j
public class ShiroConfig {

    /**
     * 由Spring管理 Shiro的生命周期
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    /**
     * 开启对 Shiro 注解的支持
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }

    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        // 强制使用cglib，防止重复代理和可能引起代理出错的问题
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

    //创建ShiroFilter（用于拦截所有请求，对受限资源进行Shiro的认证和授权判断）
    //Shiro提供了丰富的过滤器（anon等），不过在这里就需要加入我们自定义的JwtFilter了
    @Bean("shiroFilter")
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);

        // 添加自己的过滤器并且取名为jwt
        Map<String, Filter> filterMap = new HashMap<>();
        filterMap.put("jwt", new JwtFilter());
        shiroFilterFactoryBean.setFilters(filterMap);

        //配置系统的受限资源以及对应的过滤器
        Map<String, String> ruleMap = new HashMap<>();
        ruleMap.put("/user/login", "anon"); //登录路径、注册路径都需要放行不进行拦截
        ruleMap.put("/user/register", "anon");
        ruleMap.put("/**", "jwt");  // /**，一般放在最下，表示对所有资源起作用，使用JwtFilter
        shiroFilterFactoryBean.setFilterChainDefinitionMap(ruleMap);
        return shiroFilterFactoryBean;
    }


    //创建安全管理器（会自动设置到SecurityUtils中设置这个安全管理器）
    //SecurityUtils可以用来获取subject对象
    @Bean("securityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(MyRealm realm){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

        //给安全管理器设置realm
        securityManager.setRealm(realm);

        //关闭shiro的session（无状态的方式使用shiro）
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);

        return securityManager;
    }

    //创建自定义Realm，注入到spring容器中
    @Bean
    public MyRealm getRealm(){
        MyRealm realm = new MyRealm();
        //修改凭证校验匹配器（处理加密），只有使用了UsernamePasswordToken并且有对password进行加密的才需要
//        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
//        hashedCredentialsMatcher.setHashAlgorithmName("MD5");
//        hashedCredentialsMatcher.setHashIterations(1024);
//
//        realm.setCredentialsMatcher(hashedCredentialsMatcher);
        return realm;
    }
}
```



##### 7、测试

```java
@PostMapping("/login")
public ResultTemplate<String> register(@RequestBody User loginUser,
                                       HttpServletRequest request){
    ResultTemplate<String> res = new ResultTemplate<>();
    String username = loginUser.getUsername();
    String password = loginUser.getPassword();

    //根据用户名获取正确用户信息
    User user = userService.getUserInfoByName(username);
    if(user == null)
        return res.setCode(StatusCode.INVALIDUSER.getCode()).setMessage("无效用户，用户不存在");

    //盐 + 输入的密码(注意不是用户的正确密码) + 1024次散列，作为token生成的密钥
    String salt = user.getSalt();
    Md5Hash md5Hash = new Md5Hash(password, salt, 1024);

    //生成token字符串
    String token = JwtUtil.getJwtToken(username, md5Hash.toHex());   //toHex转换成16进制，32为字符
    JwtToken jwtToken = new JwtToken(token);        //转换成jwtToken（才可以被shiro识别）
    
/**可能有人会问这里为什么不直接把passsword作为密钥，或者使用一个固定的密钥。
* 是因为在后面在subject.login进入到realm中，进行认证的时候，肯定需要和password相关的进行匹配校验。
* 因为下面不是用UsernamePasswordToken，不能直接获取到传入的password，所以：
*    1. 如果使用固定密钥，那么无法实现和密码相关的校验。
*    2. 如果使用password，则有可能两个人的password相同导致误判。
* 所以这里就需要在token中就将password相关信息包含进去，这里选择作为密钥
*/

/**
* 如果是使用UsernamePasswordToken，那么在realm中就可以获取到username和password，查数据库就很容易判断。
* 但是由于返回给前端jwtToken不是UsernamePasswordToken，就还需要另外一个realm对jwtToken进行解析。
* 使用UsernamePasswordToken的，可以参考    
  https://blog.csdn.net/pengjunlee/article/details/95600843#pom.xml。我觉得很清晰
* 使用密码加密作为密钥的这种处理，同一个realm就可以实现了，各有利弊。当然了，也可以使用redis来处理。
*/

    //拿到Subject对象
    Subject subject = SecurityUtils.getSubject();

    //进行认证
    try {
        subject.login(jwtToken);
        res.setCode(StatusCode.SUCCESS.getCode()).setMessage("登录成功").setData(token);
    } catch (UnknownAccountException e){
        res.setCode(StatusCode.INVALIDUSER.getCode()).setMessage("无效用户，用户不存在");
        e.printStackTrace();
    } catch (IncorrectCredentialsException e){
        res.setCode(StatusCode.PASSWORDERROR.getCode()).setMessage("密码输入错误");
        e.printStackTrace();
    } catch (ExpiredCredentialsException  e){
        res.setCode(StatusCode.TOKENEXPIRED.getCode()).setMessage("token过期，请重新登录");
        e.printStackTrace();
    } finally {
        return res;
    }
}
```

