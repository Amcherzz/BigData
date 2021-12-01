## 时间函数

```sql
-- 获取当前时间
SELECT now()

┌───────────────now()─┐
│ 2021-12-01 14:39:10 │
└─────────────────────┘

-- 获取不通时间粒度 
date_trunc(unit, value[, timezone])
unit包括
second
minute
hour
day
week
month
quarter
year


SELECT date_trunc('week', toDate('2021-12-1'))

┌─date_trunc('week', toDate('2021-12-1'))─┐
│                              2021-11-29 │
└─────────────────────────────────────────┘

-- 格式化时间
select formatDateTime(now(), '%F')


┌─formatDateTime(now(), '%F')─┐
│ 2021-12-01                  │
└─────────────────────────────┘

-- 日期加减
select toDate('2021-12-21') - toDate('2021-01-23')


┌─minus(toDate('2021-12-21'), toDate('2021-01-23'))─┐
│                                               332 │
└───────────────────────────────────────────────────┘

--
```

| Placeholder | Description                                                  | Example    |
| ----------- | ------------------------------------------------------------ | ---------- |
| %C          | year divided by 100 and truncated to integer (00-99)         | 20         |
| %d          | day of the month, zero-padded (01-31)                        | 02         |
| %D          | Short MM/DD/YY date, equivalent to %m/%d/%y                  | 01/02/18   |
| %e          | day of the month, space-padded ( 1-31)                       | 2          |
| %F          | short YYYY-MM-DD date, equivalent to %Y-%m-%d                | 2018-01-02 |
| %G          | four-digit year format for ISO week number, calculated from the week-based year [defined by the ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Week_dates) standard, normally useful only with %V | 2018       |
| %g          | two-digit year format, aligned to ISO 8601, abbreviated from four-digit notation | 18         |
| %H          | hour in 24h format (00-23)                                   | 22         |
| %I          | hour in 12h format (01-12)                                   | 10         |
| %j          | day of the year (001-366)                                    | 002        |
| %m          | month as a decimal number (01-12)                            | 01         |
| %M          | minute (00-59)                                               | 33         |
| %n          | new-line character (‘’)                                      |            |
| %p          | AM or PM designation                                         | PM         |
| %Q          | Quarter (1-4)                                                | 1          |
| %R          | 24-hour HH:MM time, equivalent to %H:%M                      | 22:33      |
| %S          | second (00-59)                                               | 44         |
| %t          | horizontal-tab character (’)                                 |            |
| %T          | ISO 8601 time format (HH:MM:SS), equivalent to %H:%M:%S      | 22:33:44   |
| %u          | ISO 8601 weekday as number with Monday as 1 (1-7)            | 2          |
| %V          | ISO 8601 week number (01-53)                                 | 01         |
| %w          | weekday as a decimal number with Sunday as 0 (0-6)           | 2          |
| %y          | Year, last two digits (00-99)                                | 18         |
| %Y          | Year                                                         | 2018       |
| %%          | a % sign                                                     | %          |