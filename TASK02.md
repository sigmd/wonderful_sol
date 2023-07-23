## 第二章 基础查询与排序

#### 1. SELECT语句

```mysql
SELECT <列名>, 
  FROM <表名>;
```

SELECT子句中列举了希望从表中查询出的列的名称，而FROM子句则指定了选取出数据的表的名称。

#### 2.WHERE语句

```mysql
SELECT <列名>, ……
  FROM <表名>
 WHERE <条件表达式>;
```

SELECT 语句通过WHERE子句来指定查询数据的条件。在WHERE 子句中可以指定“某一列的值和这个字符串相等”或者“某一列的值大于这个数字”等条件。执行含有这些条件的SELECT语句，就可以查询出只符合该条件的记录了。

#### 3.聚合函数

- SUM：计算表中某数值列中的合计值
- AVG：计算表中某数值列中的平均值
- MAX：计算表中任意列中数据的最大值，包括文本类型和数字类型
- MIN：计算表中任意列中数据的最小值，包括文本类型和数字类型
- COUNT：计算表中的记录条数（行数）

删除重复值

```mysql
SELECT COUNT(DISTINCT product_type)
  FROM product;
```

- COUNT 聚合函数运算结果与参数有关，COUNT(*) / COUNT(1) 得到包含 NULL 值的所有行，COUNT(<列名>) 得到不包含 NULL 值的所有行。
- 聚合函数不处理包含 NULL 值的行，但是 COUNT(*) 除外。
- MAX / MIN 函数适用于文本类型和数字类型的列，而 SUM / AVG 函数仅适用于数字类型的列。
- 在聚合函数的参数中使用 DISTINCT 关键字，可以得到删除重复值的聚合结果。

#### 4.分组

```mysql
SELECT <列名1>,<列名2>, <列名3>, ……
  FROM <表名>
 GROUP BY <列名1>, <列名2>, <列名3>, ……;
```

#### 为聚合结果指定条件

HAVING 子句必须与 GROUP BY 子句配合使用，且限定的是分组聚合结果，WHERE 子句是限定数据行（包括分组列），二者各司其职，不要混淆。

#### 5.排序

```mysql
SELECT <列名1>, <列名2>, <列名3>, ……
  FROM <表名>
 ORDER BY <排序基准列1> [ASC, DESC], <排序基准列2> [ASC, DESC], ……
```

其中，参数 ASC 表示升序排列，DESC 表示降序排列，默认为升序，此时，参数 ASC 可以缺省。



***

### 练习题

#### 2.1

编写一条SQL语句，从 `product`(商品) 表中选取出“登记日期(`regist_date`)在2009年4月28日之后”的商品，查询结果要包含 `product name` 和 `regist_date` 两列。

```mysql
SELECT product_name,regist_date
FROM product
WHERE regist_data > '2009-04-28';
```

#### 2.2

请说出对product 表执行如下3条SELECT语句时的返回结果。

①

```sql
SELECT *
  FROM product
 WHERE purchase_price = NULL;
```

②

```sql
SELECT *
  FROM product
 WHERE purchase_price <> NULL;
```

③

```sql
SELECT *
  FROM product
 WHERE product_name > NULL;
```

答：均为一条记录都没有

#### 2.3

`2.2.3` 章节中的SELECT语句能够从 `product` 表中取出“销售单价（`sale_price`）比进货单价（`purchase_price`）高出500日元以上”的商品。请写出两条可以得到相同结果的SELECT语句。执行结果如下所示：

```mysql
product_name | sale_price | purchase_price 
-------------+------------+------------
T恤衫        | 　 1000    | 500
运动T恤      |    4000    | 2800
高压锅       |    6800    | 5000
```

```mysql
SELECT product_name,sale_price,purchase_price
  FROM product
 WHERE sale_price - purchase_price >= 500;
```

#### 2.4

请写出一条SELECT语句，从 `product` 表中选取出满足“销售单价打九折之后利润高于 `100` 日元的办公用品和厨房用具”条件的记录。查询结果要包括 `product_name`列、`product_type` 列以及销售单价打九折之后的利润（别名设定为 `profit`）。

```mysql
SELECT product_name,product_type,sale_price * 0.9 - purchase_price AS profit
FROM product
WHERE sale_price * 0.9 - purchase_price > 100 AND (product_type = '办公用品' OR product_type = '厨房用具');
```

#### 2.5

请指出下述SELECT语句中所有的语法错误。

```mysql
SELECT product_id, SUM（product_name）
--本SELECT语句中存在错误。
  FROM product 
 GROUP BY product_type 
 WHERE regist_date > '2009-09-01';
```

答：GROUP语句和WHERE顺序反了；字符类型不能SUM；product_id 存在于SELECT语句中，但不存在于GROUP语句

#### 2.6

请编写一条SELECT语句，求出销售单价（ `sale_price` 列）合计值大于进货单价（ `purchase_price` 列）合计值1.5倍的商品种类。执行结果如下所示。

```mysql
product_type | sum  | sum 
-------------+------+------
衣服         | 5000 | 3300
办公用品      |  600 | 320
```

```mysql
SELECT product_type,SUM(sale_price),SUM(purchase_price)
FROM product
GROUP BY product_type
HAVING SUM(sale_price)>1.5*SUM(purchase_price);
```

#### 2.7

此前我们曾经使用SELECT语句选取出了product（商品）表中的全部记录。当时我们使用了 `ORDER BY` 子句来指定排列顺序，但现在已经无法记起当时如何指定的了。请根据下列执行结果，思考 `ORDER BY` 子句的内容。

```mysql
SELECT *
FROM product
ORDER BY -regist_date,sale_price;
```

