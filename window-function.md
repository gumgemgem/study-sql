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
    [partition by <用于分组的列名>]
    [order by <用于排序的列名>]
    [frame_clause]
)
```

说明：

1. ```<窗口函数>``` 主要包括：

- 专用窗口函数，如：rank()、dense_rank()、row_number() 等
- 聚合函数，如：sum()、avg()、max()、min() 等

2. **窗口函数原则上只能写在 select 子句中**
3. frame_clause 创建窗口子句，是当前分区的子集  

## 窗口子句（frame_clause）

窗口是根据**当前行**确定的，这使得帧能够在分区内移动，具体如何移动取决于其分区内当前行的位置  
**frame_clause** 是由 `frame_units` 和 `frame_extent` 两部分组成的  

- 窗口子句起作用的函数（这些函数作用于窗口框架）
  - 聚合函数
  - 专用窗口函数：FIRST_VALUE()、LAST_VALUE()、NTH_VALUE()

- 窗口子句不起作用的函数（这些函数作用于整个分区）
  - CUME_DIST()、PERCENT_RANK()
  - RANK()、DENSE_RANK()、ROW_NUMBER()
  - LAG()、LEAD()、NTILE()

### 1. frame_units

frame_units 指示**当前行**和**窗口行**之间的关系类型  

- **ROWS**（物理窗口）  
  框架由**开始和结束行**位置定义。 偏移量是**行号**与**当前行号**的差异  
  即根据 order by 子句排序后，取当前行的前 N 行及后 N 行的数据计算（与当前行的值无关，只与排序后的行号相关）  
  
- **Range**（逻辑窗口）  
  框架由**值范围内**的行定义。 偏移量是**行值**与**当前行值**的差异  
  即指定当前行对应值的一个范围取值，列数不固定，只要行值在范围内，对应列都包含在窗口内

### 2. frame_extent

frame_extent 定义窗口框架的范围  

- frame_start：只定义窗口的开头（frame_start），此时当前行作为隐式结束  
- frame_between：定义窗口的开头（frame_start）和结尾（frame_end）；且 frame_start 必须早于 frame_end  

```sql
BETWEEN frame_start AND frame_end
```

frame_start 和 frame_end 的选择：  
|名称|描述|
|:---|:---|
|CURRENT ROW|对于 ROWS，界限是当前行；<br>对于 RANGE，界限是当前行的对等体（与当前行值相等的行）|
|UNBOUNDED PRECEDING|界限是第一个分区行|
|UNBOUNDED FOLLOWING|界限是最后一个分区行|
|N PRECEDING|对于 ROWS，界限是当前行之前的 N 行；<br>对于 RANGE，界限是值等于当前行值减去 N 的行；<br>如果当前行值为 NULL，则界限为该行的对等行；<br>若界限超过分区，则窗口框架为可用行|
|N FOLLOWING|对于 ROWS，界限是当前行之后的 N 行；<br>对于 RANGE，界限是值等于当前行值加上 N 的行；<br>如果当前行值为 NULL，则界限为该行的对等行；<br>若界限超过分区，则窗口框架为可用行|

### 3. 缺少 frame_clause 子句

在缺少窗口子句的情况下，默认窗口框架取决于 **ORDER BY** 子句是否存在：  

- ORDER BY 存在：默认窗口框架包括从**分区起始行**到**与当前行值相等的行**，**包括当前行的所有对等行**  

```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

- ORDER BY 不存在：默认窗口框架包括**所有分区行**  

```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

### 4. 示例

```sql
SELECT 
    id,
    SUM(ID) over(ORDER BY ID) default_sum,
    SUM(ID) over(ORDER BY ID 
                 RANGE BETWEEN unbounded preceding AND CURRENT ROW) range_unbound_sum,
    SUM(ID) over(ORDER BY ID 
                 ROWS BETWEEN unbounded preceding AND CURRENT ROW) rows_unbound_sum,
    SUM(ID) over(ORDER BY ID 
                 RANGE BETWEEN 1 preceding AND 2 following) range_sum,
    SUM(ID) over(ORDER BY ID 
                 ROWS BETWEEN 1 preceding AND 2 following) rows_sum
