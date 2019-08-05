# Shrio

## 一、初步认识

shiro是一个安全权限管理框架，执行身份验证、授权、密码和会话管理

## 二、基本概念

subject：主体，表示当前用户，shiro用subject封装用户传递的数据

securityManager：安全管理中心，shiro的核心，所有操作都要经过securityManager进行管理

reaml：域，就是shiro从reaml获取数据还和subject里面的token进行对比，根据验证策略判断通过或者不通过。

## 三、demo

ShiroConfig

```java
package com.example.demo.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.example.demo.contorller.UserController;
import com.example.demo.realm.UserModularRealmAuthenticator;
import com.example.demo.realm.UserRealm;
import org.apache.shiro.authc.pam.AtLeastOneSuccessfulStrategy;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.authz.Authorizer;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.realm.jdbc.JdbcRealm;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.apache.shiro.mgt.SecurityManager;

import java.util.LinkedHashMap;
import java.util.LinkedList;
import java.util.List;
import java.util.Map;

/**
 * @author 为为
 * @create 7/12
 */
@Configuration
public class ShiroConfig {

    private static Logger logger= LoggerFactory.getLogger(UserController.class);


    @Bean
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 必须设置 SecurityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // setLoginUrl 如果不设置值，默认会自动寻找Web工程根目录下的"/login.jsp"页面 或 "/login" 映射
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 设置无权限时跳转的 url;
        shiroFilterFactoryBean.setUnauthorizedUrl("/error");
        //成功后的页面
        shiroFilterFactoryBean.setSuccessUrl("/index");


        // 设置拦截器
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        //游客，开发权限
       /* filterChainDefinitionMap.put("/guest*//**", "anon");
         //用户，需要角色权限 “user”
         filterChainDefinitionMap.put("/user*//**", "roles[user]");
         //管理员，需要角色权限 “admin”
         filterChainDefinitionMap.put("/admin*//**", "roles[admin]");*/
        //开放登陆接口
        filterChainDefinitionMap.put("/login", "anon");
        //主要这行代码必须放在所有权限设置的最后，不然会导致所有 url 都被拦截
        filterChainDefinitionMap.put("/**", "roles[user]");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        System.out.println("Shiro拦截器工厂类注入成功");
        return shiroFilterFactoryBean;
    }

    @Bean
    public SecurityManager securityManager(Authorizer authorizer) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        // 设置realm.
      /*  List<Realm> list = new LinkedList();
        list.add(userRealm());
        list.add(jdbcRealm());*/
        //securityManager.setRealms(list);
        securityManager.setRealm(jdbcRealm());
        //securityManager.setRealm(userRealm());
        securityManager.setAuthorizer(authorizer);
        return securityManager;
    }

    //@Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();

        securityManager.setAuthenticator(modularRealmAuthenticator());
        logger.info("设置完自定义多reaml");
        return securityManager;
    }

    /**
     * 系统自带的Realm管理，主要针对多realm
     */
    @Bean
    public ModularRealmAuthenticator modularRealmAuthenticator() {
        //自己重写的ModularRealmAuthenticator
        UserModularRealmAuthenticator modularRealmAuthenticator = new UserModularRealmAuthenticator();
        modularRealmAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        // 设置realm.
        List<Realm> list = new LinkedList();
        list.add(userRealm());
        list.add(jdbcRealm());
        modularRealmAuthenticator.setRealms(list);
        return modularRealmAuthenticator;
    }


    @Bean
    public JdbcRealm jdbcRealm() {
        JdbcRealm jdbcRealm = new JdbcRealm();
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/shiro?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8&allowMultiQueries=true");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        jdbcRealm.setDataSource(dataSource);
        return jdbcRealm;
    }

    //@Bean
    public UserRealm userRealm() {
        return new UserRealm();
    }

    //开启shiro aop注解支持, 启用权限注解
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }

    //开启shiro aop注解支持, 启用认证注解
    @Bean
    @ConditionalOnMissingBean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAAP = new DefaultAdvisorAutoProxyCreator();
        defaultAAP.setProxyTargetClass(true);
        return defaultAAP;
    }

}
```

