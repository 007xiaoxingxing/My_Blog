---
title: Django ORM进行group by 操作
categories:
  - Program
tags:
  - django
date: 2018-08-03 17:02:15
---

今天在使用django的orm进行group by操作的时候遇到一点小问题，特此记录一下。
<!-- more -->
model的定义如下：
```python
class TestModel(models.Model):
    id = models.BigAutoField(primary_key=True)
    customer_line_id = models.BigIntegerField(blank=True, null=True)
    dstip = models.GenericIPAddressField(blank=True, null=True)
    stat_time = models.DateTimeField(blank=True, null=True)
    class Meta:
        managed = False
        db_table = 't_table'
        ordering = ['-id']
```
设想的sql：
```sql
select customer_line_id, count(customer_line_id) as count from t_table group by customer_line_id
```
设想的orm语句
```python
TestModel.objects.values('customer_line_id').annotate(count=Count('customer_line_id'))
```
django实际执行的sql：
```sql
select customer_line_id, count(customer_line_id) as count from t_table group by id order by id desc
```
实际并没有如期的按照customer_line_id进行group by。经过查找，是默认的ordering动作产生的影响，可以通过将orderby置空解决
```python
TestModel.objects.values('customer_line_id').annotate(count=Count('customer_line_id')).order_by()
```
这样就可以按照预想的情况group by指定字段了。