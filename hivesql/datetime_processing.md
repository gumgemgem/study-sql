# 日期及时间类数据的处理  

参考链接：  
1. 官网首页：https://cwiki.apache.org/confluence/display/Hive/LanguageManual
2. 官网操作符及函数（日期、字符串等）：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF     
3. 其他参考链接：https://developer.aliyun.com/article/901408#slide-12  

## 日期及时间类型  

|数据类型|描述|格式|
|---|---|---|
|DATE|用于表示日期，不包含时间部分|yyyy-MM-dd|
|TIMESTAMP|可以同时存储日期和时间信息|yyyy-MM-dd HH:mm:ss|
|INTERVAL|日期或时间频率间隔||
|时间戳|毫秒值精度；是存储1970年1月1日起的秒数或者纳秒数|122327493795|

## 日期及时间处理函数  

**注意**：以下函数中的 string date 参数可以是日期或时间的字符串类型，也可以是 date 类型，也可以是 timestamp 类型  

1. current_date / current_date()    
   返回当前日期

2. current_timestamp / current_timestamp()  
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

6. add_months()  
   在日期上加上指定月数

   **语法**：add_months(string start_date, int num_months, output_date_format)  
   **返回值**：返回为 string 类型的日期     
   **说明**：  
   - string startdate 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
   - 如果 string start_date 的天数比增加月份后的最大天数都大，那结果默认最大天数；否则和 string start_date 的天数一致  
   - Hive 4.0.0 之前的版本没有 output_date_format 参数，会忽略时间；Hive 4.0.0 及之后的版本新增了 output_date_format 参数
   - int num_months 可以是正数也可以是负数  
  
   ```sql
   select
      add_months('2025-01-08', 2) as `日期1`,
      add_months('2025-01-08 10:14:36', 2) as `日期11`,
      add_months('2025-01-31 10:14:36', 1) as `日期2`
      -- add_months('2025-01-31 10:14:36', 1, 'yyyy-MM-dd HH:mm:ss') as `日期22` -- Hive 4.0.0 及以后的版本才支持控制输出的格式

   -- 返回
   -- 2025-03-08
   -- 2025-03-08
   -- 2025-02-28
   ```

7. date_sub()  
   在日期上减去指定天数  

   **语法**：date_sub(string startdate, int days)  
   **返回值**：返回为 string 类型的日期     
   **说明**：  
   - string startdate 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  

8. date_diff()  
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

9. months_between()  
   计算两个日期之间的月份差
   
   **语法**：months_between(date1, date2)  
   **返回值**：返回为 double 类型的日期差值     
   **说明**：  
   - 如果 date1 晚于 date2，结果为正数；否则结果为负数
   - 如果两日期的天数一样或都是月份的最后一天，那么日期差值结果 double 类型的整数；否则将根据一月31天来计算小数部分，带时间的日期也会将时间差考虑进去
   
   ```sql
   select
      months_between('2025-01-08', '2025-03-08') as `月份差1`,
      months_between('2025-03-08', '2025-01-08') as `月份差11`,
      months_between('2025-02-28', '2025-01-31') as `月份差2`,
      months_between('2025-03-16', '2025-01-08') as `月份差22`,
      months_between('2025-03-16 19:48:32', '2025-01-08 09:27:56') as `月份差222`
   
   -- 返回
   -- -2.0
   -- 2.0
   -- 1.0
   -- 2.25806452
   -- 2.27196685
   ```

10. to_date()  
    返回日期时间字段中的日期部分  

    **语法**：to_date(string timestamp)   
    **返回值**：返回为 string 类型的日期     
    **说明**：  
    - string timestamp 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
   
   ```sql
   select
      to_date('2025-01-08 10:14:36') as `日期1`,
      to_date('2025-01-08') as `日期11`,
      to_date('2025/01/08 10:14:36') as `日期111`
   
   -- 返回
   -- 2025-01-08
   -- 2025-01-08
   -- null
   ```

11. day() / month() / year() / hour() / minute() / second() / weekofyear() / quarter()  
    返回给定时间的 day / month / year / hour / minute / second / weekofyear / quarter  

    **语法**：function(string date)   
    **返回值**：返回为 int 类型结果值     
    **说明**：  
    - hive 中没有专有获取季度的函数，比如 quarter()，可以通过 case...when... 来实现
    - string date 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
    - quarter() 函数应用于 Hive 1.3.0 及以后的版本  

   ```sql
   select
      year('2025-01-08 10:14:36') as `年1`,
      year('2025-01-08') as `年11`,
      year('2025/01/08') as `年111`,
      month('2025-01-08 10:14:36') as `月`,
      day('2025-01-08 10:14:36') as `日`,
      hour('2025-01-08 10:14:36') as `时`,
      minute('2025-01-08 10:14:36') as `分`,
      second('2025-01-08 10:14:36.334') as `秒`,
      weekofyear('2025-01-08 10:14:36') as `周`

   -- 返回
   -- 2025
   -- 2025
   -- null
   -- 1
   -- 8
   -- 10
   -- 14
   -- 36
   -- 2
   ```

12. last_day()   
    返回日期当月的最后一天  
   
    **语法**：last_day(string date)      
    **返回值**：返回为 stirng 类型的日期       
    **说明**：  
    - string date 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
    - string date 如果是时间，将会忽略掉时间部分  

   ```sql
   select
      last_day('2025-01-08') as `日期1`,
      last_day('2025-01-08 14:08:25') as `日期11`

   -- 返回
   -- 2025-01-31
   -- 2025-01-31
   ```