UserReaml

```java
package com.example.demo.realm;

import com.example.demo.mapper.UserMapper;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.HashSet;
import java.util.Set;

/**
 * @author 为为
 * @create 7/12
 */
public class UserRealm extends AuthorizingRealm {

    private UserMapper userMapper;

    @Override
    public String getName(){
        return "user";
    }

    @Autowired
    private void setUserMapper(UserMapper userMapper) {
        this.userMapper = userMapper;
    }
    /**
     * 获取授权信息
     *
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("————权限认证————");
        String username = (String) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //获得该用户角色
        String role = userMapper.getRole(username);
        Set<String> set = new HashSet<>();
        //需要将 role 封装到 Set 作为 info.setRoles() 的参数
        set.add(role);
        //设置该用户拥有的角色
        info.setRoles(set);
        return info;
    }

    /**
     * 获取身份验证信息
     * Shiro中，最终是通过 Realm 来获取应用程序中的用户、角色及权限信息的。
     *
     * @param authenticationToken 用户身份信息 token
     * @return 返回封装了用户信息的 AuthenticationInfo 实例
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        System.out.println("————身份认证方法————");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        // 从数据库获取对应用户名密码的用户
        String password = userMapper.getPassWordByName(token.getUsername());
        if (null == password) {
            throw new AccountException("用户名不正确");
        } else if (!password.equals(new String((char[]) token.getCredentials()))) {
            throw new AccountException("密码不正确");
        }
        return new SimpleAuthenticationInfo(token.getPrincipal(), password, getName());
    }

    public  static  void main(String[] args){
        String credentials="123456";
        String salt=null;
        String hashAlgorithmName="MD5";
        SimpleHash simpleHash = new SimpleHash(hashAlgorithmName, credentials, salt);

        System.out.println(simpleHash);
    }
}
```

UserModularRealmAuthenticator

```java
package com.example.demo.realm;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.pam.ModularRealmAuthenticator;
import org.apache.shiro.realm.Realm;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Collection;

/**
 * @author 为为
 * @create 7/15
 */
public class UserModularRealmAuthenticator extends ModularRealmAuthenticator {

    private static final Logger logger = LoggerFactory.getLogger(UserModularRealmAuthenticator.class);

    @Override
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        logger.info("UserModularRealmAuthenticator:method doAuthenticate() execute ");
        // 判断getRealms()是否返回为空
        assertRealmsConfigured();
        Collection<Realm> realms = getRealms();
        // 登录类型对应的所有Realm
        ArrayList<Realm> typeRealms = new ArrayList<>();
        for (Realm realm : realms) {

            typeRealms.add(realm);
        }
        // 判断是单Realm还是多Realm
        if (typeRealms.size() == 1) {
            logger.info("doSingleRealmAuthentication() execute ");
            return doSingleRealmAuthentication(typeRealms.get(0), authenticationToken);
        } else {
            logger.info("doMultiRealmAuthentication() execute ");
            return doMultiRealmAuthentication(typeRealms, authenticationToken);
        }
    }

}
```

## 四、自定义realm

​	写一个类，只需要继承AuthorizingRealm 实现抽象方法就可以

```java
@Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        System.out.println("————身份认证方法————");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        // 从数据库获取对应用户名密码的用户
        String password = userMapper.getPassWordByName(token.getUsername());
        if (null == password) {
            throw new AccountException("用户名不正确");
        } else if (!password.equals(new String((char[]) token.getCredentials()))) {
            throw new AccountException("密码不正确");
        }
        return new SimpleAuthenticationInfo(token.getPrincipal(), password, getName());
    }
@Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("————权限认证————");
        String username = (String) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //获得该用户角色
        String role = userMapper.getRole(username);
        Set<String> set = new HashSet<>();
        //需要将 role 封装到 Set 作为 info.setRoles() 的参数
        set.add(role);
        //设置该用户拥有的角色
        info.setRoles(set);
        return info;
    }
```



