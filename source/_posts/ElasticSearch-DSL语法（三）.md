---
title: ElasticSearch DSL语法（三）
date: 2019-10-09 21:19:14
tags:
	- ElasticSearch
	- DSL语法
categories: 
	- ElasticSearch
---

> 本文参考：https://www.cnblogs.com/shoufeng/p/11290669.html

## 普通聚合分析

### 直接聚合统计

**计算每个title下的文档数量**
```json
GET articles/_search
{
  "aggs": {
    "group_by_title": { //这里的名字自己起
      "terms": {
            "field": "title"
        }
    }
  }
}
```

**报错解决方案**
- 解决方案1：对text类型的字段开启fielddata属性。
```json
PUT articles/_mapping/article
{
    "properties": {
        "title": {
            "type": "text",
            "fielddata": true
        }
    }
}
```
- 解决方案2：使用内置keyword字段。
```json
GET articles/_search
{
  "aggs": {
    "group_by_title": { 
      "terms": {
            "field": "title.keyword"
        }
    }
  }
}
```

## 先检索，再聚合

<!-- more-->

```json
GET articles/_search
{
  "query": {
    "match": {
      "title": "源码"
    }
  },
  "aggs": {
    "group_by_title": {
      "terms": {
        "field": "title"
      }
    }
  }
}
```

### 扩展: fielddata和keyword的聚合比较

- 为某个 text 类型的字段开启fielddata字段后, 聚合分析操作会对这个字段的所有分词分别进行聚合, 获得的结果大多数情况下并不符合我们的需求。
- 使用keyword内置字段, 不会对相关的分词进行聚合, 结果可能更有用。

**推荐使用text类型字段的内置keyword进行聚合操作。**

## 嵌套聚合

**初始化数据**
```json
PUT book_shop/it_book/1
{
	"name": "深入理解Java虚拟机：JVM高级特性与最佳实践",
	"author": "周志明",
	"category": "编程语言",
	"desc": "Java图书领域公认的经典著作",
	"price": 79.0,
	"date": "2013-10-01",
	"publisher": "机械工业出版社",
	"tags": [
		"Java",
		"虚拟机",
		"最佳实践"
	]
}

PUT book_shop/it_book/2
{
	"name": "Java设计模式",
	"author": "刘伟",
	"category": "Java设计模式",
	"desc": "23种经典设计模式",
	"price": 59.0,
	"date": "2013-10-01",
	"publisher": "机械工业出版社",
	"tags": [
		"Java",
		"设计模式"
	]
}

PUT book_shop/it_book/3
{
	"name": "Spring Cloud和Docker",
	"author": "未知",
	"category": "未知",
	"desc": "未知",
	"price": 59.0,
	"date": "2013-10-01",
	"publisher": "机械工业出版社",
	"tags": [
		"Spring Cloud",
		"Docker"
	]
}
```

### 先分组，再聚合统计

