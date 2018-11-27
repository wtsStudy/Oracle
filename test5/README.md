# 实验5：PL/SQL编程

## 实验目的：
    了解PL/SQL语言结构
    了解PL/SQL变量和常量的声明和使用方法
    学习条件语句的使用方法
    学习分支语句的使用方法
    学习循环语句的使用方法
    学习常用的PL/SQL函数
    学习包，过程，函数的用法。

## - 实验场景：
- 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
- 本实验以实验四为基础

## 实验内容：
1. 创建一个包(Package)，包名是MyPack。
2. 在MyPack中创建一个函数SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销售额之和。函数SaleAmount要求输入的参数是部门号，输出部门的销售金额。
3. 在MyPack中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的ID,姓名，销售总金额。信息用dbms_output包中的put或者put_line函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。比如下面显示5个员工的信息：
4. 由于订单只是按日期分区的，上述统计是全表搜索，因此统计速度会比较慢，如何提高统计的速度呢？

## 实验过程步骤：(我的Oracle账户名： NEW_USER_WTS )
#1.创建好包，编写程序，编译保存。
    实现语句：
```sql
    create or replace PACKAGE BODY MyPack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
    N NUMBER(20,2);
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;

  PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
  AS
    LEFTSPACE VARCHAR(2000);
    SUMTRADE NUMBER(20,2);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        --查询该员工的销售总金额，放在变量 SUMTRADE 中输出值
        SELECT SUM(O.TRADE_RECEIVABLE) into SUMTRADE FROM orders o
        WHERE O.EMPLOYEE_ID=V.EMPLOYEE_ID;
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                             V.EMPLOYEE_ID||' '||v.NAME||' '||SUMTRADE||'元');
      END LOOP;
    END;
END MyPack;
```
    
创建成功截图：
    ![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test5/包创建成功截图.png )

 #2.测试：
    实现语句：
 ```sql
    --测试函数Get_SaleAmount
    select count(*) from orders;
    select MyPack.Get_SaleAmount(1) AS 部门1应收金额,MyPack.Get_SaleAmount(2) AS 部门2应收金额 from dual;
 ```
    
运行结果截图：
    ![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test5/函数测试截图.png )
    
 ```sql
    --测试过程GET_EMPLOYEES
    set serveroutput on;
DECLARE
  V_EMPLOYEE_ID NUMBER;    
BEGIN
  V_EMPLOYEE_ID := 1;
  MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;   
END;
 ```
    
运行结果截图：
    ![运行结果](https://github.com/wtsStudy/Oracle/blob/master/test5/过程测试截图.png )