## 五、认证过程

用户登录

#### 1、进入controller，获取subject，执行login方法

```java
public String login(@RequestParam(required = false) boolean rememberMe,User user){
        String userName = user.getUserName();
        Subject currentUser = SecurityUtils.getSubject();
        if(!currentUser.isAuthenticated()) {
            UsernamePasswordToken token =new UsernamePasswordToken(userName,user.getPassword());
            try {
                if(rememberMe) {
                    token.setRememberMe(true);
                }
                //主要
                currentUser.login(token);
                //
                System.out.println( currentUser.hasRole("user"));

                return "login";
            }catch (UnknownAccountException uae) {
                logger.info("对用户[" + userName + "]进行登录验证..验证未通过,未知账户");
            } catch (IncorrectCredentialsException ice) {
                logger.info("对用户[" + userName + "]进行登录验证..验证未通过,错误的凭证");
            } catch (LockedAccountException lae) {
                logger.info("对用户[" + userName + "]进行登录验证..验证未通过,账户已锁定");
            } catch (ExcessiveAttemptsException eae) {
                logger.info("对用户[" + userName + "]进行登录验证..验证未通过,错误次数过多");
            } catch (AuthenticationException ae) {
                //通过处理Shiro的运行时AuthenticationException就可以控制用户登录失败或密码错误时的情景
                logger.info("对用户[" + userName + "]进行登录验证..验证未通过,堆栈轨迹如下");
                ae.printStackTrace();
            }
        }

        return "login";
    }
```

#### 2、进入DelegatingSubject，调用securityManager.login方法

```java
public void login(AuthenticationToken token) throws AuthenticationException {
        this.clearRunAsIdentitiesInternal();
    	//主要
        Subject subject = this.securityManager.login(this, token);
    	//
        String host = null;
        PrincipalCollection principals;
        if(subject instanceof DelegatingSubject) {
            DelegatingSubject delegating = (DelegatingSubject)subject;
            principals = delegating.principals;
            host = delegating.host;
        } else {
            principals = subject.getPrincipals();
        }

        if(principals != null && !principals.isEmpty()) {
            this.principals = principals;
            this.authenticated = true;
            if(token instanceof HostAuthenticationToken) {
                host = ((HostAuthenticationToken)token).getHost();
            }

            if(host != null) {
                this.host = host;
            }

            Session session = subject.getSession(false);
            if(session != null) {
                this.session = this.decorate(session);
            } else {
                this.session = null;
            }

        } else {
            String msg = "Principals returned from securityManager.login( token ) returned a null or empty value.  This value must be non null and populated with one or more elements.";
            throw new IllegalStateException(msg);
        }
    }
```

#### 3、DefaultSecurityManager，执行login方法

```java
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info;
        try {
            //主要
            info = this.authenticate(token);
            //
        } catch (AuthenticationException var7) {
            AuthenticationException ae = var7;

            try {
                this.onFailedLogin(token, ae, subject);
            } catch (Exception var6) {
                if(log.isInfoEnabled()) {
                    log.info("onFailedLogin method threw an exception.  Logging and propagating original AuthenticationException.", var6);
                }
            }

            throw var7;
        }

        Subject loggedIn = this.createSubject(token, info, subject);
        this.onSuccessfulLogin(token, info, loggedIn);
        return loggedIn;
    }
```

#### 4、进入AuthenticatingSecurityManager，执行authenticate认证方法

```java
    public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        return this.authenticator.authenticate(token);
  }5、AbstractAuthenticator，执行authenticate方法
```