```json
GET book_shop/_search
{
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags.keyword"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

**响应结果**

```json
{
  "took": 33,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Java设计模式",
          "author": "刘伟",
          "category": "Java设计模式",
          "desc": "23种经典设计模式",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "设计模式"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "深入理解Java虚拟机：JVM高级特性与最佳实践",
          "author": "周志明",
          "category": "编程语言",
          "desc": "Java图书领域公认的经典著作",
          "price": 79,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "虚拟机",
            "最佳实践"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Spring Cloud和Docker",
          "author": "未知",
          "category": "未知",
          "desc": "未知",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Spring Cloud",
            "Docker"
          ]
        }
      }
    ]
  },
  "aggregations": {
    "group_by_tags": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Java",
          "doc_count": 2,
          "avg_price": {
            "value": 69
          }
        },
        {
          "key": "Docker",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        },
        {
          "key": "Spring Cloud",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        },
        {
          "key": "最佳实践",
          "doc_count": 1,
          "avg_price": {
            "value": 79
          }
        },
        {
          "key": "虚拟机",
          "doc_count": 1,
          "avg_price": {
            "value": 79
          }
        },
        {
          "key": "设计模式",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        }
      ]
    }
  }
}
```

### 先分组, 再统计, 最后排序

```json
GET book_shop/_search
{
  "aggs": {
    "group_by_tags": {
      "terms": {
        "field": "tags.keyword",
        "order": {
          "avg_price": "asc"
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

**响应结果**
```json
{
  "took": 33,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Java设计模式",
          "author": "刘伟",
          "category": "Java设计模式",
          "desc": "23种经典设计模式",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "设计模式"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "深入理解Java虚拟机：JVM高级特性与最佳实践",
          "author": "周志明",
          "category": "编程语言",
          "desc": "Java图书领域公认的经典著作",
          "price": 79,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "虚拟机",
            "最佳实践"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Spring Cloud和Docker",
          "author": "未知",
          "category": "未知",
          "desc": "未知",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Spring Cloud",
            "Docker"
          ]
        }
      }
    ]
  },
  "aggregations": {
    "group_by_tags": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Docker",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        },
        {
          "key": "Spring Cloud",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        },
        {
          "key": "设计模式",
          "doc_count": 1,
          "avg_price": {
            "value": 59
          }
        },
        {
          "key": "Java",
          "doc_count": 2,
          "avg_price": {
            "value": 69
          }
        },
        {
          "key": "最佳实践",
          "doc_count": 1,
          "avg_price": {
            "value": 79
          }
        },
        {
          "key": "虚拟机",
          "doc_count": 1,
          "avg_price": {
            "value": 79
          }
        }
      ]
    }
  }
}
```

### 先分组, 组内再分组, 然后统计、排序

```json
GET book_shop/_search
{
  "aggs": {
    "group_by_price": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 50,
            "to": 60
          },
           {
            "from": 60,
            "to": 70
          }
        ]
      },
      "aggs": {
        "group_by_tags": {
          "terms": {
            "field": "tags.keyword"
          },
          "aggs": {
            "avg_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```

**响应结果**
```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 3,
    "max_score": 1,
    "hits": [
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "Java设计模式",
          "author": "刘伟",
          "category": "Java设计模式",
          "desc": "23种经典设计模式",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "设计模式"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "1",
        "_score": 1,
        "_source": {
          "name": "深入理解Java虚拟机：JVM高级特性与最佳实践",
          "author": "周志明",
          "category": "编程语言",
          "desc": "Java图书领域公认的经典著作",
          "price": 79,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Java",
            "虚拟机",
            "最佳实践"
          ]
        }
      },
      {
        "_index": "book_shop",
        "_type": "it_book",
        "_id": "3",
        "_score": 1,
        "_source": {
          "name": "Spring Cloud和Docker",
          "author": "未知",
          "category": "未知",
          "desc": "未知",
          "price": 59,
          "date": "2013-10-01",
          "publisher": "机械工业出版社",
          "tags": [
            "Spring Cloud",
            "Docker"
          ]
        }
      }
    ]
  },
  "aggregations": {
    "group_by_price": {
      "buckets": [
        {
          "key": "50.0-60.0",
          "from": 50,
          "to": 60,
          "doc_count": 2,
          "group_by_tags": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "Docker",
                "doc_count": 1,
                "avg_price": {
                  "value": 59
                }
              },
              {
                "key": "Java",
                "doc_count": 1,
                "avg_price": {
                  "value": 59
                }
              },
              {
                "key": "Spring Cloud",
                "doc_count": 1,
                "avg_price": {
                  "value": 59
                }
              },
              {
                "key": "设计模式",
                "doc_count": 1,
                "avg_price": {
                  "value": 59
                }
              }
            ]
          }
        },
        {
          "key": "60.0-70.0",
          "from": 60,
          "to": 70,
          "doc_count": 0,
          "group_by_tags": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": []
          }
        }
      ]
    }
  }
}
```