FROM t
```

结果：  
| ID     | DEFAULT_SUM | RANGE_UNBOUND_SUM | ROWS_UNBOUND_SUM | RANGE_SUM | ROWS_SUM |
|:-------|------------:|------------------:|-----------------:|----------:|---------:|
| 1      |           2 |                 2 |                1 |         5 |        5 |
| 1      |           2 |                 2 |                2 |         5 |       11 |
| 3      |           5 |                 5 |                5 |         3 |       16 |
| 6      |          23 |                23 |               11 |        33 |       21 |
| 6      |          23 |                23 |               17 |        33 |       25 |
| 6      |          23 |                23 |               17 |        33 |       27 |
| 7      |          30 |                30 |               30 |        42 |       30 |
| 8      |          38 |                38 |               38 |        24 |       24 |
| 9      |          47 |                47 |               47 |        17 |       17 |

## 专用窗口函数

|名称|描述|
|:---|:---|
|CUME_DIST()|分区内累积分配值；小于等于或大于等于当前行值的累积分布|
|PERCENT_RANK()|分区内等级值；小于或大于当前行值的百分比（不包括最大值或最小值）|
|RANK()|分区内当前行的排名，有间隙|
|DENSE_RANK()|分区内当前行的排名，没有间隙|
|ROW_NUMBER()|分区内当前行的数量|
|FIRST_VALUE()|窗口框架第一行的参数值|
|LAST_VALUE()|窗口框架最后一行的参数值|
|NTH_VALUE()|窗口框架第 N 行的参数值|
|LAG()|分区内滞后当前行的参数值|
|LEAD()|分区内超前当前行的参数值|
|NTILE()|分区内当前行的储存桶编号|  

### 1、CUME_DIST()

返回当前行的值的累积分布；即（分区中）**小于等于** 或 **大于等于** 当前行的值的行数的累积分布  

说明：  
- CUME_DIST() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的
- 当 order by 升序：结果为 **小于等于** 当前行值的行数的累积分布；当 order by 降序：结果为 **大于等于** 当前行值的行数的累积分布
- 当排序列中存在 null 值，同 `rank` 的排序规则  
- 注意与 PERCENT_RANK() 的功能进行区分  

```sql
select 
    name,
    score,
    row_number() over (order by score) as `row_num`,
    cume_dist() over (order by score) as `cume_dist_val`,
    cume_dist() over (order by score desc) as `cume_dist_desc_val`
from scores;
```

结果：  
| name     | score | row_num | cume_dist_val | cume_dist_desc_val |
|:---------|------:|--------:|--------------:|-------------------:|
| Jones    |    55 |       1 |           0.2 |                   1|
| Williams |    55 |       2 |           0.2 |                   1|
| Brown    |    62 |       3 |           0.4 |                 0.8|
| Taylor   |    62 |       4 |           0.4 |                 0.8|
| Thomas   |    72 |       5 |           0.6 |                 0.6|
| Wilson   |    72 |       6 |           0.6 |                 0.6|
| Smith    |    81 |       7 |           0.7 |                 0.4|
| Davies   |    84 |       8 |           0.8 |                 0.3|
| Evans    |    87 |       9 |           0.9 |                 0.2|
| Johnson  |   100 |      10 |             1 |                 0.1|

### 2、PERCENT_RANK()

分区内的百分位数排名；即返回分区中 **小于** 或 **大于** 当前行**的值**的百分比，且分母 **不包括最高值或最低值**；计算公式为：  
```
(rank - 1) / (rows - 1)
```  

说明：  
- 其中 `rank` 是当前行的等级（相同值的排名是一样的，且**有间隙**，等同于 rank()），`rows` 是分区中的总行数
- 当排序列中存在 null 值，同 `rank` 的排序规则
- 当 order by 升序：结果为 **小于** 当前行值的百分比，且**不包括最高值**；当 order by 降序：结果为 **大于** 当前行值的百分比，且**不包括最低值**  
- 对于分区中按照顺序排好后的第一行，结果始终为 0
- 重复 `rank` 的值将返回相同的结果
- PERCENT_RANK() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的

```sql
select 
    name,
    score,
    row_number() over (order by score) as `row_num`,
    cume_dist() over (order by score) as `cume_dist_val`,
    round(percent_rank() over (order by score), 2) as `percent_rank`, 
    round(percent_rank() over (order by score desc), 2) as `percent_desc_rank`
from scores;
```

结果：  
| name     | score | row_num | cume_dist_val | percent_rank | percent_desc_rank |
|:---------|------:|--------:|--------------:|-------------:|------------------:|
| Jones    |    55 |       1 |           0.2 |         0.00 |              0.89 |
| Williams |    55 |       2 |           0.2 |         0.00 |              0.89 |
| Brown    |    62 |       3 |           0.4 |         0.22 |              0.67 |
| Taylor   |    62 |       4 |           0.4 |         0.22 |              0.67 |
| Thomas   |    72 |       5 |           0.6 |         0.44 |              0.44 |
| Wilson   |    72 |       6 |           0.6 |         0.44 |              0.44 |
| Smith    |    81 |       7 |           0.7 |         0.67 |              0.33 |
| Davies   |    84 |       8 |           0.8 |         0.78 |              0.22 |
| Evans    |    87 |       9 |           0.9 |         0.89 |              0.11 |
| Johnson  |   100 |      10 |             1 |         1.00 |              0.00 |

### 3、RANK()

分区内当前行的排名，**有间隙**；即 **相同数值相同排名，下一个新数值的排名为分区中当前数值的行数**  

说明：  
- RANK() 是一个顺序敏感函数，所以应始终使用 order by；否则所有行都是对等的
- 排序列中存在 null 值的处理方式：
  - **升序** 默认 **nulls last**（即认为 null 是 **最大值**，排在最后）；**降序** 默认 **nulls first**（即认为 null 是 **最大值**，排在最前）
  - 可自定义 null 值的排序方式：e.g. 在 **升序** 排序时，指定 null 值为 **最小值**，`order by sale nulls first`   

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
- 当排序列中存在 null 值，同 `rank` 的排序规则  

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
- 当排序列中存在 null 值，同 `rank` 的排序规则  

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

### 6、FIRST_VALUE()、LAST_VALUE()、NTH_VALUE()

- **FIRST_VALUE(col)**：返回窗口框架某列的第一行值  
- **LAST_VALUE(col)**：返回窗口框架某列的最后一行值  
- **NTH_VALUE(col, N)**：返回窗口框架某列第 N 行的参数值；如果第 N 行不存在，则函数返回NULL；N必须是正整数 

```sql
SELECT
    time, 
    subject, 
    val,
    FIRST_VALUE(val)  OVER w AS 'first',
    LAST_VALUE(val)   OVER w AS 'last',
    NTH_VALUE(val, 2) OVER w AS 'second',
    NTH_VALUE(val, 4) OVER w AS 'fourth'
