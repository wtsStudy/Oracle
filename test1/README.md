# Oracle
# 实验一：分析SQL执行计划，执行SQL语句的优化指导

## 实验内容：
- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

## 教材中的查询语句

- 查询1：

```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```
运行结果：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/query_1.png)

- 查询2：
```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
运行结果：
![运行结果](https://github.com/wtsStudy/Oracle/blob/master/query_2.PNG)

>我认为第二条查询语句更优。WHERE是一个约束声明，在查询数据库的结果返回之前对数据库中的查询条件进行约束，在结果返回之前起作用；HAVING是一个过滤声明，所谓过滤是在查询数据库的结果返回之后进行过滤，在结果返回之后起作用。

- 自己定义的查询：
```SQL
SET AUTOTRACE ON
SELECT R.REGION_NAME AS "洲地区名",COUNT(C.REGION_ID)AS "国家数"
FROM HR.REGIONS R, HR.COUNTRIES C
WHERE R.REGION_ID=C.REGION_ID
GROUP BY R.REGION_NAME, R.REGION_ID
HAVING R.REGION_ID IN ('1', '3');
```
运行结果：
![运行结果](https://github.com/wtsStudy/Oracle/edit/master/test1/query_3.PNG)

>根据前面的示例运行分析，我仿照查询2的语句，设计了我自己的查询语句，以查询人力资源管理系统中的表COUNTRY和REGION两张表，国家分别在欧洲（Europe）与亚洲(Asia)地区的个数。
