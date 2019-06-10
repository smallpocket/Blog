---
title: shiro集成无状态JWT
type: tags
tags:
  - null
date: 2018-10-03 15:52:53
categories:
description:
---
>转

# JWT

## 为什么要使用JWT

- 在前后端分离的项目当中,服务器端无法存储会话(session),而是每次请求带上相应的用户名
- 因此我们要实现完全的前后端分离，所以不可能使用session，cookie的方式进行鉴权
- JWT的鉴权,通过一个加密的秘钥来实现鉴权

## JWT的介绍

放弃Cookie,Session,使用JWT进行鉴权，完全实现无状态鉴权。

# 目标

## 效果

访问一个URL:
http://127.0.0.1:8080/hello?username=admin&params1=love&params2=girl&digest=df7f1595bd5682638556072c8ccde5edadcd807a829373d21af38fb1bc707da7

如果digest是正确的话，那么就会返回Hello,Andy,否则会login,error。

## 后台过程

1. 访问该URL
2. 首先进入AccessControlFilter，进行访问控制过滤拦截，如果不满足条件的话，那么直接就返回了，否则接着往下处理。
3. 在AccessControlFilter中我们为委托AuthorizingRealm进行身份的认证。在AuthorizingRealm中的身份验证访问进行客户端消息摘要和服务器端消息摘要的匹配。
4. 如果成功的话，那么就会到Shiro进行进一步的处理，最后到我们的Controller，然后进行返回。

### 对象的需求

1. ShiroConfiguration：在这个类中主要是注入shiro的filterFactoryBean和securityManager等对象。
2. StatelessAccessControlFilter：这个类中实现访问控制过滤，当我们访问url的时候，这个类中的两个方法会进行拦截处理。
3. StatelessAuthorizingRealm：这个类中主要是身份认证，验证信息是否合理，是否有角色和权限信息。
4. StatelessAuthenticationToken：在shiro中有一个我们常用的UsernamePasswordToken，因为我们需要这里需要自定义一些属性值，比如：消息摘要，参数Map。
5. StatelessDefaultSubjectFactory：由于我们编写的是无状态的，每人情况是会创建session对象的，那么我们需要修改createSubject关闭session的创建。
6. HmacSHA256Utils：Java 加密解密之消息摘要算法，对我们的参数信息进行处理。

# 基础配置

## pom

     <!-- spring boot web支持：mvc,aop... -->
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- shiro spring. -->
    <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring</artifactId>
       <version>1.2.2</version>
    </dependency>

    <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
      </dependency>

## contrller

Rest测试
 

    import javax.servlet.http.HttpSession;
    import org.apache.shiro.SecurityUtils;
    import org.apache.shiro.authz.annotation.RequiresRoles;
    import org.apache.shiro.session.Session;
    import org.apache.shiro.subject.Subject;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    public class HelloController {
   
      @RequestMapping("/hello")
      public String hello(String params1,String params2){
        return "hello,Andy,params1="+params1+",params1="+params2;
      }
    }

## 测试

访问[http://127.0.0.1:8080/hello](http://127.0.0.1:8080/hello )

浏览器返回:
hello,Andy,params1=null,params1=null

至此,基础配置已经完成

# 集成shiro的基本配置

## 基本配置

### 添加shiro

spring配置shiro最基本的操作就是注入ShiroFilterFactoryBean和DefaultWebSecurityManager.

因此,新建ShiroConfiguration类

    import org.apache.shiro.mgt.SecurityManager;
    import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
    import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    /**
    * shiro配置类
    * @author Angel --守护天使
    * @version v.0.1
    * @date 2017年2月25日
    */

    @Configuration
    public class ShiroConfiguration {
      
        @Bean
        public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager){
          ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
          factoryBean.setSecurityManager(securityManager);
          return factoryBean;
        }

        /**
        * shiro安全管理器:
        * 主要是身份认证的管理，缓存管理，cookie管理，
        * 所以在实际开发中我们主要是和SecurityManager进行打交道的
        * @return
        */

        @Bean
        public DefaultWebSecurityManager securityManager() {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            return securityManager;
        }
    
    }


