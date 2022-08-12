## 简介

ES是一个开源的高扩展的分布式全文搜索引擎。

## 数据格式

ES对比MySQL

- index（索引） >> Database

- Type（类型） >> Table

- Documents（文档） >> Row

- Fields（字段） >> Column

ES里的Index可以看所一个库，而Type相当于表，Documents则相当于表的行。

这里Types的概念已经逐渐弱化，ES6.X中，一个index下已经只能包含一个type，ES7.X中，Type的概念已经被删除了。

## 倒排索引

该索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址。由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因为称为倒排索引。Elasticsearch能够实现快速、高效的搜索功能，正是基于倒排索引原理。

## index

### 创建

```json
PUT doc?pretty
{
  "settings": {
    "number_of_shards": 1,  # 设置分片
    "number_of_replicas": 0 # 设置副本
  }
}
```

### 删除

```json
DELETE doc
```

## Document

### 新增

随机ID

```json
POST doc/_doc
{
  "title": "问",
  "category": "想要",
  "price": 3
}
```

自定义ID

```json
POST doc/_doc/1001
{
  "title": "问",
  "category": "想要",
  "price": 3
}

# 或者

PUT doc/_create/1002
{
    "title": "问",
  	"category": "想要",
  	"price": 3
}
```

### 简单查询

```json
# 根据ID查询
GET doc/_doc/1001
{
  
}

或

GET doc/_search
{
  "query": {
    "match": {
      "_id": "1001"
    }
  }
}
```

```json
# 查询全部
GET doc/_search
{
  
}

# or

GET doc/_search
{
  "query": {
    "match_all": {}
  }
}
```

### 修改

全量修改

```json
PUT doc/_doc/1001
{
    "title": "问",
  	"category": "想要",
  	"price": 4
}
```

部分修改

```json
POST doc/_update/1001
{
  "doc": {
    "price": 10
  }
}
```

### 删除数据

```json
DETELE doc/_doc/1001
```

### 条件查询

根据条件查询

```json
GET doc/_search
{
  "query": {
    "match": {
      "price": 10
    }
  }
}
```

查询全部

```json
GET doc/_search
{
  "query": {
    "match_all": {
      
    }
  }
}
```

### 分页查询

```json
GET doc/_search
{
  "query": {
    "match_all": {
      
    }
  },
  "from": 0,
  "size", 1
}
```

> from：表示开始位置
>
> size：表示大小
>
> from计算公式：（页码 - 1） * 每页数据条数

### 指定查询显示的字段

```json
GET doc/_search
{
	"query": {
    "match_all": {
      
    }
  },
  "_source": ["title"]
}
```

> _source：指定返回的字段，此处只返回title字段

排序

```json
GET doc/_search
{
  "query": {
    "match_all": {
      
    }
  },
  "sort": {
    "price": {
      "order": "desc"
    }
  }
}
```

> sort用于进行排序

### 多条件查询

且

```json
GET doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "小米"
          }
        },
        {
          "match": {
            "price": 19
          }
        }
      ]
    }
  }
}
```

> must是必须的意思

或

```json
GET doc/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "小米"
          }
        },
        {
          "match": {
            "price": 19
          }
        }
      ]
    }
  }
}
```

> should表示或者

### 范围查询

```json
GET doc/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "FIELD": {
              "gte": 10,
              "lte": 20
            }
          }
        }
      ]
    }
  }
}
```

> range表示范围查询

### 全查询

```json
GET doc/_search
{
  "query": {
    "match_phrase": {
      "bt": "xxxx"
    }
  }
}
```

> match_phrase：必须包含完整字符

### 高亮显示

```json
GET doc/_search
{
  "query": {
    "match": {
      "title": "xxx"
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

> highlight：表示高亮显示



## must和filter的区别：

filter： 不计算评分， 查询效率高；有缓存；  （推荐）

	+term： 精确匹配；
	
	+match： 模糊匹配， 倒排索引；

must： 要计算评分，查询效率低；无缓存；

  +term： 精确匹配 ， 要评分；

  +match：模糊匹配， 要评分；



## 聚合查询

```json
GET doc/_search
{
  "aggs": { // 聚合操作
    "NAME": {  // 名称，随意起名
      "terms": { // 分组
        "field": "FIELD" // 分组字段
      }
    }
  }
}
```

## 设置Mapping

```json
PUT doc/_mapping
{
  "properties": {
    "name": {
      "type": "text",  // text表示字符串，可分词
      "index": true
    },
    "sex": {
      "type": "keyword", // keyword表示关键词，不可分词
      "index": true,
    },
    "tel": {
      "type": "keyword",
      "index": false
    }
  }
}
```