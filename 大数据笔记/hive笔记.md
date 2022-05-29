## 什么是hive

## hive的运行原理

## hive的数据定义（DDL）

## hive的数据操作（DML）

## 查询
逐行映射、分组聚合、过滤模型
## 函数

### 聚合

### 窗口
sum(x) over();
lead(x,x,x) over();
lag(x,x,x) over();
count(x,x,x) over();
first_value(x) over();
last_value(x) over;
row_number() over ()
rank() over ();
dense_rank();

可对窗口函数进行运算操作，如：(columnA - (开窗)) as c。