```java
public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        if(token == null) {
            throw new IllegalArgumentException("Method argument (authentication token) cannot be null.");
        } else {
            log.trace("Authentication attempt received for token [{}]", token);

            AuthenticationInfo info;
            try {
                //主要
                info = this.doAuthenticate(token);
                //
                if(info == null) {
                    String msg = "No account information found for authentication token [" + token + "] by this Authenticator instance.  Please check that it is configured correctly.";
                    throw new AuthenticationException(msg);
                }
            } catch (Throwable var8) {
                AuthenticationException ae = null;
                if(var8 instanceof AuthenticationException) {
                    ae = (AuthenticationException)var8;
                }

                if(ae == null) {
                    String msg = "Authentication failed for token submission [" + token + "].  Possible unexpected error? (Typical or expected login exceptions should extend from AuthenticationException).";
                    ae = new AuthenticationException(msg, var8);
                    if(log.isWarnEnabled()) {
                        log.warn(msg, var8);
                    }
                }

                try {
                    this.notifyFailure(token, ae);
                } catch (Throwable var7) {
                    if(log.isWarnEnabled()) {
                        String msg = "Unable to send notification for failed authentication attempt - listener error?.  Please check your AuthenticationListener implementation(s).  Logging sending exception and propagating original AuthenticationException instead...";
                        log.warn(msg, var7);
                    }
                }

                throw ae;
            }

            log.debug("Authentication successful for token [{}].  Returned account [{}]", token, info);
            this.notifySuccess(token, info);
            return info;
        }
    }
```

#### 5、进入ModularRealmAuthenticator，执行doAuthenticate方法

```java
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        this.assertRealmsConfigured();
        Collection<Realm> realms = this.getRealms();
        //判断单reaml还是多reaml，多reaml有一个执行策略
        return realms.size() == 1?this.doSingleRealmAuthentication((Realm)realms.iterator().next(), authenticationToken):this.doMultiRealmAuthentication(realms, authenticationToken);
    }
```

#### 6、ModularRealmAuthenticator，doSingleRealmAuthentication方法

```java
 protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
        if(!realm.supports(token)) {
            String msg = "Realm [" + realm + "] does not support authentication token [" + token + "].  Please ensure that the appropriate Realm implementation is configured correctly or that the realm accepts AuthenticationTokens of this type.";
            throw new UnsupportedTokenException(msg);
        } else {
            //主要
            AuthenticationInfo info = realm.getAuthenticationInfo(token);
            //
            if(info == null) {
                String msg = "Realm [" + realm + "] was unable to find account data for the submitted AuthenticationToken [" + token + "].";
                throw new UnknownAccountException(msg);
            } else {
                return info;
            }
        }
    }
```

#### 7、进入AuthenticatingRealm，getAuthenticationInfo方法

```java
public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info = this.getCachedAuthenticationInfo(token);
        if(info == null) {
            //主要
            info = this.doGetAuthenticationInfo(token);
            //主要
            log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
            if(token != null && info != null) {
                this.cacheAuthenticationInfoIfPossible(token, info);
            }
        } else {
            log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
        }

        if(info != null) {
            this.assertCredentialsMatch(token, info);
        } else {
            log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
        }

        return info;
    }
```

#### 8、进入配置的reaml里面验证，执行doGetAuthenticationInfo方法

```java
 @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        System.out.println("————身份认证方法————");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        // 从数据库获取对应用户名密码的用户
        String password = userMapper.getPassWordByName(token.getUsername());
        if (null == password) {
            throw new AccountException("用户名不正确");
        } else if (!password.equals(new String((char[]) token.getCredentials()))) {
            throw new AccountException("密码不正确");
        }
        return new SimpleAuthenticationInfo(token.getPrincipal(), password, getName());
    }
```

​	一层层返回，认证结束

​	默认的subject是WebDelegatingSubject，当controller调用login方法时，调用父类DelegatingSubject的方法

![1563416900995](C:\Users\Administrator\Desktop\MyNotes\src\shiro\images\1563416900995.png)

​	默认的securityManager是DefaultWebSecurityManager

![1563416970615](C:\Users\Administrator\Desktop\MyNotes\src\shiro\images\1563416970615.png)

## 六、授权过程

