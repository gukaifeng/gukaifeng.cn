---
title: Java Servlet
mathjax: false
date: 2020-03-18 11:20:29
updated: 2020-03-18 11:20:29
tags: [Java]
categories: [网络编程]
toc: true
---



## 1. 什么是Servlet？

Servlet（Server Applet），全称Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。

Servlet运行于支持Java的应用服务器中。从实现上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。



## 2. Servlet的使用

1. 创建普通的 java 类并继承 HttpServlet；
2. Override 方法 service()；
3. 在 service() 方法中书写逻辑代码；
4. 在 web/WEB-INF/web.xml 中配置 servlet。 

<!--more-->

## 3. Servlet的运行流程

浏览器发送请求到服务器，服务器根据请求 URL 地址中的 URI 信息在 webapps 目录下找到对应的项目文件夹，然后在 web.xml 中检索对应的 servlet，找到后调用并执行 Servlet。



## 4. Servlet的生命周期

* web.xml 在服务器启动时调入内存。

* servlet 在第一次调用时调入内存，完成初始化。服务器关闭时调出内存。

    如果在 web.xml 中配置了 `load-on-startup` 则声明周期从服务器启动到服务器关闭。`<load-on-startup>2</load-on-startup>` 表示这个 servlet 第二个加载。

注：

1. Servlet中的 init() 是对 servlet 初始化的一个方法，会在 servlet 第一次加载时执行；
2. Servlet中的 destroy() 是在 servlet 被销毁时执行，也就是服务器关闭时。



## 5. doGet() 和 doPost() 方法

* service() 方法：可以处理 get/post。如果 servlet 中有 service() 方法，优先调用 service() 方法，不再调用 doGet() 或 doPost()。
* doGet() 方法：只处理 get 请求。
* doPost() 方法：只处理 post 请求。



注：如果在重写的 service() 方法中调用了父类的 service() 方法 `super.service(req, resp)`，则 service 方法处理完后，会再次根据请求方式相应地调用 doGet() 或 doPost() 方法。如果没有 doGet() 或 doPost() 方法，会报 405 错误。所以，一般情况下我们不在重写的 service() 中调用父类的 service() 方法，避免出现405错误。



## 6. Servlet常见错误

* 404：资源未找到

    原因1：在请求地址中的 servlet 的别名书写错误。

    原因2：虚拟项目名称拼写错误。

* 500：内部服务器错误

    原因1： web.xml 中的类全限定路径未找到。

    原因2：除 0 等。

* 405：请求的方式不支持

    原因1：请求方式和 servlet 中的方法不匹配所造成的。尽量使用 service() 方法处理请求且不调用父类 service() 方法。



## 7. request 对象

作用：request 对象由 tomcat 服务器创建，存储了当前请求的所有请求信息，并作为实参传递给处理请求的 servlet 中的 service() 方法。

使用：获取请求行、请求头、获取用户数据。

```java
// 请求行数据
req.getMethod(); // 获取请求方法，返回 String。如 "GET"，"POST"
req.getRequsetURL(); // 获取请求的 URL，返回 String。如 "http://localhost:8080/JavaEE/MyServlet"
req.getRequestURI(); // 获取请求的 URI，返回 String。如 "/JavaEE/MyServlet"
req.getScheme(); // 获取请求的协议，返回 String。如 "http"
```

```java
// 获取指定的请求头数据
req.getHeader(""); // 参数为 key，返回值为 value，均为 String 类型
// 获取全部的请求头数据键的枚举
Enumeration e = req.getHeaderNames();
while(e.hasMoreElements()) // 打印
    System.out.println(e.nextElement());
```

```java
// 获取用户数据（重点）
String name = req.getParameter("username");
String psd = req.getParameter("userpassword");
// 注: req.getParameter() 不能获取同键不同值的数据，如复选框
// 复选框数据应用下面的两种方法获取
String[] fav = req.getParameterValues("fav"); // 方法1，返回 String[]
req.getParameterNames("") // 方法2，用法与 req.getHeaderNames() 类似，枚举
```





## 8. response对象



```java
// 设置响应编码格式，两种方法等价
setHeader("content-type", "text/html;charset=utf-8");
setContentType("text/html;charset=utf-8");

// 设置响应
setHeader(String name, String value); // 添加响应信息，同键覆盖
addHeader(String name, String value); // 添加响应信息，同键不覆盖，用 setHeader 重设会覆盖全部的值

// 设置响应状态码
sendError(404, "bu xiang gei ni kan");

// 设置相应实体
resp.getWrite().write(String str);
```

注：处理请求操作应该还用到数据库（MVC思想）。





## 9. 登录页面服务器编写步骤

登录页面服务器编写步骤，使用 MVC 思想完成。

1. 创建数据库

2. 创建登录页面

    创建 Servlet 进行登录页面请求处理

3. 点击登录完成登录操作

    浏览器发送请求到服务器（用户信息+其他数据）

    服务器调用对应的 servlet 进行处理

    ​	设置相应编码格式

    ​	获取请求信息

    ​	处理请求信息

    ​	响应处理结果

