---
layout: post
title:  Elasticsearch lookup search log
date:   2020-12-01
Author: CBD
tags: [Elasticsearch]
---

## 使用 client 的 logger

[https://github.com/elastic/go-elasticsearch/blob/master/_examples/logging/default.go](https://github.com/elastic/go-elasticsearch/blob/master/_examples/logging/default.go)

or

[https://github.com/olivere/elastic/wiki/Logging](https://github.com/olivere/elastic/wiki/Logging)

```golang
esConfig := []elastic.ClientOptionFunc{elastic.SetURL(config.GetString("es.url"))}
if gin.Mode() != gin.ReleaseMode {
	esConfig = append(esConfig, elastic.SetTraceLog(log.New(log.Writer(), "\n", 0)))
}
Client, err = elastic.NewClient(esConfig...)
```

## 使用 Elasticsearch 的 slow log

ES 的 config 中没有单独的 log 选项, 但是有 `slow log`, 类似关系型数据库的慢查询. 测试环境可以通过修改log的触发阈值来记录每条请求.

slow log 又分 `indexing log` 和 s`earch log`.

```text
PUT /mark/_settings
{
  "index.search.slowlog.threshold.fetch.trace": "0ms",
  "index.search.slowlog.level": "trace",
  "index.indexing.slowlog.threshold.index.trace": "0ms",
  "index.indexing.slowlog.level": "trace"
}
```
