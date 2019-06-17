---
date: "2019-06-11T15:28:48+08:00"
draft: false
title: "sql小技巧(1) order by field"
tags: ["Mysql", "sql","sql小技巧"]
series: ["mysql"]
categories: ["Mysql"]
toc: true
---

## 序
在工作中，遇到了一个需求，需要通过`where in $condition`查询，
然后查询出来的结果又需要按照`$condition`来排序。

此时就可以用到`order by field`来进行排序了。

## 例子

这里我以`id`为条件做举例。

### 当不排序的时候

```sql
SELECT
	`id`,
	`name` 
FROM
	rb_robot 
WHERE
	id IN ( 51, 50, 49 ) 
```
此时返回的结果：

| id | name  |
| ------- | ------ |
| 49 | 三号 | 
| 50 | 四号 | 
| 51 | 五号 | 

### 当排序的时候

```sql
SELECT
	`id`,
	`name` 
FROM
	rb_robot 
WHERE
	id IN ( 51, 50, 49 ) 
ORDER BY
	FIELD( id, 51, 50, 49 )
```
此时返回的结果：

| id | name  |
| ------- | ------ |
| 51 | 五号 | 
| 50 | 四号 | 
| 49 | 三号 | 