#### 1、认证完成之后调用subject的isPermitted方法验证时候具有权限

```java
@RequestMapping("login")
    public String login(@RequestParam(required = false) boolean rememberMe,User user){
        String userName = user.getUserName();
        Subject subject = SecurityUtils.getSubject();
        if(!subject.isAuthenticated()) {
            UsernamePasswordToken token =new UsernamePasswordToken(userName,user.getPassword());
            try {
                if(rememberMe) {
                    token.setRememberMe(true);
                }
                subject.login(token);
                //····主要
                subject.isPermitted("user:add");
				//
                return "login";
            }catch (AuthenticationException ae) {
                //通过处理Shiro的运行时AuthenticationException就可以控制用户登录失败或密码错误时的情
                logger.info("登陆错误");
                ae.printStackTrace();
            }
        }

        return "login";
    }
```

#### 2、进入DelegatingSubject执行isPermitted的方法

​	这个步骤只调用了两个方法，返回两个true才为true

```java
   public boolean isPermitted(String permission) {
        return hasPrincipals() && securityManager.isPermitted(getPrincipals(), permission);
    }
```

#### 3、DelegatingSubject执行hasPrincipals方法

​	判断getPrincipals方法为不为空，空 返回false 

```java
protected boolean hasPrincipals() {
    return !CollectionUtils.isEmpty(getPrincipals());
}
```

#### 4、DelegatingSubject执行getPrincipals方法

​	判断getRunAsPrincipalsStack方法返回是否为空，应该不会为空，因为

​	CollectionUtils.isEmpty(runAsPrincipals) ? this.principals : runAsPrincipals.get(0);

```java
    public PrincipalCollection getPrincipals() {
        List<PrincipalCollection> runAsPrincipals = getRunAsPrincipalsStack();
        return CollectionUtils.isEmpty(runAsPrincipals) ? this.principals : runAsPrincipals.get(0);
    }
```

#### 5、DelegatingSubject执行getRunAsPrincipalsStack方法

​	获取session

​	当没进controller时会先进这个方法一次

```java
    @SuppressWarnings("unchecked")
    private List<PrincipalCollection> getRunAsPrincipalsStack() {
        Session session = getSession(false);
        if (session != null) {
            return (List<PrincipalCollection>) session.getAttribute(RUN_AS_PRINCIPALS_SESSION_KEY);
        }
        return null;
    }
```

#### 6、AuthorizingSecurityManager，执行isPermitted方法

​	第二步骤的后半部分，调用isPermitted

```java
    public boolean isPermitted(PrincipalCollection principals, String permissionString) {
        return this.authorizer.isPermitted(principals, permissionString);
    }
```

#### 7、AuthorizingSecurityManager，执行isPermitted方法

​	resolvePermission，解析 判断 哪个权限  的开始，最终返回一个WildcardPermission，主要属性parts，里面是权限和角色的集合



```java
    public boolean isPermitted(PrincipalCollection principals, String permission) {
        Permission p = getPermissionResolver().resolvePermission(permission);
        return isPermitted(principals, p);
    }
```

#### 8、WildcardPermissionResolver，执行resolvePermission方法

```java
    public Permission resolvePermission(String permissionString) {
        return new WildcardPermission(permissionString);
    }
```

#### 9、WildcardPermission，执行setParts方法

​	setParts方法就是为WildcardPermission把字符串形式的角色和权限(users:add)变为集合，设置到parts属性中

