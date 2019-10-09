---
title: ElasticSearch基本语法
date: 2019-10-09 21:18:13
tags:
	- ElasticSearch
	- 基本语法
categories: 
	- ElasticSearch
---

> 本文参考:[【易百教程-ElasticSearch教程】](https://www.yiibai.com/elasticsearch/elasticsearch-getting-start.html#article-start)

## 创建索引

`http://localhost:9200/<index>/<type>/[<id>]`

索引和类型是必须的，`id`可选，如果不指定`id`则ES会自动生成一个ID,如果不指定ID，应该使用HTTP的POST请求而不是PUT。

```json
POST /test/gaolu
{
  "name": "gaolu",
  "gender": "1",
  "age": 23
}
```
```json
{
  "_index" : "test",
  "_type" : "gaolu",
  "_id" : "R2o49WwBVPKSDjq3ebzo",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

## 更新索引

只需使用相同的ID索引它。使用与之前完全相同的索引请求，但类型扩展了JSON对象。

<!-- more-->

```json
PUT /test/gaolu/R2o49WwBVPKSDjq3ebzo
{
  "name": "gaolu",
  "gender": "1",
  "age": 23,
  "others": "180cm、75kg"
}
```
```json
{
  "_index" : "test",
  "_type" : "gaolu",
  "_id" : "R2o49WwBVPKSDjq3ebzo",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

## 由ID获取索引/文档

向同一个URL发出一个GET请求，URL的ID部分是强制性的。通过ID从ElasticSearch中检索文档可发出URL的GET请求：`http://localhost:9200/<index>/<type>/<id>`。

```
GET /test/gaolu/R2o49WwBVPKSDjq3ebzo
```
```json
{
  "_index" : "test",
  "_type" : "gaolu",
  "_id" : "R2o49WwBVPKSDjq3ebzo",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "name" : "gaolu",
    "gender" : "1",
    "age" : 23,
    "others" : "180cm、75kg"
  }
}
```

## 删除文档

为了通过ID从索引中删除单个指定的文档，使用与获取索引文档相同的URL，只是这里将HTTP方法更改为DELETE。

```
DELETE /test/gaolu/R2o49WwBVPKSDjq3ebz
```
```json
{
  "_index" : "test",
  "_type" : "gaolu",
  "_id" : "R2o49WwBVPKSDjq3ebzo",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

## 搜索

**添加数据**

```json
PUT /movies/movie/1
{
    "title": "The Godfather",
    "director": "Francis Ford Coppola",
    "year": 1972,
    "genres": ["Crime", "Drama"]
}

PUT /movies/movie/2
{
    "title": "Lawrence of Arabia",
    "director": "David Lean",
    "year": 1962,
    "genres": ["Adventure", "Biography", "Drama"]
}

PUT /movies/movie/3
{
    "title": "To Kill a Mockingbird",
    "director": "Robert Mulligan",
    "year": 1962,
    "genres": ["Crime", "Drama", "Mystery"]
}

PUT /movies/movie/4
{
    "title": "Apocalypse Now",
    "director": "Francis Ford Coppola",
    "year": 1979,
    "genres": ["Drama", "War"]
}

PUT /movies/movie/5
{
    "title": "Kill Bill: Vol. 1",
    "director": "Quentin Tarantino",
    "year": 2003,
    "genres": ["Action", "Crime", "Thriller"]
}

PUT /movies/movie/6
{
    "title": "The Assassination of Jesse James by the Coward Robert Ford",
    "director": "Andrew Dominik",
    "year": 2007,
    "genres": ["Biography", "Crime", "Drama"]
}
```

### _search端点

```
- /_search 搜索所有索引和所有类型。
http://localhost:9200/_search
- /<index>/_search 在电影索引中搜索所有类型
http://localhost:9200/movies/_search
- /<index>/<type>/_search 在电影索引中显式搜索电影类型的文档
http://localhost:9200/movies/movie/_search
```

### 搜索请求正文和ElasticSearch查询DSL



### 基本自由文本搜索

查询DSL具有一长列不同类型的查询可以使用。 对于“普通”自由文本搜索，最有可能想使用一个名称为“查询字符串查询”。

查询字符串查询是一个高级查询，有很多不同的选项，ElasticSearch将解析和转换为更简单的查询树。如果忽略了所有的可选参数，并且只需要给它一个字符串用于搜索，它可以很容易使用。

```json
POST _search
{
    "query": {
        "query_string": {
            "query": "Java"
        }
    }
}
```

### 指定搜索的字段

```json
POST _search
{
    "query": {
        "query_string": {
            "query": "Java"
            , "fields": ["tags"]
        }
    }
}
```

fields字段若是不指定，会默认自动生成名为`_all`的特殊字段，来基于所有文档中的各个字段进行匹配。