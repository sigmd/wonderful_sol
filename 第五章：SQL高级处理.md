## 第五章：SQL高级处理

### 1.窗口函数

**定义**

　　窗口函数又称为是OLAP函数，（注１：OLAP和OLTP的区别）OLAP是对数据库数据进行实时分析处理，窗口函数就是为了实现OLAP添加的标准SQL功能。（注２：MySQL８.０以下版本不能使用窗口函数）

**窗口函数和普通聚合函数的区别**

　　聚合函数可以理解成多进一出，将多条记录汇总成一条。而窗口函数是有多少条记录参与执行，就会有多少条记录返回。同时窗口函数还会有group by的分组功能和order by的排序功能。(注3：group by是分组聚合，窗口函数中是分组)

**基本语法**

```mysql
<窗口函数> OVER ([ PARTITION BY <列名> ]
                     [ ORDER BY <排序用列名> ])  
```

来对每个部分进行解读：

1. •窗口函数：就是根据需求使用的分析函数，下文会介绍。常用的有row_number(), sum()等。
2. •over(): 窗口函数的核心，用来指定函数执行的窗口范围。(窗口函数必须要有over())



**partition by子句**

​	可选参数，指示如何将查询行划分为组，类似于 GROUP BY 子句的分组功能，但是 PARTITION BY 子句并不具备 GROUP BY 子句的汇总功能，并不会改变原始表中记录的行数。

​    partition by是用来让窗口函数针对字段进行分组，然后对不同的分组进行计算。

**思考：如果不用窗口函数，直接使用group by的结果是什么样的?**

partition by和group by进行区别：partition by是分组，group by是分组聚合。



**order by子句**

可选参数，指示如何对每个分区中的行进行排序，即决定窗口内，是按那种规则(字段)来排序的。

窗口函数中的order by和普通的order by效果一致，也可以结合asc或desc实现升序和降序作用，默认是asc升序。



#### 常用函数

**1、专用函数(重点)**

**1）rank**

计算排序时，如果存在相同位次的记录，则会跳过之后的位次。

例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、4 位……

**2）dense_rank**

同样是计算排序，即使存在相同位次的记录，也不会跳过之后的位次。

例）有 3 条记录排在第 1 位时：1 位、1 位、1 位、2 位……

**3) row_number**

赋予唯一的连续位次。

例）有 3 条记录排在第 1 位时：1 位、2 位、3 位、4 位

**2、聚合函数**

聚合函数在窗口函数中的使用方法和之前的专用窗口函数一样，只是出来的结果是一个**累计**的聚合函数值。

**1) sum**  **2) count ** **3) avg**  **4) max/min**

**3、前后函数**

**1) lag**  **2）lead**  

**4、头尾函数**

**1）first_value**  **2）last_value**

**5、其他**

**1) ntile**



**窗口设置** 

row|range between 上边界规则 and 下边界规则

窗口函数中窗口的大小也可以让我们自由的设置。

1. •current row：当前行
2. •n preceding：前n行
3. •unbounded preceding：从第一条记录开始
4. •n following：往后n行
5. •unbounded following：直到结束行

```mysql
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS n PRECEDING )  
                 
<窗口函数> OVER (ORDER BY <排序用列名>
                 ROWS BETWEEN n PRECEDING AND n FOLLOWING)
```

PRECEDING（“之前”）， 将框架指定为 “截止到之前 n 行”，加上自身行

FOLLOWING（“之后”）， 将框架指定为 “截止到之后 n 行”，加上自身行



注：

- 原则上，窗口函数只能在SELECT子句中使用。

- 窗口函数OVER 中的ORDER BY 子句并不会影响最终结果的排序。其只是用来决定窗口函数按何种顺序计算。

  

### 2.GROUPING运算符

常规的GROUP BY 只能得到每个分类的小计，有时候还需要计算分类的合计，可以用 ROLLUP关键字。

```mysql
GROUP BY product_type, regist_date WITH ROLLUP;  
```



### 3.存储过程和函数

基本语法：

```mysql
[delimiter //]($$，可以是其他特殊字符）
CREATE
    [DEFINER = user]
    PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...] 
[BEGIN]
  routine_body
[END//]($$，可以是其他特殊字符）
```

