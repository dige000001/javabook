## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

## 第8章：数据库连接池


### 8.1 JDBC数据库连接池的必要性



-   在使用开发基于数据库的web程序时，传统的模式基本是按以下步骤： 

<!---->

-   -   **在主程序（如servlet、beans）中建立数据库连接**
    -   **进行sql操作**

<!---->

-   -   **断开数据库连接**

<!---->

-   这种模式开发，存在的问题: 

<!---->

-   -   普通的JDBC数据库连接使用 DriverManager 来获取，每次向数据库建立连接的时候都要将 Connection 加载到内存中，再验证用户名和密码(得花费0.05s～1s的时间)。需要数据库连接的时候，就向数据库要求一个，执行完成后再断开连接。这样的方式将会消耗大量的资源和时间。**数据库的连接资源并没有得到很好的重复利用。** 若同时有几百人甚至几千人在线，频繁的进行数据库连接操作将占用很多的系统资源，严重的甚至会造成服务器的崩溃。
    -   **对于每一次数据库连接，使用完后都得断开。** 否则，如果程序出现异常而未能关闭，将会导致数据库系统中的内存泄漏，最终将导致重启数据库。（回忆：何为Java的内存泄漏？）

<!---->

-   -   **这种开发不能控制被创建的连接对象数**，系统资源会被毫无顾及的分配出去，如连接过多，也可能导致内存泄漏，服务器崩溃。

### 8.2 数据库连接池技术


-   为解决传统开发中的数据库连接问题，可以采用数据库连接池技术。 
-    **数据库连接池的基本思想**：就是为数据库连接建立一个“缓冲池”。预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕之后再放回去。 

<!---->

-    **数据库连接池**负责分配、管理和释放数据库连接，它**允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个**。 
-   数据库连接池在初始化时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由**最小数据库连接数来设定**的。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的**最大数据库连接数量**限定了这个连接池能占有的最大连接数，当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bf2256a14c0432e823208b04f7eff0d~tplv-k3u1fbpfcp-zoom-1.image)




-   **工作原理：**


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/99d66b56bbed497a8d2840a9d1c15119~tplv-k3u1fbpfcp-zoom-1.image)



-    **数据库连接池技术的优点**\
    **1. 资源重用**\
    由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。\
    **2. 更快的系统反应速度**\
    数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间\
    **3. 新的资源分配手段**\
    对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源\
    **4. 统一的连接管理，避免数据库连接泄漏**\
    在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露 



### 8.3 多种开源的数据库连接池



-   JDBC 的数据库连接池使用 javax.sql.DataSource 来表示，DataSource 只是一个接口，该接口通常由服务器(Weblogic, WebSphere, Tomcat)提供实现，也有一些开源组织提供实现： 

<!---->

-   -   **DBCP** 是Apache提供的数据库连接池。tomcat 服务器自带dbcp数据库连接池。**速度相对c3p0较快**，但因自身存在BUG，Hibernate3已不再提供支持。
    -   **C3P0** 是一个开源组织提供的一个数据库连接池，**速度相对较慢，稳定性还可以。** hibernate官方推荐使用

<!---->

-   -   **Proxool** 是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，**稳定性较c3p0差一点**
    -   **BoneCP** 是一个开源组织提供的数据库连接池，速度快

<!---->

-   -   **Druid** 是阿里提供的数据库连接池，据说是集DBCP 、C3P0 、Proxool 优点于一身的数据库连接池，但是速度不确定是否有BoneCP快

<!---->

-   DataSource 通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把 DataSource 称为连接池
-   **DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数据库访问速度。**

<!---->

-   特别注意： 

<!---->

-   -   数据源和数据库连接不同，数据源无需创建多个，它是产生数据库连接的工厂，因此**整个应用只需要一个数据源即可。**
    -   当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close(); 但conn.close()并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池。

#### 8.3.1 C3P0数据库连接池


-   获取连接方式一



```
//使用C3P0数据库连接池的方式，获取数据库的连接：不推荐
public static Connection getConnection1() throws Exception{
	ComboPooledDataSource cpds = new ComboPooledDataSource();
	cpds.setDriverClass("com.mysql.jdbc.Driver"); 
	cpds.setJdbcUrl("jdbc:mysql://localhost:3306/test");
	cpds.setUser("root");
	cpds.setPassword("abc123");
		
//	cpds.setMaxPoolSize(100);
	
	Connection conn = cpds.getConnection();
	return conn;
}
```



