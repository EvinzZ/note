# elasticsearch java api使用，es java封装

## 目录

*   [创建索引](#创建索引)

*   [删除索引](#删除索引)

*   [判断索引是否存在](#判断索引是否存在)

*   [通过文档id获得文档内容](#通过文档id获得文档内容)

*   [求索引库文档总数](#求索引库文档总数)

*   [分组统计](#分组统计)

*   [根据条件获得文档列表](#根据条件获得文档列表)

*   [获取分页数据](#获取分页数据)

*   [获取分页数据方法](#获取分页数据方法)

*   [es日期直方图分组统计](#es日期直方图分组统计)

## 创建索引

```java
public static boolean createIndex(String index) {
        if (!isIndexExist(index)) {
            LOG.info("Index is not exits!");
        }
        CreateIndexResponse indexresponse = client.admin().indices()
                // 这个索引库的名称还必须不包含大写字母
                .prepareCreate(index).execute().actionGet();
        LOG.info("执行建立成功？" + indexresponse.isAcknowledged());
        return indexresponse.isAcknowledged();
    }
```

## 删除索引

```java
public static boolean deleteIndex(String index) {
        if (!isIndexExist(index)) {
            LOG.info("Index is not exits!");
        }
        DeleteIndexResponse dResponse = client.admin().indices().prepareDelete(index).execute().actionGet();
        if (dResponse.isAcknowledged()) {
            LOG.info("delete index " + index + " successfully!");
        } else {
            LOG.info("Fail to delete index " + index);
        }
        return dResponse.isAcknowledged();
    }
```

## 判断索引是否存在

```java
public static boolean isIndexExist(String index) {
        IndicesExistsResponse inExistsResponse = client.admin().indices().exists(new IndicesExistsRequest(index))
                .actionGet();
        if (inExistsResponse.isExists()) {
            LOG.info("Index [" + index + "] is exist!");
        } else {
            LOG.info("Index [" + index + "] is not exist!");
        }
        return inExistsResponse.isExists();
    }
```

## 通过文档id获得文档内容

```java
public static Map<String, Object> getSourceById(String index, String type, String id, String fields) {
//fields是需要获得的列，下同
        try {
            GetResponse getResponse = client.prepareGet(index, type, id).setFetchSource(fields.split(","), null).get();
            LOG.info(getResponse.getSourceAsString());
            return getResponse.getSource();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
```

## 求索引库文档总数

```java
    /**
 * 求索引库文档总数
 *
 * @param indices 传入 _index ( + _type )
 * @return
 */
    public static long docCount(String... indices) {
        try {
            return client.prepareCount(indices).get().getCount();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 0l;
    }
```

## 分组统计

```java
 /**
 * 分组统计
 * @param index 索引名称
 * @param type 类型名称，可多个，逗号分隔
 * @param groupCol 分组聚合字段，可多个，逗号分隔
 * @param distinctCol 去重统计字段
 * @param sumCol 求和字段
 * @param matchStr 过滤条件，xxx=1，可多个，逗号分隔
 * @param size size大小
 * @param startTime 开始时间戳
 * @param endTime 结束时间戳
 * @return
 */
    public static List<Bucket> subCount(String index, String type, String groupCol, String distinctCol, String sumCol, String matchStr, int size, long startTime, long endTime) {
        SearchRequestBuilder searchRequest = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequest.setTypes(type.split(","));
        }
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(
                QueryBuilders.rangeQuery("processTime")
                        .format("epoch_millis")
                        .from(startTime)
                        .to(endTime)
                        .includeLower(true)
                        .includeUpper(true));
        if (StringUtils.isNotEmpty(matchStr)) {
            for (String s : matchStr.split(",")) {
                boolQueryBuilder.must(QueryBuilders.matchQuery(s.split("=")[0], s.split("=")[1]));
            }
        }
        AggregationBuilder tb = AggregationBuilders.terms("group_name").field(groupCol);
        if (StringUtils.isNotEmpty(distinctCol)){
            tb.subAggregation(AggregationBuilders.cardinality("distinct").field(distinctCol));
        } else if (StringUtils.isNotEmpty(sumCol)){
            tb.subAggregation(AggregationBuilders.sum("sum").field(sumCol));
        }
        searchRequest.setQuery(boolQueryBuilder)
                .setSearchType(SearchType.QUERY_THEN_FETCH)
                .addAggregation(tb);
        List<Bucket> buckets = null;
        try {
            SearchResponse searchResponse = searchRequest.execute().actionGet();
            Terms terms = searchResponse.getAggregations().get("group_name");
            buckets = terms.getBuckets();
            for (Bucket bt : buckets) {
                LOG.info(bt.getKeyAsString() + " :: " + bt.getDocCount());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return buckets;
    }
```

## 根据条件获得文档列表

```java
/**
 *
 * @param index 索引名称
 * @param type 类型名称,可传入多个type逗号分隔
 * @param startTime 开始时间
 * @param endTime 结束时间
 * @param size 文档大小限制
 * @param fields 需要的字段，逗号分隔（缺省为全部字段）
 * @param matchStr 过滤条件（xxx=111,aaa=222）
 * @return
 */
    public static List<Map<String, Object>> getDocs(String index, String type, long startTime, long endTime, int size, String fields, String matchStr) {
        SearchRequestBuilder searchRequest = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequest.setTypes(type.split(","));
        }
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        boolQuery.must(
                QueryBuilders.rangeQuery("processTime")
                        .format("epoch_millis")
                        .from(startTime)
                        .to(endTime)
                        .includeLower(true)
                        .includeUpper(true));

        if (StringUtils.isNotEmpty(matchStr)) {
            for (String s : matchStr.split(",")) {
                String[] ss = s.split("=");
                if(ss.length > 1){
                    boolQuery.must(QueryBuilders.matchQuery(s.split("=")[0], s.split("=")[1]));
                }
            }
        }

        searchRequest.setQuery(boolQuery);
        if (StringUtils.isNotEmpty(fields)) {
            searchRequest.setFetchSource(fields.split(","), null);
        }
        searchRequest.setFetchSource(true);
        List<Map<String, Object>> sourceList = null;

        try {
//      System.out.println(searchRequest);
            SearchResponse response = searchRequest
                    .addSort("timestamp", SortOrder.DESC)
                    .setSize(size)
                    .execute()
                    .actionGet();
            //System.out.println(response.toString());
            SearchHits shs = response.getHits();
            sourceList = new ArrayList<>();
            for (SearchHit sh : shs) {
                sh.getSource().put("id", sh.getId());
                sourceList.add(sh.getSource());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        LOG.info("GET DOCS SIZE: " + sourceList.size());
        return sourceList;
    }
```

## 获取分页数据

```java
public class EsPage {

    // 指定的或是页面参数
    private int currentPage; // 当前页
    private int pageSize; // 每页显示多少条

    // 查询es结果
    private int recordCount; // 总记录数
    private List<Map<String,Object>> recordList; // 本页的数据列表

    // 计算
    private int pageCount; // 总页数
    private int beginPageIndex; // 页码列表的开始索引（包含）
    private int endPageIndex; // 页码列表的结束索引（包含）

    /**
 * 只接受前4个必要的属性，会自动的计算出其他3个属性的值
 *
 * @param currentPage
 * @param pageSize
 * @param recordCount
 * @param recordList
 */
    public EsPage(int currentPage, int pageSize, int recordCount, List<Map<String,Object>> recordList) {
        this.currentPage = currentPage;
        this.pageSize = pageSize;
        this.recordCount = recordCount;
        this.recordList = recordList;

        // 计算总页码
        pageCount = (recordCount + pageSize - 1) / pageSize;

        // 计算 beginPageIndex 和 endPageIndex
        // >> 总页数不多于10页，则全部显示
        if (pageCount <= 10) {
            beginPageIndex = 1;
            endPageIndex = pageCount;
        }
        // >> 总页数多于10页，则显示当前页附近的共10个页码
        else {
            // 当前页附近的共10个页码（前4个 + 当前页 + 后5个）
            beginPageIndex = currentPage - 4;
            endPageIndex = currentPage + 5;
            // 当前面的页码不足4个时，则显示前10个页码
            if (beginPageIndex < 1) {
                beginPageIndex = 1;
                endPageIndex = 10;
            }
            // 当后面的页码不足5个时，则显示后10个页码
            if (endPageIndex > pageCount) {
                endPageIndex = pageCount;
                beginPageIndex = pageCount - 10 + 1;
            }
        }
    }
    //GET、SET方法...
```

## 获取分页数据方法

```java
 /**
 * 获取分页数据
 *
 * @param index
 * @param type
 * @param currentPage
 * @param pageSize
 * @param startTime
 * @param endTime
 * @param fields
 * @return
 * @throws Exception
 */
    public static EsPage getDocsPage(String index, String type, int currentPage, int pageSize, long startTime, long endTime, String fields) throws Exception {
        SearchRequestBuilder searchRequestBuilder = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequestBuilder.setTypes(type.split(","));
        }
        searchRequestBuilder.setSearchType(SearchType.QUERY_THEN_FETCH);
        if (StringUtils.isNotEmpty(fields)) {
            searchRequestBuilder.setFetchSource(fields.split(","), null);
        }

        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        boolQuery.must(
                QueryBuilders.rangeQuery("processTime")
                        .format("epoch_millis")
                        .from(startTime)
                        .to(endTime)
                        .includeLower(true)
                        .includeUpper(true));

        searchRequestBuilder.setQuery(QueryBuilders.matchAllQuery());
        searchRequestBuilder.setQuery(boolQuery);

        // 分页应用
        searchRequestBuilder.setFrom(currentPage).setSize(pageSize);

        // 设置是否按查询匹配度排序
        searchRequestBuilder.setExplain(true);
        SearchHits searchHits = null;
        ArrayList<Map<String, Object>> sourceList = null;

        try {
// System.out.println(searchRequestBuilder);
            // 执行搜索,返回搜索响应信息
            SearchResponse response = searchRequestBuilder.execute().actionGet();
            searchHits = response.getHits();
            if (searchHits.totalHits() == 0) return null;

            // 解析对象
            sourceList = new ArrayList<Map<String, Object>>();
            SearchHit[] hits = searchHits.hits();
            for (SearchHit sh : hits) {
                sh.getSource().put("id", sh.getId());
                sourceList.add(sh.getSource());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return new EsPage(currentPage, pageSize, (int) searchHits.totalHits(), sourceList);
    }

    private static Long getStartTime() {
        Calendar todayStart = Calendar.getInstance();
        todayStart.set(Calendar.HOUR, 0);
        todayStart.set(Calendar.MINUTE, 0);
        todayStart.set(Calendar.SECOND, 0);
        todayStart.set(Calendar.MILLISECOND, 0);
        return todayStart.getTime().getTime();
    }
```

## es日期直方图分组统计

```java
/**
 * 日期直方图分组统计
 * @param index
 * @param type
 * @param histogram 间隔时间，格式：1s/1m/1h/1d/1w/1M/1q/1y,详情见 DateHistogramInterval
 * @param groupCol 目前解析只支持四层分组统计结果，可增加
 * @param match 过滤字段，Map格式，可传入多个
 * @param size
 * @param startTime
 * @param endTime
 * @return
 */
    public static List<Map<String, Object>> getHistogramSubCountList(String index, String type, String histogram, String groupCol, Map<String, Object> match, int size, long startTime, long endTime){
        List<Map<String, Object>> listMap = new LinkedList<>();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        SearchRequestBuilder searchRequest = client.prepareSearch(index);
        if (StringUtils.isNotEmpty(type)) {
            searchRequest.setTypes(type.split(","));
        }
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(
                QueryBuilders.rangeQuery("@timestamp")
                        .from(startTime)
                        .to(endTime)
                        .includeLower(true)
                        .includeUpper(true));
        if (match != null) {
            for (Map.Entry<String, Object> entry : match.entrySet()) {
                String key = entry.getKey();
                Object value = entry.getValue();
                if (value != null && !value.equals(""))
                    boolQueryBuilder.must(QueryBuilders.termsQuery(key, value));
            }
        }

        DateHistogramBuilder db = AggregationBuilders.dateHistogram("ts").field("processTime").interval(new DateHistogramInterval(histogram));

        String[] groupCols = groupCol.split(",");
        AggregationBuilder tb = null;
        AggregationBuilder stb = null;
        for (int i = 0; i < groupCols.length; i++) {
            if (tb == null) {
                tb = AggregationBuilders.terms(i + "").field(groupCols[i]).size(size);
                stb = tb;
            }
            else{
                AggregationBuilder ntb = AggregationBuilders.terms(i + "").field(groupCols[i]).size(size);
                stb.subAggregation(ntb);
                stb = ntb;
            }
        }
        db.subAggregation(tb);
```

```java
        searchRequest.setQuery(boolQueryBuilder)
                .setSearchType(SearchType.QUERY_THEN_FETCH)
                .addAggregation(db);
        SearchResponse searchResponse = searchRequest.execute().actionGet();
        LOG.debug("searchRequest = " + searchRequest);
        LOG.debug("searchResponse = " + searchResponse);
        InternalHistogram ts = searchResponse.getAggregations().get("ts");
        List<InternalHistogram.Bucket> buckets = ts.getBuckets();
        for (InternalHistogram.Bucket bt : buckets) {
            String processTime = bt.getKeyAsString();
            Terms terms = bt.getAggregations().get("0");
            for (Bucket bucket : terms.getBuckets()) {
                String srcAddress = (String) bucket.getKey();
                if(groupCols.length == 4) {
                    Terms terms1 = bucket.getAggregations().get("1");
                    for (Bucket bucket1 : terms1.getBuckets()) {
                        Terms terms2 = bucket1.getAggregations().get("2");
                        for (Bucket bucket2 : terms2.getBuckets()) {
                            Terms terms3 = bucket2.getAggregations().get("3");
                            for (Bucket bucket3 : terms3.getBuckets()) {
                                Long docCount = bucket3.getDocCount();
                                Map<String, Object> map = new HashMap<>();
                                map.put("processTime", processTime);
                                map.put(groupCols[0], bucket.getKey());
                                map.put(groupCols[1], bucket1.getKey());
                                map.put(groupCols[2], bucket2.getKey());
                                map.put(groupCols[3], bucket3.getKey());
                                map.put("docCount", docCount.intValue());
                                LOG.debug(map.toString());
                                listMap.add(map);
                            }
                        }

                    }
                } else if(groupCols.length == 3) {
                    Terms terms1 = bucket.getAggregations().get("1");
                    for (Bucket bucket1 : terms1.getBuckets()) {
                        Terms terms2 = bucket1.getAggregations().get("2");
                        for (Bucket bucket2 : terms2.getBuckets()) {
                            Long docCount = bucket2.getDocCount();
                            Map<String, Object> map = new HashMap<>();
                            map.put("processTime", processTime);
                            map.put(groupCols[0], bucket.getKey());
                            map.put(groupCols[1], bucket1.getKey());
                            map.put(groupCols[2], bucket2.getKey());
                            map.put("docCount", docCount.intValue());
                            LOG.debug(map.toString());
                            listMap.add(map);
                        }

                    }
                } else if (groupCols.length == 2) {
                    Terms terms1 = bucket.getAggregations().get("1");
                    for (Bucket bucket1 : terms1.getBuckets()) {
                        Long docCount = bucket1.getDocCount();
                        Map<String, Object> map = new HashMap<>();
                        map.put("processTime", processTime);
                        map.put(groupCols[0], bucket.getKey());
                        map.put(groupCols[1], bucket1.getKey());
                        map.put("docCount", docCount.intValue());
                        LOG.debug(map.toString());
                        listMap.add(map);
                    }
                } else {
                    Long docCount = bucket.getDocCount();
                    Map<String, Object> map = new HashMap<>();
                    map.put("processTime", processTime);
                    map.put(groupCols[0], bucket.getKey());
                    map.put("docCount", docCount.intValue());
                    listMap.add(map);
                }
            }
        }
        return listMap;
    }

```