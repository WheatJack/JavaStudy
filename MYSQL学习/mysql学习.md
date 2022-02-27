#    MYSQL学习

> 开始学习mysql了，奥力给⛽️⛽️





## 一、基础学习

### 1、通用语法及分类

#### SQL分类

![image-20220220134319454](https://tva1.sinaimg.cn/large/e6c9d24egy1gzjxc4b8mmj20m00663z3.jpg)



#### DDL语句



##### 数据库操作-CRUD

```mysql
// 查询所有数据库
show Database;
// 查询当前数据库
select database();
// 创建数据库
Create database [if not exists] 数据库名 [default charset utf8mb4][collate 排序规则];
// 删除数据库
drop database  if exists test_database;
// 使用数据库
use test_database;

```



##### 表操作-查询

```mysql
// 查询数据库所有的表
show tables;
// 查询所有表的状态
show table status;
// 查询表结构
desc table_name;
// 查询指定表的建表语句
show create table table_name;
```



##### 表操作-创建

```mysql
// 创建表--结构
CREate Table table_name (
'id' varchar(50) [comment 注释],
'id' varchar(50) [comment 注释],
'id' varchar(50) [comment 注释],
)[comment 表注释];

// 创建表
CREATE TABLE `employ` (
  `id` varchar(50) NOT NULL COMMENT '主健id',
  `name` varchar(5) DEFAULT NULL COMMENT 'name',
  `time` datetime DEFAULT NULL COMMENT '创建时间',
  `age` int DEFAULT NULL COMMENT '年纪',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='员工表';
```



###### 表数据类型及案例

**数值类型**

![image-20220220142515930](https://tva1.sinaimg.cn/large/e6c9d24egy1gzjyjpcxlsj20p907c75g.jpg)



字符串类型

![image-20220220142608824](https://tva1.sinaimg.cn/large/e6c9d24egy1gzjykom8k7j20p5063wfc.jpg)

**时间类型**

![image-20220220142649720](https://tva1.sinaimg.cn/large/e6c9d24egy1gzjylbrzehj20ic03aq39.jpg)





##### 表结构-修改

```mysql
// 新增字段
alter table table_name add 字段 类型(长度) [comment 注释] [约束];
// 只修改数据类型
alter table table_name modify 字段 类型(长度) [comment 注释] [约束];
// 修改字段名和字段类型
Alter table table_name change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];
// 删除字段
alter table table_name drop 字段名;
// 修改表名
alter table table_name rename to new_table_name;
// 删除表
drop table if exists table_name;
// 删除(截断)指定表 并重新创建该表
truncate table table_name;
```



#### DML语句

##### 添加数据

```mysql
// 给指定字段添加数据
insert into table_name (字段1,字段2) values (值1,值2......);
// 给全部字段添加数据
insert into table_name values (值1,值2......);
// 批量添加数据
insert into table_name (字段1,字段2) values (值1,值2......),(值1,值2......),(值1,值2......);
insert into table_name values (值1,值2......),(值1,值2......),(值1,值2......);
// 从别的表取数 插入到这个表
insert into table_name select * from table_name2;
```

Notes：

- 插入数据时，指定的字段顺序需要与值的顺序一一对应的
- 字符串和日期类型应该包含在引号中
- 插入的数据大小 应该在字段的规定范围内



##### 修改数据

```mysql
// 更新数据
update table_name set 字段1=值1,字段2=值2	.... [where 条件];
// 删除数据
delete from table_name  [where ];
```





#### DQL语句

```mysql
// 结构
select 
	字段列表 
from 
	table_name 
where 条件列表 
group by 分组字段列表 
having 分组后条件查询 
order by 排序字段列表 
limit 分页参数 ;
```



**查询学习：**

- 基本查询
- 条件查询 where
- 聚合函数 （count、max、min、avg、sum）
- 分组查询 group by
- 排序查询 order by
- 分页查询 limit



##### 基本查询

```mysql
// 查询全部
select * from table_name;
// 设置字段别名  as 可以省略
select name as userName from table_name;
// 去除重复 distinct
select distinct name from table_name;

```



##### 条件查询

```mysql
select * from table_name where 条件列表;
// 查询名字为两个字的员工信息---使用like 或者使用length
select * from emp where name like '__';
```

**条件列表：**

![image-20220220153026880](https://tva1.sinaimg.cn/large/e6c9d24egy1gzk0fhu4bij20ed08qgm2.jpg)



![image-20220220153048518](https://tva1.sinaimg.cn/large/e6c9d24egy1gzk0fv6c9kj20do03u0st.jpg)



##### 聚合函数

> 将**一列**数据作为一个整体，进行纵向计算
>
> **所有的null值是不参与聚合函数的**

![image-20220220153712548](https://tva1.sinaimg.cn/large/e6c9d24egy1gzk0mip1bgj20f204wjrf.jpg)



```mysql
// 语法
select max(age) from table_name;
```



##### 分组查询

```mysql
// 语法
select 字段列表 from table_name where 条件  group by 分组字段名 having  分组过滤条件;
```



**WHERE与HAVING的区别：**

- 执行时机不同：where 是分组之前进行过滤的 不满足where条件 不参与分组；having是分组之后对结果进行过滤
- 判断条件不同：where不能对聚合函数进行判断，而having可以



**Notes：**

- 执行顺序：where>聚合函数>having
- 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义



##### 排序查询

```mysql
// 语法  默认asc  
Select 字段列表 from table_name  order by 字段1 asc,字段2 desc;
```



##### 分页查询

```mysql
// 语法
select * from table_name limit 起始索引,查询记录数;

```



**Notes:**

- 起始索引 从0开始 起始索引= 查询页码-1  * 每页显示的条数
- 分页查询是数据库的方言，不同的数据库有不同的实现 Mysql是Limit
- 如果查询的是第一页数据，起始索引可以省略，直接简写为 limit 10



##### 执行顺序

![image-20220220203618076](https://tva1.sinaimg.cn/large/e6c9d24egy1gzk99ryqehj20t80crwfo.jpg)





#### DCL语句

> Data Control Language ，用来管理数据库用户、控制数据库的访问权限等等

##### 基础CRUD

```mysql
// 使用mysql 库
use mysql;
use mysql;
// 查询用户
select * from user;

// 创建用户 test_user
create user 'test_user'@'主机名：localhost' identified  by '123456';

// 修改用户密码
alter user 'test_user 用户名'@'主机名: localhost' identified  with mysql_native_password By '新密码';

// 删除用户
drop user 'test_user 用户名'@'主机名: localhost';
```



##### 权限控制

权限列表

![image-20220220214614372](https://tva1.sinaimg.cn/large/e6c9d24egy1gzkbahykr6j20t409xwf0.jpg)



```mysql
// 查询权限
SHOW GRANTS FOR 'root'@'%';

// 授予权限 如果授予所有的 就使用*.*
GRANTS 权限列表 ON 数据库.表名 TO '用户名：test_user'@'主机名：localhost';
grant all on *.* to 'test_user'@'localhost';

//撤销权限
REVOKE 权限列表 ON 数据库.表名 FROM '用户名：test_user'@'主机名：localhost';
REVOKE all ON *.* FROM 'test_user'@'localhost';

```



### 2、函数

> 可以直接被调用的程序或代码



#### 字符串函数

![image-20220220220543391](https://tva1.sinaimg.cn/large/e6c9d24egy1gzkburecjrj20t80aewfq.jpg)



```mysql
// 字符串拼接
select concat('aaaa', name) from employ where id = 1;
select concat('hello', 'world');
// 转大小写
select lower('hello');
select upper('hello');
// 左右填充
select  lpad('aaa',10,'___');
select  rpad('aaa',10,'___');
// 去掉空格
select  trim(' hello world  ');
// 截取字符串
select  substr('11aaaaaa',1,5);
```



**案例：使得员工编号小于5位的前面自动补0，例如 1变成00001；**

```mysql
update employ set  id = lpad(id,5,'0');
```



#### 数值函数

![image-20220223212333518](https://tva1.sinaimg.cn/large/e6c9d24egy1gznrhydzjlj20ri07fwex.jpg)

```mysql
// 向上取整 --13
select ceil(12.1);
// 向下取整  --1
select floor(1.1);
//返回x/y的模 --20
select mod(20,100);
// 返回0-1的随机数
select rand();
// 参数x的四舍五入，保留指定小数 ----1.2222
select round(1.222222,4);
```

生成6位随机数

```mysql
select lpad(round((select rand()*1000000),0),6,'0');
```



#### 日期函数

![image-20220223213455206](https://tva1.sinaimg.cn/large/e6c9d24egy1gznrtp75f4j20rr0bbmy7.jpg)

```mysql
// 当前日期
select curdate();
// 当前时间
select curtime();
// 当前日期和时间
select now();
// 获取data的年份
select year(now());
// 获取data的月份
select month(now());
// 获取data的日份
select day(now());
// 是时间相加
select date_add(now(),INTERVAL 100 YEAR );
select date_add(now(),INTERVAL 100 day );
select date_add(now(),INTERVAL 100 month );
// 时间差
select datediff(now(),now());
```



#### 流程函数

![image-20220223214454850](https://tva1.sinaimg.cn/large/e6c9d24egy1gzns458rvyj20rs06pq3n.jpg)



```mysql
-- IFNULL
select ifnull('',111);
select ifnull('2222',111);
select ifnull(null,'22');
-- IF
select if(true,'1','0');
-- CASE WHEN
select name, (case age when 10 then '10岁' when 11 then '11岁' else '不是这个年纪' end) as age
from employ;

```



### 3、约束

> 作用于表中字段上的规则，用于限制存储在表中的数据

#### 概述

![image-20220223215903635](https://tva1.sinaimg.cn/large/e6c9d24egy1gznsitdjlgj20ok081q3y.jpg)

Notes：

**约束是作用于表字段上的，可以在创建表/修改表的时候添加约束**

 

#### 约束演示

![image-20220223220302958](/Users/gaoshang/Library/Application Support/typora-user-images/image-20220223220302958.png)

```mysql
CREATE TABLE `table_user` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT 'id',
  `name` varchar(10) NOT NULL COMMENT 'name',
  `age` int DEFAULT NULL COMMENT '年纪',
  `status` char(1) DEFAULT '1' COMMENT 'status',
  `gender` char(1) NOT NULL COMMENT 'gender',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`),
  CONSTRAINT `table_user_chk_1` CHECK (((`age` > (0 & `age`)) <= 120))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='用户表';
```



#### 外键约束

> 目前不在使用强关联物理外键约束

```mssql
-- 添加外键
alter table  employ add constraint  foreign_key_id foreign key (id) references table_user(id);
-- 删除外键
alter table  employ drop foreign key  foreign_key_id;
```



##### 删除/更新行为

![image-20220223223305281](https://tva1.sinaimg.cn/large/e6c9d24egy1gznti7j9hyj20sr06x3zs.jpg)



### 4、多表查询



#### 多表关系

*关系分为：*

- 一对一**（单表拆分，在任意一方加入外键盘，关联另一方的外键，并设置外键为unique）**
- 一对多
- 多对多**（需要中间表维护关系）**



#### 多表查询概述

```mysql
-- 两个表关联 ---笛卡尔积
select * from employ,table_user;
```



#### 内连接

> 查询AB交集的部分数据

##### 隐式内连接

```mysql
select * from employ e,gift_history g where e.id =g.id;
```

  

##### 显式内连接

```mysql
select * from employ e inner join  gift_history g on  e.id =g.id;
```





#### 外连接

##### 左外连接

> 查询左表所有数据，以及两张表交集部分数据

```mysql
select * from employ  e left join table_user P on e.id = P.id where e.name is not null ;
```



##### 右外连接

> 查询右表所有数据，以及两张表交集部分数据

```mysql
select * from employ  e right outer join  table_user P on e.id = P.id where e.name is not null ;
```



#### 自连接

> 自己与自己关联查询

```mysql
-- 显示自连接
select * from employ  e1 join employ e2 on e1.id = e2.id;
-- 隐式自连接
select * from employ  e1 , employ e2 on e1.id = e2.id;
```

业务场景：比如员工表，需要查询员工的上级领导是谁，那么就自连接，根据自己的manager_id 等于自己表的id ，那么就可以查询对应的信息出来

Notes：自连接查询，可以是内连接查询，也可以是外连接查询



#### 联合查询

> 对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集

union，union all

对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集

```mysql
select * from employ where age>10
union
select * from employ where age<10
union
select a.id,a.age,a.name,a.status
from table_user a where a.id>1;
```

Notes：

- 对于联合查询的多张表的数据必须保持一致，字段类型也需要保持一致
- union all 会将全部的数据直接合并在一起，union会对合并之后的数据去重



#### 子查询

> sql中嵌套select语句

```mysql
select * from table_name where column =(select id form table_name2 where ....);
```



##### 标量子查询

查询结果为单个值 ---一种条件 确定

```mysql
select * from employ where id > (select id from employ where age ='11');
```



##### 列子查询

查询结果为一列--- in、not in 

```mysql
select * from employ where id in (select id from employ where age >1);
-- all 用法 满足查询出来的所有条件
select * from employ where id > all (select id from employ where age ='1');
-- some 用法 满足查询出来的其中一个条件
select * from employ where id > some (select id from employ where age ='1');
```



##### 行子查询

查询结果为一行

```mysql
-- 新语法
select * from employ where  (name,age)!=(select name,age from employ where id =00011);

```



##### 表子查询

查询结果为多行多列

```mysql
-- 多条件in 🐮🍺
select * from employ where  (name,age) in(select name,age from employ where id is not null);
-- 子查询当一个表
select * from (select * from employ where time<now()) as a;
```





