## 无状态
首先,restful风格里面,是没有session和cookie等相关东西的,所以在配置当中要关闭这些东西

需要配置以下的几个地方

1. SubjectContext在创建的时候，需要关闭session的创建，这个主要是由DefaultWebSubjectFactory的createSubject进行管理。
2. 需要禁用使用Sessions 作为存储策略的实现，这个主要由securityManager的subjectDao的sessionStorageEvaluator进行管理的。
3. 需要禁用掉会话调度器，这个主要由sessionManager进行管理。

### 配置

我们需要先定义一个StatelessDefaultSubjectFactory类，此类继承于DefaultWebSubjectFactory，我们重写createSubject的方法，通过SubjectContext关闭session的创建

    import org.apache.shiro.subject.Subject;
    import org.apache.shiro.subject.SubjectContext;
    import org.apache.shiro.web.mgt.DefaultWebSubjectFactory;
    /**
    *
    通过调用context.setSessionCreationEnabled(false)表示不创建会话；如果之后调用
    Subject.getSession()将抛出DisabledSessionException异常。
    * @author Angel --守护天使
    * @version v.0.1
    * @date 2017年2月25日
    */

    public class StatelessDefaultSubjectFactory extends DefaultWebSubjectFactory{

        @Override
        public Subject createSubject(SubjectContext context) {
          //不创建session.
          context.setSessionCreationEnabled(false);
        System.out.println("shiro.config.subjectFactory.createSubject.SessionCreationEnabled.false");
          return super.createSubject(context);
        }

    }

- 调整下ShiroConfiguration，首先我们要注入StatelessDefaultSubjectFactory；其次就是将StatelessDefaultSubjectFactory交给DefaultWebSecurityManager进行管理；最后使用securityManager获取到subjectDao禁用session的存储策略
- （注意新加的代码为：Add.2.x）

    import org.apache.shiro.mgt.DefaultSessionStorageEvaluator;
    import org.apache.shiro.mgt.DefaultSubjectDAO;
    import org.apache.shiro.mgt.SecurityManager;
    import org.apache.shiro.session.mgt.DefaultSessionManager;
    import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
    import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
    import org.apache.shiro.web.mgt.DefaultWebSubjectFactory;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    /**
    * shiro配置类.
    * @author Angel --守护天使
    * @version v.0.1
    * @date 2017年2月25日
    */

    @Configuration
    public class ShiroConfiguration {

        @Bean
        public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager){
          ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
          factoryBean.setSecurityManager(securityManager);
          return factoryBean;
        }

        /**
        * shiro安全管理器:
        * 主要是身份认证的管理，缓存管理，cookie管理，
        * 所以在实际开发中我们主要是和SecurityManager进行打交道的
        * @return
        */
        @Bean
        public DefaultWebSecurityManager securityManager() {
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            //Add.2.2
            securityManager.setSubjectFactory(subjectFactory());
            //Add.2.5
            securityManager.setSessionManager(sessionManager());
          
            /*
            * 禁用使用Sessions 作为存储策略的实现，但它没有完全地禁用Sessions
            * 所以需要配合context.setSessionCreationEnabled(false);
            */
            //Add.2.3
            ((DefaultSessionStorageEvaluator)((DefaultSubjectDAO)securityManager.getSubjectDAO()).getSessionStorageEvaluator()).setSessionStorageEnabled(false);
            return securityManager;
        }
      
        /**
        * Add.2.1
        * subject工厂管理器.
        * @return
        */
        @Bean
        public DefaultWebSubjectFactory subjectFactory(){
          StatelessDefaultSubjectFactory subjectFactory = new StatelessDefaultSubjectFactory();
          return subjectFactory;
        }
        /**
        * Add.2.4
        * session管理器：
        * sessionManager通过sessionValidationSchedulerEnabled禁用掉会话调度器，
        * 因为我们禁用掉了会话，所以没必要再定期过期会话了。
        * @return
        */
        @Bean
        public DefaultSessionManager sessionManager(){
          DefaultSessionManager sessionManager = new DefaultSessionManager();
          sessionManager.setSessionValidationSchedulerEnabled(false);
          return sessionManager;
        }
      
    }