FROM observations
WINDOW w AS (PARTITION BY subject 
             ORDER BY time
             ROWS UNBOUNDED PRECEDING);
```

结果：  
|time      |  subject |      val |    first |     last |   second |   fourth |
|:---------|---------:|---------:|---------:|---------:|---------:|---------:|
| 07:00:00 |    st113 |       10 |       10 |       10 |     NULL |     NULL |
| 07:15:00 |    st113 |        9 |       10 |        9 |        9 |     NULL |
| 07:30:00 |    st113 |       25 |       10 |       25 |        9 |     NULL |
| 07:45:00 |    st113 |       20 |       10 |       20 |        9 |       20 |
| 07:00:00 |    xh458 |        0 |        0 |        0 |     NULL |     NULL |
| 07:15:00 |    xh458 |       10 |        0 |       10 |       10 |     NULL |
| 07:30:00 |    xh458 |        5 |        0 |        5 |       10 |     NULL |
| 07:45:00 |    xh458 |       30 |        0 |       30 |       10 |       30 |
| 08:00:00 |    xh458 |       25 |        0 |       25 |       10 |       30 |

### 7. LAG()、LEAD()

- **LAG(col [, N[, default]])**：返回分区内某列当前行**滞后（或先于）** N 行的值
  - 如果没有这样的行，则返回值为 default：比如当 N = 3 时，对于分区中的前两行而言，返回的都是 default 值；且 defualt 可为某列的值  
  - 如果 N 或 default 缺少，则默认值分别为 1 和 NULL  
  - N 必须是非负整数；如果 N 为 0，col 则评估当前行  

- **LEAD(col [, N[, default]])**：返回分区内某列当前行**跟随（或后于）** N 行的值
  - 如果没有这样的行，则返回值为 default：比如当 N = 3 时，对于分区中的最后两行而言，返回的都是 default 值；且 defualt 可为某列的值  
  - 如果 N 或 default 缺少，则默认值分别为 1 和 NULL  
  - N 必须是非负整数；如果 N 为 0，col 则评估当前行  

```sql
SELECT
    n,
    LAG(n, 1, 0)      OVER w AS 'lag',
    LEAD(n, 1, 0)     OVER w AS 'lead',
    n + LAG(n, 1, 0)  OVER w AS 'next_n',
    n + LEAD(n, 1, 0) OVER w AS 'next_next_n'
FROM fib
WINDOW w AS (ORDER BY n)
```

结果：  
|        n |      lag |     lead |   next_n |next_next_n |
|---------:|---------:|---------:|---------:|-----------:|
|        1 |        0 |        1 |        1 |          2 |
|        1 |        1 |        2 |        2 |          3 |
|        2 |        1 |        3 |        3 |          5 |
|        3 |        2 |        5 |        5 |          8 |
|        5 |        3 |        8 |        8 |         13 |
|        8 |        5 |        0 |       13 |          8 |

### 8. NTILE()

**NTILE(N)** 将分区划分为 N 组（存储桶），为分区中的每一行分配其存储桶编号，并返回其分区中当前行的存储区编号  

- N 是必须是正整数；桶号的范围是 1 到 N  
- NTILE(N) 是一个顺序敏感函数，所以应始终使用 order by  
- 桶号如何分配？  
  - 如果分区行的总数可被 N 整除，则行将在组之间平均分配  
  - 如果分区行的数量不能被 N 整除，则 NTILE(N) 函数将生成两个大小的组，差异为1；且数量较大的组总是**以 ORDER BY 子句指定的顺序**位于较小的组之前  

```sql
SELECT
    val,
    ROW_NUMBER() OVER w AS 'row_number',
    NTILE(2)     OVER w AS 'ntile2',
    NTILE(4)     OVER w AS 'ntile4'
FROM numbers
WINDOW w AS (ORDER BY val)
```

结果：  
|      val |row_number |   ntile2 |   ntile4 |
|---------:|----------:|---------:|---------:|
|        1 |         1 |        1 |        1 |
|        1 |         2 |        1 |        1 |
|        2 |         3 |        1 |        1 |
|        3 |         4 |        1 |        2 |
|        3 |         5 |        1 |        2 |
|        3 |         6 |        2 |        3 |
|        4 |         7 |        2 |        3 |
|        4 |         8 |        2 |        4 |
|        5 |         9 |        2 |        4 |
