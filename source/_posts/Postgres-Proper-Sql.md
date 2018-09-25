---
title: Postgres的特色语法
categories:
  - Program
tags:
  - sql
date: 2018-06-06 13:00:34
---

Postgres号称世界上最好的开源数据库，我们自己的产品也在使用，于是乎我特地去学习了一下Postgres的特色语法，这里从书上摘抄了一些语句，忘记的时候可以翻一下。

<!--more-->

##### DISTINCT ON

指定字段值去重

```sql
select distinct on (left(tract_id, 5))

left(tract_id, 5) as country, tract_id,tract_name

from census.lu_tracts

order by country, tract_id;

```



##### delete using

using可以将需要借助的一个或者多个中间表纳入到一个delete语句中

```
delete from census.facts

using census.lu_fact_types as ft

where fatcs.fact_type_id = ft.fact_type_id and ft.short_name = 's01';
```



##### 将修改影响到的记录行返回给用户

delete，insert，update操作得记录行可以返回给用户

```sql
update census.lu_fact_types as f

set short_name = replace(replace(lower(f.fact_subcats[4], ' ', '_', ':', '')))

where f.fact_subcats[3] = 'Hispanic or latino:' and f.fact_subcats[4] > ''

RETURNING fact_type_id, short_name;

```



##### 直接将查询结果转换为json格式

```sql
select array_to_json(array_agg(f)) as cat from(

select max(fact_type_id) as max_type, category

from census.lu_fact_types

group by category

)as f;

select json_agg(f) as cats from table as f;

```



##### 窗口函数

基本窗口函数

```sql
select tract_id, val, avg(val) OVER() as val_avg 

from census.facts

where fact_type_id=86;

partition by子句

select tract_id, val, avg(val), over(partition by left(tract_id, 5)) as val_avg_contry

from census.facts 

where fact_type_id=2 order by tract_id;

```



order by 子句

```sql
select row_number() over(order by tract_name) as rnum, tract_name

from census.lu_tracts

order by rnum limit 4;

```



联用partition by和order by

```sql
select tract_id, val,

sum(val) over(partition by left(tract_id, 5) order by val) as sum_country_ordered

from census.facts

where fact_type_id =2

order by left(tract_id, 5), val;

```



可以看出上面输出的合计值是逐行累加的，这就是在over子句中使用order by的效果，即窗口可见域是从排序后的记录集的头条记录开始，到order by字段值与当前记录值匹配的那行记录为止，因此最终呈现为动态累加的效果。

还可以通过range或者rows关键字来显示执行窗口的可见记录域。例如rows between row and 5 following

命名窗口，lead，lag

```sql
select * from(

select

row_number() over(wt) as rnum,

substring(tract_id,1,5) as country_code,

tract_id,

lag(tract_id, 2) over wt as tract_2_before,

lead(tract_id) over wt as tract_after,

from census.lu_tracts

window wt as (partition by substring(tract_id, 1,5) order by tract_id) --将窗口命名为wt

) as x

where rnum between 2 and 3 and country_code in ('25007', '25025')

order by coutry_code, rnum;

```



##### cte表达式（公用表达式）

 	cte的分类：基本CTE，可写CTE，递归CTE

  	cte的 基本用法：

```sql
with cte as(

select tract_id,substring(tract_id,1,5) as country_code,

count(*) over(partition by substring(tract_id, 1,5)) as cnt_tracts

from census.lu_tracts

)

select max(tract_id) as last_tract, coutry_code, cnt_tracts

from cte

where cnt_tracts >100

group by coutry_code, cnt_tracts;

```



多个cte对的用法

```sql
with

cte1 as (

select

tract_id,

substring(tract_id, 1,5) as country_code,

count(*) over(partition by substring(tract_id,1,5)) as cnt_tracts

from census.lu_tracts

),

cte2 as (

select

max(tract_id) as last_tract,

country_code,

cnt_tracts

from cte1

where cnt_tracts < 8 group by country_code, cnt_tracts

)

select c.last_tract, f.fact_type_id,f.val

from census.facts as f inner join cte2 on f.tact_id = c.last_tract;

```

参考书籍：《PostgreSQL即学即用》