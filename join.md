# Join 

## 1、left join、right join、inner join、full join 的区别

- full (outer) join(全连接) 返回包括** 左表和右表中的所有 **的记录  
- left (outer) join(左联接) 返回包括** 左表中的所有记录和右表中联结字段相等 **的记录  
- right (outer) join(右联接) 返回包括** 右表中的所有记录和左表中联结字段相等 **的记录  
- inner join(等值联接) 返回** 两个表中联结字段相等 **的记录  

**注意**：  

1. ** 联结字段相等 **的记录如果存在重复出现的记录，则会在主表中添加相应的记录  
2. 表中的字段属性是** 唯一的 **，查询后生成的新表** 不继承 **该字段的唯一性  

e.g. 表 a（id 字段具有唯一性）和 表 b 如下

|id|value|          
|---|---|           
|1|66|              
|2|24|

|id|number|
|---|---|
|1|2|
|2|4|
|1|3|
|2|5|

```sql
select * from a left join b on a.id = b.id;
```

|id|value|id|number|        
|---|---|---|---|           
|1|66|1|2|
|2|24|2|4|
|1|66|1|3|             
|2|24|2|5|

查询后生成的新表没有继承表 a 字段 id 的唯一性  
等价于

```sql
select * from a right join b on a.id = b.id;
```

|id|value|id|number|        
|---|---|---|---|           
|1|66|1|2|
|1|66|1|3|             
|2|24|2|4|
|2|24|2|5|
