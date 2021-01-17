# Shiro

## 导包

```xml
<!--shiro subject SecurityManager realm-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.7.0</version>
</dependency>
        <!--thymeleaf-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!--thymeleaf-shiro-->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>

```

## 配置 两个类

```java
import com.mycode.pojo.model.User;
import com.mycode.service.UserService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthenticatingRealm;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.slf4j.Logger;

import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import sun.rmi.runtime.Log;

import java.util.List;

public class UserRealm extends AuthorizingRealm {

    private static final Logger logger = LoggerFactory.getLogger(UserRealm.class);

    @Autowired
    private UserService userService;

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        logger.info("授权了");
        return null;
    }
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        logger.info("认证了");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        List<User> userList = userService.findUserByName(token.getUsername());
        if (userList==null){
            return null;//用户名错误
        }
        return new SimpleAuthenticationInfo("", userList.get(0).getPassword(), "");
    }
}
```

```java
package com.mycode.config;


import com.mycode.pojo.model.User;
import com.mycode.service.UserService;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthenticatingRealm;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.subject.Subject;
import org.slf4j.Logger;

import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import sun.rmi.runtime.Log;

import java.util.List;

public class UserRealm extends AuthorizingRealm {

    private static final Logger logger = LoggerFactory.getLogger(UserRealm.class);

    @Autowired
    private UserService userService;

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        logger.info("授权了");
        //将subject的Principal里面到的userList取出来，进行授权
        Subject subject = SecurityUtils.getSubject();
        User user = (User) subject.getPrincipal();
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.addStringPermission(user.getPermission());
        logger.info("授权完毕"+simpleAuthorizationInfo.toString());
        return simpleAuthorizationInfo;
    }
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        logger.info("认证了");
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        List<User> userList = userService.findUserByName(token.getUsername());
        if (userList==null){
            return null;//用户名错误
        }
        //把查询到的用户放入principal为方便之后在上面那个方法进行授权
        return new SimpleAuthenticationInfo(userList.get(0), userList.get(0).getPassword(), "");
    }
}

```

![image-20201221185000063](C:\Users\DMQi\AppData\Roaming\Typora\typora-user-images\image-20201221185000063.png)



	## 页面相关

```java
@Controller
public class UserController {

    private static final Logger logger = LoggerFactory.getLogger(UserController.class);

    @GetMapping({"/", "/index"})
    public String index(Model model){
        model.addAttribute("index", "helloword");
        return "index";
    }

    @GetMapping("/user/add")
    public String add(){
        return "user/add";
    }

    @GetMapping("/user/update")
    public String update(){
        return "user/update";
    }

    @RequestMapping("/toLogin")
    public String toLogin(){
        return "login";
    }

    @PostMapping("/login")
    public String login(String username, String password, Model model){
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);

        try {
            subject.login(token);
            return "/index";
        } catch (UnknownAccountException e) {
            logger.error(e.getMessage());
            model.addAttribute("msg", "用户名错误");
            return "/login";
        }catch (IncorrectCredentialsException e){
            logger.error(e.getMessage());
            model.addAttribute("msg", "密码错误");
            return "/login";
        }
    }

}
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml"
      xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>

<div th:text="${index}"></div>
<p shiro:notAuthenticated="">
   <a th:href="@{/toLogin}">登录</a>
</p>
<div>
    <a shiro:hasPermission="user:user" th:href="@{/user/list}">获取用户信息</a>
    <a shiro:hasPermission="user:add" th:href="@{/user/add}">新增用户</a>
    <a shiro:hasPermission="user:delete" th:href="@{/user/delete}">删除用户</a>
    <a shiro:hasPermission="user:update" th:href="@{/user/update}">删除用户</a>
</div>

</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>

<p th:text="${msg}" style="color:red"></p>

<form th:action="@{/login}" method="post">
    <p>用户名：<input type="text" name="username"></p>
    <p>密码：<input type="password" name="password"></p>
    <p><input type="submit"></p>
</form>

</body>
</html>
```