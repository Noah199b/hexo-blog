---
title: ElasticSearch DSL语法（一）
date: 2019-10-09 21:18:41
tags:
	- ElasticSearch
	- DSL语法
categories: 
	- ElasticSearch
---

> 本文参考：
> - https://blog.csdn.net/jiaminbao/article/details/80105636
> - https://www.cnblogs.com/shoufeng/p/11096521.html

## DSL语句校验

```json
GET /movies/movie/_validate/query?explain
{
  "query": {
    "match": {
      "title": "t"
    }
  }
}
```

## DSL简单用法

### 查询所有
```json
GET /movies/movie/_search
{
  "query": {
    "match_all": {
    }
  }
}
```

### 查询标题包含of的电影，并按年份降序排序
```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": "of"
    }
  },
  "sort": [
    {
      "year": {
        "order": "desc"
      }
    }
  ]
}
```

### 分页查询

<!-- more-->

```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": "of"
    }
  },
  "size": 1,    ## 查询个数
  "from": 0     ## 从第几个开始(from = （页数-1）*size)
}
```

### 指定查询结果字段
```json
GET /movies/movie/_search
{
  "query": {
    "match_all": {
    }
  },
  "_source": ["title","year"]
}
```

### 相关符号
| 符号表示 | 代表含义 |
| -------- | -------- |
| gte      | 大于等于 |
| gt       | 大于     |
| lte      | 小于等于 |
| lt       | 小于     |

### 查询名称包含of且年份在1950-1970之间的电影
```json
GET /movies/movie/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "title": "of"
        }
      },
      "filter": {
        "range": {
          "year": {
            "gte": 1950,
            "lte": 1970
          }
        }
      }
    }
  }
}
```

## full-text search 搜索
- 全文检索，倒排索引
- 索引中包含任意一个匹配拆分后的词就可以出现在结果中，只是匹配度越高越排前面。

例如：同时包含`The`和`of`的数据会显示在最前面。
```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": "The of"
    }
  }
}
```
```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.3861481,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "6",
        "_score" : 1.3861481,
        "_source" : {
          "title" : "The Assassination of Jesse James by the Coward Robert Ford",
          "director" : "Andrew Dominik",
          "year" : 2007,
          "genres" : [
            "Biography",
            "Crime",
            "Drama"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "2",
        "_score" : 0.5619609,
        "_source" : {
          "title" : "Lawrence of Arabia",
          "director" : "David Lean",
          "year" : 1962,
          "genres" : [
            "Adventure",
            "Biography",
            "Drama"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "title" : "The Godfather",
          "director" : "Francis Ford Coppola",
          "year" : 1972,
          "genres" : [
            "Crime",
            "Drama"
          ]
        }
      }
    ]
  }
}
```

## phrase search 短语搜索(精确查询)

- 不会对查询串进行分词, 而是直接精确匹配查找。

例如：`The`和`Assassination`不会进行分词，而是进行全匹配。
```json
GET /movies/movie/_search
{
  "query": {
    "match_phrase": {
      "title": "The Assassination"
    }
  }
}
```
```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.3921447,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "6",
        "_score" : 1.3921447,
        "_source" : {
          "title" : "The Assassination of Jesse James by the Coward Robert Ford",
          "director" : "Andrew Dominik",
          "year" : 2007,
          "genres" : [
            "Biography",
            "Crime",
            "Drama"
          ]
        }
      }
    ]
  }
}
```

## operator 控制匹配规则

- and : 作用等于 match_phrase ，全匹配。
```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "The Assassination",
        "operator": "and"
      }
    }
  }
}
```
- or : 作用等于match ，部分匹配。
```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "The Assassination",
        "operator": "or"
      }
    }
  }
}
```

## Highlight Search 高亮搜索

- 给匹配拆分后的查询词增加高亮的 html 标签。例如:`<em>The</em>`

```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "The Assassination",
        "operator": "or"
      }
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```
```json
{
  "took" : 208,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.7486695,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "6",
        "_score" : 1.7486695,
        "_source" : {
          "title" : "The Assassination of Jesse James by the Coward Robert Ford",
          "director" : "Andrew Dominik",
          "year" : 2007,
          "genres" : [
            "Biography",
            "Crime",
            "Drama"
          ]
        },
        "highlight" : {
          "title" : [
            "<em>The</em> <em>Assassination</em> of Jesse James by <em>the</em> Coward Robert Ford"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "title" : "The Godfather",
          "director" : "Francis Ford Coppola",
          "year" : 1972,
          "genres" : [
            "Crime",
            "Drama"
          ]
        },
        "highlight" : {
          "title" : [
            "<em>The</em> Godfather"
          ]
        }
      }
    ]
  }
}
```

## minimum_should_match 指定命中的百分比

