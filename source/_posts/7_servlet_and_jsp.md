---
title: Java Web - Servlet and JSP
date: 2019-10-11 22:54:25
tags: [Java, Web, Servlet, JSP]
categories: [Web Backend]
---

之前我们搭建好了自己的 Java Web 开发环境，现在当然要进行实际应用啦。下面就开始我们 Java Web 之旅第二站 - Servlet and JSP。

<!--more-->

## Servlet

Servlet 是基于 Java 的 Web 组件，作为 Web 容器的功能模块，它运行在服务端，处理来自客户端的网络请求，并返回相应数据包。任何 Servlet 组件都实现了 `javax.servlet.Servlet` 接口， Web 容器可以统一管理这些 Servlet 组件。

### Servlet 生命周期

Servlet 整个生命周期由 Web 容器管理，它会经历以下 4 个阶段:

1. 载入阶段 load
  Web 容器部署后，将加载所有 Servlet。
2. 初始化阶段 init
  Web 容器在首次接收到来自客户端请求或者 Web 容器启动时指定(web.xml 中设置), 将调用对应 Servlet 类的 init 方法，实例化 Servlet 对象并初始化数据。
3. 处理请求 service
  Web 容器启用一个线程来处理客户端请求，根据 Request URI 调用对于 Servlet 实例的 service 方法，并传入 HttpServletRequest 和 HttpServletResponse 参数， 默认的 service 方法将请求分发到对应的 doGet 或 doPost 方法中。
4. 销毁阶段 destroy
  当 Web 容器终止 或者 Servlet 重新装载时，Web 容器将调用前面 Servlet 实例的 destroy 方法，销毁实例。最后 servlet 实例将被 JVM 垃圾回收。

![Servlet 生命周期](/images/posts/7_servlet_and_jsp/servlet_life_cycle.png)

### IDEA 搭建 Servlet 项目

使用 IDEA 可以很方便地搭建出 Servlet 项目骨架。让我们开始吧～

首先新建一个项目，选择 JavaEE Web Application 骨架，然后在此基础上集成 Maven 环境，最后会得到如下的项目结构:

```
├── pom.xml
├── servlet_demo.iml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       └── java
└── web
    ├── index.jsp
    └── WEB-INF
        └── web.xml
```

下面描述下各文件的作用:

1. pom.xml: Maven 的配置文件，用于管理项目依赖等
2. servlet_demo.iml: IDEA 的项目设置文件，保存模块路径，依赖关系和其他设置等
3. src: 保存功能或测试代码的目录
4. web: 保存前端页面或资源的目录
5. web.xml: Java Web 项目的配置文件，用于配置 Servlet, 路由映射等

在开始编写代码前，我们需要先引入 servlet-api.jar 到项目中。打开 pom.xml 并按下 `ALT` + `Insert`, 在弹窗中选择 `Dependency`, 搜索 `servlet` 并选中相应的依赖包，最后在 Maven 面板中选择同步依赖即可<sup>[[2]](#work_with_maven_dependencies)</sup>。

### First Servlet

准备工作已经做好了，现在让我们着手编写第一个 Servlet 吧。

首先在 web 目录下添加一个 HTML 页面，它有个登录表单提交用户输入的用户名和密码:

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
    <form action="login" method="post">
        Username: <input type="text" name="username">
        Password: <input type="password" name="password">
        <input type="submit" value="Login">
    </form>
</body>
</html>
```

然后在 src/main/java 目录下创建一个 Servlet Java 文件:

```java
// LoginServlet.java
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class LoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        PrintWriter writer = resp.getWriter();

        writer.println("loging in...");
        if (username.equals("admin") && password.equals("123")) {
            writer.println("authorized");
        } else {
            writer.println("failed");
        }

        resp.setContentType("text/html");
        writer.close();
    }
}
```

前面提到一个 Servlet 类需要实现 `javax.servlet.Servlet` 接口，上面定义的 `LoginServlet` 继承的 `HttpServlet` 类就实现了 `javax.servlet.Servlet`, 此外实现了此接口的还有 `GenericServlet` 类。而 `GenericServlet` 比起 `HttpServlet`，它没有派发 service 方法，需要自己实现。

这里的 `LoginServlet` 获取到请求中的 `username` 和 `password` 参数，并对其验证，输出验证结果。

别忘了还需要映射 servlet 到路由上哦，更新 web.xml 如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <servlet>
        <servlet-name>LoginServlet</servlet-name>
        <servlet-class>LoginServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>LoginServlet</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>
</web-app>
```

