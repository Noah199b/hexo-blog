---
title: ElasticSearch DSL语法（二）
date: 2019-10-09 21:19:02
tags:
	- ElasticSearch
	- DSL语法
categories: 
	- ElasticSearch
---

> 本文参考：https://www.cnblogs.com/shoufeng/p/11266136.html

## 数值范围查询

**查询1950<year<2000的电影**

```json
GET movies/_search
{
  "query": {
    "range": {
      "year": {
        "gte": 1950,
        "lte": 2000
      }
    }
  }
}
```

## 时间范围查询

**初始数据**

```json
PUT /articles/article/1
{
    "title": "深入浅出JVM",
    "author": "张三",
    "post_date": "2019-08-11",
}
PUT /articles/article/2
{
    "title": "Zookeeper入门",
    "director": "李四
    "post_date": "2019-09-03",
}
PUT /articles/article/3
{
    "title": "Mybatis源码解读",
    "director": "王五",
    "post_date": "2018-01-11",
}
PUT /articles/article/4
{
    "title": "Nginx从入门到精通",
    "director": "赵六",
    "post_date": "2019-03-12",
}
PUT /articles/article/5
{
    "title": "Tomcat使用及源码分析",
    "director": "二愣子",
    "post_date": "2019-05-14",
}
```

### 简单示例

<!-- more -->

**查询一天内发布的文章**

```json
GET articles/_search
{
    "query": {
        "range": {
            "post_date": {
                "gte": "now-1d/d", 
                "lt":  "now/d"
            }
        }
    }
}
```
```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "articles",
        "_type": "article",
        "_id": "2",
        "_score": 1,
        "_source": {
          "title": "Zookeeper入门",
          "director": "李四",
          "post_date": "2019-09-03"
        }
      }
    ]
  }
}
```
### 关于时间的数学表达式(date-math)

- `now` : 系统当前时间
- `2019-10-01||` : 日期字符串||  表示任意日期。

```json
GET articles/_search
{
    "query": {
        "range": {
            "post_date": {
                "gte": "2019-01-01||", 
                "lt":  "now/d"
            }
        }
    }
}
```

## ElasticSearch支持数学表达式的时间单位

| 表达式 | 含义 | 表达式 | 含义 |
| ------ | ---- | ------ | ---- |
| y      | 年   | M      | 月   |
| w      | 星期 | d      | 天   |
| h      | 小时 | H      | 小时 |
| m      | 分钟 | s      | 秒   |

在日期之后可以选择一个或多个数学表达式：

- `+1h` : 加一小时
- `-1d` : 减一天
- `/d` : 四舍五入到最近一天的起始

## 关于时间的四舍五入

对日期中的日、月、小时等进行四舍五入时，取决于范围的结尾是包含（include）还是排除（exclude）。

- 向上舍入：移动到舍入范围的最后一毫秒；
- 向下舍入：移动到舍入范围的第一毫秒。

**示例：**

- `"gt": "2019-9-5||/M"` : 大于日期取舍，需向上舍入，结果是2019-9-30T23:59:59.999，不包含9月份。
- `"gte": "2019-9-5||/M"` : 大于等于日期取舍，需向下舍入，结果是2019-9-1，包含9月份。
- `"lt": "2019-9-5||/M"` : 小于日期取舍，需向下舍入，结果是2019-9-1，不包含9月份。
- `"lte": "2019-9-5||/M"` : 小于等于日期取舍，需向上舍入，结果是2019-9-30T23:59:59.999，包含9月份。

## 日期格式化范围查询（format）

```json
GET articles/_search
{
    "query": {
        "range": {
            "post_date": {
                "gte": "2018-01-01", 
                "lt":  "2019",
                "format": "yyyy-MM-dd||yyyy"
            }
        }
    }
}
```

> 注意: 如果日期中缺失了部分年、月、日, 缺失的部分将被填充为unix系统的初始值, 也就是1970年1月1日。<br>
> 例如: 将`dd`设置为format，像`"lt": 5`被转换为 1970-01-05T00:00:00.000Z。

## 时区范围查询(time_zone)

如果日期field的格式允许, 也可以通过在日期值本身中指定时区, 从而将日期从另一个时区的时间转换为UTC时间, 或者为其指定特定的time_zone参数。

**需要注意的是, now是不受time_zone影响的。**

## 检索（query）和过滤（filter）的区别

> 参考： https://www.cnblogs.com/shoufeng/p/11278046.html

1) 业务关心的、需要根据匹配的相关度进行排序的搜索条件 放在 query 中;
2) 业务不关心、不需要根据匹配的相关度进行排序的搜索条件 放在 filter 中.