参考文档：
[https://zhuanlan.zhihu.com/p/602457022](https://zhuanlan.zhihu.com/p/602457022)
[通俗易懂的学会：SQL窗口函数](https://www.zhihu.com/tardis/zm/art/92654574?source_id=1005)

## 1、窗口函数

### 1. <窗口函数>的位置，可以放以下两种函数：

1） 专用窗口函数，比如rank, dense_rank, row_number等

2） 聚合函数，如sum. avg, count, max, min等

### 2. 窗口函数有以下功能：

1）同时具有分组（partition by）和排序（order by）的功能

2）不减少原表的行数，所以经常用来在每组内排名

### 3. 注意事项

1. 窗口函数原则上只能写在select子句中

### 4. 窗口函数使用场景

1）业务需求“**在每组内排名”**，比如：

2. 排名问题：每个部门按业绩来排名

3. topN问题：找出每个部门排名前N的员工进行奖励

### 5. 示例：rank()、dense_rank()、row_number()

```SQL
select *,
   rank() over (order by 成绩 desc) as ranking,
   dense_rank() over (order by 成绩 desc) as dese_rank,
   row_number() over (order by 成绩 desc) as row_num
from 班级表
```
    
![[Pasted image 20260406132130.png]]

### 6. 示例：聚合函数sum()、avg()、count()、max()、min()作为窗口函数
```SQL
select *,
   sum(成绩) over (order by 学号) as current_sum,
   avg(成绩) over (order by 学号) as current_avg,
   count(成绩) over (order by 学号) as current_count,
   max(成绩) over (order by 学号) as current_max,
   min(成绩) over (order by 学号) as current_min
from 班级表
```
    
![[Pasted image 20260406132054.png]]
    
## 2、增删改查

1. ##   增：
    

```SQL
INSERT INTO fly_table1 (`course`,`teacher`,`price`) VALUES ('linux C/C++','fly',0.0);
```

2. ##   删
    
    ```SQL
    DELETE FROM `fly_table1` where id = 2;
    ```
    

3. ##   改
    

```SQL
UPDATE `fly_table1` SET price=price+100,course='linux MySQL' WHERE id =3;
```

4. ##   查
    

```SQL
SELECT price FROM fly_table1 WHERE course='linux C/C++';
```

## 3、表—创建，删除

1. ## table - 创建
```sql
CREATE TABLE `table_name` (column_name column_type);
```
```sql
CREATE TABLE IF NOT EXISTS `fly_table1` (
`id` INT UNSIGNED AUTO_INCREMENT COMMENT '编号',
`course` VARCHAR(100) NOT NULL COMMENT '课程',
`teacher` VARCHAR(40) NOT NULL COMMENT '讲师',
`price` DECIMAL(8,2) NOT NULL COMMENT '价格',
PRIMARY KEY ( `id` )
)ENGINE=innoDB DEFAULT CHARSET=utf8 COMMENT = '课程表';
```

2. ## table - 删除
```sql
DROP TABLE `table_name`;
```
3. ## 清空数据表
```sql
TRUNCATE TABLE `table_name`; -- 截断表 以页为单位（至少有两行数据），有自增索引的话，从初始值开始累加
DELETE TABLE `table_name`; -- 逐行删除，有自增索引的话，从之前值继续累加
```
