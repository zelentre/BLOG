---
title: MySQL学习
date:  2021-07-05 10:37:49  
categories: 
 - DB
 - MySQL
tag：MySQL
---

# MySQL学习

<!--more-->

## 数据库事务

数据库事务是指作为单个逻辑工作单元执行的一系列操作，这些操作要么全做要么全不做，是一个不可分割的工作单位。

数据库事务的四大特性(简称ACID)是： 

**(1) 原子性(Atomicity)**

事务的原子性指的是，事务中包含的程序作为数据库的逻辑工作单位，它所做的对数据修改操作要么全部执行，要么完全不执行。这种特性称为原子性。

例如银行取款事务分为2个步骤(1)存折减款(2)提取现金。不可能存折减款，却没有提取现金。2个步骤必须同时完成或者都不完成。

**(2)一致性(Consistency)**    

事务的一致性指的是在一个事务执行之前和执行之后数据库都必须处于一致性状态。这种特性称为事务的一致性。假如数据库的状态满足所有的完整性约束，就说该数据库是一致的。

例如完整性约束a+b=10，一个事务改变了a，那么b也应随之改变。

**(3)分离性(亦称独立性Isolation)**

分离性指并发的事务是相互隔离的。即一个事务内部的操作及正在操作的数据必须封锁起来，不被其它企图进行修改的事务看到。假如并发交叉执行的事务没有任何控制，操纵相同的共享对象的多个并发事务的执行可能引起异常情况。

**(4)持久性(Durability)**

持久性意味着当系统或介质发生故障时，确保已提交事务的更新不能丢失。即一旦一个事务提交，DBMS保证它对数据库中数据的改变应该是永久性的，即对已提交事务的更新能恢复。持久性通过数据库备份和恢复来保证。 

## 什么是事务隔离？

任何支持事务的数据库，都必须具备四个特性，分别是：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability），也就是我们常说的事务ACID，这样才能保证事务（（Transaction）中数据的正确性。

而事务的隔离性就是指，多个并发的事务同时访问一个数据库时，一个事务不应该被另一个事务所干扰，每个并发的事务间要相互进行隔离。

如果没有事务隔离，会出现什么样的情况呢？

假设我们现在有这样一张表（T），里面记录了很多牛人的名字，我们不进行事务的隔离看看会发生什么呢？

第一天，事务A访问了数据库，它干了一件事情，往数据库里加上了新来的牛人的名字，但是没有提交事务。

insert into T values (4, '牛D');

这时，来了另一个事务B，他要查询所有牛人的名字。

select Name from T;

这时，如果没有事务之间没有有效隔离，那么事务B返回的结果中就会出现“牛D”的名字。这就是“**脏读（dirty read）**”。

第二天，事务A访问了数据库，他要查看ID是1的牛人的名字，于是执行了

select Name from T where ID = 1;

这时，事务B来了，因为ID是1的牛人改名字了，所以要更新一下，然后提交了事务。

update T set Name = '不牛' where ID = 1;

接着，事务A还想再看看ID是1的牛人的名字，于是又执行了

select Name from T where ID = 1;

结果，两次读出来的ID是1的牛人名字竟然不相同，这就是**不可重复读（unrepeatable read）**。

第三天，事务A访问了数据库，他想要看看数据库的牛人都有哪些，于是执行了

select * from T;

这时候，事务B来了，往数据库加入了一个新的牛人。

insert into T values(4, '牛D');

这时候，事务A忘了刚才的牛人都有哪些了，于是又执行了。

select * from T;

结果，第一次有三个牛人，第二次有四个牛人。

相信这个时候事务A就蒙了，刚才发生了什么？这种情况就叫“幻读（phantom problem）”。

为了防止出现脏读、不可重复读、幻读等情况，我们就需要根据我们的实际需求来设置数据库的隔离级别。

数据库都有哪些隔离级别呢？

一般的数据库，都包括以下四种隔离级别：

读未提交（Read Uncommitted）读提交（Read Committed）可重复读（Repeated Read）串行化（Serializable）如何使用这些隔离级别，那就需要根据业务的实际情况来进行判断了。

我们接下来就看看这四个隔离级别的具体情况

读未提交（Read Uncommitted）

读未提交，顾名思义，就是可以读到未提交的内容。

因此，在这种隔离级别下，查询是不会加锁的，也由于查询的不加锁，所以这种隔离级别的一致性是最差的，可能会产生“脏读”、“不可重复读”、“幻读”。

如无特殊情况，基本是不会使用这种隔离级别的。

读提交（Read Committed）

读提交，顾名思义，就是只能读到已经提交了的内容。

