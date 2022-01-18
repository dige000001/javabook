## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

**必要jar包：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/826cf2152e6e40f4b2b67f962779861c~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf898e85c59c4888bde8b26d33bd54a5~tplv-k3u1fbpfcp-zoom-1.image)




**在 spring 配置文件配置数据库连接池**

```
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" 
 destroy-method="close">
 <property name="url" value="jdbc:mysql:///user_db" />
 <property name="username" value="root" />
 <property name="password" value="root" />
 <property name="driverClassName" value="com.mysql.jdbc.Driver" />
</bean>
```

**配置 JdbcTemplate 对象，注入 DataSource**

```
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
 <!--注入 dataSource-->
 <property name="dataSource" ref="dataSource"></property>
</bean>
```

开启组件扫描

```
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

****

#

**创建 service 类，创建 dao 类，在 dao 注入 jdbcTemplate 对象，在service类注入dao对象**

```
@Service
public class BookService {
 //注入 dao
 @Autowired
 private BookDao bookDao;
}
```

```
@Repository
public class BookDaoImpl implements BookDao {
 //注入 JdbcTemplate
 @Autowired
 private JdbcTemplate jdbcTemplate;
}
```

**对应数据库创建实体类**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d01a046c3194432b1b78c0ca60bf06c~tplv-k3u1fbpfcp-zoom-1.image)




# JdbcTemplate

### 添加操作：

调用 JdbcTemplate 对象里面 update 方法实现添加操作

update(String sql, Object... args)

参数分别为sql语句和可变参数

```
@Repository
public class BookDaoImpl implements BookDao {
 //注入 JdbcTemplate
 @Autowired
 private JdbcTemplate jdbcTemplate;
 //添加的方法
    
 @Override
 public void add(Book book) {
 //1 创建 sql 语句
 String sql = "insert into t_book values(?,?,?)";
 //2 调用方法实现
 Object[] args = {book.getUserId(), book.getUsername(), 
book.getUstatus()};
 int update = jdbcTemplate.update(sql,args)
 System.out.println(update);
 }
}
```

### 修改

```
@Override
public void updateBook(Book book) {
 String sql = "update t_book set username=?,ustatus=? where user_id=?";
 Object[] args = {book.getUsername(), book.getUstatus(),book.getUserId()};
 int update = jdbcTemplate.update(sql, args);
 System.out.println(update);
}
```

### 删除 

```
@Override
public void delete(String id) {
 String sql = "delete from t_book where user_id=?";
 int update = jdbcTemplate.update(sql, id);
 System.out.println(update);
}
```

###

### 查询


#### 查询表里面有多少条记录

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42b1b6672dab402fa9106036191400ca~tplv-k3u1fbpfcp-zoom-1.image)

第一个参数：sql 语句 

第二个参数：返回类型 Class 

```
@Override
public int selectCount() {
String sql = "select count(*) from t_book";
 Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
 return count;
}
```

#### 查询返回对象 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2a44c81b02a49819f8cc303fbca5431~tplv-k3u1fbpfcp-zoom-1.image)

RowMapper 是接口，针对返回不同类型数据，使用这个接口里面实现类完成数据封装 

```
@Override
public Book findBookInfo(String id) {
 String sql = "select * from t_book where user_id=?";
 //调用方法
 Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
 return book;
}
```

#### 查询返回对象集合 

```
@Override
public List<Book> findAllBook() {
 String sql = "select * from t_book";
 //调用方法
 List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
 return bookList;
}
```

###

###

### 批量操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/160916e2384e4c2896c2ca5d9c7805db~tplv-k3u1fbpfcp-zoom-1.image)



#### 批量添加操作




```
@Override
public void batchAddBook(List<Object[]> batchArgs) {
 String sql = "insert into t_book values(?,?,?)";
 int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
 System.out.println(Arrays.toString(ints));
}
```

```
//批量添加测试
List<Object[]> batchArgs = new ArrayList<>();
Object[] o1 = {"3","java","a"};
Object[] o2 = {"4","c++","b"};
Object[] o3 = {"5","MySQL","c"};
batchArgs.add(o1);
batchArgs.add(o2);
batchArgs.add(o3);
//调用批量添加
bookService.batchAdd(batchArgs);
```

#### 批量修改操作 

```
@Override
public void batchUpdateBook(List<Object[]> batchArgs) {
 String sql = "update t_book set username=?,ustatus=? where user_id=?";
 int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
 System.out.println(Arrays.toString(ints));
}
```

#### 批量删除操作 

```
@Override
public void batchDeleteBook(List<Object[]> batchArgs) {
 String sql = "delete from t_book where user_id=?";
 int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
 System.out.println(Arrays.toString(ints));
}
```



# 事务

在 dao 创建两个方法：加钱和减钱的方法，在 service 创建方法（转账的方法） 

```
@Repository
public class UserDaoImpl implements UserDao {
 @Autowired
 private JdbcTemplate jdbcTemplate;
 //lucy 转账 100 给 mary
 //少钱
 @Override
 public void reduceMoney() {
 String sql = "update t_account set money=money-? where username=?";
 jdbcTemplate.update(sql,100,"lucy");
 }
 //多钱
 @Override
 public void addMoney() {
 String sql = "update t_account set money=money+? where username=?";
 jdbcTemplate.update(sql,100,"mary");
 }
}
```

```
@Service
public class UserService {
 //注入 dao
 @Autowired
 private UserDao userDao;
 //转账的方法
 public void accountMoney() {
 //lucy 少 100
 userDao.reduceMoney();
 //mary 多 100
 userDao.addMoney();
 }
}
```

### 事务操作（Spring 事务管理介绍） 

1、事务添加到 JavaEE 三层结构里面 Service 层（业务逻辑层） 

2、在 Spring 进行事务管理操作 （1）有两种方式：编程式事务管理和声明式事务管理（使用） 

3、声明式事务管理 （1）基于注解方式（使用） （2）基于 xml 配置文件方式 

4、在 Spring 进行声明式事务管理，底层使用 AOP 原理 

5、Spring 事务管理 API 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88cf57a77997494daa72d2987a8611db~tplv-k3u1fbpfcp-zoom-1.image)

### 事务操作（注解声明式事务管理） 

#### 1、在 spring 配置文件配置事务管理器 

```
<!--创建事务管理器-->
<bean id="transactionManager" 
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
 <!--注入数据源-->
 <property name="dataSource" ref="dataSource"></property>
