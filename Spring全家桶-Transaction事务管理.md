---
title: Spring全家桶-Transaction事务管理
abbrlink: 2271151f
date: 2020-09-19 18:33:03
categories:
  - Java
  - Spring
tags: [Java, Transaction]

---

# 事务

## 数据库事务

### 性质

原子性: 数据库操作都完成或者都不完成. 
一致性: 事务前后,数据状态一致
隔离性: 不同事务间的可见性
持久性: 事务提交后,数据库永久保存

### 隔离级别
`MySQL`  默认可重复读, `Oracle` 默认读已提交

| 级别     | 解决       | 描述                                                         |
| -------- | ---------- | ------------------------------------------------------------ |
| 读未提交 | - - - -    | 读到别人没有提交的事务数据                                   |
| 读已提交 | 脏读       | 读到别人已经提交的事务数据                                   |
| 可重复读 | 不可重复读 | 如果进行多次查询,别人如果已经提交修改,仍读不大,保证事务内读取一致性 |
| 线性读   | 幻读       | 事务必须线性执行                                             |




### 数据库端事务
```mysql
#  开启事务
START TRANSACTION;

# 操作1
UPDATE t_user t
SET t.amount = t.amount + 100
WHERE t.username = '张三';

# 操作2
UPDATE t_user t
SET t.amount = t.amount - 100
WHERE t.username = '李四' ;

# 事务操作 回滚 / 提交
ROLLBACK;
COMMIT;
```

### 示例: `JDBC` 事务操作

```java
/**
 * JDBC 数据库事物
 * 尝试断点处,前一个SQL事物是否提交,不同SQL间事物隔离级别
 */
public class LocalJdbcTransaction {

    private static String sqlA = "UPDATE t_user t SET t.amount = t.amount + 100 WHERE t.username = ?";

    private static String sqlB = "UPDATE t_user t SET t.amount = t.amount - 100 WHERE t.username = ?";

    private static String sqlC = "select t.id, t.username, t.amount from t_user t";

    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        // 数据库及驱动信息
        String url = "jdbc:mysql://localhost:3306/springboot_transaction?serverTimezone=Asia/Shanghai&characterEncoding=UTF-8";
        String username = "root";
        String password = "123456";
        String driver = "com.mysql.cj.jdbc.Driver";
        // 加载类
        Class.forName(driver);
        // 获得连接
        Connection connection = DriverManager.getConnection(url, username, password);

        // 启用事物,禁用默认提交,必须手动提交/回滚当前连接
        connection.setAutoCommit(false);
        // 设置SQL参数并执行
        PreparedStatement preparedStatementA = connection.prepareStatement(sqlA);
        preparedStatementA.setString(1, "张三");
        preparedStatementA.executeUpdate();

        // 在事物内查询
        PreparedStatement preparedStatementC = connection.prepareStatement(sqlC);
        ResultSet resultSet = preparedStatementC.executeQuery();
        while (resultSet.next()) {
            System.out.println(resultSet.getInt("id"));
            System.out.println(resultSet.getString("username"));
            System.out.println(resultSet.getInt("amount"));
        }

        // 设置SQL参数并执行
        PreparedStatement preparedStatementB = connection.prepareStatement(sqlB);
        preparedStatementB.setString(1, "李四");
        preparedStatementB.executeUpdate();
        // 提交事务
        connection.commit();
        // 关闭连接
        connection.close();
    }
}
```

# `Spring` 事务

- 支持多种资源事务管理/同步. 如 `Redis` , `Kafka` , `JDBC`
- 声明式事务管理
- 方便集成 `Spring` 框架

## 本地事务管理

1. 业务系统通过注解或者主动调用事务管理器 `PlatFormTransactionManager` .
2. 事务管理器, 根据具体的事务类型配置对应的事务实现 (`DataSourceTransactionManager` , `JapTransactionManager` ..)  , 进行事务操作, 调用资源管理器.
3. 数据库资源管理器(`Connection`), 执行对数据库底层的事务操作.

### 事务抽象类

`org.springframework.transaction.TransactionManager` (空接口)其子类中定义了事务操作. 
常用 `org.springframework.transaction.PlatformTransactionManager` 平台事务管理器.