这是各种系统中最常用的一种隔离级别，也是SQL Server和Oracle的默认隔离级别。这种隔离级别能够有效的避免脏读，但除非在查询中显示的加锁，如：

select * from T where ID=2 lock in share mode;

select * from T where ID=2 for update;

不然，普通的查询是不会加锁的。

那为什么“读提交”同“读未提交”一样，都没有查询加锁，但是却能够避免脏读呢？

这就要说道另一个机制“快照（snapshot）”，而这种既能保证一致性又不加锁的读也被称为“**快照读（Snapshot Read）**”

假设没有“快照读”，那么当一个更新的事务没有提交时，另一个对更新数据进行查询的事务会因为无法查询而被阻塞，这种情况下，并发能力就相当的差。

而“快照读”就可以完成高并发的查询，不过，“读提交”只能避免“脏读”，并不能避免“不可重复读”和“幻读”。

可重复读(Repeated Read)

可重复读，顾名思义，就是专门针对“不可重复读”这种情况而制定的隔离级别，自然，它就可以有效的避免“不可重复读”。而它也是MySql的默认隔离级别。

在这个级别下，普通的查询同样是使用的“快照读”，但是，和“读提交”不同的是，当事务启动时，就不允许进行“修改操作（Update）”了，而“不可重复读”恰恰是因为两次读取之间进行了数据的修改，因此，“可重复读”能够有效的避免“不可重复读”，但却避免不了“幻读”，因为幻读是由于“插入或者删除操作（Insert or Delete）”而产生的。	

串行化（Serializable）

这是数据库最高的隔离级别，这种级别下，事务“串行化顺序执行”，也就是一个一个排队执行。

这种级别下，“脏读”、“不可重复读”、“幻读”都可以被避免，但是执行效率奇差，性能开销也最大，所以基本没人会用。

总结一下

为什么会出现“脏读”？因为没有“select”操作没有规矩。

为什么会出现“不可重复读”？因为“update”操作没有规矩。

为什么会出现“幻读”？因为“insert”和“delete”操作没有规矩。

“读未提（Read Uncommitted）”能预防啥？啥都预防不了。

“读提交（Read Committed）”能预防啥？使用“快照读（Snapshot Read）”，避免“脏读”，但是可能出现“不可重复读”和“幻读”。

“可重复读（Repeated Red）”能预防啥？使用“快照读（Snapshot Read）”，锁住被读取记录，避免出现“脏读”、“不可重复读”，但是可能出现“幻读”。

“串行化（Serializable）”能预防啥？排排坐，吃果果，有效避免“脏读”、“不可重复读”、“幻读”，不过效果谁用谁知道。

delemiter 分隔符 除\符号外，任何字符都可以用作语句分隔符

## 存储过程

drop procedure if exists  name  删除存储过程

```sql
/*
 使用存储过程
    MySQL支持IN（传递给存储过程）、OUT（从存储过程传出，如下面的例子）、INOUT（对存储过程传入和传出）类型的参数。存储过程的代码位于BEGIN和END语句内，它们是一系列SELECT语句，用来检索值，然后保存到相应的变量（通过指定INTO关键字）
 参数的数据类型：
 	存储过程的参数允许的数据类型与表钟使用的数据类型相同。注意，记录集不是允许的类型，因此，不能通过一个参数返回多个行和列。这是就是下面例子使用3个参数（和3条SELECT语句）的原因。
*/
DROP PROCEDURE IF EXISTS productpricing;
CREATE　PROCEDURE productpricing(
    OUT pl DECIMAL(8,2),OUT ph DECIMAL(8,2),OUT pa DECIMAL(8,2)
) 
BEGIN 
	SELECT　Min(prod_price)
	INTO pl
	FROM products;
	SELECT Max(prod_price)
	INTO ph
	FROM products;
	SELECT Avg(prod_price)
	INTO pa
	FROM products;
END;
/*
	调用存储过程，必须指定三个变量名（参数名可以随便指定）
	分析：由于此存储过程要求3个参数，因此必须正好传递3个参数，所以这条CALL语句给出3个参数。他们是存储过程将保存结果的3个变量的名称
	变量名 所有MySQL变量都必须以@开始。
*/
CALL productpricing(@pl,@ph,@pa)
/*
	在调用时，这条语句并不显示任何数据。它返回以后可以显示（或在其他处理中使用）的变量。
	如下，搜索产品的平均价格
*/
SELECT @pa;
# 获取三个值
SELECT @pl,@ph,@pa;
# IN 使用
CREATE　PROCEDURE ordertotal(
    IN onumber INT,
    OUT ototal DECIMAL(8,2)
) 
BEGIN 
	SELECT SUM(item_price*quantity)
	FROM orderitems
	WHERE order_num = onumber
	INTO　ototal;
END;
CALL ordertotal(20005,@total);
SELECT @total;
-- 游标cursor。。
```