成功关闭session等

## 测试

测试原理：如果是无状态的话，那么在调用代码：currentUser.getSession()是会抛出异常的。所以很好测试，直接在HellController中加入如下方法即可测试

        /**
        * 此方法执行的时候，会抛出异常：
        * Session creation has been disabled for the current subject.
        * @param session
        * @return
        */
        @RequestMapping("/hello3")
        public String hello3(){
          Subject currentUser = SecurityUtils.getSubject(); 
          Session session = currentUser.getSession();
          System.out.println(session);
          return"hello3,Andy";
        }

访问:[http://127.0.0.1:8080/hello3](http://127.0.0.1:8080/hello3)

报错:

Session creation has been disabled for the current subject.

恭喜成功,达成目标,无状态的shiro

# 请求控制拦截。

## 准备工作
### 工具类:对参数信息处理——加密解密之消息摘要算法

    package example.shiro.config;

    import java.util.List;
    import java.util.Map;
    import javax.crypto.Mac;
    import javax.crypto.SecretKey;
    import javax.crypto.spec.SecretKeySpec;
    import org.apache.commons.codec.binary.Hex;

    /**
    * @Title : 
    * Created by Hyper on 2018/10/3 16:54
    */
    public class HmacSHA256Utils {

      public static String digest(String key, String content) {
        try {
          Mac mac = Mac.getInstance("HmacSHA256");
          byte[] secretByte = key.getBytes("utf-8");
          byte[] dataBytes = content.getBytes("utf-8");
          SecretKey secret = new SecretKeySpec(secretByte, "HMACSHA256");
          mac.init(secret);
          byte[] doFinal = mac.doFinal(dataBytes);
          byte[] hexB = new Hex().encode(doFinal);
          return new String(hexB, "utf-8");
        } catch (Exception e) {
          throw new RuntimeException(e);
        }
      }

      @SuppressWarnings("unchecked")
      public static String digest(String key, Map<String, ?> map) {
        StringBuilder s = new StringBuilder();
        for (Object values : map.values()) {
          if (values instanceof String[]) {
            for (String value : (String[]) values) {
              s.append(value);
            }
          } else if (values instanceof List) {
            for (String value : (List<String>) values) {
              s.append(value);
            }
          } else {
            s.append(values);
          }
        }
        return digest(key, s.toString());
      }
    }

### 保存我们的身份信息,用户名，客户端传入的消息摘要，还有客户端传入的参数map等

    import org.apache.shiro.authc.AuthenticationToken;

    import java.util.Map;

    /**
    * 用于授权的Token对象
    * 用户身份即用户名；
    * 凭证即客户端传入的消息摘要。
    *
    * @Time :Created by Hyper on 2018/10/3 16:55
    */

    public class StatelessAuthenticationToken implements AuthenticationToken {

        private static final long serialVersionUID = 1L;

        //用户身份即用户名；
        private String username;

        //参数.
        private Map<String, ?> params;

        //凭证即客户端传入的消息摘要。
        private String clientDigest;

        public StatelessAuthenticationToken() {
        }

        public StatelessAuthenticationToken(String username, Map<String, ?> params, String clientDigest) {
            super();
            this.username = username;
            this.params = params;
            this.clientDigest = clientDigest;
        }

        public StatelessAuthenticationToken(String username, String clientDigest) {
            super();
            this.username = username;
            this.clientDigest = clientDigest;
        }

        @Override
        public Object getPrincipal() {
            return username;
        }

        @Override
        public Object getCredentials() {
            return clientDigest;
        }

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public Map<String, ?> getParams() {
            return params;
        }

        public void setParams(Map<String, ?> params) {
            this.params = params;
        }

        public String getClientDigest() {
            return clientDigest;
        }

        public void setClientDigest(String clientDigest) {
            this.clientDigest = clientDigest;
        }

    }

## 核心

实现访问控制过滤器，拦截我们的请求，我们主要是处理onAccessDenied（）方法，接收到请求的参数，组装成StatelessAuthenticationToken，然后委托为Realm进行处理

### 思路

1. 客户端生成的消息摘要；
2. 客户端传入的用户身份；
3. 客户端请求的参数列表；
4. 生成无状态Token
5. 委托给Realm进行登录

### 实现

#### 访问控制过滤器

    import org.apache.shiro.web.filter.AccessControlFilter;

    import javax.servlet.ServletRequest;
    import javax.servlet.ServletResponse;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    import java.util.HashMap;
    import java.util.Map;

    /**
    * @Time : Created by Hyper on 2018/10/3 16:55
    */
    public class StatelessAccessControlFilter extends AccessControlFilter {

        /**
        * 先执行：isAccessAllowed 再执行onAccessDenied
        * <p>
        * isAccessAllowed：表示是否允许访问；mappedValue就是[urls]配置中拦截器参数部分，
        * 如果允许访问返回true，否则false；
        * <p>
        * 如果返回true的话，就直接返回交给下一个filter进行处理。
        * 如果返回false的话，回往下执行onAccessDenied
        */
        @Override
        protected boolean isAccessAllowed(ServletRequest request, ServletResponse response,
                                          Object mappedValue)
                throws Exception {
            System.out.println("StatelessAuthcFilter.isAccessAllowed()");
            return false;
        }

        /**
        * onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回true表示需要继续处理；
        * 如果返回false表示该拦截器实例已经处理了，将直接返回即可。
        */
        @Override
        protected boolean onAccessDenied(ServletRequest request, ServletResponse response)
                throws Exception {
            System.out.println("StatelessAuthcFilter.onAccessDenied()");
            //1、客户端生成的消息摘要
            String clientDigest = request.getParameter("digest");
            //2、客户端传入的用户身份
            String username = request.getParameter("username");
            //3、客户端请求的参数列表
            Map<String, String[]> params = new HashMap<String, String[]>(request.getParameterMap());
            //为什么要移除呢？签名或者消息摘要算法的时候不能包含digest.
            params.remove("digest");
            //4、生成无状态Token
            StatelessAuthenticationToken token = new StatelessAuthenticationToken(username, params,
                    clientDigest);
    //     UsernamePasswordToken token = new UsernamePasswordToken(username,clientDigest);
            try {
                //5、委托给Realm进行登录
                getSubject(request, response).login(token);
            } catch (Exception e) {
                e.printStackTrace();
                //6、登录失败
                onLoginFail(response);
                //就直接返回给请求者.
                return false;
            }
            return true;
        }

        /**
        * 登录失败时默认返回401 状态码
        */
        private void onLoginFail(ServletResponse response) throws IOException {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            httpResponse.getWriter().write("login error");
        }
    }

#### Realm进行登录

请求就到了我们的Realm代码，所以我们需要编写一个Realm来进行身份验证下，这里的核心就是获取到AccessControlFilter传递过来的StatelessAuthenticationToken中的参数进行消息摘要，然后生成对象SimpleAuthenticationInfo交给Shiro进行比对

    import org.apache.shiro.authc.AuthenticationException;
    import org.apache.shiro.authc.AuthenticationInfo;
    import org.apache.shiro.authc.AuthenticationToken;
    import org.apache.shiro.authc.SimpleAuthenticationInfo;
    import org.apache.shiro.authz.AuthorizationInfo;
    import org.apache.shiro.authz.SimpleAuthorizationInfo;
    import org.apache.shiro.realm.AuthorizingRealm;
    import org.apache.shiro.subject.PrincipalCollection;


    /**
    * @Time :Created by Hyper on 2018/10/3 16:56
    */
    public class StatelessAuthorizingRealm extends AuthorizingRealm {

        /**
        * 仅支持StatelessToken 类型的Token，
        * 那么如果在StatelessAuthcFilter类中返回的是UsernamePasswordToken，那么将会报如下错误信息：
        * Please ensure that the appropriate Realm implementation is configured correctly or
        * that the realm accepts AuthenticationTokens of this type.StatelessAuthcFilter.isAccessAllowed()
        */
        @Override
        public boolean supports(AuthenticationToken token) {
            return token instanceof StatelessAuthenticationToken;
        }

        /**
        * 身份验证
        */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token)
                throws AuthenticationException {
            System.out.println("StatelessRealm.doGetAuthenticationInfo()");
            StatelessAuthenticationToken statelessToken = (StatelessAuthenticationToken) token;
            //不能为null,否则会报错的.
            String username = (String) statelessToken.getPrincipal();
            //根据用户名获取密钥（和客户端的一样）
            String key = getKey(username);
            //在服务器端生成客户端参数消息摘要
            String serverDigest = HmacSHA256Utils.digest(key, statelessToken.getParams());
            System.out.println(serverDigest + "," + statelessToken.getCredentials());
            //然后进行客户端消息摘要和服务器端消息摘要的匹配
            SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
                    username,
                    serverDigest,
                    getName());
            return authenticationInfo;
        }


        /**
        * 授权
        */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
            System.out.println("StatelessRealm.doGetAuthorizationInfo()");
            //根据用户名查找角色，请根据需求实现
            String username = (String) principals.getPrimaryPrincipal();
            SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
            //这里模拟admin账号才有role的权限.
            if ("admin".equals(username)) {
                authorizationInfo.addRole("admin");
            }
            return authorizationInfo;
        }

        /**
        * 得到密钥，此处硬编码一个.
        *
        * @param username
        * @return
        */
        private String getKey(String username) {
            return "andy123456";
        }

    }