- `IN` 是入参。每个参数默认都是一个 `IN` 参数。如需设定一个参数为其他类型参数，请在参数名称前使用关键字 `OUT` 或 `INOUT` 。一个IN参数将一个值传递给一个过程。存储过程可能会修改这个值，但是当存储过程返回时，调用者不会看到这个修改。
- `OUT` 是出参。一个 `OUT` 参数将一个值从过程中传回给调用者。它的初始值在过程中是 `NULL` ，当过程返回时，调用者可以看到它的值。
- `INOUT` ：一个 `INOUT` 参数由调用者初始化，可以被存储过程修改，当存储过程返回时，调用者可以看到存储过程的任何改变。



### 4.预处理声明 PREPARE Statement

MySQL 从4.1版本开始引入了 `PREPARE Statement` 特性，使用 `client/server binary protocol` 代替 `textual protocol`，其将包含占位符 （） 的查询传递给 MySQL 服务器

基本语法：

```mysql
PREPARE stmt_name FROM preparable_stmt
```

`MySQL PREPARE Statement` 使用步骤如下：

1. PREPARE – 准备需要执行的语句预处理声明。
2. EXECUTE – 执行预处理声明。
3. DEALLOCATE PREPARE – 释放预处理声明。

这里使用 `shop` 中的 `product` 表进行演示。

首先，定义预处理声明如下：

```sql
PREPARE stmt1 FROM 
	'SELECT 
   	    product_id, 
            product_name 
	FROM product
        WHERE product_id = ?';
```

其次，声明变量 `pcid`，代表商品编号，并将其值设置为 `0005`：

```sql
SET @pcid = '0005'; 
```

第三，执行预处理声明：

```sql
EXECUTE stmt1 USING @pcid;
```

第四，为变量 `pcid` 分配另外一个商品编号：

```sql
SET @pcid = '0008'; 
```

第五，使用新的商品编号执行预处理声明：

```sql
EXECUTE stmt1 USING @pcid;
```

最后，释放预处理声明以释放其占用的资源：

```sql
DEALLOCATE PREPARE stmt1;
```



# 练习题

1.请说出针对本章中使用的 product（商品）表执行如下 SELECT 语句所能得到的结果。

```mysql
SELECT  product_id
       ,product_name
       ,sale_price
       ,MAX(sale_price) OVER (ORDER BY product_id) AS Current_max_price
  FROM product;
```

答：按商品编号进行排序，并给出到目前商品的最大销售价格

2.继续使用product表，计算出按照登记日期（regist_date）升序进行排列的各日期的销售单价（sale_price）的总额。排序是需要将登记日期为NULL 的“运动 T 恤”记录排在第 1 位（也就是将其看作比其他日期都早）

```mysql
SELECT  product_id
       ,product_name
			 ,regist_date
       ,sale_price
       ,SUM(sale_price) OVER (ORDER BY regist_date) AS sum_price
  FROM product;
```

3.思考题

① 窗口函数不指定PARTITION BY的效果是什么？

② 为什么说窗口函数只能在SELECT子句中使用？实际上，在ORDER BY 子句使用系统并不会报错。

答：针对排序列进行全局排序 如果在 WHERE, GROUP BY, HAVING 使用了窗⼝函数，就是说提前进行了⼀次排序，排序之后再去除记录、汇总、汇总过滤，第⼀次排序结果就是错误的，没有实际意义。而ORDER BY 语句执行顺序在SELECT 语句之后，自然是可以使用的。

4.使用存储过程创建20个与 `shop.product` 表结构相同的表

````mysql
DELIMITER//
CREATE PROCEDURE set_table()
BEGIN
  DECLARE i CHAR(2);
  DECLARE TABLE_NAME VARCHAR(20);
  SET i = '01';            
  WHILE i <= 20 DO
  SET TABLE_NAME = CONCAT('table',i);
  SET @csql = CONCAT('create table ',TABLE_NAME,' like shop.product');
  PREPARE create_stmt FROM @csql;
  EXECUTE create_stmt;
  SET i = LPAD(CAST(i+1 AS CHAR),2,0);
  END WHILE;    
END//
````