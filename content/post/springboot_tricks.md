---
title: "[Springboot]操作记录"
date: 2019-02-26T15:40:15+08:00
draft: false 
---
[TOC]

## 一、springboot获取http参数的方法

### 1.将参数写做变量名的形参

GET POST 参数为空时变量值为null

```java
  /**
    * 1.直接把表单的参数写在Controller相应的方法的形参中
    * @param username
    * @param password
    * @return
    */
    @RequestMapping("/login")
    public String login(String username,String password) {
        System.out.println("username is:"+username);
        System.out.println("password is:"+password);
        ... ...
    }
```

### 2.使用注解@RequestParam绑定参数

GET POST

不传值时将会报错，可以通过设置属性required=false解决,例如: @RequestParam(value="username", required=false)

```json
{
    "timestamp":"2019-02-26T07:56:44.039+0000",
    "status":400,
    "error":"Bad Request",
    "message":"Required String parameter 'sTime' is not present",
    "path":"/login"
}
```

```java
/**
  * @param username
  * @param password
  * @return
  */
  @RequestMapping("/login")
  public String login(
      @RequestParam('username') String username,
      @RequestParam('username') String username) {
      System.out.println("username is:"+username);
      System.out.println("password is:"+password);
      ... ...
  }
```

### 3.使用HttpServletRequest接收参数

GET POST

```java
 /**
   * @param request
   * @return
   */
   @RequestMapping("/login")
   public String login(HttpServletRequest request) {
       String username = request.getParameter("username");
       String password = request.getParameter("password");
       System.out.println("username is:"+username);
       System.out.println("password is:"+password);
       ... ...
    }
```

### 4.通过一个bean接收参数

```java
package hello.model;

public class User {
    
    private String username;
    private String password;
    
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    
}
```

```java
 /**
   * @param user
   * @return
   */
    @RequestMapping("/login")
    public String login(User user) {
        System.out.println("username is:"+user.getUsername());
        System.out.println("password is:"+user.getPassword());
        ... ...
    }
```

### 5.使用@ModelAttribute注解获取POST请求的FORM表单数据

POST

```html
<form action ="<%=request.getContextPath()%>/hello/login" method="post"> 
     用户名:<input type="text" name="username"/><br/>
     密码:<input type="password" name="password"/><br/>
     <input type="submit" value="提交"/> 
</form> 
```

```java
 /**
   * @param user
   * @return
   */
   @RequestMapping(value="/login",method=RequestMethod.POST)
   public String addUser5(@ModelAttribute("user") User user) {
       System.out.println("username is:"+user.getUsername());
       System.out.println("password is:"+user.getPassword());
       ... ...
   }

```

### 6.通过@PathVariable获取路径中的参数

GET

```java
 /**
   * 4、通过@PathVariable获取路径中的参数
   * @param username
   * @param password
   * @return
   */
   @RequestMapping(value="/login/{username}/{password}",method=RequestMethod.GET)
   public String addUser4(@PathVariable String username,@PathVariable String password) {
       System.out.println("username is:"+username);
       System.out.println("password is:"+password);
       ... ...
   }
```
## 二、使用Apache2进行端口转发

需要将本地服务进行公网展示的时候，springboot默认配置是使用8080端口进行http服务，但是有的内网穿透工具仅支持80端口映射，于是需要`配置springboot服务使用80端口`或者`使用Apache2进行端口转发`。

简单配置即可，将接受到serverName的请求（同域名）转发给8080端口。

```xml
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so
<VirtualHost *:80>
	ServerName vz2kq4.natappfree.cc
        ProxyPass / http://localhost:8080/
        ProxyPassReverse / http://localhost:8080/
</VirtualHost>
```

需要proxy和proxy_http模块的支持。

## 三、实现分页功能

### 1 return json实现分页功能

### 2 return 前端页面实现分页功能