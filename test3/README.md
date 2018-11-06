# 实验3：创建分区表

## 实验目的：

掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 实验内容：
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用**你自己的账号创建本实验的表**，表创建在上述3个分区，自定义分区策略。
- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。

## 实验步骤
## 1.创建主表和从表
用户名：NEW_USER_WTS; 设置主表主键 order_id, 从表外键 order_id; 主表的分区策略是按照日期范围--2016，2017，2018，分别对应分区USERS, USERS02,USERS03。
在主表orders和从表order_details之间建立引用分区
在NEW_USER_WTS用户中创建两个表：orders（订单表）和order_details（订单详表），两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储。
创建orders表的语句是：

```sql
CREATE TABLE ORDERS
(
order_id NUMBER(10,0)NOT NULL,
customer_name VARCHAR2(40 BYTE)NOT NULL,
customer_tel VARCHAR2(40 BYTE)NOT NULL,
order_date DATE NOT NULL,
employee_id NUMBER(6,0) NOT NULL,
discount NUMBER(8,2)DEFAULT 0,
trade_receivable NUMBER(8,2)DEFAULT 0
)
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
BUFFER_POOL DEFAULT
)
PARTITION BY RANGE (order_date)  //RANGE分区类型
(
PARTITION partition_before_2017 VALUES LESS THAN (
TO_DATE(' 2017-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS,

PARTITION partition_before_2018 VALUES LESS THAN (
TO_DATE(' 2018-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS02,

PARTITION partition_before_2019 VALUES LESS THAN (
TO_DATE(' 2019-01-01 00: 00: 00', 'SYYYY-MM-DD HH24: MI: SS',
'NLS_CALENDAR=GREGORIAN'))TABLESPACE USERS02
);
```

创建order_details表的语句如下：
```sql
CREATE TABLE order_details
(
id NUMBER(10,0)NOT NULL,
order_id NUMBER(10,0)NOT NULL,
product_id VARCHAR2(40 BYTE)NOT NULL,
product_num NUMBER(8,2) NOT NULL,
product_price NUMBER(8,2) NOT NULL,
CONSTRAINT order_details_fk1 FOREIGN KEY (order_id)
REFERENCES orders ( order_id )
ENABLE
)
TABLESPACE USERS
PCTFREE 10 
INITRANS 1
STORAGE( BUFFER_POOL DEFAULT )
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1);
```


## 2.往两张表插入万条以上数据

先插入主表数据，再插入从表数据，三个表空间的数据分布是平均的。
```sql
//主表中插入单条数据的sql语句
INSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('WANG', '152', to_date ( '2016-12-20 18:31:34' , 'YYYY-MM-DD HH24:MI:SS' ), 001, 16, 6);
INSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('ZHAO', '152', to_date ( '2017-12-20 18:31:34' , 'YYYY-MM-DD HH24:MI:SS' ), 001, 17, 7);
INSERT INTO orders(customer_name, customer_tel, order_date, employee_id, trade_receivable, discount) VALUES('LI', '152', to_date ( '2018-12-20 18:31:34' , 'YYYY-MM-DD HH24:MI:SS' ), 001, 18, 8);
//主表中重复插入，达到万条数据，，说明：创建了序列 SEQ_ID， 触发器tr_IDADD（当插入主表单条数据时候自动插入order_id值【唯一值】）
insert into orders
select *
from orders;
```

- 主表数据概览：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/分区主表数据概览.png )

```sql
//从表中插入单条数据的sql语句
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(233, 233, 233, 255.66);
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(332, 332, 332, 266.55);
insert into order_details(id, PRODUCT_ID, PRODUCT_NUM, PRODUCT_PRICE) VALUES(323, 323, 323, 265.56);
//从表中重复插入，达到万条数据，，说明：创建了序列 SEQ_ID， 触发器tr_DETAILS_IDADD（当插入从表单条数据时候自动插入order_id值【唯一值】）
insert into order_details
select *
from order_details;
```

- 从表数据概览：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/分区从表数据概览.png )

## 3.联合查询主表和从表（分区）
```sql
//查询PARTITION_BEFORE_2017分区中两张表的数据（部分列）
SELECT
    orders.order_id,
    orders.order_date,
    order_details.order_id,
    TRADE_RECEIVABLE,
    PRODUCT_ID,
    PRODUCT_PRICE
FROM orders partition (PARTITION_BEFORE_2017) LEFT JOIN order_details partition (PARTITION_BEFORE_2017)
ON (orders.order_id = order_details.order_id);
```

- 查询结果概览：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/分区查询_PartitionBefore2017.png )

- 查询执行计划：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/分区查询_执行计划.png )

## 4.联合查询两张表（不分区）；对比实验分析
```sql
//查询两张表的数据（所有），表未分区
select * from orders_nopartition, order_details_nopartition where orders_nopartition.order_id = order_details_nopartition.order_id(+);
```
- 查询结果概览：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/未分区查询_查询结果.png )

- 查询执行计划：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test3/未分区查询_执行计划.png )

- 对比分析：
两张表中数据均为12288条，从表ORDER_DETAILS跟主表ORDERS建立了主外键，从表的分区策略同
主表一样。
我对分区表的partition_before_2017分区进行了查询，查询结果完全正确，可以看出CPU的cost=392,
consistent gets=153，相较于未分区的表查询---CPU的cost=34,consistent gets=91，，分区表查
询的资源占比明显高出很多。不过分区查询的时间平均为1.5秒，未分区查询的时间平均为3.5秒，显然分区
后表的查询速度快了不少。
对于表的分区，逻辑上表实质还是一张完整的表，不过是把数据按照某种分区判定方法（范围分区，Hash分区，
List分区，混合分区）而存放在多个表空间上面，好处明显就是查询时候不会扫描完整张表，提高了查询速度。
