# 日期及时间类数据的处理  

参考链接：https://developer.aliyun.com/article/901408#slide-12  

## 日期及时间类型  

|数据类型|描述|格式|
|---|---|---|
|DATE|用于表示日期，不包含时间部分|yyyy-MM-dd|
|TIMESTAMP|可以同时存储日期和时间信息|yyyy-MM-dd HH:mm:ss|
|INTERVAL|日期或时间频率间隔||
|时间戳|毫秒值精度；是存储1970年1月1日起的秒数或者纳秒数|122327493795|

## 日期及时间处理函数  

1. current_date  
   返回当前日期

2. current_timestamp  
   返回当前时间  

3. unix_timestamp()  
   将**日期字符串**转换为 **Unix 时间戳**  

   **语法**：unix_timestamp([string date], [string pattern])  
   **返回值**：返回的时间戳为 bigint 类型  
   **说明**：
       - 两个参数都省略时，表示将**当前时间**转化为时间戳
       - 只省略 string pattern 时，string date 必须是 TIMESTAMP 类型的数据，且格式必须是 yyyy-MM-dd HH:mm:ss  

   ```sql
   select
        current_date as `当前日期`,
        current_timestamp as `当前时间`,
        unix_timestamp() as `当前时间的时间戳`,
        unix_timestamp('2025-01-07', 'yyyy-MM-dd') as `时间戳1`,
        unix_timestamp('2025-01-07') as `时间戳11`,
        unix_timestamp('2025-01-07 14:29:06', 'yyyy-MM-dd HH:mm:ss') as `时间戳2`,
        unix_timestamp('2025/01/07 14:29:06', 'yyyy/MM/dd HH:mm:ss') as `时间戳22`,
        unix_timestamp('2025-01-07 14:29:06') as `时间戳222`,
        unix_timestamp('2025/01/07 14:29:06') as `时间戳2222`

   -- 返回
   -- 2025-01-07
   -- 2025-01-07 15:06:51.574
   -- 1736233623
   -- 1736179200
   -- null
   -- 1736231346
   -- 1736231346
   -- 1736231346
   -- null
   ```

4. from_unixtime()  
   将 **Unix 时间戳**转换为**日期字符串**  

   **语法**：from_unixtime(bigint unixtime[, string format])  
   **返回值**：返回为 string 类型的日期     
   **说明**： 
       - string format 省略时，默认返回 yyyy-MM-dd HH:mm:ss 格式的 string 类型  

   ```sql
   select
        from_unixtime(1736231346, 'yyyy-MM-dd') as `日期1`,
        from_unixtime(1736231346) as `当前日期11`,
        from_unixtime(1736231346, 'yyyy-MM-dd HH:mm:ss') as `时间`,
        from_unixtime(unix_timestamp('2025-01-07', 'yyyy-MM-dd'), 'yyyyMMdd') as `日期2`   -- 将 yyyy-MM-dd 格式转化为 yyyyMMdd

   -- 返回
   -- 2025-01-07
   -- 2025-01-07 14:29:06
   -- 2025-01-07 14:29:06
   -- 20250107
   ```

5. date_add()  
   在日期上加上指定天数  

   **语法**：date_add(string startdate, int days)  
   **返回值**：返回为 string 类型的日期     
   **说明**：
       - string startdate 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  

   ```sql
   select
        date_add('2025-01-07', 3) as `日期1`,
        date_add('2025-01-07 17:40:26', 3) as `日期2`,
        date_add('2025/01/07', 3) as `日期3`,
        date_add('20250107', 3) as `日期4`

   -- 返回
   -- 2025-01-10
   -- 2025-01-10
   -- null
   -- null
   ```

6. date_sub()  
   在日期上减去指定天数  

   **语法**：date_sub(string startdate, int days)  
   **返回值**：返回为 string 类型的日期     
   **说明**：
       - string startdate 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  

6. date_diff()  
   计算两个日期之间的天数差  

   **语法**：date_diff(string enddate, string startdate)  
   **返回值**：返回为 int 类型的数值     
   **说明**：
       - string enddate、string startdate 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  

   ```sql
   select
        datediff('2025-01-10', '2025-01-07') as `日期间隔1`,
        datediff('2025-01-10 17:40:26', '2025-01-07  13:30:28') as `日期间隔2`,
        datediff('2025/01/10', '2025/01/07') as `日期间隔3`

   -- 返回
   -- 3
   -- 3
   -- null
   ```
