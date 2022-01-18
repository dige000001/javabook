## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

命令：
java：运行java程序

javac：编译java程序


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/479569d35999488d924e07eb76689c9b~tplv-k3u1fbpfcp-zoom-1.image)

java是解释型语言：将java文件编译成class文件后，jvm对class文件每一行都进行解释，解释成本地操作系统能识别的语句

**注：现在的即时编译JIT指的是** **在提前将字节码编译成本地操作系统能识别的语句，这样就不用跑一句解释一句，速度能提升很大**

编译型（如C++）：直接将源代码文件编译成操作系统可直接运行的可执行文件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6493c55680124df7a830659f8cbc33ee~tplv-k3u1fbpfcp-zoom-1.image)



名词解释

J2SDK就是java 2 sdk，现在叫jdk，jdk包含jre

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b923c2d804174baba287beccdd1a58d4~tplv-k3u1fbpfcp-zoom-1.image)




Path：配置可执行文件的默认查找路径 如java.exe放在xxx/bin目录下，在环境变量中的path属性加上这个路径之后，在cmd里输入java，它就会自动从这个路径找java.exe这个可执行文件

ClassPath：是JVM的一个环境变量，用于方便查找用户的类

<https://docs.oracle.com/javase/tutorial/essential/environment/paths.html>

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a3aa256ee104f12bcd92e263600d743~tplv-k3u1fbpfcp-zoom-1.image)