| 类                           | 实现子类                       | 说明                           |
| ---------------------------- | ------------------------------ | ------------------------------ |
| `PlatFormTransactionManager` |                                | 平台事务管理接口,定义事务行为. |
|                              | `DataSourceTransactionManager` | `JDBC` 事务管理器              |
|                              | `JapTransactionManager`        | `JPA` 事务管理器               |
|                              | `JmsTransactionManager`        | `JMS` 消息事务管理器           |
|                              | `JtaTransactionManager`        | `JTA` 事务管理器               |
| `TransactionDefinition`      |                                | 事务管理器设置属性             |
| `TransactionStatus`          |                                | 事务运行的状态                 |

`PlatFormTransactionManager` 平台事物管理接口, 定义了本地事务行为

```java
/** 平台事务管理 */
public interface PlatformTransactionManager {
    // 通过定义创建事务状态
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    // 提交
    void commit(TransactionStatus status) throws TransactionException;
    // 回滚
    void rollback(TransactionStatus status) throws TransactionException;
}
```
`TransactionDefinition` 事务属性定义

```java
/** 事务属性 */
public interface TransactionDefinition {
    // 事务传播属性
    int getPropagationBehavior();
    // 隔离级别
    int getIsolationLevel();
    // 事务名
    String getName();
    // 超时时间
    int getTimeout();
    // 是否只读
    boolean isReadOnly();
}
```
`TransactionStatus` 事务运行状态定义

```java
/** 事务状态 */
public interface TransactionStatus extends SavepointManager {
    // 是否为新建事务
    boolean isNewTransaction();
    // 是否含有存盘点
    boolean hasSavepoint();
    // 只读事务
    void setRollbackOnly();
    // 是否只读事务
    boolean isRollbackOnly();
    // 事务是否完成
    boolean isCompleted();
}
```
### 隔离级别

| 属性                                               | 隔离级别           |
| -------------------------------------------------- | ------------------ |
| `TransactionDefinition.ISOLATION_DEFAULT`          | 数据库默认隔离级别 |
| `TransactionDefinition.ISOLATION_READ_UNCOMMITTED` | 读未提交           |
| `TransactionDefinition.ISOLATION_READ_COMMITTED`   | 读已提交           |
| `TransactionDefinition.ISOLATION_REPEATABLE_READ`  | 可重复读           |
| `TransactionDefinition.ISOLATION_SERIALIZABLE`     | 串行读             |

### 事务传播
一个`ServiceA`调用另一个`ServiceB`,如果都被标记为事务,那么这个`B`事务行为如何界定,称为传播行为.

| 属性                                                   | 传播行为                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `TransactionDefinition.PROPAGATION_REQUIRED (Default)` | A调用B时,如果A已经正在事务中,那么B则不会创建事务.共用同一个;否则自己创建事务 |
| `TransactionDefinition.PROPAGATION_SUPPORTS`           | A调用B时,如果A已经正在事务中,则B也进入该事务;如果A未在事务中,则B也不在事务中. |
| `TransactionDefinition.PROPAGATION_MANDATORY`          | A调用B时,A必须有一个事务,否则抛出异常                        |
| `TransactionDefinition.PROPAGATION_REQUIRES_NEW`       | 无论A是否含有事务,B会起一个新事务.仅限于`JTA`事务管理器      |
| `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`      | 无论A是否含有事务,B均不在一个事务内执行,执行完成后将挂起的A事务继续执行 |
| `TransactionDefinition.PROPAGATION_NEVER`              | 不在事务内执行,不存在事务                                    |
| `TransactionDefinition.PROPAGATION_NESTEDED`           | 嵌套事务,通过 `JDBC` 支持的 `savepoint` 进行存盘点的嵌套事务. |

 ### 示例: `Spring` 事务操作

通过 `@Transactional` 进行事务操作.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import top.jionjion.jta.bean.Customer;
import top.jionjion.jta.dao.CustomerRepository;

/** 通过标签形式,管理事务 */
@Service
public class CustomerServiceInAnnotation {
	
    /** JPA操作 */
    @Autowired
    private CustomerRepository customerRepository;

    /** 保存 */
    @Transactional
    public Customer save(Customer customer) {
        return customerRepository.save(customer);
    }
}

```

通过 `PlatformTransactionManager` 平台事务管理器, 进行事务操作

```java
/** 通过代码方式管理事务 */
@Service
public class CustomerServiceInCode {

    /** JPA操作 */
    @Autowired
    private CustomerRepository customerRepository;

    /** 平台事务管理器 */
    @Autowired
    private PlatformTransactionManager transactionManager;