其中 welcome-file 就是网站首页。

最后点击启动 Tomcat 运行我们的站点吧～(别忘了，运行前配置下 Tomcat)

## JSP

JSP - Java Server Pages 是一种特殊的 Servlet, 主要用于处理 Web 视图部分。它允许在 HTML, XML 中嵌入 Java 代码，调用 Java API。

### JSP 生命周期

JSP 有着类似 Servlet 一样的生命周期，区别在于 JSP 比起 Servlet 多了编译阶段。 JSP 会经历以下 4 个阶段:

1. 编译阶段
  当客户端请求 JSP 页面时， Web 容器会检查此 JSP，若 JSP 未被编译或编译后修改， Web 容器将解析此 JSP，并将其转换 Servlet 再编译。
2. 初始化阶段
  Web 容器编译完 JSP 后，将会调用 jspInit 方法初始化 JSP。
3. 处理请求
  Web 容器调用 _jspService 方法来处理每个请求， _jspService 接收了 HttpServletRequest 和 HttpServletResponse 两个参数。
4. 销毁阶段
  Web 容器终止或 JSP 被重新编译时，Web 容器将调用 jspDestroy 方法来销毁之前的 JSP。

### JSP 语法

JSP 在 XML 中嵌入 Java 代码，它也有着自己特殊的语法，下面我们简单介绍下:

1. 基本格式: `<% 代码片段 %>`
2. 声明语法: `<%! int c = 1; %>`
3. 插值语法: `<%= (new java.util.Date()).toLocaleString() %>`
4. 指令语法: `<%@ page attribute="value" %>`, 用于描述页面属性等
5. 行为语法: `<% jsp:include page="value" %>`

### JSP 隐含对象

JSP 上下文中存在一些隐含对象，我们用它们控制 JSP 行为。

1. request: HttpServletRequest 实例
2. response: HttpServletResponse 实例
3. out:	PrintWriter 实例，用于把结果输出至网页上
4. session:	HttpSession 实例
5. application: ServletContext 实例，与应用上下文有关
6. config: ServletConfig 实例
7. pageContext: PageContext 实例，提供对 JSP 页面所有对象以及命名空间的访问
8. page: 类似于 Java 类中的 this 关键字
9. Exception: Exception 对象，代表发生错误的 JSP 页面中对应的异常对象

### JSP 编码问题

如果我们要在页面正常显示中文，我们需要在 JSP 文件头部添加以下代码:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```

### First JSP

我们已经了解 JSP 的基本概念了，现在就编写第一个 JSP 吧。

首先在前面的 index.html 中增加下面的表单，接收输入的用户名提交后跳转到 JSP 页面:

```html
<!-- index.html -->
...
    <form action="welcome.jsp">
        Username: <input type="text" name="username">
        <input type="submit" value="Welcome">
    </form>
</body>
</html>
```

在 JSP 页面中，我们将接收到的用户名输出页面中:

```jsp
<%-- welcome.jsp --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    Hi <%= request.getParameter("username") %>, Welcome to JSP
</body>
</html>
```

## 参考

1. [Servlet and JSP Tutorial- How to Build Web Applications in Java?](https://www.edureka.co/blog/servlet-and-jsp-tutorial/)
2. <a name='work_with_maven_dependencies'></a>[Work with Maven dependencies](https://www.jetbrains.com/help/idea/work-with-maven-dependencies.html)
3. [JSP 生命周期](https://www.runoob.com/jsp/jsp-life-cycle.html)
4. [JSP 语法](https://www.runoob.com/jsp/jsp-syntax.html)
5. [JSP中两种include的区别](https://www.cnblogs.com/lazycoding/archive/2011/04/04/two_include.html)