4. 在 Servlet 中完成用户登录校验（需连接数据库）



## 10. Servlet 流程总结

1. 浏览器发起请求到服务器；
2. 服务器接收浏览器请求，进行解析，创建 request 对象存储请求数据；
3. 服务器调用对应的 Servlet 进行处理，并将 request 对象作为实参传递给 Servlet 方法；
4. Servlet 的方法处理请求。
    1. 设置请求编码格式；
    2. 设置响应编码格式；
    3. 获取请求信息；
    4. 处理请求信息；
        1. 直接响应
        2. 请求转发
        3. 重定向
    5. 响应处理结果。



* 数据流转流程

    浏览器 -----> 服务器 -----> 数据库

    浏览器 <----- 服务器 <----- 数据库





## 11. 解决请求乱码

1. 使用 String 类将获取的数据重新编码（重要数据建议使用此方法不会出错）

    ```JAVA
    // uname 为待转码 String 字符串
    // 浏览器默认编码格式为 iso8859-1
    uname = new String(uname.getBytes("iso8859-1"), "utf-8");
    ```

2. GET 方法（两步）

    1. 在 service() 函数开头写上 `req.setCharacterEncoding("utf-8")`;
    2. 在 tomcat/conf/server.xml 文件中，在 Connector 标签中增加属性 `useBodyEncodingForURI=true`。

3. POST 方法

    在 service() 函数开头写上 `req.setCharacterEncoding("utf-8")`;





## 12. 请求转发

实现多个 servlet 联动操作处理请求，这样避免代码冗余，让 servlet 职责更加明确。

`req.getRequestDispather("要转发的地址（相对路径）").forward(req, resp);`

特点：一次请求，**浏览器地址栏信息不改变**。

注意：请求转发后直接 return 结束即可，继续处理无意义。

请求转发中的数据流转（作用于：一次请求内）：

`req.setAttribute(Object name, Object value);`

`req.getAttribute(Object obj);`



## 13. 重定向

如果当前请求Servlet 无法进行处理，或如果使用请求转发会造成表单数据重复提交，怎么办？使用重定向解决。

`resp.sendRedirect("location");` 如果是本地资源 location 是相对路径（一般从项目根目录开始写，如 /javaee/main），如果是网络资源，是网络地址。

特点：两次请求，浏览器地址栏信息改变，两个 request 对象。

如果请求中有表单数据，而数据又比较重要，不能重复提交，建议重定向；

如果请求被 servlet 接收后，无法处理，建议重定向。

重定向中，若需要实现多个请求之间的数据共享，需要使用 cookie。



## 14. Cookie

Cookie用于解决不同发送请求之间的数据共享问题。

### 14.1. Cookie 的创建和存储

1. 创建 Cookie 对象，一个 Cookie 存储一个键值对。

    ```java
    Cookie c = new Cookie(String name, String value);
    ```

2. 设置有效期。

    ```java
    c.setMaxAge(int expiry); // 单位 s。不设置为临时存储，设置后为定时存储
    ```

3. 设置有效路径

    ```java
    c.setPath(String uri);
    ```

4. 添加 Cookie 信息到响应 response。

    ```java
    resp.addCookie(c);
    ```



### 14.2. Cookie 的获取



1. 获取 Cookie 信息数组

    ```java
    Cookie[] cks = req.getCookies();
    ```

2. 遍历数据获取 Cookie 信息

    ```java
    if(cks != null) {
        for(Cookie c: cks) {
            String name = c.getName();
            String value = c.getValue();
            System.out.println(name + ":" + value);
        }
    }
    ```




### 14.3. Cookie 的特点

1. Cookie是浏览器端的数据存储技术；
2. 存储的数据声明在服务器端；
3. 临时存储：存储在浏览器的运行内存中，浏览器关闭即失效；
4. 定时存储：设置的有效期内存在，存储在客户端的硬盘中。在有效期内符合路径要求的请求都会附带该信息。
5. 默认 Cookie 信息存储好之后，每次请求都会附带，除非设置了有效路径。





## 15. Session

### 15.1. Session的原理

用户第一次访问服务器，服务器会创建一个 Session 对象给此用户，并将该 Session 对象的 JSESSIONID 使用 Cookie 技术存储到浏览器中，保证用户的其他请求能够获取到同一个 Session 对象，也保证了不同请求能够获取到共享的数据。

### 15.2. Session的特点

1. 存储在服务器端；
2. 服务器进行创建；
3. 依赖 Cookie 技术；
4. 一次会话；
5. 默认存储时间是 30 分钟。

### 15.3. Session的作用

解决了一个用户不同请求处理的数据共享问题。



### 15.4. Session的使用

1. 创建/获取  Session 对象

    ```java
    // 如果请求中有 Session 标识符 JSESSIONIN，则返回其对应的 Session 对象；
    // 如果请求中没有，则创建新的 Session 对象，并将其 JSESSIONID 作为 Cookie 存储到浏览器中。
    // 如果 Session 对象失效了，也会重新创建一个 Session 对象并将其 JSESSIONID 作为 Cookie 存储到浏览器中。
    // 注：JSESSIONID 存储在了 Cookie 的临时存储空间中，浏览器关闭即失效。
    HttpSession hs = req.getSession();
    ```

