# 窗口函数

参考资料：  

[1、SQL窗口函数](https://zhuanlan.zhihu.com/p/92654574)  
[2、MySQL8.0 官网 Window Function](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)  
[3、MySQL8.0 官网 Window Function 中文翻译](http://www.deituicms.com/mysql8cn/cn/web.html)  
[4、以下代码及结果来源资料](https://www.begtut.com/mysql/mysql-dense_rank-function.html)

## 定义

窗口函数（Window Function），又叫 OLAP 函数（Online Anallytical Processing，联机分析处理）  
可以对数据库的数据进行实时分析处理

## 作用

在实际工作中，经常会遇到需要**在分组内进行排名**，比如：

- 排名问题：每个部门按照业绩来排名
- TopN 问题：找出每个部门排名前 N 的员工进行奖励

当遇到此类问题时，group by 无法实现，sql window function 应运而生

## 基本语法

基本语法：  

```sql
<窗口函数> over (
    partition by <用于分组的列名>
    order by <用于排序的列名>
)
```

说明：

1. ```<窗口函数>``` 主要包括：

- 专用窗口函数，如：rank()、dense_rank()、row_number() 等
- 聚合函数，如：sum()、avg()、max()、min() 等

2. 窗口函数原则上只能写在 select 子句中

## 专用窗口函数

|名称|描述|
|:---|:---|
|CUME_DIST()|分区内累积分配值|
|PERCENT_RANK()|分区内等级值|
|RANK()|分区内当前行的排名，有间隙|
|DENSE_RANK()|分区内当前行的排名，没有间隙|
|ROW_NUMBER()|分区内当前行的数量|
|FIRST_VALUE()|窗口框架第一行的参数值|
|LAST_VALUE()|窗口框架最后一行的参数值|
|NTH_VALUE()|窗口框架第 N 行的参数值|
|LAG()|分区内滞后当前行的参数值|
|LEAD()|分区内超前当前行的参数值|
|NTILE()|分区内当前行的储存捅编号|  

### 1、CUME_DIST()

返回当前行的值的累积分布；即（分区中）**小于等于** 当前行的值的行数的累积分布  

说明：  
- CUME_DIST() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的  
- 注意与 PERCENT_RANK() 的功能进行区分  

```sql
select 
    name,
    score,
    row_number() over (order by score) as `row_num`,
    cume_dist() over (order by score) as `cume_dist_val`
from scores;
```

结果：  
| name     | score | row_num | cume_dist_val |
|:---------|------:|--------:|--------------:|
| Jones    |    55 |       1 |           0.2 |
| Williams |    55 |       2 |           0.2 |
| Brown    |    62 |       3 |           0.4 |
| Taylor   |    62 |       4 |           0.4 |
| Thomas   |    72 |       5 |           0.6 |
| Wilson   |    72 |       6 |           0.6 |
| Smith    |    81 |       7 |           0.7 |
| Davies   |    84 |       8 |           0.8 |
| Evans    |    87 |       9 |           0.9 |
| Johnson  |   100 |      10 |             1 |

### 2、PERCENT_RANK()

分区内的百分位数排名；即返回分区中 **小于** 当前行**的值**的百分比，且分母 **不包括最高值**；计算公式为：  
```
(rank - 1) / (rows - 1)
```  

说明：  
- 其中 `rank` 是当前行的等级（相同值的排名是一样的，且**有间隙**，等同于 rank()），`rows` 是分区中的总行数  
- 对于分区中按照顺序排好后的第一行，结果始终为 0
- 重复 `rank` 的值将返回相同的结果
- PERCENT_RANK() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的

```sql
select 
    name,
    score,
    row_number() over (order by score) as `row_num`,
    cume_dist() over (order by score) as `cume_dist_val`,
    round(percent_rank() over (order by score), 2) as `percent_rank`
from scores;
```

结果：  
| name     | score | row_num | cume_dist_val | percent_rank |
|:---------|------:|--------:|--------------:|-------------:|
| Jones    |    55 |       1 |           0.2 |         0.00 |
| Williams |    55 |       2 |           0.2 |         0.00 |
| Brown    |    62 |       3 |           0.4 |         0.22 |
| Taylor   |    62 |       4 |           0.4 |         0.22 |
| Thomas   |    72 |       5 |           0.6 |         0.44 |
| Wilson   |    72 |       6 |           0.6 |         0.44 |
| Smith    |    81 |       7 |           0.7 |         0.67 |
| Davies   |    84 |       8 |           0.8 |         0.78 |
| Evans    |    87 |       9 |           0.9 |         0.89 |
| Johnson  |   100 |      10 |             1 |         1.00 |

### 3、RANK()

分区内当前行的排名，**有间隙**；即 **相同数值相同排名，下一个新数值的排名为分区中当前数值的行数**  

说明：  
- RANK() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的  

```sql
select 
    sales_employee,
    fiscal_year,
    sale,
    rank() over (
        partition by fiscal_year
        order by sale DESC
    ) as sales_rank
from sales;
```  

结果：  
| sales_employee | fiscal_year | sale   | sales_rank |
|:---------------|------------:|-------:|-----------:|
| John           |        2016 | 200.00 |          1 |
| Alice          |        2016 | 150.00 |          2 |
| Bob            |        2016 | 100.00 |          3 |
| Bob            |        2017 | 150.00 |          1 |
| John           |        2017 | 150.00 |          1 |
| Alice          |        2017 | 100.00 |          3 |
| John           |        2018 | 250.00 |          1 |
| Alice          |        2018 | 200.00 |          2 |
| Bob            |        2018 | 200.00 |          2 |

### 4、DENSE_RANK()

返回分区中当前行的排名，**没有间隙**；即 **相同数值相同排名，下一个新数值的排名加 1**  

说明：  
- DENSE_RANK() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的  

```sql
select
    sales_employee,
    fiscal_year,
    sale,
    dense_rank() over (
        partition by fiscal_year
        order by sale DESC
    ) as sales_rank
from sales;
```

结果：  
| sales_employee | fiscal_year | sale   | sales_rank |
|:---------------|------------:|-------:|-----------:|
| John           |        2016 | 200.00 |          1 |
| Alice          |        2016 | 150.00 |          2 |
| Bob            |        2016 | 100.00 |          3 |
| Bob            |        2017 | 150.00 |          1 |
| John           |        2017 | 150.00 |          1 |
| Alice          |        2017 | 100.00 |          2 |
| John           |        2018 | 250.00 |          1 |
| Alice          |        2018 | 200.00 |          2 |
| Bob            |        2018 | 200.00 |          2 |

### 5、ROW_NUMBER()

分区中当前行的编号，行数从 1 开始到分区行数结尾；也可作为分区内排名的一种函数  

说明：  
- ROW_NUMBER() 是一个顺序敏感函数，所以应始终使用 order by；否则行的编号是不正确的
- ROW_NUMBER() 常用来**删除各个分区内的重复行，将非唯一行转换为唯一行**，见下例  

```sql
select 
    id,
    name
from (
    select
        id,
        name,
        row_number() over (partition by name order by name) as row_num
    FROM rowNumberDemo
) as t
where row_num <> 1
```

原始数据：  
| id   | name |
|-----:|:-----|
|    1 | A    |
|    2 | B    |
|    3 | B    |
|    4 | C    |
|    5 | C    |
|    6 | C    |
|    7 | D    |

结果：  
| id   | name |
|-----:|:-----|
|    1 | A    |
|    2 | B    |
|    4 | C    |
|    7 | D    |

### 6、FIRST_VALUE()
