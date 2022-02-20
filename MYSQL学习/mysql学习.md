#    MYSQL学习

> 开始学习mysql了，奥力给⛽️⛽️



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
CREate Table table_name (
'id' varchar(50) comment '注释',
'id1' varchar(50) comment '注释',
'id2' varchar(50) comment '注释'
)comment '表注释';
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
alter table_name drop 字段名;
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
insert into table_name select *from table_name2;
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



#### 日期函数



#### 流程函数



