## 触发器

​	触发器是MySQL响应以下任意语句而自动执行的一条MySQL语句（或位于BEGIN和END语句之间的一组语句）：delete、insert、update，其他MySQL语句不支持触发器。

### 一、创建触发器

- 唯一的触发器名

- 触发器的关联表

- 触发器应该相应的活动（delete、insert、update）

- 触发器何时执行（处理之前或之后）

  **保持每个数据库的触发器名唯一**	在MySQL5中，触发器名必须在每个表中唯一，但不是在每个数据库中唯一。这表示同一数据库中的两个表可具有相同名字的触发器。这在其他每个数据库触发器名必须唯一的DBMS中是不允许的，而且以后的MySQL版本很有可能会使命名规则更为严格。因此，现在最好是在数据库范围内使用唯一的触发器名。

  [触发器参考](https://zhuanlan.zhihu.com/p/158670286)

##　管理事务处理

​	**并非所有引擎都支持事务处理**	MySQL支持几种基本的数据库引擎。MyISAM和InnoDB是两种最常使用的引擎。前者不支持明确的事务处理管理，后者支持。若你的应用中需要事务处理功能，则一定要使用正确的引擎类型。

​	事务处理可以用来维护数据库的完整性，它保证成批的MySQL操作要么完全执行，要么完全不执行。

- 事务（transaction）指一组SQL语句。

- 回退（rollback）指撤销指定SQL语句的过程。

- 提交（commit）指将为存储的SQL语句结果写入数据库表。

- 保留点（savepoint）指事务处理中设置的历史占位符（placehoder），你可以对它发布回退（与回退整个事务处理不同）。

### 使用ROLLBACK

  ```sql
  SELECT * FROM ordertotals;
  START TRANSACTION;
  DELETE FROM ordertotals;
  SELECT * FROM ordertotals;
  ROLLBACK;
  SELECT * FROM ordertotals;
  ```

  **可回退的语句**	事务处理用来管理insert、update和delete语句。你不能回退select语句（这样做也没有什么意义）你不能回退create或drop操作。事务处理块中可以使用这两条语句，但如果你执行回退，它们不会被撤销。

### 使用COMMIT

​	一般的MySQL语句都是直接针对数据库表执行和编写的。这就是所谓的隐含提交（implicit commit），即提交（写或保存）操作时自动进行的。但是在事务处理模块中，提交不会隐含的进行。为进行明确的提交，使用commit语句

```sql
start transaction;
delete from orderitems where order_num = 20010;
delete from orders where order_num = 20010;
commit;
/*
	在这个例子中，从系统中完全删除订单20010。因为涉及更新两个数据库表orders和orderitems，所以使用事务处理块俩保证订单不被部分删除。最后的commit语句仅在不出错时写出更改。如果第一条delete起作用，但第二条失败，则delete不会提交（实际上，它是被自动撤销的）。
*/
```

​	**隐含事务关闭**	当commit或rollback语句执行后，事务会自动关闭(将来的更改会隐含提交)。

### 使用保留点

​	为了支持回退部分事务处理，必须能在事务处理块中合适的位置放置占位符。这样，如果需要回退，可以回退到某个占位符。这些占位符称为保留点。为了创建占位符，可如下使用savepoint语句
`savepoint delete1;`每个保留点都取标识它的唯一名字，以便在回退时，MySQL知道要回退到何处。回退到本例给出的保留点  `rollback to delete1;`

​	**保留点越多越好**	可以子啊MySQL代码中设置任意多的保留点，越多越好。保留点越多，就越能按照自己的意愿灵活的进行回退。

​	**释放保留点**	保留点在事务处理完成（执行一条rollback或commit）后自动释放。自MySQL5以来，也可以用 release savepoint明确地释放保留点。

### 更改默认的提交行为

​	默认的MySQL行为是自动提交所有更改。任何时候你执行一条MySQL语句，该语句实际上都是针对表执行的，而且所作的更改立即生效。为指示MySQL不自动提交更改，需要使用以下语句：

```sql
set autocommit = 0;
/*
	autocommit标志决定是否自动提更改，不管有没有commit语句。设置autocommit为0（MySQL 0为false，非0为true）指示MySQLMySQL不自动提交更改（直到autocommit被设置为真为止）
*/
```

​	**标志为连接专用**	autocommit标志是针对每个连接而不是服务器的。