    /** 保存 */
    public Customer save(Customer customer){
        Customer result = null;
        // 事务定义
        DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
        // 隔离级别
        transactionDefinition.setIsolationLevel(TransactionDefinition.ISOLATION_DEFAULT);
        // 传播行为
        transactionDefinition.setPropagationBehavior(TransactionDefinition.PROPAGATION_SUPPORTS);
        // 平台事务管理器,获取事务
        TransactionStatus transaction = transactionManager.getTransaction(transactionDefinition);

        try {
            // 业务方法
            result = customerRepository.save(customer);
            // 提交
            transactionManager.commit(transaction);
        } catch (Exception e){
            // 回滚
            transactionManager.rollback(transaction);
        }
        return result;
    }
}
```



## 外部事务管理

`JTA` - `Java事务管理API`  (`Java Transaction API`) 
由外部事务管理器提供事务管理,调用 `Spring` 的事务接口.
事务管理器多由应用服务器提供,常用 `JNDI` 等方式获得外部事务管理器.

### 两阶段提交

通过两阶段提交完成多数据库事务一致性.即首先进行一阶段提交,在所有数据源没有抛出异常后,执行二阶段提交;
如果一阶段提交中抛出异常,则二阶段均进行回滚保持事务一致性.(性能瓶颈)

**具体**
`Spring`事务接口 -> `SpringJTA` -> `JTA`事务管理器 -> 资源管理器 -> 数据库

### `JTA` 接口
`javax.transaction.TransactionManager` 接口,定义了全局的事务的方法
```java
public interface TransactionManager {
    // 开始事务
    void begin() throws NotSupportedException, SystemException;
    // 提交事务
    void commit() throws RollbackException, HeuristicMixedException, HeuristicRollbackException, SecurityException, IllegalStateException, SystemException;
    // 获得状态
    int getStatus() throws SystemException;
    // 开启一个事务
    Transaction getTransaction() throws SystemException;
    // 挂起后继续
    void resume(Transaction tobj) throws InvalidTransactionException, IllegalStateException, SystemException;
    // 回滚
    void rollback() throws IllegalStateException, SecurityException, SystemException;
    // 设置只读
    void setRollbackOnly() throws IllegalStateException, SystemException;
    // 设置超时时间
    void setTransactionTimeout(int seconds) throws SystemException;
    // 挂起
    Transaction suspend() throws SystemException;
}
```

需要在 `pom` 中添加依赖.常用 `atomikos` . 进行强一致性事务. 剩余操作与一般数据库操作一致.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```



# 分布式事务

## `GUID` 生成策略

标识全局唯一, 表示数据的唯一性.

- 数据库自增序列
- 使用 `UUID` 生成.
时间戳+机器特征,32位字符
- 使用 `MongoDB` 的 `ObjectID`
时间戳+机器ID+进程ID+序列. 24位字符
- 分布式服务
`Redis` 中的 `INCR` 自增变量

**笔记:** 通过将32位的字符串转为 `Long` 类型数字,以便提高性能

## 分布式对象

将对象信息存入分布式数据库中, 以便各子系统间同时调用. 

`Redis` : `Redisson` 库, 提供锁对象.

## 幂等

多次调用返回同样的结果,并不会对系统造成重复影响.

- 方法幂等
- 接口幂等
- 服务幂等

### 接口幂等

通过对传入参数的 `dto` 对象生成(通过系统生成对象唯一)或者获取(唯一字段标识)唯一ID, 并存入已处理的 `Map` 中,
用以标识其是否已经被处理过.来保证接口操作幂等.

```java
// 服务端实现幂等
public class OrderService {
    private Map distributedMap; // 用于存放已经处理的id
    @Transactional
    public void ticketOrder (BuyTicketDTo dto) {
        String uuid = createUUID(dto); // 创建或获取数据的唯一 ID
        if (distributedMap.contains(uuid)) {
            Order order = createOrder(dto); // 本服务内还没处理这个操作
            userService.charge(dto); // 调用user微服务
        }
    }
}
```

通过在SQL执行时更新状态( `status` 状态更新为已支付)并在条件中指定具体前状态( `status` 源状态为未支付). 保证一次执行后,后续不再变动

```sql
-- 数据库端实现幂等
update user_order 
   set amount = 100, status = 'paid' -- 原子性操作
 where status = 'unpaid'
```

### 服务间幂等

**服务间调用事务**

