---
layout: post
title:  Elasticsearch RESTFul API Basic
date:   2020-12-10
Author: CBD
tags: [Elasticsearch]
---

Elasticsearch 的 RESTFul API 虽然提供了方便的查询接口, 但是多变的组合方式很让人头疼.

ES 使用了 RESTFul 风格的 HTTP Method, 但跟经典设计有些区别:

* 没有 MATCH, 用 POST 加 `_update` 代替
* PUT 新建索引, 新建或覆盖文档 (必须指明ID)
* POST 新建或修改文档, 其他设置

## 查看索引是否存在

`HEAD /marks`

## 创建索引

`PUT /marks`

```json
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "created_at": {
        "type": "date"
      },
      "updated_at": {
        "type": "date"
      },
      "user_id": {
        "type": "long"
      },
      "url": {
        "type": "keyword"
      },
      "tag": {
        "type": "keyword"
      },
      "hash_key": {
        "type": "keyword"
      },
      "selection": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "comment": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart",
        "fields": {
          "keyword": {
            "ignore_above": 256,
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 查看索引列表

`GET /_cat/indices`

response 为按行分隔的文本, 非 JSON

## 查看索引详情

`GET /marks`

* 只查看索引映射: `GET /marks/_mappings`
* 只查看索引设置: `GET /marks/_settings`

## 删除索引

`DELETE /marks`

## 新建文档

PUT 必须制定文档 `_id`, POST 可选自动生成 `_id`(推荐指定).

7.x 以后的版本限制每个索引必须使用相同的类型, 即用 `_doc` 占位.

`PUT /marks/_doc/1`

```json
{
  "id": 1,
  "created_at": "2020-12-08T15:57:09.706469055+08:00",
  "updated_at": "2020-12-08T15:57:09.706469055+08:00",
  "user_id": 123,
  "url": "http://localhost",
  "tag": "red",
  "selection": "My rabbit jumps",
  "comment": "静安嘉里中心"
}
```

`POST /marks/_doc/2`

```json
{
  "id": 2,
  "created_at": "2020-12-08T15:57:09.706469055+08:00",
  "updated_at": "2020-12-08T15:57:09.706469055+08:00",
  "user_id": 123,
  "url": "http://localhost",
  "tag": "green",
  "selection": "Jumping jack rabbits",
  "comment": "国金中心"
}
```

## 删除文档 By _id

`DELETE /marks/_doc/1`

## 修改文档部分字段

`POST /marks/_update/1`

```json
{
  "doc": {
    "new_filed": "simple str"
  }
}
```

## 根据具体值查找

根据 id 的精确值查找:

SQL: `select * from marks where id = 1`

ES :

`GET /marks/_search`

```json
{
  "query": {
    "term": {
      "id": {
        "value": 1
      }
    }
  }
}
```

根据 id 数组的精确值查找:

SQL: `select * from marks where id in (1, 2)`

ES:

`GET /marks/_search`

```json
GET /marks/_search
{
  "query": {
    "terms": {
      "id": [
        1,
        2
      ]
    }
  }
}
```

## 查询模板

常用的查询都可以用 bool 查询包装, 很多工具也是这样做的.

* must_not 一定不命中, 如果命中即排除;
* must 要求其中每个查询都命中, 即 AND 的关系; 会计算分数;
* should 如果命中则有加分, 即 OR 的关系;
* filter 同 must 但不计算分数;

sql: `SELECT*FROM marks LIMIT 20 OFFSET 0`

`GET /marks/_search`

```json
GET /marks/_search
{
  "query": {
    "bool": {
      "must_not": [],
      "must": [],
      "should": [],
      "filter": []
    }
  },
  "from": 0,
  "size": 20,
  "sort": [],
  "aggs": {}
}
```

## term

marks 中 comment, 一份数据有两种索引方式:

* text: 根据分析器先拆分 token 再录入倒排索引;
* keyword: 原样存储.

term 是 精确值匹配, 在 `comment.keyword` 中搜索:

sql:

```sql
select * from marks where `comment.keyword` = '国金中心'
```

ES:

`GET /marks/_search`

```json
{
  "query": {
    "term": {
      "comment.keyword": {
        "value": "国金中心"
      }
    }
  }
}
```

## 分析器

`GET /marks/_analyze`

```json
{
  "analyzer": "ik_smart",
  "text": [
    "静安中心"
  ]
}
```

## match

match 是构建在 term 的基础上的. 录入文档时, 文档经过 `analyzer` 分词, 搜索文本时, 文本经过 `search_analyzer` 分词, 然后分词和分词 term 精确匹配.

比如在 comment 上搜索: `静安中心`.

先经过 `search_analyzer` 得到 token `静安` 和 `中心`, 然后把他们带入倒排索引分别匹配:

sql: `select * from marks where comment = '静安' or comment = '中心'`

(这里的 sql 已经不能准确表达了, 因为 comment 是一组值.)

ES:

`GET /marks/_search`

```json
{
  "query": {
    "match": {
      "comment": "静安中心"
    }
  }
}
```

如果没有专门指定 `search_analyzer`, 那么他就跟随字段的 `analyzer`.

## multi_match

`GET /marks/_search`

```json
{
  "query": {
    "multi_match": {
      "query": "静安中心",
      "type": "best_fields", 
      "fields": [
        "comment",
        "selection"
      ]
    }
  }
}
```

`muti_match` 是对 `match` 的包装, 他在多个字段上执行 match, 根据 type 进行多种策略的算分和返回.

* best_fields, 默认的type, 只看得分最高的字段;
* most_fields, 得分最高的字段占大头, 其他字段有相应的加分, 忽略不相关的长尾;
* cross_fields, 多字段融合, 综合计算词频.

## 查询分析

`POST /marks/_validate/query?explain`

```json
{
  "query": {
    "multi_match": {
      "query": "静安中心",
      "type": "most_fields",
      "fields": [
        "comment",
        "selection"
      ]
    }
  }
}
```

开发阶段可以利用 `_validate` 查看 query 是否合法, 查看解析后的搜索逻辑.