```java
····················· 
public WildcardPermission(String wildcardString) {
        this(wildcardString, DEFAULT_CASE_SENSITIVE);
    }

    public WildcardPermission(String wildcardString, boolean caseSensitive) {
        setParts(wildcardString, caseSensitive);
    }

················执行方法前的操作

 protected void setParts(String wildcardString, boolean caseSensitive) {
 		//去掉空格
        wildcardString = StringUtils.clean(wildcardString);

        if (wildcardString == null || wildcardString.isEmpty()) {
            throw new IllegalArgumentException("Wildcard string cannot be null or empty. Make sure permission strings are properly formatted.");
        }
		//转换大小写
        if (!caseSensitive) {
            wildcardString = wildcardString.toLowerCase();
        }
		//分割角色和权限，转化为list   user:add,update-->user   add,update
        List<String> parts = CollectionUtils.asList(wildcardString.split(PART_DIVIDER_TOKEN));

        this.parts = new ArrayList<Set<String>>();
        for (String part : parts) {
        //再分割权限
            Set<String> subparts = CollectionUtils.asSet(part.split(SUBPART_DIVIDER_TOKEN));

            if (subparts.isEmpty()) {
                throw new IllegalArgumentException("Wildcard string cannot contain parts with only dividers. Make sure permission strings are properly formatted.");
            }
            //添加权限
            this.parts.add(subparts);
        }

        if (this.parts.isEmpty()) {
            throw new IllegalArgumentException("Wildcard string cannot contain only dividers. Make sure permission strings are properly formatted.");
        }
    }
```

#### 10、AuthorizingRealm执行getAuthorizationInfo方法

​	前面把要判断的权限设置完了

​	这步是执行自己的reaml的授权方法，添加权限和角色，返回info，里面有角色和权限，凭证

```java
protected AuthorizationInfo getAuthorizationInfo(PrincipalCollection principals) {

        if (principals == null) {
            return null;
        }

        AuthorizationInfo info = null;

        if (log.isTraceEnabled()) {
            log.trace("Retrieving AuthorizationInfo for principals [" + principals + "]");
        }

        Cache<Object, AuthorizationInfo> cache = getAvailableAuthorizationCache();
        if (cache != null) {
            if (log.isTraceEnabled()) {
                log.trace("Attempting to retrieve the AuthorizationInfo from cache.");
            }
            Object key = getAuthorizationCacheKey(principals);
            info = cache.get(key);
            if (log.isTraceEnabled()) {
                if (info == null) {
                    log.trace("No AuthorizationInfo found in cache for principals [" + principals + "]");
                } else {
                    log.trace("AuthorizationInfo found in cache for principals [" + principals + "]");
                }
            }
        }


        if (info == null) {
            // Call template method if the info was not found in a cache
            //···········主要 调用自己的reaml
            info = doGetAuthorizationInfo(principals);
            //···············
            // If the info is not null and the cache has been created, then cache the authorization info.
            if (info != null && cache != null) {
                if (log.isTraceEnabled()) {
                    log.trace("Caching authorization info for principals: [" + principals + "].");
                }
                Object key = getAuthorizationCacheKey(principals);
                cache.put(key, info);
            }
        }

        return info;
    }
```

#### 11、进入自己的reaml，给用户添加角色和权限

```java
 @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("————权限认证————");
        String username = (String) principalCollection.getPrimaryPrincipal();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        Set<String> set = new HashSet<>();
        set.add("user:add");
        set.add("user:update");
        set.add("user:delete");
        info.setStringPermissions(set);
        set = new HashSet<>();
        //设置该用户拥有的角色
        set.add("user");
        info.setRoles(set);
        return info;
    }

```

#### 12、AuthorizingRealm，执行isPermitted方法

​	把info变为Permission集合，implies方法返回最后的判断结果

```java
protected boolean isPermitted(Permission permission, AuthorizationInfo info) {
    	//检查、转换刚添加的权限
        Collection<Permission> perms = getPermissions(info);
        if (perms != null && !perms.isEmpty()) {
            for (Permission perm : perms) {
                if (perm.implies(permission)) {
                    return true;
                }
            }
        }
        return false;
    }
```

#### 13、AuthorizingRealm，执行getPermissions方法

