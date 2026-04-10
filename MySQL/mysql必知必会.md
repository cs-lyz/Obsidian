## SELECT

## DISTINCT
```sql
SELECT   DISTINCT vend_id ,prod_price FROM products;
```
- `DISTINCT` 作用于后面**所有列的组合**
- 它会返回 **vend_id 和 prod_price 组合起来唯一**的记录

## LIMIT
```sql
SELECT  vend_id  FROM products  LIMIT 5; -- 返回前五行
SELECT  vend_id  FROM products  LIMIT 6 OFFSET 5; --返回第5行开始的6行数据 
```
## ORDER
```sql
SELECT  name,price FROM products ORDER BY name;-- 默认就是升序
SELECT  name,price FROM products ORDER BY name,price DESC;
--先按名字升序，再按价格降价排序
```
## WHERE
```sql
SELECT   id ,prod_price FROM products where name='fafa' 
	ORDER BY id ；--要在where后面 默认不区分大小写 FAFA也会被检测出来
	
SELECT   id ,price FROM products where price (NOT) between 5 and 10
SELECT   id ,price FROM products where price (NOT) IS NULL--检测空值，不会被特意过滤出来，除非用这个
between
OR
AND 优先级比 OR 高
IN(10,15) 包括10,15
NOT
```
一般在Mysql过滤数据

## LIKE
```sql
SELECT  vend_id ,prod_price FROM products WHERE name LIKE 'jet%'
-- % 匹配任意字符出现的任意次数
-- _ 只匹配单个字符

```
## Concat
```sql
SELECT  Concat(vend_id ,'(',prod_price ,')') FROM products WHERE name LIKE 'jet%'

SELECT  Concat(name ,':(',prod_price ,')') AS title FROM products WHERE name LIKE 'jet%'    -- java可以引用这个列   一般在MySQL里面做这个比较快

SELECT id,nums,price,nums*price AS total_price FROM orderitems
```

## DATE函数
```sql
AddDate()...
CurDate()...
Day()...
Date()... 提取日期部分
Time()...
存储的日期格式为 '2005-09-01'
SELECT id,num FROM orders WHERE Date(order_date)  BETWEEN '2005-09-01' AND '2005-09-8';
```

## 行数，总和，最大最小平均
```sql
AVG()
COUNT()  统计行数
MAX()
MIN
SUM
SELECT AVG(price) AS avg_price FROM products;

SELECT COUNT(email) AS has_email FROM customers;
--只对具有email的客户计数  忽略NULl值
-- COUNT(*) 对表中行计数，包含NULL
```

## GROUP,HAVING
```sql
SELECT  id ,COUNT(price) AS total_price FROM products GROUP BY id;
-- 按商品ID分组，统计每个ID对应的商品价格数量
-- 用HAVING过滤分组
SELECT  id ,COUNT(price) AS total_price FROM products GROUP BY id 
	HAVING COUNT(price)>=100
	ORDER BY total_price;

```

<mark style="background: #BBFABBA6;">SELECT语句次序</mark>
```sql
SELECT 
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

## 子查询
不能嵌套过多子查询，性能不好

## 联结
```sql
SELECT vend_name,prod_name,prod_price
FROM vendors,products
WHERE vendors.vend_id=products.vend_id

```


## Match,Against
```sql
SELECT note_text FROM usernotes
WHERE Match(note_text) Against('rabbit')  --返回的数据按照匹配程度排序
--跟LIKE不一样

```

## INSERT
```sql
INSERT INTO customers（cust_id,
   cust_contact,
   cust_email,
   cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country）
SELECT cust_id,
   cust_contact,
   cust_email,
   cust_name,
   cust_address,
   cust_city,
   cust_state,
   cust_zip,
   cust_country
FROM custnew;             -- 把查出来的所有数据插入到另一个表里面
```

## UPDATE
**IGNORE** 关键字　如果用UPDATE语句更新多行，并且在更新这些行中的一行或多行时出现错误，则整个UPDATE操作会被取消（错误发生前更新的所有行会被恢复到它们原来的值）。即使发生错误，也要继续进行更新，可以使用IGNORE 关键字，如下所示
## PRIMARY KEY
主键可以由多列组成，但是各列的组合必须唯一

## 视图
包含的是一个SQL查询，与联结同样的查询
作用，隐藏复杂的SQL，

![[1774249191099_d.png]]
![[b81d42b78bf26d69b2f16ad241de043c.jpg]]

## 函数
```mysql
CREATE PROCEDURE productpricing()
BEGIN
	SELECT Avg(prod_price) AS priceaverage
	FROM products;
END;
-- 使用
CALL productpricing();
-- 删除
DROP PROCEDURE productpricing；
-- 查看
SHOW CREATE PROCEDURE productpricing；


-- IN 传递给存储过程 OUT 从存储过程传出 INOUT 从存储过程传入传出
DELIMITER //
CREATE PROCEDURE productpricing（
   OUT pl DECIMAL(8,2),
   OUT ph DECIMAL(8,2),
   OUT pa DECIMAL(8,2)
）
BEGIN
   SELECT Min(prod_price)
   INTO pl
   FROM products;
   SELECT Max(prod_price)
   INTO ph
   FROM products;
   SELECT Avg(prod_price)
   INTO pa
   FROM products;
