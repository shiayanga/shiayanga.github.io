---
layout: post
title: 记一次ElasticSearch参数调优
subtitle: 
author: Shiayanga
categories: [ElasticSearch]
banner:
tags: [ElasticSearch]
sidebar: []
---

# 问题描述
生产环境中的ES在查询大数据量时一直报错

```json
{
	"error": {
		"root_cause": [],
		"type": "search_phase_execution_exception",
		"reason": "",
		"phase": "fetch",
		"grouped": true,
		"failed_shards": [],
		"caused_by": {
			"type": "too_many_buckets_exception",
			"reason": "Trying to create too many buckets. Must be less than or equal to: [65535] but was [65536]. This limit can be set by changing the [search.max_buckets] cluster level setting.",
			"max_buckets": 65535
		}
	},
	"status": 503
}
```
经过查询，是在聚合查询过程中创建了过多的存储桶，超过了系统的默认值65535
# search.max_buckets 参数
官方文档中提到 [Search settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-settings.html#search-settings-max-buckets)
![image.png](https://pic-lee.oss-cn-beijing.aliyuncs.com/202501031001969.png)
该参数是在单个响应中允许的最大的聚合存储桶数量，默认是65535，尝试返回超过此限制的请求将返回错误，也就是上面的报错。

在ElasticSearch中，桶就是指定聚合的分组，例如：

| id  | name  | age |
| --- | ----- | --- |
| 1   | zhang | 20  |
| 2   | wang1 | 30  |
| 3   | li    | 40  |
| 4   | zhao  | 40  |
| 5   | wang1 | 20  |
假设以ID聚合，就是5个桶；以name聚合，就是4个桶；以age聚合，就是3个桶。这样就可以直观的理解search.max_buckets最大能有几个桶了。

# 结论
search.max_buckets 参数默认值是65535，所以在某些条件下会创建超出该值的存储桶。

但是在某些场景下为了保证聚合后数据的完整性，可以适当的调整该参数的值。

当我们把这个值由65535调整到700000后，查询就可以正常使用了。

不过，如果数据量巨大，这个参数设置的也巨大，查询时会触发**熔断机制甚至是OOM**；在这种情况下，可以配合query条件+agg的方式实现查询。