2. 设置 Session 的存储时间

    ```java
    hs.setMaxInactiveInterval(int seconds); // 设置存储时间(秒)
    // 注：seconds 内如果再次访问 Session，重新计时。没有使用则销毁。
    ```

3. Session 强制失效

    ```Java
    hs.invalidate();
    ```

4. 存储数据

    ```java
    hs.setAttribute(String name, Object obj);
    ```

5. 获取数据

    ```java
    hs.getAttribute(String name); // 返回 Object，要强制类型转换
    ```

6. 使用时机

    一般用户在登录 web 项目时会将用户的个人信息存储到 Session 中，供该用户的其他请求使用。

7. 作用域

    一次会话。在 JSESSIONID 和 Session 对象不失效的情况下为整个项目。

8. Session 的失效处理

    如果获取的 Session 中对象为 null，重定向到登录界面。





Session 解决了一个用户的不同请求的数据共享问题，只要在 JSESSIONID 不失效和 Session 对象不失效的情况下，用户的任意请求在处理时都能获取到同一个 Session 对象。



## 16. ServletContext

ServletContext用于解决不同用户间的数据共享问题。

ServletContext 对象由服务器进行创建，一个项目只有一个对象ServletContext对象。不管在项目的任意位置进行获取得到的都是同一个对象，那么不同用户发起的请求获取到的也就是同一个对象，该对象由用户共同拥有。

ServletContext生命周期：服务器启动到服务器关闭。

ServletContext作用域：项目内。

### 16.1. 获取ServletContext对象



```java
// 第一种方式（常用）
ServletContext sc1 = this.getServletContext();
// 第二种方式
ServletContext sc2 = this.getServletConfig().getServletContext();
// 第三种方式（常用）
ServletContext sc3 = req.getSession().getServletContext();
```

### 16.2. 使用ServletContext对象完成数据共享

```java
// 存储数据
sc.setAttribute(String name, Object value);
// 获取数据
sc.getAttribute(String name); // 返回 Object
```

### 16.3. ServletContext其他功能



#### 16.3.1. 获取 web.xml 的全局配置数据



```xml
<!-- 在 web.xml 中配置全局数据 -->
<!-- 一组 context-param 标签只能配置一个数据 -->
<!-- 作用：将静态数据和代码解耦 -->
<context-param>
  <param-name>name</param-name>
  <param-value>gukaifeng</param-value>
</context-param>
```

```java
sc.getInitParameter(String name); // 根据配置名获取配置数据，返回 String
sc.getInitParemeterNames(); // 返回键名的枚举
```



#### 16.3.2. 获取项目根目录下的绝对路径

```java
String path = sc.getRealPath("文件在项目的相对路径");
```



#### 16.3.3. 获取项目根目录下资源的流对象

```java
InputStream is = sc.getResourceAsStream("文件在项目的相对路径");
// 注：此方式只能获取项目根目录下的资源流对象，class 文件的流对象需要使用类加载器获取
```





## 17. ServletConfig 对象

ServletConfig 对象是 Servlet 的专属配置对象，每个 Servlet 都单独拥有一个 ServletConfig 对象，用来获取 web.xml 中的配置信息。

使用：

1. 获取 ServletConfig 对象

    ```java
    ServletConfig sc = this.getServletConfig();
    ```

2. 获取 web.xml 中的配置数据

    1. 在 web.xml 中配置 servlet 独有的属性

        在 `<servlet> </servlet>`中添加 `<initparam></intiparam>` 标签。

        ```XML
        <servlet>
            <servlet-name>login</servlet-name>
            <servlet-class>com.servlet.ServletLogin</servlet-class>
            <init-param>
                <param-name>config</param-name>
                <param-value>utf-8</param-value>
            </init-param>
        </servlet>
        <servlet-mapping>
            <servlet-name>login</servlet-name>
            <url-pattern>/login</url-pattern>
        </servlet-mapping>
        ```

    2. 获取数据

        ```java
        ServletConfig sc = this.getServletConfig();
        String code = sc.getInitParameter("config");
        ```





## 18. web.xml 文件

* 作用：存储项目相关的配置信息，保护 Servlet。解耦一些数据对程序的依赖。

* 使用位置：每个 web 项目中，Tomcat 服务器中（在服务器目录 conf 目录中）。

    * web 项目下的 web.xml 文件为局部配置，针对本项目的配置。
    * Tomcat 下的 web.xml 文件为全局配置，配置公共信息。

* 内容（核心组件）

    1. 全局上下文配置（全局配置参数）；
    2. Servlet 配置；
    3. 过滤器配置；
    4. 监听器配置。

* 加载顺序

    Web 容器会按 ServletContext -> Context-param -> listener -> filter -> servlet 这个顺序加载组件。这些元素在 web.xml 文件中的任意位置。