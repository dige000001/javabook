## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

Session和cookies的关系：<https://www.zhihu.com/question/19786827>

Setsession.jsp

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>

<%
   session.setAttribute("name", "teemo");
%>

<a href="getSession.jsp">跳转到获取session的页面</a>
```

session对象保存数据的方式，有点像Map的键值对(key-value)\
"name"是键，"teemo" 是值

getSession.jsp

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>
 
<%
    String name = (String)session.getAttribute("name");
%>
 
session中的name: <%=name%>
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eada5ad22e0b4c82ae624fe00315ffee~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/563188b6a65247f38e4a5a71147c863d~tplv-k3u1fbpfcp-zoom-1.image)




浏览器没开启cookies时怎么启用sessionid

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>

<%
   session.setAttribute("name", "teemo");
%>

<a href="<%=response.encodeURL("getSession.jsp")%>">跳转到获取session的页面</a>
```

response.encodeURL方法会把getSession.jsp这个url转换为

getSession.jsp;jsessionid=22424AEA86ADBE89F335EEB649D997A8

这样服务器就可以获取到sessionid了

****

**encodeUrl:**

Java Servlet API 中引用 Session 机制来追踪客户的状态。Servlet API 中定义了 javax.servlet.http.HttpSession 接口，Servlet 容器必须实现这个接口。当一个 Session 开始时，Servlet 容器将创建一个 HttpSession 对象，Servlet 容器为 HttpSession 分配一个唯一标识符，称为 Session ID。Servlet 容器将 Session ID 作为 Cookie 保存在客户的浏览器中。每次客户发出 HTTP 请求时，Servlet 容器可以从 HttpRequest 对象中读取 Session ID，然后根据 Session ID 找到相应的 HttpSession 对象，从而获取客户的状态信息。 当客户端浏览器中禁止 Cookie，Servlet 容器无法从客户端浏览器中取得作为 Cookie 的 Session ID，也就无法跟踪客户状态。 

本质：Java Servlet API 中提出了跟踪 Session 的另一种机制，如果客户端浏览器不支持 Cookie，Servlet 容器可以重写客户请求的 URL，把 Session ID 添加到 URL 信息中。 

HttpServletResponse 接口提供了重写 URL 的方法

：**public java.lang.String encodeURL(java.lang.String url)**

该方法的实现机制为： 

● 先判断当前的 Web 组件是否启用 Session，如果没有启用 Session，直接返回参数 url。 

● 再判断客户端浏览器是否支持 Cookie，如果支持 Cookie，直接返回参数 url；如果不支持 Cookie，就在参数 url 中加入 Session ID 信息，然后返回修改后的 url。 

我们可以对网页中的链接稍作修改，解决以上问题： 

修改前： 

<a href=“maillogin.jsp“> 

修改后： 

<a href=“<%=response.encodeURL(“maillogin.jsp“)%>“> 

[\
](https://blog.csdn.net/shb_derek1/article/details/8025459)



