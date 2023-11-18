# ShardingJdbc学习

> https://shardingsphere.apache.org/

## 1.4 Sharding-JDBC介绍

### 1.4.1 Sharding-JDBC介绍

Sharding-JDBC是当当网研发的开源分布式数据库中间件，从 3.0 开始Sharding-JDBC被包含在 Sharding-Sphere 中，之后该项目进入进入Apache孵化器，4.0版本之后的版本为Apache版本。

ShardingSphere是一套开源的分布式数据库中间件解决方案组成的生态圈，它由Sharding-JDBC、Sharding- Proxy和Sharding-Sidecar(计划中)这3款相互独立的产品组成。 他们均提供标准化的数据分片、分布式事务和 数据库治理功能，可适用于如Java同构、异构语言、容器、云原生等各种多样化的应用场景。

官方地址:https://shardingsphere.apache.org/document/current/cn/overview/

咱们目前只需关注Sharding-JDBC，它定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端 直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种 ORM框架。

Sharding-JDBC的核心功能为数据分片和读写分离，通过Sharding-JDBC，应用可以透明的使用jdbc访问已经分库 分表、读写分离的多个数据源，而不用关心数据源的数量以及数据如何分布。

- 适用于任何基于Java的ORM框架，如: Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
- 基于任何第三方的数据库连接池，如:DBCP, C3P0, BoneCP, Druid, HikariCP等。
- 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer和PostgreSQL。

![image-20231118161517728](./img/5.png)

上图展示了Sharding-Jdbc的工作方式，使用Sharding-Jdbc前需要人工对数据库进行分库分表，在应用程序中加入 Sharding-Jdbc的Jar包，应用程序通过Sharding-Jdbc操作分库分表后的数据库和数据表，由于Sharding-Jdbc是对 Jdbc驱动的增强，使用Sharding-Jdbc就像使用Jdbc驱动一样，在应用程序中是无需指定具体要操作的分库和分表 的。



## 2.Sharding-JDBC快速入门

### 2.1 需求说明

本章节使用Sharding-JDBC完成对订单表的水平分表，通过快速入门程序的开发，快速体验Sharding-JDBC的使用 方法。

人工创建两张表，t_order_1和t_order_2，这两张表是订单表拆分后的表，通过Sharding-Jdbc向订单表插入数据， 按照一定的分片规则，主键为偶数的进入t_order_1，另一部分数据进入t_order_2，通过Sharding-Jdbc 查询数 据，根据 SQL语句的内容从t_order_1或t_order_2查询数据。



### 2.2.环境搭建

#### 2.2.2 创建数据库

```sql

CREATE DATABASE `db1` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
-- auto-generated definition
create table t_order_1
(
    order_id bigint         not null comment '订单id'
        primary key,
    price    decimal(10, 2) not null comment '订单价格',
    user_id  bigint         not null comment '下单用户id',
    status   varchar(50)    not null comment '订单状态'
)
    charset = utf8mb4
    row_format = DYNAMIC;

-- auto-generated definition
create table t_order_2
(
    order_id bigint         not null comment '订单id'
        primary key,
    price    decimal(10, 2) not null comment '订单价格',
    user_id  bigint         not null comment '下单用户id',
    status   varchar(50)    not null comment '订单状态'
)
    charset = utf8mb4
    row_format = DYNAMIC;
```

#### 2.2.3.引入maven依赖

> 见GitHub的详细工程代码：https://github.com/WheatJack/JavaStudyProject/tree/master/ShardingJDBCStudy

```xml
 <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>4.1.1</version>
        </dependency>
```

详细的CRUD见：GitHub的详细工程代码：https://github.com/WheatJack/JavaStudyProject/tree/master/ShardingJDBCStudy

### 	2.4.流程分析

通过日志分析，Sharding-JDBC在拿到用户要执行的sql之后干了哪些事儿:

1. 解析sql，获取**片键值**，在本例中是order_id
2. Sharding-JDBC通过规则配置 t_order_$->{order_id % 2 + 1}，知道了当order_id为偶数时，应该往 t_order_1表插数据，为奇数时，往t_order_2插数据。
3. 于是Sharding-JDBC根据order_id的值改写sql语句，改写后的SQL语句是真实所要执行的SQL语句。 
4. 执行改写后的真实sql语
5. 将所有真正执行sql的结果进行汇总合并，返回。