13. next_day()  
    获取给定日期之后的下一个指定的工作日  
   
    **语法**：next_day(string start_date, string day_of_week)   
    **返回值**：返回为 stirng 类型的日期       
    **说明**：  
    - string start_date 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
    - string start_date 如果是时间，将会忽略掉时间部分
    - string day_of_week 为指定工作日，可以是英文全称或者英文缩写，如：'Monday' 或 'Mon'  
   
   ```sql
   select
      next_day('2025-01-08', 'Monday') as `日期1`,
      next_day('2025-01-08', 'Mon') as `日期11`,
      next_day('2025-01-08 14:08:25', 'Tuesday') as `日期2`

   -- 返回
   -- 2025-01-13
   -- 2025-01-13
   -- 2025-01-14
   ```

14. trunc()  
    截取日期到指定的粒度  
   
    **语法**：trunc(string date, string format)   
    **返回值**：返回为 string 类型的日期或时间       
    **说明**：  
    - string format 不能省略  
    - 截取指定日期当年第一天，string format 为：'YY'、'YYYY'、'YEAR'（必须大写）；截取指定日期当月第一天，string format 为：'MM'、'MON'、'MONTH'（必须大写）；string format 目前只有这几种格式，没有其他格式  
    - string date 可以是日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式  
   
   ```sql
   select
      trunc('2025-02-08', 'YY') as `日期1`,
      trunc('2025/02/08', 'YY') as `日期11`,
      trunc('2025-02-08 10:14:36', 'YY') as `日期2`,
      trunc('2025-02-08 10:14:36', 'YYYY') as `日期22`,
      trunc('2025-02-08 10:14:36', 'YEAR') as `日期222`,
      trunc('2025-02-08 10:14:36', 'MM') as `日期3`,
      trunc('2025-02-08 10:14:36', 'MON') as `日期33`,
      trunc('2025-02-08 10:14:36', 'MONTH') as `日期333`
   
   -- 返回
   -- 2025-01-01
   -- null
   -- 2025-01-01
   -- 2025-01-01
   -- 2025-01-01
   -- 2025-02-01
   -- 2025-02-01
   -- 2025-02-01
   ```

15. date_format()  
    将日期或时间或时间戳格式转化为指定的字符串格式   
   
    **语法**：date_format(date/timestamp/string ts, string pattern)  
    **返回值**：返回为 string 类型的结果         
    **说明**：  
    - string pattern 为 hive 中常用的格式化字符串（见下表）  
    - date/timestamp/string ts 可以是日期或时间或字符串类型的日期或时间，但必须是 yyyy-MM-dd 或 yyyy-MM-dd HH:mm:ss 的格式   
   
   ```sql
   select
   	date_format('2025-01-08', 'E') as `日期1`,
   	date_format('2025-01-08', 'EEEE') as `日期11`,
   	date_format('2025-01-08 14:08:25', 'EEE') as `日期111`,
   	date_format('2025-01-08 14:08:25', 'u') as `日期1111`,   -- 获取日期或时间是一周中的周几
   	date_format('2025-01-08', 'MM') as `日期2`,
   	date_format('2025-01-08 14:08:25', 'HH') as `日期22`,
   	date_format('2025-01-08 14:08:25', 'hh') as `日期222`,
   	date_format('2025/01/08', 'yyyy-MM-dd') as `日期3`,
   	date_format('2025-01-08', 'yyyy/MM/dd') as `日期33`   -- 将 yyyy-MM-dd 格式转化为 yyyy/MM/dd 格式
   
   -- 返回
   -- Wed
   -- Wednesday
   -- Wed
   -- 3
   -- 01
   -- 14
   -- 02
   -- null
   -- 2025/01/08
   ```

## 常用的格式化字符串  

|字符串|说明|示例|
|---|---|---|
|**yyyy**|**四位数的年份**|如 2025|
|yy|两位数的年份|如 25|
|**MM**|**两位数的月份**|如 01 到 12|
|MMM|月份的缩写|如 Jan、Feb|
|MMMM|月份的全称|如 January、February|
|**dd**|**两位数的日期**|如 01 到 31|
|**u**|**星期的数字**|如 1 到 7|
|E/EEE|星期的缩写|如 Mon、Tue|
|EEEE|星期的全称|如 Monday、Tuesday|
|**HH**|**两位数的小时**|24小时制，如 00 到 23|
|hh|两位数的小时|12小时制，如 01 到 12|
|**mm**|**两位数的分钟**|如 00 到 59|
|**ss**|**两位数的秒**|如 00 到 59|

## 日期或时间的常用操作

- 获取指定日期当月的第一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(trunc(string date, 'MM'), 'yyyyMMdd')
  ```

- 获取指定日期当月的最后一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(last_day(string date), 'yyyyMMdd')
  ```

- 获取上月的第一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(trunc(add_months(current_date, -1), 'MM'), 'yyyyMMdd')
  ```

- 获取上月的最后一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(last_day(add_months(current_date, -1)), 'yyyyMMdd')
  date_format(date_sub(trunc(current_date, 'MM'), 1), 'yyyyMMdd')
  ```

- 获取上周同一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(date_sub(current_date, 7), 'yyyyMMdd')
  ```

- 获取上月同一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(add_months(current_date, -1), 'yyyyMMdd')
  ```

- 获取上季度同一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(add_months(current_date, -3), 'yyyyMMdd')
  ```

- 获取去年同一天，并输出格式为 yyyyMMdd  
  ```sql
  date_format(add_months(current_date, -12), 'yyyyMMdd')
  ```