-   获取连接方式二



```
//使用C3P0数据库连接池的配置文件方式，获取数据库的连接：推荐
private static DataSource cpds = new ComboPooledDataSource("helloc3p0");
public static Connection getConnection2() throws SQLException{
	Connection conn = cpds.getConnection();
	return conn;
}
```



其中，src下的配置文件为：【c3p0-config.xml】



```
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
	<named-config name="helloc3p0">
		<!-- 获取连接的4个基本信息 -->
		<property name="user">root</property>
		<property name="password">abc123</property>
		<property name="jdbcUrl">jdbc:mysql:///test</property>
		<property name="driverClass">com.mysql.jdbc.Driver</property>
		
		<!-- 涉及到数据库连接池的管理的相关属性的设置 -->
		<!-- 若数据库中连接数不足时, 一次向数据库服务器申请多少个连接 -->
		<property name="acquireIncrement">5</property>
		<!-- 初始化数据库连接池时连接的数量 -->
		<property name="initialPoolSize">5</property>
		<!-- 数据库连接池中的最小的数据库连接数 -->
		<property name="minPoolSize">5</property>
		<!-- 数据库连接池中的最大的数据库连接数 -->
		<property name="maxPoolSize">10</property>
		<!-- C3P0 数据库连接池可以维护的 Statement 的个数 -->
		<property name="maxStatements">20</property>
		<!-- 每个连接同时可以使用的 Statement 对象的个数 -->
		<property name="maxStatementsPerConnection">5</property>

	</named-config>
</c3p0-config>
```




#### 8.3.2 DBCP数据库连接池



-   DBCP 是 Apache 软件基金组织下的开源连接池实现，该连接池依赖该组织下的另一个开源系统：Common-pool。如需使用该连接池实现，应在系统中增加如下两个 jar 文件： 

<!---->

-   -   Commons-dbcp.jar：连接池的实现
    -   Commons-pool.jar：连接池实现的依赖库

<!---->

-   **Tomcat 的连接池正是采用该连接池来实现的。** 该数据库连接池既可以与应用服务器整合使用，也可由应用程序独立使用。
-   数据源和数据库连接不同，数据源无需创建多个，它是产生数据库连接的工厂，因此整个应用只需要一个数据源即可。

<!---->

-   当数据库访问结束后，程序还是像以前一样关闭数据库连接：conn.close(); 但上面的代码并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池。
-   配置属性说明

| 属性                         | 默认值   | 说明                                                                           |
| -------------------------- | ----- | ---------------------------------------------------------------------------- |
| initialSize                | 0     | 连接池启动时创建的初始化连接数量                                                             |
| maxActive                  | 8     | 连接池中可同时连接的最大的连接数                                                             |
| maxIdle                    | 8     | 连接池中最大的空闲的连接数，超过的空闲连接将被释放，如果设置为负数表示不限制                                       |
| minIdle                    | 0     | 连接池中最小的空闲的连接数，低于这个数量会被创建新的连接。该参数越接近maxIdle，性能越好，因为连接的创建和销毁，都是需要消耗资源的；但是不能太大。 |
| maxWait                    | 无限制   | 最大等待时间，当没有可用连接时，连接池等待连接释放的最大时间，超过该时间限制会抛出异常，如果设置-1表示无限等待                     |
| poolPreparedStatements     | false | 开启池的Statement是否prepared                                                      |
| maxOpenPreparedStatements  | 无限制   | 开启池的prepared 后的同时最大连接数                                                       |
| minEvictableIdleTimeMillis |       | 连接池中连接，在时间段内一直空闲， 被逐出连接池的时间                                                  |
| removeAbandonedTimeout     | 300   | 超过时间限制，回收没有用(废弃)的连接                                                          |
| removeAbandoned            | false | 超过removeAbandonedTimeout时间后，是否进 行没用连接（废弃）的回收                                 |



-   获取连接方式一：




```
public static Connection getConnection3() throws Exception {
	BasicDataSource source = new BasicDataSource();
		
	source.setDriverClassName("com.mysql.jdbc.Driver");
	source.setUrl("jdbc:mysql:///test");
	source.setUsername("root");
	source.setPassword("abc123");
		
	//
	source.setInitialSize(10);
		
	Connection conn = source.getConnection();
	return conn;
}
```



-   获取连接方式二：