### 2.5.其他集成方式

Sharding-JDBC不仅可以与spring boot良好集成，它还支持其他配置方式，共支持以下四种集成方式。

**Java 配置**

添加配置类:

```java
package com.example.shardingjdbcstudy.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.apache.shardingsphere.api.config.sharding.KeyGeneratorConfiguration;
import org.apache.shardingsphere.api.config.sharding.ShardingRuleConfiguration;
import org.apache.shardingsphere.api.config.sharding.TableRuleConfiguration;
import org.apache.shardingsphere.api.config.sharding.strategy.InlineShardingStrategyConfiguration;
import org.apache.shardingsphere.shardingjdbc.api.ShardingDataSourceFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * <p>
 * JDBC的代码配置
 * </p>
 *
 * @author JackGao
 * @since 11/18/23
 **/
@Configuration
public class ShardingJdbcConfig {
    /**
     * <p> 定义数据源 <p>
     *
     * @author JackGao
     * @since 11/18/23 16:46
     */
    Map<String, DataSource> createDataSourceMap() {
        DruidDataSource dataSource1 = new DruidDataSource();
        dataSource1.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource1.setUrl("jdbc:mysql://localhost:3306/order_db?useUnicode=true");
        dataSource1.setUsername("root");
        dataSource1.setPassword("root");
        Map<String, DataSource> result = new HashMap<>();
        result.put("m1", dataSource1);
        return result;
    }

    /**
     * <p> 定义主键生成策略 <p>
     *
     * @author JackGao
     * @since 11/18/23 16:47
     */
    private static KeyGeneratorConfiguration getKeyGeneratorConfiguration() {
        return new
                KeyGeneratorConfiguration("SNOWFLAKE", "order_id");
    }

    /**
     * <p> 定义t_order表的分片策略 <p>
     *
     * @author JackGao
     * @since 11/18/23 16:47
     */
    TableRuleConfiguration getOrderTableRuleConfiguration() {
        TableRuleConfiguration result = new TableRuleConfiguration("t_order", "m1.t_order_$‐> {1..2} ");
        result.setTableShardingStrategyConfig(new
                InlineShardingStrategyConfiguration("order_id", "t_order_$‐>{order_id % 2 + 1}"));
        result.setKeyGeneratorConfig(getKeyGeneratorConfiguration());
        return result;
    }

    /**
     * <p> 定义sharding‐Jdbc数据源 <p>
     *
     * @author JackGao
     * @since 11/18/23 16:48
     */
    @Bean
    DataSource getShardingDataSource() throws SQLException {
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        shardingRuleConfig.getTableRuleConfigs().add(getOrderTableRuleConfiguration());
        //spring.shardingsphere.props.sql.show = true
        Properties properties = new Properties();
        properties.put("sql.show", "true");
        return ShardingDataSourceFactory.createDataSource(createDataSourceMap(), shardingRuleConfig, properties);
    }


}	
```

由于采用了配置类所以需要屏蔽原来application.properties文件中spring.shardingsphere开头的配置信息。 还需要在SpringBoot启动类中屏蔽使用spring.shardingsphere配置项的类:

```java

/**
 * <p> 如果使用Java配置类的方法配置类 ShardingJdbc 那么需要屏蔽shardingJdbc的自己的配置类
 * 切记是该包下：import org.apache.shardingsphere.shardingjdbc.spring.boot.SpringBootConfiguration;
 * <p>
 * @author JackGao
 * @since 11/18/23 16:54
 */
@SpringBootApplication(exclude = {SpringBootConfiguration.class})
public class ShardingJdbcStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShardingJdbcStudyApplication.class, args);
    }

}
```



## 3.Sharding-JDBC执行原理

### 3.1 基本概念

在了解Sharding-JDBC的执行原理前，需要了解以下概念:

**逻辑表：**水平拆分的数据表的总称。例:订单数据表根据主键尾数拆分为10张表，分别是 t_order_0 、 t_order_1 到 t_order_9 ，他们的逻辑表名为 t_order 。

**真实表：**在分片的数据库中真实存在的物理表。即上个示例中的 t_order_0 到 t_order_9 。

**数据节点：**数据分片的最小物理单元。由数据源名称和数据表组成，例: database1.t_order_0 。
