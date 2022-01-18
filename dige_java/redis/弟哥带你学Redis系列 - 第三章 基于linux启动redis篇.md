## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# redis服务启动



**默认配置启动**

redis-server

redis-server --port 6379

redis-server --port 6380 ......

**指定配置文件启动**

redis-server redis.conf

redis-server redis-6379.conf

redis-server redis-6380.conf .....

redis-server conf/redis-6379.conf

redis-server config/redis-6380.conf ......

# redis客户端连接

**默认连接**

redis-cli

**连接指定服务器**

redis-cli -h 127.0.0.1

redis-cli -p 6379

redis-cli -h 127.0.0.1 p 6379

# Redis服务端配置

**基本配置**

daemonize yes

以守护进程方式启动，使用本启动方式,redis将以服务的形式存在，日志将不再打印到命令窗口中

bind 127.0.0.1

绑定主机地址

port 6780

设定当前服务启动端口号

dir "/自定义目录/redis/data"

设定当前服务文件保存位置，包含日志文件、持久化文件(后面详细讲解)等

logfile "6***.log "

设定日志文件名，便于查阅

loglevel debug | verbose | notice | warning

设置服务器以指定日志记录级别

**注意:日志级别开发期设置为verbose即可，生产环境中配置为notice，简化日志输出量，降低写日志O的频度**

daemonize yes | no

设置服务器以守护进程的方式运行



databases 16

include /path/server-端口号.conf

导入并加载指定配置文件信息，用于快速创建redis公共配置较多的redis实例配置文件，便于维护

# Redis客户端配置

设置同一时间最大客户端连接数，默认无限制。当客户端连接到达上限，Redis会关闭新的连接

maxclients 0

客户端闲置等待最大时长，达到最大值后关闭连接。如需关闭该功能，设置为0

timeout 300

