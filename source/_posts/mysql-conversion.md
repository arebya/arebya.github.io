---
title: mysql_conversion
date: 2018-03-30 19:28:00
tags: mysql
---

### 隐式类型转换

#### 数据类型字段

如果以数字开关的，后面的字符将被截断，只取前面的数字值，如果不以数字开关的将被置为0

table表中的columnA字段为int类型
```
select * from table where columnA='7788Ab'
```
相当于
```
select * from table where columnA=7788
```

而
```
select * from table where columnA='Ab7788'
```
相当于
```
select * from table where columnA=0
```
#### 字符类型字段
table表中的columnB字段为varchar类型,且有索引idx_columnB
```
select * from table where columnB='7788Ab'
```
可以正常使用索引
而
```
select * from table where columnB=7788
```
发生隐式类型转换，不能使用索引

#### 官方规则

- 两个参数至少有一个是 NULL 时，比较的结果也是 NULL，例外是使用 <=> 对两个 NULL 做比较时会返回 1，这两种情况都不需要做类型转换
- 两个参数都是字符串，会按照字符串来比较，不做类型转换
- 两个参数都是整数，按照整数来比较，不做类型转换
- 十六进制的值和非数字做比较时，会被当做二进制串
- 有一个参数是 TIMESTAMP 或 DATETIME，并且另外一个参数是常量，常量会被转换为 timestamp
- 有一个参数是 decimal 类型，如果另外一个参数是 decimal 或者整数，会将整数转换为 decimal 后进行比较，如果另外一个参数是浮点数，则会把 decimal 转换为浮点数进行比较
- 所有其他情况下，两个参数都会被转换为浮点数再进行比较