1. 减少服务间事务调用
2. 将服务器事务改为消息队列调用
注意: 消息中间件支持事务 / 重试机制 / 业务异常时回滚

**回滚策略**

1. 错误消息写入到失败队列
2. 定时任务检索超时订单, 对未完成订单作自动回滚
3. 错误消息保留,人工处理

## 分布式事务

### 一致性选择
强一致性: 所有服务操作均在一个原子内完成
弱一致性: 服务间隔调用,在各间隔内数据一致性.(难点:如果某一环节出现问题,如何处理)
最终一致性: 通过重试,人工干预,保证最终数据一致.

### 强一致性策略

**使用 `Spring JTA` **

1. 通过应用服务器实现 `JTA` 事务管理,如 `JBoss`
2. 通过第三方事务管理器实现 `JTA` 事务,如 `Atomikos` (推荐) , `Bitronix` 等

`Atomikos`: 新建个线程,调用 `Spring` 提供的事务管理器,使用两阶段提交,完成事务同步,但是性能较差.

### 弱一致性策略

消息驱动:
将数据库操作写入消息队列中,保持不同阶段下.消息与数据库操作的一致性,最终将事务整体保持一致.

事件溯源:
将不同数据库操作作为事件,写入同一个源中.分别控制不同阶段内的事务.

`TCC`模式: (`Try-Confirm-Cancel`)
服务间调用,尝试操作,如果没有问题,则确认.否则调用取消.

### 策略选择

强一次性: JTA (性能查,限于单次服务)
弱一次性/最终一次性: 最大努力一次提交,链式事务.(需要设计事务处理机制)

## 多数据源事务策略

### 最后资源博弈
`XA` 协议是指其中一个数据库支持 `JTA` 事务. 先提交非 `JTA` 事务,再提交 `JTA` 事务.这样,如果中途炸了,可以在最后的`JTA` 事务中回滚.

### 共享底层资源
不同事务间共享同一个底层资源,如将消息和数据都存入数据库中.需要消息服务支持数据库管理.

### 最大努力一次提交
将多个事务,尽可能依次同步提交,减少出错概率.
在多个事务中,引入消息重试机制,如果事务炸了,尝试再次重试..需要考虑数据重复操作问题.

### 链式提交事务
定义一个事务链,多个事务在一个事务管理器中依次执行.

# 分布式事务实例

## 链式提交事务

多数据源的数据库事务处理,尝试将其通过链式管理分次提交.

多数据源 `jdbc` 管理

```java
/**
 *  数据库配置,在中使用
 *  org.springframework.jdbc.datasource.DataSourceUtils#doGetConnection(javax.sql.DataSource)
 *
 * @author Jion
 */
@Configuration
public class DBConfiguration {

    /** 定义用户数据源数据库连接属性 */
    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.ds-user")
    public DataSourceProperties userDataSourceProperties() {
        return new DataSourceProperties();
    }

    /** 定义用户数据源数据库连接源 */
    @Bean
    @Primary
    public DataSource userDataSource() {
        return userDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
    }

    /** 定义用户数据源JDBC模板 */
    @Bean
    public JdbcTemplate userJdbcTemplate(@Qualifier("userDataSource") DataSource userDataSource){
        return new JdbcTemplate(userDataSource);
    }

    /** 定义数据库事务管理器 */
    @Bean
    public PlatformTransactionManager transactionManager(){
        PlatformTransactionManager userTransactionManager = new DataSourceTransactionManager(userDataSource());
        PlatformTransactionManager orderDataSourceTransactionManager = new DataSourceTransactionManager(orderDataSource());
        // 链式事务,这样可以将两个事务依次打开,依次执行,依次提交,或者依次回滚.
        return new ChainedTransactionManager(userTransactionManager, orderDataSourceTransactionManager);
    }

    /** 定义订单数据源数据库连接属性 */
    @Bean
    @ConfigurationProperties(prefix = "spring.ds-order")
    public DataSourceProperties orderDataSourceProperties() {
        return new DataSourceProperties();
    }

    /** 定义订单数据源数据库连接源 */
    @Bean
    public DataSource orderDataSource() {
        return orderDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
    }

    /** 定义订单数据源JDBC模板 */
    @Bean
    public JdbcTemplate orderJdbcTemplate(@Qualifier("orderDataSource") DataSource orderDataSource){
        return new JdbcTemplate(orderDataSource);
    }
}
```

`jdbc` 与 `jpa` 事务管理

