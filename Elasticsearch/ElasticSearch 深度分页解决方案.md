# ElasticSearch 深度分页解决方案

## 目录

*   [前言](#前言)

*   [From/Size参数](#fromsize参数)

*   [深度分页问题](#深度分页问题)

*   [Scroll](#scroll)

*   [Scroll Scan](#scroll-scan)

*   [Sliced Scroll](#sliced-scroll)

*   [Search After](#search-after)

*   [总结](#总结)

*   [ES7版本变更](#es7版本变更)

*   [性能对比](#性能对比)

*   [向前翻页](#向前翻页)

*   [总结](#总结-1)

*   [个人思考](#个人思考)

## 前言

Elasticsearch 是一个实时的分布式搜索与分析引擎，在使用过程中，有一些典型的使用场景，比如分页、遍历等。

在使用关系型数据库中，我们被告知要注意甚至被明确禁止使用深度分页，同理，在 Elasticsearch 中，也应该尽量避免使用深度分页。

## From/Size参数

在ES中，分页查询默认返回最顶端的10条匹配hits。

如果需要分页，需要使用from和size参数。

*   from参数定义了需要跳过的hits数，默认为0；

*   size参数定义了需要返回的hits数目的最大值。

一个基本的ES查询语句是这样的：

```json
POST /my_index/my_type/_search
{
    "query": { "match_all": {}},
    "from": 100,
    "size":  10
}
```

上面的查询表示从搜索结果中取第100条开始的10条数据。

**「那么，这个查询语句在ES集群内部是怎么执行的呢？」**

在ES中，搜索一般包括两个阶段，query 和 fetch 阶段，可以简单的理解，query 阶段确定要取哪些doc，fetch 阶段取出具体的 doc。

![Query阶段](image/image_ekG4yBWQ-A.png "Query阶段")

如上图所示，描述了一次搜索请求的 query 阶段：·

1.  Client 发送一次搜索请求，node1 接收到请求，然后，node1 创建一个大小为`from + size`的优先级队列用来存结果，我们管 node1 叫 coordinating node。

2.  coordinating node将请求广播到涉及到的 shards，每个 shard 在内部执行搜索请求，然后，将结果存到内部的大小同样为`from + size` 的优先级队列里，可以把优先级队列理解为一个包含`top N`结果的列表。

3.  每个 shard 把暂存在自身优先级队列里的数据返回给 coordinating node，coordinating node 拿到各个 shards 返回的结果后对结果进行一次合并，产生一个全局的优先级队列，存到自身的优先级队列里。

在上面的例子中，coordinating node 拿到`(from + size) * 6`条数据，然后合并并排序后选择前面的`from + size`条数据存到优先级队列，以便 fetch 阶段使用。

另外，各个分片返回给 coordinating node 的数据用于选出前`from + size`条数据，所以，只需要返回唯一标记 doc 的`_id`以及用于排序的`_score`即可，这样也可以保证返回的数据量足够小。

coordinating node 计算好自己的优先级队列后，query 阶段结束，进入 fetch 阶段。

query 阶段知道了要取哪些数据，但是并没有取具体的数据，这就是 fetch 阶段要做的。

![Fetch阶段](image/image_KNlqvi7XGn.png "Fetch阶段")

上图展示了 fetch 过程：

1.  coordinating node 发送 GET 请求到相关shards。

2.  shard 根据 doc 的`_id`取到数据详情，然后返回给 coordinating node。

3.  coordinating node 返回数据给 Client。

coordinating node 的优先级队列里有`from + size` 个`_doc _id`，但是，在 fetch 阶段，并不需要取回所有数据，在上面的例子中，前100条数据是不需要取的，只需要取优先级队列里的第101到110条数据即可。

需要取的数据可能在不同分片，也可能在同一分片，coordinating node 使用 **「multi-get」** 来避免多次去同一分片取数据，从而提高性能。

**「这种方式请求深度分页是有问题的：」**

我们可以假设在一个有 5 个主分片的索引中搜索。当我们请求结果的第一页（结果从 1 到 10 ），每一个分片产生前 10 的结果，并且返回给 协调节点 ，协调节点对 50 个结果排序得到全部结果的前 10 个。

现在假设我们请求第 1000 页—结果从 10001 到 10010 。所有都以相同的方式工作除了每个分片不得不产生前10010个结果以外。然后协调节点对全部 50050 个结果排序最后丢弃掉这些结果中的 50040 个结果。

**「对结果排序的成本随分页的深度成指数上升。」**

**「注意1：」**

size的大小不能超过`index.max_result_window`这个参数的设置，默认为10000。

如果搜索size大于10000，需要设置`index.max_result_window`参数

```json

PUT _settings
{
    "index": {
        "max_result_window": "10000000"
    }
}  
```

**「注意2：」**

`_doc`将在未来的版本移除，详见：

*   [https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0](https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0 "https://www.elastic.co/cn/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0")

*   [https://elasticsearch.cn/article/158](https://elasticsearch.cn/article/158 "https://elasticsearch.cn/article/158")

![](image/image_PGsG4aZSPb.png)

## 深度分页问题

Elasticsearch 的From/Size方式提供了分页的功能，同时，也有相应的限制。

举个例子，一个索引，有10亿数据，分10个 shards，然后，一个搜索请求，from=1000000，size=100，这时候，会带来严重的性能问题：CPU，内存，IO，网络带宽。

在 query 阶段，每个shards需要返回 1000100 条数据给 coordinating node，而 coordinating node 需要接收`10 * 1000`，100 条数据，即使每条数据只有 `_doc _id` 和 `_score`，这数据量也很大了？

**「在另一方面，我们意识到，这种深度分页的请求并不合理，因为我们是很少人为的看很后面的请求的，在很多的业务场景中，都直接限制分页，比如只能看前100页。」**

比如，有1千万粉丝的微信大V，要给所有粉丝群发消息，或者给某省粉丝群发，这时候就需要取得所有符合条件的粉丝，而最容易想到的就是利用 from + size 来实现，不过，这个是不现实的，这时，可以采用 Elasticsearch 提供的其他方式来实现遍历。

深度分页问题大致可以分为两类：

*   **「随机深度分页：随机跳转页面」**

*   **「滚动深度分页：只能一页一页往下查询」**

**「下面介绍几个官方提供的深度分页方法」**

## Scroll

我们可以把scroll理解为关系型数据库里的cursor，因此，scroll并不适合用来做实时搜索，而更适合用于后台批处理任务，比如群发。

这个分页的用法，**「不是为了实时查询数据」**，而是为了\*\*「一次性查询大量的数据（甚至是全部的数据」\*\*）。

因为这个scroll相当于维护了一份当前索引段的快照信息，这个快照信息是你执行这个scroll查询时的快照。在这个查询后的任何新索引进来的数据，都不会在这个快照中查询到。

但是它相对于from和size，不是查询所有数据然后剔除不要的部分，而是记录一个读取的位置，保证下一次快速继续读取。

不考虑排序的时候，可以结合`SearchType.SCAN`使用。

scroll可以分为初始化和遍历两部，初始化时将\*\*「所有符合搜索条件的搜索结果缓存起来（注意，这里只是缓存的doc\_id，而并不是真的缓存了所有的文档数据，取数据是在fetch阶段完成的）」\*\*，可以想象成快照。

在遍历时，从这个快照里取数据，也就是说，在初始化后，对索引插入、删除、更新数据都不会影响遍历结果。

**「基本使用」**

```json

POST /twitter/tweet/_search?scroll=1m
{
    "size": 100,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
```

初始化指明 index 和 type，然后，加上参数 scroll，表示暂存搜索结果的时间，其它就像一个普通的search请求一样。

会返回一个`_scroll_id`，`_scroll_id`用来下次取数据用。

**「遍历」**

```json
POST /_search?scroll=1m
{
    "scroll_id":"XXXXXXXXXXXXXXXXXXXXXXX I am scroll id XXXXXXXXXXXXXXX"
}
```

这里的`scroll_id`即 上一次遍历取回的`_scroll_id`或者是初始化返回的`_scroll_id`，同样的，需要带 scroll 参数。

重复这一步骤，直到返回的数据为空，即遍历完成。

**「注意，每次都要传参数 scroll，刷新搜索结果的缓存时间」**。另外，**「不需要指定 index 和 type」**。

设置scroll的时候，需要使搜索结果缓存到下一次遍历完成，**「同时，也不能太长，毕竟空间有限。」**

**「优缺点」**

缺点：

1.  **「scroll\_id会占用大量的资源（特别是排序的请求）」**

2.  同样的，scroll后接超时时间，频繁的发起scroll请求，会出现一些列问题。

3.  **「是生成的历史快照，对于数据的变更不会反映到快照上。」**

**「优点：」**

适用于非实时处理大量数据的情况，比如要进行数据迁移或者索引变更之类的。

## Scroll Scan

ES提供了scroll scan方式进一步提高遍历性能，但是scroll scan不支持排序，因此scroll scan适合不需要排序的场景

**「基本使用」**

Scroll Scan 的遍历与普通 Scroll 一样，初始化存在一点差别。

```json

POST /my_index/my_type/_search?search_type=scan&scroll=1m&size=50
{
 "query": { "match_all": {}}
}


```

需要指明参数：

*   `search_type`：赋值为scan，表示采用 Scroll Scan 的方式遍历，同时告诉 Elasticsearch 搜索结果不需要排序。

*   scroll：同上，传时间。

*   size：与普通的 size 不同，这个 size 表示的是每个 shard 返回的 size 数，最终结果最大为 `number_of_shards * size`。

**「Scroll Scan与Scroll的区别」**

1.  Scroll-Scan结果\*\*「没有排序」\*\*，按index顺序返回，没有排序，可以提高取数据性能。

2.  初始化时只返回 `_scroll_id`，没有具体的hits结果

3.  size控制的是每个分片的返回的数据量，而不是整个请求返回的数据量。

## Sliced Scroll

如果你数据量很大，用Scroll遍历数据那确实是接受不了，现在Scroll接口可以并发来进行数据遍历了。

每个Scroll请求，可以分成多个Slice请求，可以理解为切片，各Slice独立并行，比用Scroll遍历要快很多倍。

```json
POST /index/type/_search?scroll=1m
{
    "query": { "match_all": {}},
    "slice": {
        "id": 0,
        "max": 5
    }   
}
 
POST ip:port/index/type/_search?scroll=1m
{
    "query": { "match_all": {}},
    "slice": {
        "id": 1,
        "max": 5
    }   
}
```

上边的示例可以单独请求两块数据，最终五块数据合并的结果与直接scroll scan相同。

其中max是分块数，id是第几块。

> 官方文档中建议max的值不要超过shard的数量，否则可能会导致内存爆炸。

## Search After

`Search_after`是 ES 5 新引入的一种分页查询机制，其原理几乎就是和scroll一样，因此代码也几乎是一样的。

**「基本使用：」**

第一步：

```json
POST twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "es"
        }
    },
    "sort": [
        {"date": "asc"},
        {"_id": "desc"}
    ]
}
```

返回出的结果信息 ：

```json
{
      "took" : 29,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 5,
          "relation" : "eq"
        },
        "max_score" : null,
        "hits" : [
          {
            ...
            },
            "sort" : [
              ...
            ]
          },
          {
            ...
            },
            "sort" : [
              124648691,
              "624812"
            ]
          }
        ]
      }
    }
```

上面的请求会为每一个文档返回一个包含sort排序值的数组。

这些sort排序值可以被用于`search_after`参数里以便抓取下一页的数据。

比如，我们可以使用最后的一个文档的sort排序值，将它传递给`search_after`参数：

```json
GET twitter/_search
{
    "size": 10,
    "query": {
        "match" : {
            "title" : "es"
        }
    },
    "search_after": [124648691, "624812"],
    "sort": [
        {"date": "asc"},
        {"_id": "desc"}
    ]
}
```

若我们想接着上次读取的结果进行读取下一页数据，第二次查询在第一次查询时的语句基础上添加`search_after`，并指明从哪个数据后开始读取。

**「基本原理」**

es维护一个实时游标，它以上一次查询的最后一条记录为游标，方便对下一页的查询，它是一个无状态的查询，因此每次查询的都是最新的数据。

由于它采用记录作为游标，因此\*\*「SearchAfter要求doc中至少有一条全局唯一变量（每个文档具有一个唯一值的字段应该用作排序规范）」\*\*

**「优缺点」**

**「优点：」**

1.  无状态查询，可以防止在查询过程中，数据的变更无法及时反映到查询中。

2.  不需要维护`scroll_id`，不需要维护快照，因此可以避免消耗大量的资源。

**「缺点：」**

1.  由于无状态查询，因此在查询期间的变更可能会导致跨页面的不一值。

2.  排序顺序可能会在执行期间发生变化，具体取决于索引的更新和删除。

3.  至少需要制定一个唯一的不重复字段来排序。

4.  它不适用于大幅度跳页查询，或者全量导出，对第N页的跳转查询相当于对es不断重复的执行N次search after，而全量导出则是在短时间内执行大量的重复查询。

`SEARCH_AFTER`不是自由跳转到任意页面的解决方案，而是并行滚动多个查询的解决方案。

## 总结

| 分页方式      | 性能 | 优点                                             | 缺点                                                         | 场景                                   |
| ------------- | ---- | ------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| from + size   | 低   | 灵活性好，实现简单                               | 深度分页问题                                                 | 数据量比较小，能容忍深度分页问题       |
| scroll        | 中   | 解决了深度分页问题                               | 无法反应数据的实时性（快照版本）维护成本高，需要维护一个 scroll\_id | 海量数据的导出需要查询海量结果集的数据 |
| search\_after | 高   | 性能最好不存在深度分页问题能够反映数据的实时变更 | 实现复杂，需要有一个全局唯一的字段连续分页的实现会比较复杂，因为每一次查询都需要上次查询的结果，它不适用于大幅度跳页查询 | 海量数据的分页                         |

## ES7版本变更

参照：[https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results "https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results")

在`7.*`版本中，ES官方不再推荐使用Scroll方法来进行深分页，而是推荐使用带PIT的`search_after`来进行查询；

从`7.*`版本开始，您可以使用`SEARCH_AFTER`参数通过上一页中的一组排序值检索下一页命中。

使用`SEARCH_AFTER`需要多个具有相同查询和排序值的搜索请求。

如果这些请求之间发生刷新，则结果的顺序可能会更改，从而导致页面之间的结果不一致。

为防止出现这种情况，您可以创建一个时间点(PIT)来在搜索过程中保留当前索引状态。

```json
POST /my-index-000001/_pit?keep_alive=1m

返回一个PIT ID：
{
  "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

在搜索请求中指定PIT：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
     "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
     "keep_alive": "1m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos", "numeric_type" : "date_nanos" }}
  ]
}
```

## 性能对比

分别分页获取`1 - 10`，`49000 - 49010`，`99000 - 99010`范围各10条数据（前提10w条），性能大致是这样：

![](image/image_XpNs6Lcu5x.png)

## 向前翻页

对于向前翻页，ES中没有相应API，但是根据官方说法（[https://github.com/elastic/elasticsearch/issues/29449），ES中的向前翻页问题可以通过翻转排序方式来实现即：](https://github.com/elastic/elasticsearch/issues/29449），ES中的向前翻页问题可以通过翻转排序方式来实现即： "https://github.com/elastic/elasticsearch/issues/29449），ES中的向前翻页问题可以通过翻转排序方式来实现即：")

1.  对于某一页，正序`search_after`该页的最后一条数据id为下一页，则逆序`search_after`该页的第一条数据id则为上一页。

2.  国内论坛上，有人使用缓存来解决上一页的问题：[https://elasticsearch.cn/question/7711](https://elasticsearch.cn/question/7711 "https://elasticsearch.cn/question/7711")

![](image/image_9WmIM6_Hfu.png)

## 总结

1.  如果数据量小（from+size在10000条内），或者只关注结果集的TopN数据，可以使用from/size 分页，简单粗暴

2.  数据量大，深度翻页，后台批处理任务（数据迁移）之类的任务，使用 scroll 方式

3.  数据量大，深度翻页，用户实时、高并发查询需求，使用 search after 方式

## 个人思考

Scroll和`search_after`原理基本相同，他们都采用了游标的方式来进行深分页。

这种方式虽然能够一定程度上解决深分页问题。但是，它们并不是深分页问题的终极解决方案，深分页问题\*\*「必须避免！!」\*\*。

对于Scroll，无可避免的要维护`scroll_id`和历史快照，并且，还必须保证`scroll_id`的存活时间，这对服务器是一个巨大的负荷。

对于`Search_After`，如果允许用户大幅度跳转页面，会导致短时间内频繁的搜索动作，这样的效率非常低下，这也会增加服务器的负荷，同时，在查询过程中，索引的增删改会导致查询数据不一致或者排序变化，造成结果不准确。

`Search_After`本身就是一种业务折中方案，它不允许指定跳转到页面，而只提供下一页的功能。

Scroll默认你会在后续将所有符合条件的数据都取出来，所以，它只是搜索到了所有的符合条件的`doc_id`(这也是为什么官方推荐用`doc_id`进行排序，因为本身缓存的就是`doc_id`，如果用其他字段排序会增加查询量)，并将它们排序后保存在协调节点(coordinate node)，但是并没有将所有数据进行fetch，而是每次scroll，读取size个文档，并返回此次读取的最后一个文档以及上下文状态，用以告知下一次需要从哪个shard的哪个文档之后开始读取。

这也是为什么官方不推荐scroll用来给用户进行实时的分页查询，而是适合于大批量的拉取数据，因为它从设计上就不是为了实时读取数据而设计的。