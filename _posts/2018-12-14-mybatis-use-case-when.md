---
layout: post
title: Mybatis 使用 case when 报错问题
date: 2018-12-14
categories:
- devlog
tags: [mybatis]
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  views: '2'
author:
  login: slayer
  email: dongyado@gmail.com
  display_name: slayer
---

近期使用 spring mybatis 开发一个小游戏数据处理系统，其中涉及一个字段增减。

使用的数据库是 mysql, seat_used 为无符号整型，用来记录已经使用的座位，因为更改这个字段有可能是多个程序并发更改，
所以使用一个座位需要 -1，空出座位时 +1

类似
```sql
update table_name set seat_used = seat_used + offset where id = 1;
```

其中 offset 可能是正数和负数，这样就遇到一个问题，有些时候为了测试或者数据临时更改会导致 seat_used 在等于 0 的时候也会被 -1，
直接导致 mysql out of range 报错:
 
 ```text
### Error updating database.  Cause: com.mysql.jdbc.MysqlDataTruncation: 
Data truncation: BIGINT UNSIGNED value is out of range in '(`table_name`.`seat_used` + -(1))'
```

初步解决这个问题的方法有多种：

1. 更新之前检查 seat_used 是否等于 0， 大于 0 就更新，这样在并发的情况下，可能会导致该更新的时候不会更新，比如在检查 
seat_used = 1 的后的这一刻，恰好有个线程更改了这个字段为 0，这样肯定会导致更新出错。

2. 把字段类型更改为有符号整型，这就需要在数据输出和更新的时候做好严格的检查，否则极少数情况下会出现数据为负数的问题，
我选择的是从根本上解决这个问题，不允许有负数这种脏数据出现在数据库。

3. 在更新的时候使用 case when 来决定是否更新。

更改 xml mapper 的文件如下：

```text
      <if test="seatUsed != null">
        seat_used = case when seat_used = 0 and #{seatUsed,jdbcType=INTEGER} < 0 then 0 else  seat_used + #{seatUsed,jdbcType=INTEGER} end,
      </if>
```

也就是，当 seat_used = 0 而且 更新值 < 0 的时候，还是更新为0，否则就 设置为 seat_used + 更新值。

执行后会报解析错误

```text
nested exception is org.apache.ibatis.builder.BuilderException: Error creating document instance. 
 Cause: org.xml.sax.SAXParseException; lineNumber: 335; columnNumber: 79; 
 The content of elements must consist of well-formed character data or markup.

```

错误指向 “ < 0 ”，刚开始以为是 mybatis 不支持这种语法，后来才回过神这是 xml 的解析错误，这样就简单了，加上 CDATA 标签解决这个问题，
对于 CDATA 里面的数据，xml 不会再解析：


```text
      <if test="seatUsed != null">
        <![CDATA[
        seat_used = case when seat_used = 0 and #{seatUsed,jdbcType=INTEGER} < 0 then 0 else  seat_used + #{seatUsed,jdbcType=INTEGER} end,
        ]]>
      </if>
```

以上就是解决 mysql 无符号字段自减导致 out of range错误的方法之一，网上主流方案是更改字段类型，根据自己的需求选择合适的方案即可。