END//
DELIMITER ;

CALL productpricing（@pricelow,
                    @pricehigh,
                    @priceaverage）;  -- 调用后不显示任何数据，要显示数据
SELECT @priceaverage;


DELIMITER //
CREATE PROCEDURE ordertotal（
   IN onumber INT,
   OUT ototal DECIMAL(8,2)
）
BEGIN
   SELECT Sum(item_price*quantity)
   FROM orderitems
   WHERE order_num = onumber
   INTO ototal;
END //
DELIMITER ;

CALL ordertotal(20005, @total);



DELIMITER //

-- 名称: ordertotal表
-- 参数: onumber = 订单号
--      taxable = 如果不用交税就为0，如果需要交税则为1
--      ototal = 订单总金额变量

CREATE PROCEDURE ordertotal（
   IN onumber INT,
   IN taxable BOOLEAN,
   OUT ototal DECIMAL(8,2)
） COMMENT 'Obtain order total, optionally adding tax'
BEGIN

   -- 声明 total 变量
   DECLARE total DECIMAL(8,2);
   -- 声明税率百分比
   DECLARE taxrate INT DEFAULT 6;

   -- 获取订单总金额
   SELECT Sum(item_price*quantity)
   FROM orderitems
   WHERE order_num = onumber
   INTO total;
   -- 需要交税吗？
   IF taxable THEN
      --是的，所以在总金额中加上税率
      SELECT total+(total/100*taxrate) INTO total;
   END IF;

   -- 最后，保存到输出变量
   SELECT total INTO ototal;

END //

DELIMITER ;
```
## 游标
```mysql
DELIMITER //

CREATE PROCEDURE processorders()
BEGIN
   -- 声明局部变量
   DECLARE done BOOLEAN DEFAULT 0;
   DECLARE o INT;
   DECLARE t DECIMAL(8,2);

   -- 声明游标
   DECLARE ordernumbers CURSOR
   FOR
   SELECT order_num FROM orders;

   -- 声明继续处理程序
   DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done=1;
   -- **当发生 SQLSTATE '02000' 错误时，将变量 `done` 的值设置为 1，然后继续执行后续代码。**
   -- `'02000'` 是 SQL 标准中表示 **“未找到数据”**（No Data）的状态码。  
   -- 常见场景：使用游标（CURSOR）执行 `FETCH` 时，如果没有更多行可获取，就会触发这个状态码。
   -- 创建一张表来存储结果
   CREATE TABLE IF NOT EXISTS ordertotals
      （order_num INT, total DECIMAL(8,2)）;

   -- 打开游标
   OPEN ordernumbers;

   -- 遍历所有行
   REPEAT

      -- 获取订单号
      FETCH ordernumbers INTO o;

      -- 获取此订单的总金额
      CALL ordertotal(o, 1, t);

      --在ordertotals表中插入订单和总金额
      INSERT INTO ordertotals(order_num, total)
      VALUES(o, t);

   -- 循环结束  当 done 为真时结束循环
   UNTIL done END REPEAT;

   -- 关闭游标
   CLOSE ordernumbers;

END //

DELIMITER ;
```

## 触发器
```mysql
CREATE TRIGGER newproduct AFTER INSERT ON products -- AFTER INSERT在插入之后执行
FOR EACH ROW SET @result = 1; -- 每次插入设置result值为1

DELIMITER // -- 让插入的Ca或ca变为大写CA

CREATE TRIGGER newvendor AFTER INSERT ON vendors
FOR EACH ROW   -- 如果一次插入多行（例如 `INSERT INTO vendors VALUES (...), (...), (...);`），触发器会为**每一行**分别执行一次。
BEGIN
   UPDATE vendors SET vend_state=Upper(vend_state) WHERE
      vend_id = NEW.vend_id;-- - `NEW` 关键字代表**当前正在插入的那一行**的虚拟行记录。通过 `NEW.vend_id` 可以获取该行的 `vend_id` 值。
END //

DELIMITER ;

```

```mysql
DELIMITER //

CREATE TRIGGER deleteorder BEFORE DELETE ON orders  -- 在删除之前进行 如果这个插入记录失败 那么删除也会失败
FOR EACH ROW
BEGIN
   INSERT INTO archive_orders（order_num,
                              order_date,
                              cust_id）
   VALUES（OLD.order_num,  -- 访问被删除的行 全都是只读的 不能更新
          OLD.order_date,
          OLD.cust_id）;
END//

DELIMITER ;
```


在UPDATE 触发器代码中，你可以引用一张名为OLD的虚拟表访问更新前（UPDATE语句之前）的值，引用一张名为NEW的虚拟表访问更新后的值；
• 在BEFORE UPDATE 触发器中，NEW中的值也可以被更新，这样你就可以更改将要用于UPDATE语句中的值；
• OLD中的值全都是只读的，不能更新。

## 事务
事务处理用来管理 INSERT语句、UPDATE语句和DELETE语句。你不能回滚 SELECT语句。（这样做也没有什么意义。）你也不能回滚 CREATE语句或DROP语句。事务处理块中可以使用这两个语句，但如果你执行回滚，则它们不会被撤销。
```mysql
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM orders WHERE order_num = 20010;
COMMIT;
```