#### 配置到ShiroConfiguration(Add.4.x)

### 测试

http://127.0.0.1:8080/hello?username=admin&params1=love&params2=girl&digest=df7f1595bd5682638556072c8ccde5edadcd807a829373d21af38fb1bc707da7

digest是根据参数生成的,更换值则会login error

## 权限控制篇

### 配置ShiroConfiguration

在shiroConfiguration中加入【开启shiro aop注解支持】和【自动代理所有的advisor】

具体代码为:Add.5.x

### 在方法(controller)中加入注解@RequiresRoles("admin")

      @RequestMapping("/hello4")
      @RequiresRoles("admin")
    // @RequiresPermissions("userInfo:add")//权限管理;要求拥有userInfo:add权限才可以执行
      public String hello4() {
        return "hello4,Andy";
      }

## 测试

### 正确的地址

http://127.0.0.1:8080/hello4?username=admin&params1=love&params2=girl&digest=df7f1595bd5682638556072c8ccde5edadcd807a829373d21af38fb1bc707da7

### 错误的

http://127.0.0.1:8080/hello4?username=zs&params1=love&params2=girl&digest=df7f1595bd5682638556072c8ccde5edadcd807a829373d21af38fb1bc707da7

报错.UnauthorizedException: Subject does not have role [admin]

# 源码

 [个人源码实战练习](https://github.com/smallpocket/spring-boot-demo/spring-boot-shiro)

# 参考 #
1. [Spring Boot之Shiro无状态（1）【从零开始学Spring Boot】](http://412887952-qq-com.iteye.com/blog/2359084)
2. [Spring Boot之Shiro无状态（2）【从零开始学Spring Boot】](http://412887952-qq-com.iteye.com/blog/2359097)
3. [Spring Boot之Shiro无状态（3）【从零开始学Spring Boot】](http://412887952-qq-com.iteye.com/blog/2359098)
4. [Spring Boot之Shiro无状态（4）【从零开始学Spring Boot】](http://412887952-qq-com.iteye.com/blog/2359099)