</bean>
```

#### 2、在 spring 配置文件，开启事务注解 

（1）在 spring 配置文件引入名称空间 tx 

```
<beans xmlns="http://www.springframework.org/schema/beans" 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xmlns:context="http://www.springframework.org/schema/context" 
 xmlns:aop="http://www.springframework.org/schema/aop" 
 xmlns:tx="http://www.springframework.org/schema/tx" 
 xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd 
 http://www.springframework.org/schema/context 
http://www.springframework.org/schema/context/spring-context.xsd 
 http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop.xsd
http://www.springframework.org/schema/tx 
http://www.springframework.org/schema/tx/spring-tx.xsd">
```

（2）开启事务注解 

```
<tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

#### 3、 在 service 类上面（或者 service 类里面方法上面）添加事务注解

（1）@Transactional，这个注解添加到类上面，也可以添加方法上面 

（2）如果把这个注解添加类上面，这个类里面所有的方法都添加事务 

（3）如果把这个注解添加方法上面，为这个方法添加事务 

```
@Service
@Transactional
public class UserService {
```

在这个注解里面可以配置事务相关参数 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1813ef6291e420bbfde577ba7d82922~tplv-k3u1fbpfcp-zoom-1.image)

下面逐一讲解：

-    **propagation：事务传播行为**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b47bcb2784b419890314ff23e2f255c~tplv-k3u1fbpfcp-zoom-1.image)

如：

@Transcation(propagation = Propagation.REQUIRED)

-   **ioslation：事务隔离级别**

事务有特性成为隔离性，多事务操作之间不会产生影响。不考虑隔离性产生很多问题：脏读、不可重复读、虚（幻）读 

解决：通过设置事务隔离级别，解决读问题 ：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5ec748739674c91ac29f7d308f66ef3~tplv-k3u1fbpfcp-zoom-1.image)

@Transcation(isolation = Isolation.REPEATABLE_READ)

-   **timeout：超时时间**

（1）事务需要在一定时间内进行提交，如果不提交进行回滚 

（2）默认值是 -1 ，设置时间以秒单位进行计算 

-   **readOnly：是否只读**

（1）读：查询操作，写：添加修改删除操作 

（2）readOnly 默认值 false，表示可以查询，可以添加修改删除操作 

（3）设置 readOnly 值是 true，设置成 true 之后，只能查询 

-   **rollbackFor： 设置出现哪些异常进行事务回滚**
-   **noRollbackFor： 设置出现哪些异常不进行事务回滚**


### XML配置方式

第一步 配置事务管理器 

第二步 配置通知 

第三步 配置切入点和切面 

```
<!--1 创建事务管理器-->
<bean id="transactionManager" 
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
 <!--注入数据源-->
 <property name="dataSource" ref="dataSource"></property>
</bean>

<!--2 配置通知-->
<tx:advice id="txadvice">
 <!--配置事务参数-->
 <tx:attributes>
 <!--指定哪种规则的方法上面添加事务-->
 <tx:method name="accountMoney" propagation="REQUIRED"/>
 <!--<tx:method name="account*"/>-->
 </tx:attributes>
</tx:advice>

<!--3 配置切入点和切面-->
<aop:config>
 <!--配置切入点-->
 <aop:pointcut id="pt" expression="execution(* 
com.atguigu.spring5.service.UserService.*(..))"/>
 <!--配置切面-->
 <aop:advisor advice-ref="txadvice" pointcut-ref="pt"/>
</aop:config>
```

### 完全注解

创建配置类，使用配置类替代 xml 配置文件 

```
@Configuration //配置类
@ComponentScan(basePackages = "com.atguigu") //组件扫描
@EnableTransactionManagement //开启事务
public class TxConfig {
 //创建数据库连接池
 @Bean
 public DruidDataSource getDruidDataSource() {
 DruidDataSource dataSource = new DruidDataSource();
 dataSource.setDriverClassName("com.mysql.jdbc.Driver");
 dataSource.setUrl("jdbc:mysql:///user_db");
 dataSource.setUsername("root");
 dataSource.setPassword("root");
 return dataSource;
 }
    
 //创建 JdbcTemplate 对象
 @Bean
 public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
 //到 ioc 容器中根据类型找到 dataSource
 JdbcTemplate jdbcTemplate = new JdbcTemplate();
 //注入 dataSource
 jdbcTemplate.setDataSource(dataSource);
  return jdbcTemplate;
 }
    
 //创建事务管理器
 @Bean
 public DataSourceTransactionManager 
getDataSourceTransactionManager(DataSource dataSource) {
 DataSourceTransactionManager transactionManager = new 
DataSourceTransactionManager();
 transactionManager.setDataSource(dataSource);
 return transactionManager;
 }
}
```

若有收获，就点个赞吧

