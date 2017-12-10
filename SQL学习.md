# SQL学习
## DISTINCT

> 用于返回唯一不同的值

```
SELECT DISTINCT Company FROM Orders
```

## ORDER BY

> 用于根据置顶的列对结果集进行排序，加DESC关键字降序排序

```
SELECT Company, OrderNumber Orders ORDER BY Company
```

## INSERT INTO

> 用于向表格中插入新的行

```
INSERT INTO table_name (列1,2,3,...)VALUES (值1,2,3,...)
```

## UPDATE

> 修改表中数据

```
SELECT table_name SET 列 = 新值 WHERE 列 = 某值
```

## TOP子句

> 规定要返回的记录的数目

```
SELECT colunm_name(s) FROM table_name LIMIT number
```

## LIKE操作符

> 在WHERE子句中搜索列中的指定模式

```
SELECT column_name(s) FROM table_name WHERE column_name LIKE pattern
```

## UNION & UNION ALL

> 合并两个或多个SELECT查询的结果

```
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2

SELECT column_name(s) FROM table_name1
UNION ALL
SELECT column_name(s) FROM table_name2
```

## SELECT INTO

> 只做备份复件

```
SELECT *
INTO new_table_name
FROM old_table_name
```