```java
 protected Collection<Permission> getPermissions(AuthorizationInfo info) {
        Set<Permission> permissions = new HashSet<Permission>();

        if (info != null) {
            Collection<Permission> perms = info.getObjectPermissions();
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }
            //````````主要
            perms = resolvePermissions(info.getStringPermissions());
           //////
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }

            perms = resolveRolePermissions(info.getRoles());
            if (!CollectionUtils.isEmpty(perms)) {
                permissions.addAll(perms);
            }
        }

        if (permissions.isEmpty()) {
            return Collections.emptySet();
        } else {
            return Collections.unmodifiableSet(permissions);
        }
    }
```

#### 14、AuthorizingRealm，执行resolvePermissions方法

```java
private Collection<Permission> resolvePermissions(Collection<String> stringPerms) {
    Collection<Permission> perms = Collections.emptySet();
    PermissionResolver resolver = getPermissionResolver();
    if (resolver != null && !CollectionUtils.isEmpty(stringPerms)) {
        perms = new LinkedHashSet<Permission>(stringPerms.size());
        for (String strPermission : stringPerms) {
            //······主要 resolvePermission就是设置parts属性
            Permission permission = getPermissionResolver().resolvePermission(strPermission);
            //······
            perms.add(permission);
        }
    }
    return perms;
}
```

#### 15、重复7、8步骤

#### 16、WildcardPermission，执行implies方法

​	要判断的和新添加的权限都处理好了，最后一步进行判读是否具有权限

```java
public boolean implies(Permission p) {
    // By default only supports comparisons with other WildcardPermissions
    if (!(p instanceof WildcardPermission)) {
        return false;
    }

    WildcardPermission wp = (WildcardPermission) p;

    List<Set<String>> otherParts = wp.getParts();

    int i = 0;
    for (Set<String> otherPart : otherParts) {
        // If this permission has less parts than the other permission, everything after the number of parts contained
        // in this permission is automatically implied, so return true
        if (getParts().size() - 1 < i) {
            return true;
        } else {
            Set<String> part = getParts().get(i);
            if (!part.contains(WILDCARD_TOKEN) && !part.containsAll(otherPart)) {
                return false;
            }
            i++;
        }
    }

    // If this permission has more parts than the other parts, only imply it if all of the other parts are wildcards
    for (; i < getParts().size(); i++) {
        Set<String> part = getParts().get(i);
        if (!part.contains(WILDCARD_TOKEN)) {
            return false;
        }
    }

    return true;
}
```

​	判断结束，返回

## 七、权限、角色注解处理过程

## 八、html页面使用shiro标签

​	1、引入依赖

```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

​	2、html修改头

```xml
  <html lang="en" xmlns:th="http://www.thymeleaf.org"
          xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
```

​	3、添加bean

```java
 @Bean
 public ShiroDialect shiroDialect() {
    return new ShiroDialect();
 }
```

## 九、使用缓存

#### 1、使用ehcache

​	1）、导入依赖	

```
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.3.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-api</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
```

​	2）写ehcache的配置文件 ehcache-shiro.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache updateCheck="false" name="shiroCache">

    <diskStore path="D:\shiro\ehcache" />
    <!--     <diskStore path="java.io.tmpdir"/> -->

    <!--
    eternal：缓存中对象是否为永久的，如果是，超时设置将被忽略，对象从不过期。
    maxElementsInMemory：缓存中允许创建的最大对象数
    overflowToDisk：内存不足时，是否启用磁盘缓存。
    timeToIdleSeconds：缓存数据的钝化时间，也就是在一个元素消亡之前，  两次访问时间的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是 0 就意味着元素可以停顿无穷长的时间。
    timeToLiveSeconds：缓存数据的生存时间，也就是一个元素从构建到消亡的最大时间间隔值，这只能在元素不是永久驻留时有效，如果该值是0就意味着元素可以停顿无穷长的时间。
    memoryStoreEvictionPolicy：缓存满了之后的淘汰算法。
    diskPersistent:设定在虚拟机重启时是否进行磁盘存储，默认为false
    diskExpiryThreadIntervalSeconds: 属性可以设置该线程执行的间隔时间(默认是120秒，不能太小
    1 FIFO，先进先出
    2 LFU，最少被使用，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
    3 LRU，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
    -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
    />

    <cache name="activeSessionCache"
           maxElementsInMemory="10000"
           eternal="true"
           overflowToDisk="false"
           diskPersistent="true"
           diskExpiryThreadIntervalSeconds="600"/>

    <cache name="shiro.authorizationCache"
           maxElementsInMemory="100"
           eternal="false"
           timeToLiveSeconds="600"
           overflowToDisk="false"/>

</ehcache>
```