```java
**
 *  数据库配置,在中使用
 *  org.springframework.jdbc.datasource.DataSourceUtils#doGetConnection(javax.sql.DataSource)
 *
 *  JDBC和JPA共同使用
 * @author Jion
 */
@Configuration
public class DBConfiguration {

    /** 定义用户数据源数据库连接属性 */
    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.ds-user")
    public DataSourceProperties userDataSourceProperties() {
        return new DataSourceProperties();
    }

    /** 定义用户数据源数据库连接源 */
    @Bean
    @Primary
    public DataSource userDataSource() {
        return userDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
    }

    /** Bean实体管理工厂 */
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(){
        HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
        // 不生成表结构
        adapter.setGenerateDdl(false);
        // 事务管理器工厂类
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setJpaVendorAdapter(adapter);
        factory.setDataSource(userDataSource());
        // 包扫描路径
        factory.setPackagesToScan("top.jionjion.jpa.bean");
        return factory;
    }

    /** 事务管理器 */
    @Bean
    public PlatformTransactionManager transactionManager(){
        // JPA 事务管理器
        JpaTransactionManager userTransactionManager = new JpaTransactionManager();
        // 实体管理工厂
        userTransactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
        // JDBC事务管理器
        DataSourceTransactionManager orderTransactionManager = new DataSourceTransactionManager(orderDataSource());
        // 链式事务管理器. 先放入,后提交
        return new ChainedTransactionManager(userTransactionManager, orderTransactionManager);
    }


    /** 定义订单数据源数据库连接属性 */
    @Bean
    @ConfigurationProperties(prefix = "spring.ds-order")
    public DataSourceProperties orderDataSourceProperties() {
        return new DataSourceProperties();
    }

    /** 定义订单数据源数据库连接源 */
    @Bean
    public DataSource orderDataSource() {
        return orderDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
    }

    /** 定义订单数据源JDBC模板 */
    @Bean
    public JdbcTemplate orderJdbcTemplate(@Qualifier("orderDataSource") DataSource orderDataSource){
        return new JdbcTemplate(orderDataSource);
    }
}
```



## 消息驱动事务
将ActiveMQ的消息同步到数据库中,保持事务一致.

将数据库存入后,放入消息中.
参考项目 dxt-db-jms 

消息配置

```java
import org.apache.activemq.ActiveMQConnectionFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.TransactionAwareConnectionFactoryProxy;
import org.springframework.jms.core.JmsTemplate;
import javax.jms.ConnectionFactory;

/** JMS 消息配置 */
@Configuration
public class JmsConfiguration {


    /** JMS连接工厂 */
    @Bean
    public ConnectionFactory connectionFactory(){
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://127.0.0.1:61616");
        TransactionAwareConnectionFactoryProxy proxy = new TransactionAwareConnectionFactoryProxy();
        proxy.setTargetConnectionFactory(connectionFactory);
        proxy.setSynchedLocalTransactionAllowed(true);
        return proxy;
    }

    /** 重新配置模板工厂,启用事务 */
    @Bean
    public JmsTemplate jmsTemplate(){
        JmsTemplate jmsTemplate = new JmsTemplate(connectionFactory());
        // 由事务控制Session连接
        jmsTemplate.setSessionTransacted(true);
        return jmsTemplate;
    }
}
```

服务层

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import top.jionjion.jms.bean.Customer;
import top.jionjion.jms.dao.CustomerRepository;

import java.util.List;

/** 数据库与消息重试入库 */
@Service
public class CustomerService {

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private JmsTemplate jmsTemplate;

    /** 消息触发,收到队列消息后执行 */
    @Transactional(rollbackFor = Exception.class)
    @JmsListener(destination = "customer:msg:new")
    public void handleCreateCustomer(String msg){
        System.out.println("收到消息:" + msg);
        Customer customer = new Customer();
        customer.setUsername("Jion");
        customer.setDeposit(-2000);
        customerRepository.save(customer);
        if (customer.getDeposit() <= 0){
            throw new RuntimeException("余额不足");
        }
    }

    /** 创建用户 */
    @Transactional(rollbackFor = Exception.class)
    public Customer createCustomer(Customer customer) {
        Customer result = customerRepository.save(customer);
        if(result.getDeposit() <= 0){
            throw new RuntimeException("余额不足");
        }
        // 发送消息
        jmsTemplate.convertAndSend("customer:msg:reply", "创建成功.");

        return result;
    }
}
```

