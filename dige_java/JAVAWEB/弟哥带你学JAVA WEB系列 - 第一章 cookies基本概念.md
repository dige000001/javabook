## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0a14d01ca45402f860336800ecfef74~tplv-k3u1fbpfcp-zoom-1.image)




在web目录下创建一个文件 setCookie.jsp\


```
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>

<%
	Cookie c = new Cookie("name", "Gareen");
	c.setMaxAge(60 * 24 * 60);
	c.setPath("/");
	response.addCookie(c);
%>

<a href="getCookie.jsp">跳转到获取cookie的页面</a>
```

Cookie c = new Cookie("name", "Gareen");\
创建了一个cookie,名字是"name" 值是"Gareen"\


c.setMaxAge(24 * 60 * 60);\
表示这个cookie可以保留一天，如果是0，表示浏览器一关闭就销毁\


c.setPath("/");\
Path表示访问服务器的所有应用都会提交这个cookie到服务端，如果其值是 /a, 那么就表示仅仅访问 /a 路径的时候才会提交 cookie\


response.addCookie(c);\
通过response把这个cookie保存在浏览器端\

访问地址：

http://127.0.0.1/setCookie.jsp




在web目录下创建文件getCookie.jsp\
然后访问网页:


http://127.0.0.1/getCookie.jsp




```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" import="javax.servlet.http.Cookie"%>
 
<%
    Cookie[] cookies = request.getCookies();
    if (null != cookies)
        for (int d = 0; d <= cookies.length - 1; d++) {
            out.print(cookies[d].getName() + ":" + cookies[d].getValue() + "<br>");
        }
%>
```


Cookie[] cookies = request.getCookies();
表示获取所有浏览器传递过来的cookie


if (null != cookies )
如果浏览器端没有任何cookie，得到的Cookie数组是null


for (int d = 0; d <= cookies.length - 1; d++) {

out.print(cookies[d].getName() + ":" + cookies[d].getValue() + "<br>");

}


遍历所有的cookie
可以看到name:Gareen，这个在setCookie.jsp中设置的cookie\
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6727be81138f48538d9438bd1511fba3~tplv-k3u1fbpfcp-zoom-1.image)\
注； JSESSIONID 这个不是我们自己设置的cookie，这是tomcat设置的cookie，会在下一章[session](https://how2j.cn/k/jsp/jsp-session/583.html)的学习中用到