```
//使用dbcp数据库连接池的配置文件方式，获取数据库的连接：推荐
private static DataSource source = null;
static{
	try {
		Properties pros = new Properties();
		
		InputStream is = DBCPTest.class.getClassLoader().getResourceAsStream("dbcp.properties");
			
		pros.load(is);
		//根据提供的BasicDataSourceFactory创建对应的DataSource对象
		source = BasicDataSourceFactory.createDataSource(pros);
	} catch (Exception e) {
		e.printStackTrace();
	}
		
}
public static Connection getConnection4() throws Exception {
		
	Connection conn = source.getConnection();
	
	return conn;
}
```




其中，src下的配置文件为：【dbcp.properties】




```
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true&useServerPrepStmts=false
username=root
password=abc123

initialSize=10
#...
```




#### 8.3.3 Druid（德鲁伊）数据库连接池




Druid是阿里巴巴开源平台上一个数据库连接池实现，它结合了C3P0、DBCP、Proxool等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池，**可以说是目前最好的连接池之一。**




```
package com.atguigu.druid;

import java.sql.Connection;
import java.util.Properties;

import javax.sql.DataSource;

import com.alibaba.druid.pool.DruidDataSourceFactory;

public class TestDruid {
	public static void main(String[] args) throws Exception {
		Properties pro = new Properties();		 pro.load(TestDruid.class.getClassLoader().getResourceAsStream("druid.properties"));
		DataSource ds = DruidDataSourceFactory.createDataSource(pro);
		Connection conn = ds.getConnection();
		System.out.println(conn);
	}
}
```



其中，src下的配置文件为：【druid.properties】



```
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
username=root
password=123456
driverClassName=com.mysql.jdbc.Driver

initialSize=10
maxActive=20
maxWait=1000
filters=wall
```


-   详细配置参数：

| **配置**                        | **缺省** | **说明**                                                                                                                                                                         |
| ----------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| name                          |        | 配置这个属性的意义在于，如果存在多个数据源，监控的时候可以通过名字来区分开来。   如果没有配置，将会生成一个名字，格式是：”DataSource-” +   System.identityHashCode(this)                                                                  |
| url                           |        | 连接数据库的url，不同数据库不一样。例如：mysql :   jdbc:mysql://10.20.153.104:3306/druid2      oracle :   jdbc:oracle:thin:@10.20.149.85:1521:ocnauto                                             |
| username                      |        | 连接数据库的用户名                                                                                                                                                                      |
| password                      |        | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter。详细看这里：[https://github.com/alibaba/druid/wiki/使用ConfigFilter](https://github.com/alibaba/druid/wiki/%E4%BD%BF%E7%94%A8ConfigFilter) |
| driverClassName               |        | 根据url自动识别   这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置下)                                                                                                  |
| initialSize                   | 0      | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时                                                                                                                             |
| maxActive                     | 8      | 最大连接池数量                                                                                                                                                                        |
| maxIdle                       | 8      | 已经不再使用，配置了也没效果                                                                                                                                                                 |
| minIdle                       |        | 最小连接池数量                                                                                                                                                                        |
| maxWait                       |        | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。                                                                                          |
| poolPreparedStatements        | false  | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。                                                                                                 |
| maxOpenPreparedStatements     | -1    | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100                                                             |
| validationQuery               |        | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。                                                                                 |
| testOnBorrow                  | true   | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。                                                                                                                                    |
| testOnReturn                  | false  | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能                                                                                                                                     |
| testWhileIdle                 | false  | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。                                                                             |
| timeBetweenEvictionRunsMillis |        | 有两个含义： 1)Destroy线程会检测连接的间隔时间2)testWhileIdle的判断依据，详细看testWhileIdle属性的说明                                                                                                         |
| numTestsPerEvictionRun        |        | 不再使用，一个DruidDataSource只支持一个EvictionRun                                                                                                                                         |
| minEvictableIdleTimeMillis    |        |                                                                                                                                                                                |
| connectionInitSqls            |        | 物理连接初始化的时候执行的sql                                                                                                                                                               |
| exceptionSorter               |        | 根据dbType自动识别   当数据库抛出一些不可恢复的异常时，抛弃连接                                                                                                                                           |
| filters                       |        | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有：   监控统计用的filter:stat日志用的filter:log4j防御sql注入的filter:wall                                                                                          |
| proxyFilters                  |        | 类型是List，如果同时配置了filters和proxyFilters，是组合关系，并非替换关系                                                                                                                               |

