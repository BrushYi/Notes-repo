#### 力扣1179. 重新格式化部门表
链接：https://leetcode.cn/problems/reformat-department-table
![](sql刷题笔记_img/2022-06-14-15-00-47.png)
![](sql刷题笔记_img/2022-06-14-15-01-31.png)

要求：编写一个 SQL 查询来重新格式化表，使得新的表中有一个部门 id 列和一些对应 每个月 的收入（revenue）列。

- 解题
```sql
select id,
sum(if(month="Jan",revenue,null)) as "Jan_Revenue",
sum(if(month="Feb",revenue,null)) as "Feb_Revenue",
sum(if(month="Mar",revenue,null)) as "Mar_Revenue",
sum(if(month="Apr",revenue,null)) as "Apr_Revenue",
sum(if(month="May",revenue,null)) as "May_Revenue",
sum(if(month="Jun",revenue,null)) as "Jun_Revenue",
sum(if(month="Jul",revenue,null)) as "Jul_Revenue",
sum(if(month="Aug",revenue,null)) as "Aug_Revenue",
sum(if(month="Sep",revenue,null)) as "Sep_Revenue",
sum(if(month="Oct",revenue,null)) as "Oct_Revenue",
sum(if(month="Nov",revenue,null)) as "Nov_Revenue",
sum(if(month="Dec",revenue,null)) as "Dec_Revenue"
from
Department
group by id;
```

- 要点：通过case-when或者if行转列。

#### 力扣1795. 每个产品在不同商店的价格
链接：https://leetcode.cn/problems/rearrange-products-table
![](sql刷题笔记_img/2022-06-14-15-03-55.png)

要求：请你重构 Products 表，查询每个产品在不同商店的价格，使得输出的格式变为(product_id, store, price) 。如果这一产品在商店里没有出售，则不输出这一行。

- 解题
```sql
select product_id,'store1' as store ,store1 as price from Products where store1 is not null
union all
select product_id,'store2' as store ,store2 as price from Products where store2 is not null
union all
select product_id,'store3' as store ,store3 as price from Products where store3 is not null
```

- 要点：通过联结进行列转行。