​	3）、添加bean

```java
@Bean
    public EhCacheManager cacheManager(){
        EhCacheManager manager = new EhCacheManager();
        manager.setCacheManagerConfigFile("classpath:ehcache-shiro.xml");
        return manager;
    }
```

​	4）、securityManager添加缓存管理器

```java
@Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm());
        securityManager.setCacheManager(cacheManager());
        return securityManager;
    }
```

​	**如果在运行过程中 主体的权限发生改变，可以在自定义reaml中实现清理缓存方法，在授予权限之后调用，用户可以不用退出**

```java
public void clearCache() {
        PrincipalCollection principals = SecurityUtils.getSubject().getPrincipals();
        super.clearCache(principals);
    }
```

#### 2、使用redis缓存

​	1、导入pom

```xml
<dependency>
            <groupId>org.crazycake</groupId>
            <artifactId>shiro-redis</artifactId>
            <version>2.4.2.1-RELEASE</version>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.38</version>
        </dependency>
```

​	2、添加bean

```java
@Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm());
        securityManager.setCacheManager(cacheManager());
        securityManager.setSessionManager(sessionManager());
        return securityManager;
    }
    /**
     * cacheManager 缓存 redis实现
     * 使用的是shiro-redis开源插件
     *
     * @return
     */
    public RedisCacheManager cacheManager() {
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        redisCacheManager.setRedisManager(redisManager());
        return redisCacheManager;
    }
    /**
     * 配置shiro redisManager
     * 使用的是shiro-redis开源插件
     * @return
     */
    public RedisManager redisManager() {
        RedisManager redisManager = new RedisManager();
       //new FastJsonRedisSerializer();
        redisManager.setHost("192.168.214.30");
        redisManager.setPort(6379);
        // 配置缓存过期时间
        redisManager.setExpire(1800);
        redisManager.setTimeout(0);
        redisManager.setPassword("root");
        return redisManager;
    }
    /**
     * RedisSessionDAO shiro sessionDao层的实现 通过redis
     * 使用的是shiro-redis开源插件
     */
    @Bean
    public RedisSessionDAO redisSessionDAO() {
        RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
        redisSessionDAO.setRedisManager(redisManager());
        return redisSessionDAO;
    }
    @Bean
    public DefaultWebSessionManager sessionManager() {
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        sessionManager.setSessionDAO(redisSessionDAO());
        return sessionManager;
    }
```

## 十、RememberMe功能

​	1、添加cookie管理器bean

```java
@Bean
        public SimpleCookie rememberMeCookie(){
            //System.out.println("ShiroConfiguration.rememberMeCookie()");
            //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
            SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
            //<!-- 记住我cookie生效时间30天 ,单位秒;-->
            simpleCookie.setMaxAge(259200);
            return simpleCookie;
        }
    @Bean
    public CookieRememberMeManager rememberMeManager(){
        //System.out.println("ShiroConfiguration.rememberMeManager()");
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(rememberMeCookie());
        //rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
        //cookieRememberMeManager.setCipherKey(Base64.decode("2AvVhdsgUs0FSA3SDFAdag=="));
        return cookieRememberMeManager;
    }

```

​	2、securityManager添加cookie管理器

```java
@Bean
public SecurityManager securityManager() {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(userRealm());
    securityManager.setCacheManager(cacheManager());
    securityManager.setSessionManager(sessionManager());
    //
    securityManager.setRememberMeManager(rememberMeManager());
    //
    return securityManager;
}
```