- 用来指定最少要匹配多少比例的分词, 才算符合条件并返回结果.

```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "the",
        "minimum_should_match": "50%"
      }
    }
  }
}
```

## multi_match 多字段匹配

- 用来对多个字段同时进行匹配: 任意一个字段中存在相应的分词, 就可作为结果返回。

例如：只要字段`title`或`director`存在`James`或`Coppola`就会返回。
```json
GET /movies/movie/_search
{
  "query": {
    "multi_match": {
      "query": "James Coppola",
      "fields": ["title","director"]
    }
  }
}
```
```json
{
  "took" : 69,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 0.8781843,
    "hits" : [
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "4",
        "_score" : 0.8781843,
        "_source" : {
          "title" : "Apocalypse Now",
          "director" : "Francis Ford Coppola",
          "year" : 1979,
          "genres" : [
            "Drama",
            "War"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "6",
        "_score" : 0.69607234,
        "_source" : {
          "title" : "The Assassination of Jesse James by the Coward Robert Ford",
          "director" : "Andrew Dominik",
          "year" : 2007,
          "genres" : [
            "Biography",
            "Crime",
            "Drama"
          ]
        }
      },
      {
        "_index" : "movies",
        "_type" : "movie",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "title" : "The Godfather",
          "director" : "Francis Ford Coppola",
          "year" : 1972,
          "genres" : [
            "Crime",
            "Drama"
          ]
        }
      }
    ]
  }
}

```

## bool query 布尔查询

- must：必须匹配，相当于sql中的 `=`；
- must_not：必须不匹配，相当于sql中的`!=`；
- should：不强匹配，相当于sql中的`or`；
- filter：过滤，将满足一定条件的文档筛选出来。

### 简单示例

```json
GET /movies/movie/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{
          "title": "The"
        }
      },
      "must_not": {
        "match": {"director": "Coppola"}
      },
      "should": {
        "range": {
          "year": {
            "gte": 2000
          }
        }
      },
      "filter": {
        "bool": {
          "must":{
            "range": {
              "year": {
                "gte": 2005,
                "lte": 2008
              }
            }
          }
        }
      }
    }
  }
}
```

##  match_phrase + slop 

例如：`The` Assassination `of` Jesse James by the Coward Robert Ford.

若要匹配`The of`是匹配不到的，因为中间还有一个单词`Assassination`，无法进行全匹配。但是有时候我们就是要这种不完全匹配，只要求他们尽可能靠谱，中间有几个单词是没啥问题的，那就可以用到 slop。slop = 2 表示中间如果间隔 2 个单词以内也算是匹配的结果。

```json
GET /movies/movie/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "the of",
        "slop": 1
      }
    }
  }
}
```

##  term query 不分词检索

- 把检索串当作一个整体来执行检索, 即不会对检索串分词。

```json
GET /movies/movie/_search
{
  "query": {
    "term": {
      "title.keyword": "Lawrence of Arabia"
    }
  }
}
```

## terms query 相当于in查询

- 相当于多个term检索, 类似于SQL中in关键字的用法。

```json
GET /movies/movie/_search
{
  "query": {
    "terms": {
      "title.keyword": ["Lawrence of Arabia","Apocalypse Now"]
    }
  }
}
```

## prefix query 前缀检索

- 检索以某关键字开始的文档。—— 扫描所有倒排索引, 性能较差。

```json
GET /movies/movie/_search
{
  "query": {
    "prefix": {
      "title": {
        "value": "the"
      }
    }
  }
}
```

## wildcard query 通配符检索

- 扫描所有倒排索引, 性能较差。

```json
GET /movies/movie/_search
{
  "query": {
    "wildcard": {
      "title": "*"
    }
  }
}
```

## regexp query 正则检索

- 扫描所有倒排索引, 性能较差。

```json
GET /movies/movie/_search
{
  "query": {
    "regexp": {
      "title": "[a-z]+"
    }
  }
}
```

## fuzzy query 纠错检索

- fuzziness的默认值是2 —— 表示最多可以纠错两次。

> 说明: fuzziness的值太大, 将削弱检索条件的作用, 也就是说纠错次数太多, 就会导致限定检索结果的检索条件被改变, 失去了限定作用。

```json
GET /movies/movie/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Th",
        "fuzziness": 1, 
        "operator": "or"
      }
    }
  }
}
```

## boost评分权重 控制文档的优先级别

- 通过boost参数, 令满足某个条件的文档的得分更高, 从而使得其排名更靠前。

```json
GET /movies/movie/_search
{
  "query": {
        "bool": {
            "must": [
                { "match": { "title": "The"} }
            ], 
            "should": [
                { 
                   "match": { 
                        "director": {
                            "query": "Ford", 
                            "boost": 2
                        } 
                    }
                }
            ]
        }
    }
}
```