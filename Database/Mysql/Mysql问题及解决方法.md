# 重置自增id

```sql
alter table table_name AUTO_INCREMENT = new_vul;

-- 示例
alter table user AUTO_INCREMENT = 1000;
```

# 树结构循环查询

> 数据成树状结构，在不确定子节点是否存在，存在多少级下查询。使用`WITH RECURSIVE`递归查询。

```mysql
-- 测试
with RECURSIVE t1 AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM t1 WHERE n < 5
)
SELECT * FROM t1;
```

```mysql
with recursive t1 as (
    select * from  course_category p where id= '1-2'
    union all
    select t.* from course_category t inner join t1 on t1.id = t.parentid
)
select *  from t1 order by  t1.id,t1.orderby
```
![mysql0.png](..%2F..%2Fimg%2FMysql%2Fmysql0.png)