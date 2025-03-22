---
title: ElasticSearch
tags: elasticsearch
categories: elasticsearch
---

# [ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/index.html)

## 环境搭建

### 单机部署

1. 下载 elasticsearch
    ```bash
    curl -O https://mirrors.huaweicloud.com/elasticsearch/6.2.4/elasticsearch-6.2.4.tar.gz
    ```

2. 解压
    ```bash
    tar zxvf elasticsearch-6.2.4.tar.gz
    ```

3. 运行
    ```bash
    cd ./elasticsearch-6.2.4/bin

    ./elasticsearch -d # 以 daemon 方式启动
    # ./elasticsearch
    ```

4. 访问
    ```bash
    curl http://127.0.0.1:9200/
    ```

### 搭建可视化控制台 Kibana

1. 下载
    ```bash
    curl -O 'https://mirrors.huaweicloud.com/kibana/6.2.4/kibana-6.2.4-windows-x86_64.zip'
    ```

2. 解压
    ```bash
    unzip kibana-6.2.4-windows-x86_64.zip
    ```

3. 运行
    ```bash
    cd ./kibana-6.2.4-windows-x86_64/bin
    ./kibana
    ```

4. 访问
    ```cmd
    start http://127.0.0.1:5601/
    ```

## 语法

> 索引、类型、文档、字段关系如下  
> - Indices 索引，类似关系型数据中的数据库  
>     - Types 类型（≥7.0 版本已移除），类似关系型数据中的数据表  
>         - Documents 文档，类似关系型数据中的表记录行  
>             - Fields 字段，类似关系型数据中的表字段

### DDL(数据定义语言)

- 查看所有索引库
    ```bash
    curl http://127.0.0.1:9200/_cat/indices?v
    ```

- 创建索引库
    ```bash
    # # 创建索引库，如 commerce
    curl -XPUT http://127.0.0.1:9200/commerce

    # 创建带有类型、映射的索引
    curl -XPUT http://127.0.0.1:9200/commerce -H 'Content-Type: application/json' -d '{
        "settings": {
            "number_of_shards": 2, # 分片个数，默认为 5
            "number_of_replicas": 2 # 副本个数，默认为 1
        },
        "mappings": {
            "offering": { # 类型，7.0+ 版本无需该参数。如果需要设置默认类型，类型名称应为 "_default_"
                "properties": {
                    "name": {
                        "type": "text", # 不指定分词器时，会使用默认的 standard 分词器
                        # "index": true, # 是否索引。默认为 true，未索引的字段不可查询
                        # "store": true, # 是否存储。默认为 false，可以查询但是原始数据不能返回，但是这通常无关紧要，因为 _source 默认会存储
                        # "copy_to": "description" # 索引过程中，将字段值拷贝到指定字段
                        "fields": {
                            "suggest": {
                                "type": "completion", # Completion 建议器用
                                "analyzer": "english"
                            }
                        }
                    },
                    "description": {
                        "type": "text",
                        "analyzer": "english", # 指定使用 english 分词器
                        "fielddata": true # text 类型不能用于聚合，需要开启 fielddata 以支持聚合，但会额外占用存储。默认为关闭状态。
                    },
                    "brand": {
                        "type": "keyword" # 不需要分词的字段将 type 设置为 keyword，可以节省空间和提高写性能
                    },
                    "product": {
                        "type": "nested",
                        "properties": {
                            "price": {
                                "type": "double"
                            },
                            "stock": {
                                "type": "long"
                            }
                        }
                    },
                    "timestamp": {
                        "type": "date",
                        # "format": "yyyy-MM-dd HH:mm:ss||epoch_millis" # 格式为 `yyyy-MM-dd HH:mm:ss` 或毫秒数
                    }
                }
            }
        }
    }'
    ```

- 修改索引库
    ```bash
    # 修改索引库映射
    curl -XPOST http://127.0.0.1:9200/commerce/offering/_mapping -H 'Content-Type: application/json' -d '{
        "properties": {
            "spu": { # SPU 编码
                "type": "text"
            },
            "tag": { # 标签、关键词
                "type": "keyword"
            }
        }
    }'

    # 修改索引的副本数
    curl -XPUT http://127.0.0.1:9200/commerce/_settings -H 'Content-Type: application/json' -d '{
        "number_of_replicas": 1
    }'

    # 修改索引刷新间隔时间（当数据添加到索引后并不能马上被查询到，等到索引刷新后才会被查询到，默认为 1 秒）
    curl -XPUT http://127.0.0.1:9200/commerce/_settings -H 'Content-Type: application/json' -d '{
        "index": {
            "refresh_interval": "5s" # 配置间隔时间为 5 秒。单位支持 ms（毫秒）、s（秒）、m（分钟），默认单位为毫秒。值为 -1 时，表示不刷新索引。
        }
    }'
    
    # 开启字段的缓存，用于 text 类型字段的聚合索引（默认 text 类型不能用于聚合）
    curl -XPUT http://127.0.0.1:9200/commerce/offering/_mapping -H 'Content-Type: application/json' -d '{
        "properties": {
            "description": { # 字段名为 description
                "type": "text",
                "fielddata": true # 开启 fielddata 缓存
            }
        }
    }'
    ```

- 查看索引库映射
    ```bash
    curl -XGET http://127.0.0.1:9200/commerce/_mapping?pretty
    ```

- 删除索引库
    ```bash
    # 删除索引库，如 commerce
    curl -XDELETE http://127.0.0.1:9200/commerce

    # 删除所有索引
    curl -XDELETE http://127.0.0.1:9200/_all
    curl -XDELETE http://127.0.0.1:9200/*
    ```

- 关闭、打开索引
    ```bash
    # 关闭索引
    curl -XPOST http://127.0.0.1:9200/commerce/_close

    # 打开索引
    curl -XPOST http://127.0.0.1:9200/commerce/_open
    ```

- 重建（迁移）索引
    ```bash
    # 对当前的索引 commerce 添加别名
    curl -XPOST http://127.0.0.1:9200/_aliases -H 'Content-Type: application/json' -d '{
        "actions": [
            {
                "add": {
                    "index": "commerce",
                    "alias": "commerce_lastest"
                }
            }
        ]
    }'

    # 新增⼀个索引
    NEW_INDEX_NAME=commerce_`date +%Y%m%d%H%M%S`
    INDEX_MAPPINGS=`curl -XGET http://127.0.0.1:9200/commerce/_mappings | sed 's/^{"commerce"://' | sed 's/}$//'`
    curl -XPUT "http://127.0.0.1:9200/commerce_$(date +%Y%m%d%H%M%S)" -H 'Content-Type: application/json' -d "$INDEX_MAPPINGS"

    # 同步数据至新索引，wait_for_completion 表示是否同步执行（true）还是异步执行（false）
    curl -XPOST http://127.0.0.1:9200/_reindex?wait_for_completion=true -H 'Content-Type: application/json' -d '{
        "source": {
            "index": "commerce"
        },
        "dest": {
            "index": "'$NEW_INDEX_NAME'"
        }
    }'

    # 替换别名
    curl -XPOST http://127.0.0.1:9200/_aliases -H 'Content-Type: application/json' -d '{
        "actions": [
            {
                "add": {
                    "index": "'$NEW_INDEX_NAME'",
                    "alias": "commerce_lastest"
                }
            },
            {
                "remove": {
                    "index": "commerce",
                    "alias": "commerce_lastest"
                }
            }
        ]
    }'

    # 删除旧的索引
    curl -XDELETE http://127.0.0.1:9200/commerce

    # 验证新的索引
    curl -XPOST http://127.0.0.1:9200/commerce_lastest/_search?pretty -H 'Content-Type: application/json' -d '{
        "query": {
            "match_all": { }
        }
    }'
    ```

### DML(数据操纵语言)

- 新增
    ```bash
    # 可以不用预先创建索引库，es 将会自动创建索引库
    curl -XPOST http://127.0.0.1:9200/commerce/offering -H 'Content-Type:application/json' -d '{
        "name": "Apple iPhone 8",
        "price": 823.88
    }'

    curl -XPUT http://127.0.0.1:9200/commerce/offering/1?refresh -H 'Content-Type:application/json' -d '{ # 参数 refresh 表示添加数据时忽略 refresh_interval 配置，直接触发刷新索引
        "name": "Apple iPhone X",
        "price": 1129.08
    }'
    ```

- 修改
    ```bash
    # 根据 id 修改
    curl -XPOST http://127.0.0.1:9200/commerce/offering/1/_update -H 'Content-Type:application/json' -d '{
        "doc": {
            "price": 1098.56
        }
    }'

    # 根据条件修改，如果没有 description 字段，则修改其 description 字段值为 NA
    curl -XPOST http://127.0.0.1:9200/commerce/offering/_update_by_query -H 'Content-Type:application/json' -d '{
        "script": {
            "source": "ctx._source[\"description\"] = \"NA\"" # 语法参考 https://www.elastic.co/guide/en/elasticsearch/reference/6.2/painless-api-reference.html
        },
        "query": {
            "bool": {
                "must_not": [{
                    "exists": {
                        "field": "description"
                    }
                }]
            }
        }
    }'
    ```

- 查询
    ```bash
    # 根据 id 查询
    curl -XGET http://127.0.0.1:9200/commerce/offering/7081550

    # 查询
    curl http://127.0.0.1:9200/commerce/_search?pretty -H 'Content-Type: application/json' -d '{ # 参数 pretty 用于格式化显示结果
        "query": {
            "match": {
                "name": "iPhone"
            }
        }
    }'
    ```

- 计数
    ```bash
    curl http://127.0.0.1:9200/commerce/_count -H 'Content-Type: application/json' -d '{
        "query": {
            "match": {
                "name": "iPhone"
            }
        }
    }'
    ```

- 删除
    ```bash
    # P.S. Deleting a document doesn’t immediately remove the document from disk; it just marks it as deleted. Elasticsearch will clean up deleted documents in the background as you continue to index more data.
    curl -XDELETE http://127.0.0.1:9200/commerce/offering/7081550

    # 根据条件删除
    curl -XPOST http://127.0.0.1:9200/commerce/_delete_by_query?conflicts=proceed \ # conflicts=proceed 表示强制执行删除（执行批量删除的时候，可能会发生版本冲突）
    -H 'Content-Type: application/json' -d '{
        "query": {
            "match": {
                "name": "iPhone"
            }
        }
    }'

    # 删除文档的时候，是将新文档写入，同时将旧文档标记为已删除。 磁盘空间是否释放取决于新旧文档是否在同一个segment file里面，因此ES后台的segment merge在合并segment file的过程中有可能触发旧文档的物理删除。但因为一个shard可能会有上百个segment file，还是有很大几率新旧文档存在于不同的segment里而无法物理删除。想要手动释放空间，只能是定期做一下force merge，并且将max_num_segments设置为1。
    curl -XPOST http://127.0.0.1:9200/_forcemerge?only_expunge_deletes=true&max_num_segments=1
    ```

- 批量导入
    ```bash
    # curl -XPOST http://127.0.0.1:9200/commerce/offering/_bulk?pretty -H 'Content-Type: application/x-ndjson' --data-binary $'
    curl -XPOST http://127.0.0.1:9200/_bulk -H 'Content-Type: application/json' -d '
    {"index":{"_index":"commerce","_type":"offering","_id":"1600139446635"}}
    {"name":"HUAWEI Mate 40 Pro+","description":"HUAWEI Mate 40 Pro+ Kirin 9000 SoC chip super-sensing movie image wired and wireless dual super fast charge mobile phones","price":2050.00,"brand":"Huawei","rom":["256GB"],"ram":["12GB"],"color":["Black","White"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600065238546"}}
    {"name":"Honor 9i","description":"In stock Original Huawei Honor 9i Mobile Phone 64GB 128GB Face Recogntion Phone 5.84 inch Android 8.0 Huawei 4G Smartphone","price":119.00,"brand":"Honor","rom":["64GB"],"ram":["4GB"],"color":["Black","Blue"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"62454454292"}}
    {"name":"HUAWEI Mate 20X","description":"Global New HUAWEI Mate 20X 7.2 inch 4000mAh Battery Android Smartphone 4G Mobile Phone","price":467.00,"brand":"Huawei","rom":["64GB","128GB"],"ram":["8GB"],"color":["Silver","Blue"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600062036029"}}
    {"name":"Oneplus 8 Pro","description":"Original Oneplus 8 Pro 5G Mobile Phone 6.78 inch 865 Octa Core Four Rear Camera NFC Smartphone","price":599.00,"brand":"Oneplus","rom":["128GB","256GB"],"ram":["8GB","12GB"],"color":["Black","Blue","Green"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600193343226"}}
    {"name":"OnePlus Nord N10","description":"HOT OnePlus Nord N10 5G Mobile Phone 6.49 inch 90Hz Smooth Display 6GB 128GB Snapdragon 690 64MP Smartphone Oneplus Nord N10","price":269.00,"brand":"Oneplus","rom":["128GB"],"ram":["6GB"],"color":["Midnight Ice"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600170560559"}}
    {"name":"Xiaomi Poco M3","description":"Xiomi mobile phone poco Celular poco M3 phones 128GB 64GB on sale global version xiaomi poco m3","price":148.00,"brand":"Xiomi","rom":["64GB"],"ram":["6GB"],"color":["Blue"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"62347912813"}}
    {"name":"Redmi Note 9S","description":"Global Version Xiaomi Redmi Note 9S 4GB 64GB Full Screen AI Voice Assistant Mobile Phone","price":155.00,"brand":"Redmi","rom":["64GB"],"ram":["4GB"],"color":["Black","Blue","Grey"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600151821373"}}
    {"name":"Realme 7","description":"Realme 7 6.5 Inch Perforated Screen 8GB RAM 128GB 48MP Camera Mobile Phone","price":219,"brand":"Realme","rom":["128GB"],"ram":["8GB"],"color":["Blue","White"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"1600207086497"}}
    {"name":"Realme GT","description":"Original realme GT 5G Mobile Phone 12GB 256GB 6.43\"120Hz SuperAMOLED Snapdragon 888 Octa Core 65W Fast Charger NFC realme GT","price":560.00,"brand":"Realme","rom":["256GB"],"ram":["12GB"],"color":["Blue","White","Yellow"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"10023605369768"}}
    {"name":"Apple iPhone 12","description":"Apple iPhone 12 All China Netcom 5g mobile phone black all China Netcom 128G","price":913.76,"brand":"Apple","rom":["128GB","256GB"],"ram":["4GB"],"color":["Black","White","Red","Green","Blue"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"100005492551"}}
    {"name":"Apple iPhone 11","description":"Apple iPhone 11 (A2223) 128GB Black Mobile Unicom Telecom 4G mobile phone dual card dual standby [Airpods package]","price":857.30,"brand":"Apple","rom":["64GB","128GB","256GB"],"ram":["4GB"],"color":["Black","White","Red","Green","Blue","Purple"]}
    {"index":{"_index":"commerce","_type":"offering","_id":"10026915217186"}}
    {"name":"OPPO Find X3","description":"Oppo find X3 series 5g mobile phone oppo curved screen findx2pro findx3pro find X3 Mirror Black (8GB + 128GB) 5g all China Netcom [quick delivery from stock + 2-year warranty + 50% refund after sun exposure]","price":686.54,"brand":"Oppo","rom":["128GB","256GB"],"ram":["8GB","12GB"],"color":["Black","Blue","White"]}
    '
    ```
    ```bash
    # 导入 json 文件
    curl -O 'http://storage.ikyxxs.com/es/bookdata.json
    echo ''>> bookdata.json # https://stackoverflow.com/questions/48810804/missing-newline-for-adding-with-bulk-api

    curl -XPOST 'localhost:9200/test/book/_bulk?pretty' -H 'Content-Type: application/x-ndjson' --data-binary @bookdata.json
    curl -XPOST 'localhost:9200/test/book/_count?pretty
    ```

#### URL 查询

```bash
# 查询全部
curl -XGET http://127.0.0.1:9200/commerce/_search
curl -XGET http://127.0.0.1:9200/commerce/_search?q=*

# 查询所有字段中包含关键字 iphone 的文档(当索引一个文档，ES 把所有字符串字段值连接起来放在一个大字符串中，它被索引为一个特殊的字段 _all)
curl -XGET http://127.0.0.1:9200/commerce/_search?q=iphone

# 查询 name 字段中包含 oppo、vivo 或 "iphone 8"，timestamp 晚于 2014-09-10，_all 字段包含 android 或 ios 的文档
# curl -XGET 'http://127.0.0.1:9200/commerce/_search?pretty&q=name:(oppo vivo "iphone 8") AND timestamp:>2014-09-10 OR (android ios)'

# 查询 name 字段包含 oppo 或 vivo、timestamp 晚于 2014-09-10、_all 字段包含 android 或 ios
# curl -XGET http://127.0.0.1:9200/commerce/_search?q=+name:(oppo vivo) +timestamp:>2014-09-10 +(android ios)

# 根据时间范围查询
# curl -XGET http://127.0.0.1:9200/commerce/_search?q=timestamp:["2021-01-01 00:00:00" TO *]
```

#### DSL 查询

- 基本查询
    - 精准查询（term、terms）
        ```json
        GET /commerce/_search
        {
            "query": {
                "term": {
                    "brand": "Apple"
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "terms": { // terms 查询是 term 的扩展，可以支持多个 value 匹配，只需要一个匹配就可以了
                    "brand": ["Apple", "Huawei"]
                }
            }
        }
        ```
    - 分词匹配查询（match）
        ```json
        GET /commerce/_search
        {
            "query": {
                "match_all": { } // match_all 用于查询全部信息
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "match": { // 单个字段进行分词匹配查询
                    "name": "iphone"
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "multi_match": { // 多字段进行匹配查询
                    "query": "iphone",
                    "fields": ["name", "description"]
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "match_phrase": { // 短语匹配查询，ElasticSearch 引擎首先分析（analyze）查询字符串，从分析后的文本中构建短语查询，这意味着必须匹配短语中的所有分词，并且保证各个分词的相对位置不变
                    // "description": "4GB 64GB"
                    "description": {
                        "query": "8GB 128GB",
                        "slop": 2 // 表示 "8GB 128GB" 这个短语中，"128GB" 移动了 1 次，即最多移动了不超过 2 次，就可以跟 "8GB RAM 128GB" 匹配上了
                    }
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "match": {
                    "name": {
                        "query": "iphone",
                        "fuzziness": "AUTO"
                    }
                }
            }
        }
        ```
    - 模糊查询（fuzzy）
        ```json
        GET /commerce/_search
        {
            "query": {
                "fuzzy": {
                    "name": {
                        "value": "iphene",
                        "fuzziness": 1 // 最大编辑距离（莱文斯坦编辑距离），即可以允许纠正（入参 value）错误拼写的字符个数，默认为 2，推荐值 1。可设置为 "AUTO"，表示字符串长度为 1-2 时最大编辑距离为 0，长度为 3-5 时最大编辑距离为 1，长度大于 5 时最大编辑距离为 2（例如 AUTO 模式下，入参 "abcd" 能匹配 "abcde" 但不能匹配 "abcdef"，而入参 "abcdef" 能匹配 "abcd"）
                    }
                }
            }
        }
        ```
    - 通配符查询（wildcard）
        ```json
        GET /commerce/_search
        {
            "query": {
                "wildcard": {
                    "brand": "Real*" // 字符 '?' 将会匹配任何字符，'*' 将会匹配零个或者多个字符
                }
            }
        }
        ```
    - 布尔查询（bool）
        ```json
        GET /commerce/_search
        {
            "query": {
                "bool": {
                    "must": [ // must 表示查询条件为 and 关系
                        { "match": { "name": "iphone" }},
                        { "match": { "description": "unicom" }}
                    ]
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "bool": {
                    "filter": [ // 同 must，但子句将不计算得分
                        {
                            "terms": {
                                "brand": ["Huawei", "Honor"]
                            }
                        }
                    ],
                    // // should 与 filter/must 混用时，会导致 should 失效，解决方法如下
                    // // 方法一：新增 minimum_should_match 设置
                    // "should": [
                    //     { "match": { "name": "iphone" }},
                    //     { "match": { "name": "oppo"}}
                    // ],
                    // "minimum_should_match": 1 // 值为整数或百分数，如 1 或 "50%"。表示至少满足 should 中 1 个语句，或者 50% 的语句。
                    // // 方法二：将 should 嵌在 must 语句中
                    // "must": {
                    //     "bool": {
                    //         "should": [
                    //             { "match": { "name": "iphone" }},
                    //             { "match": { "name": "oppo"}}
                    //         ]
                    //     }
                    // }
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "bool": {
                    "should": [ // should 表示查询条件为 or 关系
                        { "match": { "name": "iphone" }},
                        { "match": { "name": "oppo"}}
                    ]
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "bool": {
                    "must_not": [
                        { "match": { "name": "iphone" }},
                        { "match": { "name": "oppo" }}
                    ]
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "bool": {
                    "must": [
                        {
                            "term": {
                                "brand": "Apple"
                            }
                        },
                        {
                            "bool": { // 嵌套 bool 查询，查询品牌为 Apple 并且价格不高于 900 的文档
                                "must_not": [
                                    {
                                        "range": {
                                            "price": {
                                                "gte": 900.00
                                            }
                                        }
                                    }
                                ]
                            }
                        }
                    ]
                }
            }
        }
        ```
    - 前缀查询（prefix）
        ```json
        GET /commerce/_search
        {
            "query": {
                "prefix": {
                    "name": "real"
                }
            }
        }
        ```
    - 正则查询（regexp）
        ```json
        GET /commerce/_search
        {
            "query": {
                "regexp": {
                    "name": "[a-z]+"
                }
            }
        }
        ```
    - 查询字符串查询（query_string）
        ```json
        GET /commerce/_search
        {
            "query": {
                "query_string": {
                    "query": "(xiaomi AND voice) OR movie" // 查询包含 "xiaomi" 和 "voice" 或者 "movie"
                }
            }
        }
        ```
        ```json
        GET /commerce/_search
        {
            "query": {
                "query_string": {
                    "query": "xiaomi OR movie", // 查询 name 和 description 中包含 "xiaomi" 和 "movie" 的文档
                    "fields": ["name", "description"]
                }
            }
        }
        ```
    - 范围查询（range）
        ```json
        GET /commerce/_search
        {
            "query": {
                "range": {
                    "timestamp": {
                        "gte": "1999-01-01",
                        "lte": "2000-01-01"
                        // "gt": "now-1M/d", // 当前时间的上一天，四舍五入到最近的一天（`+1h`: 加 1 小时，`-1d`: 减 1 天，`/d`: 四舍五入到最近的一天。表达式支持的时间单位有 `y`: 年，`M`: 月，`w`: 星期，`d`: 天，`h`: 小时，`H`: 小时，`m`: 分，`s`: 秒）
                        // "lt": "now/d" // 当前时间，四舍五入到最近的一天
                    }
                }
            }
        }
        ```
    - id 查询
        ```json
        GET /commerce/_search
        {
            "query": {
                "ids": {
                    "values": ["1600139446635", "1600062036029"]
                }
            }
        }
        ```

- 排序
    ```json
    GET /commerce/_search
    {
        "query": {
            "match_all": { }
        },
        "sort": {
            "price": { // 根据价格倒序排序
                "order": "desc"
            }
        }
    }
    ```

- 返回指定字段
    ```json
    GET /commerce/_search
    {
        "query": {
            "match_all": { }
        },
        "_source": ["name", "price"]
    }
    ```

- 分页
    ```json
    GET /commerce/_search
    {
        "query": {
            "match_all": { }
        },
        "from": 0,
        "size": 5
    }
    ```
    > 该分页查询方法，在深度分页场景下，查询效率低：每次查询，es 需要执行 from + size 条数据然后处理后返回。  
    > 同时 es 限制了分页的深度，默认配置最大值 max_result_window 为 10000：from + size ≤ 10000，即默认配置下查询第 ≥10000 条数据时会抛异常。

- 聚合查询
    ```json
    // 统计词频
    GET /commerce/_search
    {
        "size": 0,
        "aggs": {
            "description_words": { // 自定义聚合名称
                "terms": {
                    "size": 10,
                    "field": "description"
                }
            }
        }
    }
    
    // 按时间统计
    GET /commerce/_search
    {
        "size": 0,
        "query": {
            "match": {
                "name": "iPhone"
            }
        },
        "aggs": {
            "xxx": {
                "date_histogram": {
                    "field": "timestamp",
                    "interval": "day", // `year` 或 `1y`: 1 年、`quarter` 或 `1q`: 1 季度、`month` 或 `1M`: 1 月份、`week` 或 `1w`: 1 星期、`day` 或 `1d`: 1 天、`hour` 或 `1h`: 1 小时、`minute` 或 `1m`: 1 分钟、`second` 或 `1s`: 1 秒，例如 `5m` 表示每 5 分钟，`day` 表示每天
                    "format": "yyyy-MM-dd", // yyyy-MM-dd HH:mm:ss.SSSZ
                    "time_zone": "+08:00"
                }
            }
        }
    }
    ```

- 查询结果高亮
    ```json
    GET /commerce/_search
    {
        "query": {
            "match": {
                "name": "iphone"
            }
        },
        "highlight": {
            // "pre_tags": [
            //     "<em class=\"c_color\">"
            // ],
            // "post_tags": [
            //     "</em>"
            // ],
            "fields": {
                "name": {}
            }
        }
    }
    ```

- 其它
    ```json
    // 多字段组合查询
    GET /commerce/_search
    {
        "query": {
            "bool": {
                "must": [ // must 表示 and，should 表示 or
                    {
                        "match": {
                            "description": "Camera"
                        }
                    }, {
                        "wildcard": {
                            "brand": "Real*"
                        }
                    }
                ]
            }
        },
        "sort": {
            "timestamp": {
                "order": "desc"
            }
        },
        "from": 0,
        "size": 10
    }

    // 根据品牌去重并展示每个品牌的一条记录
    GET /commerce/_search
    {
        "query": {
            "match_all": { }
        },
        "collapse": {
            "field": "brand", // 要进行折叠的字段
            "inner_hits": { // 折叠的参数集
                "name": "test", // 自定义 hits 的名称
                "ignore_unmapped": true, // 默认为 false，如果存在一些数据没有折叠字段的会报错，设置为 true 可以避免类似的报错
                "from": 0,
                "size": 0, // from 和 size 用来控制想要返回的折叠列表，这里我的需求是重复 brand 相同仅返回头条，所以两个参数均设置为 0，如果有需求折叠列表的可以通过这里控制
                "version": false,
                "explain": false,
                "track_scores": true,
                "sort": [{ // 折叠列表的排序，折叠列表中要把谁显示在第一个的排序，比如这样做是将该折叠列表的数据按字段 price 倒序排列
                    "price": {
                        "order": "desc"
                    }
                }]
            }
        }
    }

    // 聚合查询所有品牌
    GET /commerce/_search
    {
        "size": 0,
        "aggs": {
            "brands": { // 自定义组名为 brands
                "terms": {
                    "field": "brand"
                }
            }
        }
    }

    // 聚合去重，展示每个品牌下最高价格的 1 条记录
    GET /commerce/_search
    {
        "size": 0,
        "aggs": {
            "brands": {
                "terms": {
                    "field": "brand"
                },
                "aggs": {
                    "product": {
                        "top_hits": {
                            "sort": [
                                {
                                    "price": {
                                        "order": "desc"
                                    }
                                }
                            ],
                            "size": 1 // 每个品牌下展示 1 条商品记录
                        }
                    }
                }
            }
        }
    }

    // 统计每种品牌的平均价格
    GET /commerce/_search
    {
        "size": 0,
        "aggs": {
            "popular_brand": {
                "terms": {
                    "field": "brand"
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

    // 按照日期聚合分组，求出每个月个数
    // GET /commerce/_search
    // {
    //     "size": 0,
    //     "aggs": {
    //         "date_sales": {
    //             "date_histogram": {
    //                 "field": "timestamp",
    //                 "calendar_interval": "month",
    //                 "format": "yyyy-MM-dd",
    //                 "min_doc_count": 0,
    //                 "extended_bounds": {
    //                     "min": "2019-01-01",
    //                     "max": "2019-12-31"
    //                 }
    //             }
    //         }
    //     }
    // }

    // 统计每个季度每个品牌的销售额，及每个季度销售总额
    // GET /commerce/_search
    // {
    //     "size": 0,
    //     "aggs": {
    //         "date_sales": {
    //             "date_histogram": {
    //                 "field": "timestamp",
    //                 "calendar_interval": "quarter",
    //                 "format": "yyyy-MM-dd",
    //                 "min_doc_count": 0,
    //                 "extended_bounds": {
    //                     "min": "2019-01-01",
    //                     "max": "2020-12-31"
    //                 }
    //             },
    //             "aggs": {
    //                 "group_by_brand": {
    //                     "terms": {
    //                         "field": "brand"
    //                     },
    //                     "aggs": {
    //                         "sum_price": {
    //                             "sum": {
    //                                 "field": "price"
    //                             }
    //                         }
    //                     }
    //                 },
    //                 "total_sum_price": {
    //                     "sum": {
    //                         "field": "price"
    //                     }
    //                 }
    //             }
    //         }
    //     }
    // }
    ```

### 其它

<details>
<summary>深度分页</summary>

- 使用 skip + size 深度分页时（大于阈值 max_result_window，默认为 10000）导致查询失败，可以配置修改 max_result_window 阈值
    ```bash
    curl -XPUT "http://127.0.0.1:9200/commerce/_settings" -d '{
        "index": {
            "max_result_window": 50000
        }
    }'
    ```

- 深度分页 scroll（游标查询，无法实现实时查询）  
    如果我们分页要请求大数据集或者一次请求要获取较大的数据集，scroll 都是一个非常好的解决方案。
    使用 scroll 滚动搜索，可以先搜索一批数据，然后下次再搜索一批数据，以此类推，直到搜索出全部的数据来 scroll 搜索会在第一次搜索的时候，保存一个当时的视图快照，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的。每次发送 scroll 请求，我们还需要指定一个 scroll 参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了。
    ```json
    GET /commerce/offering/_search?scroll=5m # scroll=5m 表示该窗口过期时间为 5 分钟
    {
        "query": {
            "match_all": {}
        },
        "size": 2
    }
    # 返回 _scroll_id，如 DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAC0YFmllUjV1QTIyU25XMHBTck1XNHpFWUEAAAAAAAAtGRZpZVI1dUEyMlNuVzBwU3JNVzR6RVlBAAAAAAAALRsWaWVSNXVBMjJTblcwcFNyTVc0ekVZQQAAAAAAAC0aFmllUjV1QTIyU25XMHBTck1XNHpFWUEAAAAAAAAtHBZpZVI1dUEyMlNuVzBwU3JNVzR6RVlB
    ```
    ```json
    GET /_search/scroll
    {
        "scroll": "5m",
        "scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAC0YFmllUjV1QTIyU25XMHBTck1XNHpFWUEAAAAAAAAtGRZpZVI1dUEyMlNuVzBwU3JNVzR6RVlBAAAAAAAALRsWaWVSNXVBMjJTblcwcFNyTVc0ekVZQQAAAAAAAC0aFmllUjV1QTIyU25XMHBTck1XNHpFWUEAAAAAAAAtHBZpZVI1dUEyMlNuVzBwU3JNVzR6RVlB"
    }
    ```

- 深度分页 search_after（假分页）  
    search_after 是一种假分页方式，根据上一页的最后一条数据来确定下一页的位置，同时在分页请求的过程中，如果有索引数据的增删改查，这些变更也会实时的反映到游标上。为了找到每一页最后一条数据，每个文档必须有一个全局唯一值，官方推荐使用 _uid 作为全局唯一值，但是只要能表示其唯一性就可以。
    为了演示，我们需要给上文中的 commerce 索引增加一个 uid 字段表示其唯一性。
    ```json
    GET /commerce/offering/_search
    {
        "query": {
            "match_all": {}
        },
        "size": 2,
        "sort": [
            {
                "uid": "desc"
            }
        ]
    }
    ```
    ```json
    GET /commerce/offering/_search
    {
        "query": {
            "match_all": {}
        },
        "size": 2,
        "search_after": [1005], # 下一次分页，需要将上述分页结果集的最后一条数据的值带上。
        "sort": [
            {
                "uid": "desc"
            }
        ]
    }
    ```

</details>

<details>
<summary>词权重</summary>

> [查询时权重提升](https://www.elastic.co/guide/cn/elasticsearch/guide/current/query-time-boosting.html)

```json
GET /commerce/_search
{
    "query": {
        "bool": {
            "should": [
                {
                    "term": {
                        "brand": {
                            "value": "Apple",
                            "boost": 4
                        }
                    }
                },
                {
                    "match": {
                        "description": {
                            "query": "iphone",
                            "boost": 2 // description 查询 iphone 语句的重要性是 name 查询 huawei、vivo 的 2 倍，因为它的权重提升值为 2 。默认没有设置 boost 的查询语句的值为 1
                        }
                    }
                },
                {
                    "match": {
                        "name": "huawei vivo"
                    }
                }
            ]
        }
    },
    "explain": true, // 设置 explain: true，可返回评分计算过程
    "sort": { // 当有多个排序字段时，按字段出现顺序为优先级进行排序
        "_score": { // 首先根据得分排序
            "order": "desc" // 按照评分降序排序
        },
        "price": { // 然后根据价格排序
            "order": "asc" // 升序排序
        }
    }
}
```

> - _score 算法，可通过查询 DSL 中设置 explain: true 查看评分计算过程
>     - BM25（ES≥5.0（即 Lucene≥6.0）版本默认评分算法）
>     - TF/IDF（≥7.0 版本已废弃）

<blockquote>

<details>
<summary>评分计算 explanation</summary>

> https://www.elastic.co/guide/en/elasticsearch/reference/current/search-explain.html

> https://www.cnblogs.com/wangjiuyong/articles/7055724.html

```json
{
    "took": 69,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 4, // 查找到的文档共有 4 个
        "max_score": null,
        "hits": [
            {
                "_shard": "[commerce][1]",
                "_node": "_NK2n9X4SoCW6PKoRBuF2w",
                "_index": "commerce",
                "_type": "offering",
                "_id": "10023605369768",
                "_score": 2.3536685,
                "_source": {
                    "name": "Apple iPhone 12",
                    "description": "Apple iPhone 12 All China Netcom 5g mobile phone black all China Netcom 128G",
                    ...
                },
                "sort": [
                    2.3536685,
                    913.76
                ],
                "_explanation": {
                    "value": 2.3536685,
                    "description": "sum of:",
                    "details": [
                        {
                            "value": 2.3536685,
                            "description": "weight(description:iphon in 3) [PerFieldSimilarity], result of:", // 在文档（内部 id 为 3）中搜索字段 description 包含关键词 iphon 的权重评分结果如下
                            "details": [
                                {
                                    "value": 2.3536685,
                                    "description": "score(doc=3,freq=1.0 = termFreq=1.0\n), product of:",
                                    "details": [
                                        {
                                            "value": 2.0,
                                            "description": "boost",
                                            "details": []
                                        },
                                        {
                                            "value": 1.0296195,
                                            "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
                                            "details": [
                                                {
                                                    "value": 2.0,
                                                    "description": "docFreq", // 满足当前查询条件（description 中包含搜索词 iphon ）的文档个数
                                                    "details": []
                                                },
                                                {
                                                    "value": 6.0,
                                                    "description": "docCount", // 数据对应的分片下的文档总个数
                                                    "details": []
                                                }
                                            ]
                                        },
                                        {
                                            "value": 1.1429797,
                                            "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
                                            "details": [
                                                {
                                                    "value": 1.0,
                                                    "description": "termFreq=1.0", // 搜索词 iphon 在字段 description 中出现的次数
                                                    "details": []
                                                },
                                                {
                                                    "value": 1.2,
                                                    "description": "parameter k1",
                                                    "details": []
                                                },
                                                {
                                                    "value": 0.75,
                                                    "description": "parameter b",
                                                    "details": []
                                                },
                                                {
                                                    "value": 20.166666,
                                                    "description": "avgFieldLength", // 当前数据所在分片下，所有文档的字段 description 分词并且去除停用词部分的 terms 总个数，除以文档总数
                                                    "details": []
                                                },
                                                {
                                                    "value": 14.0, // 这里可能不是整数，是因为：lucene 为了降低存储的空间，实现了区间映射功能，即在存储字段的长度时，没有存储实际长度，而是存储了一个 byte 类型的值（0-255），每个值对应了 BM25Similarity 中 NORM_TABLE 数组的下标 index
                                                    "description": "fieldLength", // 满足查询条件的文档的字段 description 的长度（分词并去除停用词部分 terms 个数）
                                                    "details": []
                                                }
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    ]
                }
            },
            ...
        ]
    }
}
```

BM25Similarity（NORM_TABLE）根据真实长度计算 fieldLength 值算法参考如下（ElasticSearch 5.3 版本）

```javascript
var SmallFloat = {
    byte315ToFloat: function (b) {
        if (b == 0) return 0.0;
        let bits = (b & 0xff) << (24 - 3);
        bits += (63 - 15) << 24;
        return this.intBitsToFloat(bits);
    },
    floatToByte315: function (f) {
        let bits = this.floatToRawIntBits(f);
        let smallfloat = bits >> (24 - 3);
        if (smallfloat <= ((63 - 15) << 3)) {
            return (bits <= 0) ? 0 : 1;
        }
        if (smallfloat >= ((63 - 15) << 3) + 0x100) {
            return -1;
        }
        return (smallfloat - ((63 - 15) << 3));
    },
    intBitsToFloat: function (b) {
        let buf = new ArrayBuffer(4);
        (new Uint32Array(buf))[0] = b;
        return (new Float32Array(buf))[0];
    },
    floatToRawIntBits: function (f) {
        let buf = new ArrayBuffer(4);
        (new Float32Array(buf))[0] = f;
        return (new Uint32Array(buf))[0];
    }
}
var BM25Similarity = {
    getNormTable: function () {
        let NORM_TABLE = new Array(256);
        for (let i = 1; i < 256; i++) {
            let f = SmallFloat.byte315ToFloat(i);
            NORM_TABLE[i] = 1.0 / (f * f);
        }
        NORM_TABLE[0] = 1.0 / NORM_TABLE[255]; // otherwise inf
        return NORM_TABLE;
    },
    getFieldLength: function (realFieldLength) {
        let idx = SmallFloat.floatToByte315(Math.sqrt(1.0 / realFieldLength)); // the index of NORM_TABLE in BM25Similarity
        if (idx == 0) {
            idx = 255;
        }
        let f = SmallFloat.byte315ToFloat(idx);
        return 1.0 / (f * f);
    },
    getMightRealFieldLength: function (fieldLength, max = 100) {
        let idx = this.getNormTable().findIndex((v, i) => (v + "").startsWith(fieldLength + ""));
        if (idx == 255) {
            idx = 0;
        }
        let o = [];
        for (let i = 1; i < max; i++) {
            if (SmallFloat.floatToByte315(Math.sqrt(1.0 / i)) == idx) {
                o.push(i);
            }
        }
        return o;
    }
}
BM25Similarity.getFieldLength(5); // 5.224489795918367
BM25Similarity.getMightRealFieldLength(5.2244897); // [5]
```

</details>

<details>
<summary>设置评分算法</summary>

> https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-similarity.html

```json
PUT /commerce
{
    "settings": {
        "index": {
            "similarity": {
                "my_similarity": {
                    "type": "BM25",
                    "k1": 1.2, // （默认为 1.2）
                    "b": 0 // （默认为 0.7）设置 b 为 0，则评分不受词频影响
                }
            }
        }
    },
    "mappings": {
        "offering": {
            "properties": {
                "description": {
                    "type": "text",
                    "similarity": "my_similarity"
                }
            }
        }
    }
}
```

</details>

<details>
<summary>使用 function_score 自定义计算评分</summary>

> https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#query-dsl-function-score-query

```json
GET /commerce/_search
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5", 
            "functions": [
                {
                    "filter": { "match": { "test": "bar" } },
                    "random_score": {}, 
                    "weight": 23
                },
                {
                    "filter": { "match": { "test": "cat" } },
                    "weight": 42
                }
            ],
            "max_boost": 42,
            "score_mode": "max",
            "boost_mode": "multiply",
            "min_score": 42
        }
    }
}
```

</details>

<details>
<summary>使用 script_score 自定义计算评分</summary>

> https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html

</details>

</blockquote>

</details>

<details>
<summary>联想词/建议器</summary>

- term 建议器（基于 analyze 分析过后的单个 term 去提供建议）
    ```json
    GET /commerce/_search
    {
        "suggest": {
            "my_suggestion": { // 自定义搜索建议，可以为多个
                "text": "huaw real",
                "term": {
                    "suggest_mode": "missing", // 可选值 missing（默认）、popular、always，其中 missing 表示如果 rock 在索引的字典中已存在，则不返回
                    "field": "name" // 联想字段需要支持分词，如 text 类型
                    // "max_edit": 2 // 可选参数，可选值为 1、2（默认），表示 text 中的词与索引字典中值的编辑距离，小于或等于这个值才会被建议返回
                }
            }
        }
    }
    ```

- phrase 建议器（在 term 建议器基础上，考量多个 term 是否同时存在、相邻程度、词频等）
    ```json
    // GET commerce/_search
    // {
    //     "suggest": {
    //         "text": "jeva null point exception",
    //         "simple_phrase": {
    //             "phrase": {
    //                 "field": "title",
    //                 "size": 3,
    //                 "direct_generator": [
    //                     {
    //                         "field": "title",
    //                         "suggest_mode": "always",
    //                         "min_word_length": 4
    //                     }
    //                 ],
    //                 "collate": {
    //                     "query": {
    //                         "source": {
    //                             "match": {
    //                                 "{{field_name}}": "{{suggestion}}"
    //                             }
    //                         }
    //                     },
    //                     "params": {
    //                         "field_name": "title"
    //                     },
    //                     "prune": true
    //                 }
    //             }
    //         }
    //     }
    // }
    ```

- completion 建议器
    1. 配置字段类型为 completion 类型
        ```json
        PUT /commerce
        {
            "mappings": {
                "offering": {
                    "properties": {
                        "name": {
                            "type": "text",
                            "analyzer": "english",
                            "fields": {
                                "suggest": {
                                    "type": "completion",
                                    "analyzer": "english"
                                }
                            }
                        }
                    }
                }
            }
        }
        ```
    2. 查询
        ```json
        GET /commerce/_search
        {
            "suggest": {
                "my_suggestion": {
                    "prefix": "hua",
                    "completion": {
                        "field": "name.suggest",
                        "fuzzy": { // 可选，表示开启模糊匹配
                            // "fuzziness": "AUTO" // 默认为 AUTO
                        }
                    }
                }
            }
        }
        ```

- context 建议器

</details>

<details>
<summary>同义词</summary>

1. 定义一个同义词分析器
    ```json
    PUT /commerce
    {
        "settings": {
            "analysis": {
                "filter": {
                    "my_synonym_filter": { // 自定义了一个语汇单元过滤器
                        "type": "synonym",  // 指定过滤器使用同义词类型
                        "synonyms": [ // 定义同义词。同义词不具备传递性，同一组同义词不应拆分为多行(组)写
                            "ipod, i-pod, i pod => ipod", // 单向同义词: 索引或查询时，箭头左侧的词将会被映射成箭头右侧的词
                            "马铃薯, 土豆, potato" // 双向同义词: 索引时会同时建立同义词的倒排索引，查询时会同时对同义词的倒排索引匹配
                        ]
                    }
                },
                "analyzer": {
                    "my_synonyms": { // 自定义了一个使用 my_synonym_filter 过滤器的自定义分析器
                        "tokenizer": "standard",
                        "filter": [
                            "lowercase",
                            "my_synonym_filter"
                        ]
                    }
                }
            }
        }
    }
    ```
2. 测试使用同义词分析器
    ```json
    GET /_analyze
    {
        "analyzer": "my_synonyms",
        "text": "Elizabeth is the English queen"
    }
    ```

</details>

<details>
<summary>停用词</summary>

</details>

<details>
<summary>分词器</summary>

- 使用标准分析器分析文本
    ```bash
    curl -XGET http://127.0.0.1:9200/_analyze?pretty -H 'Content-Type: application/json' -d '{
        "analyzer": "standard",
        "text": "this is a text"
    }'
    ```

- 配置、使用 english 分析器（索引时分词、查询时分词）
    ```json
    POST /commerce
    {
        "mappings": {
            "offering": {
                "properties": {
                    "content": {
                        "type": "text",
                        "analyzer": "english", // 索引时分词
                        "search_analyzer": "english" // 查询时分词
                    }
                }
            }
        }
    }
    ```
    ```json
    POST /commerce/_search
    {
        "query": {
            "match": {
                "content": {
                    "query": "hello world",
                    "analyzer": "english", // 查询时使用指定分词器分词
                }
            }
        }
    }
    ```

> 分词流程：*text* => char_filter(Character Filter/字符过滤器) => tokenizer(Tokenizer/分词器) => *token(词元)* => filter(Token Filter/分词过滤器) => *term(词)*

- <span id="create_custom_analyzer">创建自定义分析器</span>
    > analyzer 分析器包含零或多个 character filters，一个 tokenizer，零或多个 token filters
    ```json
    PUT /commerce
    {
        "settings": {
            "analysis": {
                "char_filter": {
                    "my_char_filter": {
                        "type": "mapping",
                        "mappings": ["& => and"]
                    }
                },
                "tokenizer": {
                    "my_tokenizer": {
                        "type": "nGram", // N-gram 模型，参考 https://blog.csdn.net/songbinxu/article/details/80209197
                        "min_gram": "2",
                        "max_gram": "3",
                        "token_chars": ["letter", "digit"]
                    }
                },
                "filter": {
                    "my_token_filter": {
                        "type": "stop",
                        "stopwords": ["the", "a"]
                    }
                },
                "analyzer": {
                    "my_analyzer": {
                        "type": "custom",
                        "char_filter": ["html_strip", "my_char_filter"],
                        "tokenizer": "my_tokenizer",
                        "filter": ["lowercase", "my_token_filter"]
                    }
                }
            }
        }
    }
    ```

- 分析文本
    ```json
    GET /_analyze
    {
        "analyzer": "my_analyzer",
        "text": "oppo & apple",
        "explain": true
    }
    ```
    ```json
    GET /_analyze
    {
        "char_filter": ["my_char_filter"],
        "tokenizer": "my_tokenizer",
        "filter": [
            "lowercase",
            "my_token_filter",
            {
                "type": "stemmer",
                "name": "english"
            }
        ],
        "text": "oppo & apple",
        "explain": true
    }
    ```

<blockquote>

<details open>
<summary><a href="https://www.elastic.co/guide/en/elasticsearch/reference/6.2/analysis-analyzers.html">ES 内置的分析器 analyzer</a></summary>

- standard  
    未设置分析器时默认使用此分析器。在空格、符号处切，中文部分切割为一个一个的汉字。由 standard tokenizer, standard filter, lower case filter, stop filter 组成。
- simple  
    在空格、符号、数字处切割，中文部分不会切割为一个一个的汉字。由 lower case tokenizer 组成。
- stop  
    在空格、符号、数字、英文介词和冠词处切割，中文部分不会切割为一个一个的汉字。由 lower case tokenizer, stop filter 组成。
- keyword  
    不分词，内容整体作为一个 token。
- whitespace  
    只在空格处切割。
- [lang](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html)  
    语言分析器有很多种，把语言全小写就是，比如 english、chinese。english、chinese 的效果都一样：在空格、符号、英文介词和冠词处切割，中文切割为一个一个的汉字。
- pattern  
    根据正则表达式来切割，默认使用的正则表达式是 \W+，在匹配 \W+ 的地方切割。\w 包括英文字母、阿拉伯数字和 _，\W 是任意一个非 \w 字符，中文字符也算 \W。
- snowball  
    由 standard tokenizer, standard filter, lower case filter, stop filter, snowball filter 组成。
- custom  
    自定义分词器，参考[创建自定义分词器](#create_custom_analyzer)。由零或多个 char_filter，一个 tokenizer, 零或多个 filter 组成。

<blockquote>

<details>
<summary><a href="https://www.elastic.co/guide/en/elasticsearch/reference/6.2/analysis-charfilters.html">ES 内置的字符串过滤器 char_filter</a></summary>

- mapping  
    根据配置的映射关系替换字符。
- html_strip  
    去掉 HTML 元素。
- pattern_replace  
    用正则表达式处理字符串
    ```json
    // 驼峰分词
    {
        "type": "pattern_replace",
        "pattern": "(?<=\\p{Lower})(?=\\p{Upper})",
        "replacement": " "
    }
    ```
    ```json
    // 特殊符号分词
    {
        "type": "pattern_replace",
        "pattern": "(?:\\p{Punct})",
        "replacement": " "
    }
    ```

</details>

<details>
<summary><a href="https://www.elastic.co/guide/en/elasticsearch/reference/6.2/analysis-tokenizers.html">ES 内置的分词器 tokenizer</a></summary>

- standard
- edgeNGram
- keyword  
    不分词
- letter  
    按单词分
- lowercase  
    letter tokenizer, lower case filter
- nGram
- whitespace  
    以空格为分隔符拆分
- pattern  
    定义分隔符的正则表达式
- uax_url_email  
    不拆分 url 和 email
- path_hierarchy  
    处理类似 `/path/to/somthing` 样式的字符串

</details>

<details>
<summary><a href="https://www.elastic.co/guide/en/elasticsearch/reference/6.2/analysis-tokenfilters.html">ES 内置的分词过滤器 filter</a></summary>

- standard
- asciifolding
- length  
    去掉太长或者太短的
- lowercase  
    转成小写
- nGram
- edgeNGram
- porterStem  
    波特词干算法
- shingle  
    定义分隔符的正则表达式
- stop  
    停用词，从 tokens 中删除停用词
    ```json
    {
        "type": "stop",
        "stopwords": [ // 停用词列表。可选，字符串或字符数组类型。默认为 "_english_"。如果值为 "_none_" 则表示停用词为空。
            "_english_", // english 停用词列表
            "and", "is", "the" // 自定义停用词
        ],
        "stopwords_path": "stopwords.txt" // 停用词词库路径。可选，字符串类型。词库文件必须为 UTF-8 编码。文件路径相对于 $ES_HOME/config 目录。修改词库文件后需要关闭和重新打开索引以更新停用词。
    }
    ```
- word_delimiter  
    将一个单词再拆成子分词，如 WiFi => Wi, Fi
    ```json
    {
        "type": "word_delimiter"
    }
    ```
- stemmer
    ```json
    {
        "type": "stemmer",
        "name": "english"
    }
    ```
- stemmer_override
- keyword_marker
- keyword_repeat
- kstem
- snowball
- phonetic  
    [插件](https://github.com/elasticsearch/elasticsearch-analysis-phonetic)
- synonyms  
    处理同义词
    ```json
    {
        "type": "synonym",
        "synonyms": [
            "ipod, i-pod, i pod => ipod",
            "马铃薯, 土豆, potato"
        ]
        // "synonyms_path": "synonyms.txt"
    }
    ```
- dictionary_decompounder, hyphenation_decompounder  
    分解复合词
    ```json
    {
        "type": "dictionary_decompounder",
        "word_list": [
            "wi" // 如 `wifi` 分词为 `wifi`, `wi`
        ],
        // "word_list_path": "words.txt",
        "min_word_size": 2 // 最小单词长度，默认为 5。这里需要小于等于 2，否则 wifi 中的 wi 不能被拆分出来
    }
    ```
- reverse  
    反转字符串
- elision  
    去掉缩略语
- truncate  
    截断字符串
- unique  
- pattern_capture
- pattern_replace  
    用正则表达式替换
- trim  
    去掉空格
- limit  
    限制token数量
- hunspell  
    拼写检查
- common_grams
- arabic_normalization, persian_normalization

</details>

</blockquote>

</details>

</blockquote>

- Payload
    - 搜索关键词 "To be, or not to be" 会被停用词全部忽略，从而导致无法搜到正确结果  
        中文意思是“生存还是毁灭”，出自莎士比亚的名言
    - 搜索关键词 "create" 或 "created" 会被分词为 "creat"  
        使用 Porter stemmer 词干算法，会将 "create" 还原为 "creat"，但不会影响分词索引结果
    - 搜索关键词 "西门子"(中文)、"ximenzi"(拼音)、"siemens"(英文)、"xmz"(拼音简写)、"西闷子"(中文纠错)、"ximenzhi"(拼音纠错)、"西"(中文前缀) 都能匹配或联想到“西门子”相关记录
    - 搜索关键词 "汽车改装鲨鱼鳍"，ik 分词器中的 ik_smart analyzer 会将其分词为“汽车”、“改装”、“鲨”、“鱼鳍”，在自定义词库 ext_dict 中写入“鲨鱼鳍”后，优化分词结果为“汽车”、“改装”、“鲨鱼鳍”
    - 搜索关键词 "海洛因" 会阻断查询并提示该词汇为敏感词汇
    - 分词与屏蔽敏感词汇？"科技处女干事每月经过下属都要亲口交代24口交换机等技术性器件的安装工作"

</details>

<details>
<summary>数据预处理（Pipeline）</summary>

1. 创建一个 Pipeline 并命名为 my_timestamp_pipeline，用于自动生成时间戳
    ```json
    PUT /_ingest/pipeline/my_timestamp_pipeline
    {
        "description": "Adds a field to a document with the time of ingestion",
        "processors": [
            {
                "set": {
                    "field": "timestamp",
                    "value": "{{_ingest.timestamp}}" // 该时间是 UTC+0 时间，晚于国内 8 小时
                }
            }
        ]
    }
    ```
2. 新增数据时使用 my_timestamp_pipeline
    ```json
    PUT /commerce/offering/1?pipeline=my_timestamp_pipeline
    {
        "name": "Apple iPhone 8",
        "price": 823.00
    }
    ```

</details>

<details>
<summary>配置使用 ik 分词器</summary>

> 参考 https://github.com/medcl/elasticsearch-analysis-ik

1. 安装
    1. 下载插件
        ```bash
        curl -O -L https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.4/elasticsearch-analysis-ik-6.2.4.zip
        ```
    2. 解压至 $ES_HOME/plugins/ik 目录下
        ```bash
        unzip -d elasticsearch-6.2.4/plugins elasticsearch-analysis-ik-6.2.4.zip && mv elasticsearch-6.2.4/plugins/elasticsearch elasticsearch-6.2.4/plugins/ik
        ```
    3. 重启 elasticsearch
2. 测试
    ```bash
    curl -XGET http://127.0.0.1:9200/_analyze?pretty -H 'Content-Type: application/json' -d '{
        "analyzer": "ik_max_word",
        "text": "\u4e2d\u534e\u4eba\u6c11\u5171\u548c\u56fd\u56fd\u52a1\u9662\uff0c\u5373\u4e2d\u592e\u4eba\u6c11\u653f\u5e9c\uff0c\u662f\u6700\u9ad8\u56fd\u5bb6\u6743\u529b\u673a\u5173\u7684\u6267\u884c\u673a\u5173\uff0c\u662f\u6700\u9ad8\u56fd\u5bb6\u884c\u653f\u673a\u5173\u3002" # 中华人民共和国国务院，即中央人民政府，是最高国家权力机关的执行机关，是最高国家行政机关。
    }'
    ```
    > ik 支持以下 analyzer 和 tokenizer
    > - analyzer: `ik_smart`（粗颗粒度拆分，适合 Phrase 查询）, `ik_max_word`（细颗粒度拆分，适合 Term 查询）
    > - tokenizer: `ik_smart`, `ik_max_word`
3. 配置词库  
   编辑 ${ES_HOME}/plugins/ik/config/IKAnalyzer.cfg.xml，配置内容如下
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
   <properties>
       <comment>IK Analyzer 扩展配置</comment>

       <!-- 本地词库 -->
       <entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry><!-- 用户可以在这里配置自己的扩展字典 -->
       <entry key="ext_stopwords">custom/ext_stopword.dic</entry><!-- 用户可以在这里配置自己的扩展停止词字典-->

       <!-- 远程词库（热更新 IK 分词） -->
       <!--
           其中 location 是指一个 url，如 http://yoursite.com/getCustomDict，该请求只需满足以下两点即可完成分词热更新：
               - 该 http 请求需要返回两个头部(header)，一个是 `Last-Modified`，一个是 `ETag`，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库。
               - 该 http 请求返回的内容格式是一行一个分词，换行符用 `\n` 即可。
           满足上面两点要求就可以实现热更新分词了，不需要重启 ES 实例。
           P.S. 可以将需自动更新的热词放在一个 UTF-8 编码的 .txt 文件里，放在 nginx 或其他简易 http server 下，当 .txt 文件修改时，http server 会在客户端请求该文件时自动返回相应的 Last-Modified 和 ETag。可以另外做一个工具来从业务系统提取相关词汇，并更新这个 .txt 文件
       -->
       <entry key="remote_ext_dict">location</entry><!-- 用户可以在这里配置远程扩展字典 -->
       <entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry><!-- 用户可以在这里配置远程扩展停止词字典 -->
   </properties>
   ```
4. 使用
    1. 创建索引
        ```bash
        curl -XPUT http://127.0.0.1:9200/ik_test_idx
        ```
    2. 创建 mapping
        ```bash
        curl -XPOST http://127.0.0.1:9200/ik_test_idx/fulltext/_mapping -H 'Content-Type: application/json' -d '{
            "properties": {
                "content": {
                    "type": "text",
                    "analyzer": "ik_max_word", # 索引时分词
                    "search_analyzer": "ik_smart" # 查询时分词
                }
            }
        }'
        ```
    3. 创建记录
        ```json
        POST /ik_test_idx/fulltext/1
        {
            "content": "中华人民共和国国务院，即中央人民政府，是最高国家权力机关的执行机关，是最高国家行政机关。"
        }
        ```
    4. 高亮查询
        ```json
        POST /ik_test_idx/fulltext/_search
        {
            "query": {
                "match": {
                    "content": "国家"
                }
            },
            "highlight": {
                // "pre_tags": ["<tag1>", "<tag2>"],
                // "post_tags": ["</tag1>", "</tag2>"],
                "fields": {
                    "content": {}
                }
            }
        }
        ```
    5. 联想词
        1. 创建基于 ik 插件的自动补全 mapping
            ```bash
            curl -XPOST http://127.0.0.1:9200/ik_test_idx/fulltext/_mapping -d'
            {
                "suggest": {
                    "properties": {
                        "content": {
                            "type": "text",
                            "analyzer": "ik_max_word",
                            "search_analyzer": "ik_smart",
                            "fields": {
                                "suggest": {
                                    "type": "completion",
                                    "analyzer": "ik_max_word",
                                    "search_analyzer": "ik_smart",
                                    "payloads": false
                                }
                            }
                        }
                    }
                }
            }'
            ```
        2. 通过 _suggest 对 Completion Suggester 数据进行搜索
            ```bash
            curl -XPOST "http://127.0.0.1:9200/ik_test_idx/_suggest" -d'
            {
                "my-suggest": {
                    "text": "中华",
                    "completion": {
                        "field": "content.suggest",
                        "size": 10 // 返回结果数量
                    }
                }
            }'
            ```

</details>

<details>
<summary>索引模板</summary>

1. 创建索引模板，新建的索引名称需要匹配 "goods*" 才可以使用该模板
    ```json
    PUT _template/goods
    {
        "index_patterns": "goods*",
        "settings": {
            "index.number_of_replicas": "1",
            "index.number_of_shards": "5",
            "index.translog.flush_threshold_size": "512mb",
            "index.translog.sync_interval": "60s",
            "index.codec": "best_compression",
            "analysis": {
                "filter": {
                    "edge_ngram_filter": {
                        "type": "edge_ngram",
                        "min_gram": 1,
                        "max_gram": 50
                    },
                    "simple_pinyin_filter": {
                        "type": "pinyin",
                        "keep_first_letter": true,
                        "keep_separate_first_letter": false,
                        "keep_full_pinyin": false,
                        "keep_original": false,
                        "limit_first_letter_length": 50,
                        "lowercase": true
                    },
                    "full_pinyin_filter": {
                        "type": "pinyin",
                        "keep_first_letter": false,
                        "keep_separate_first_letter": false,
                        "keep_full_pinyin": true,
                        "none_chinese_pinyin_tokenize": true,
                        "keep_original": false,
                        "limit_first_letter_length": 50,
                        "lowercase": true
                    }
                },
                "char_filter": {
                    "charconvert": {
                        "type": "mapping",
                        "mappings_path": "char_filter_text.txt"
                    }
                },
                "tokenizer": {
                    "ik_max_word": {
                        "type": "ik_max_word",
                        "use_smart": true
                    }
                },
                "analyzer": {
                    "ngramIndexAnalyzer": {
                        "type": "custom",
                        "tokenizer": "keyword",
                        "filter": [
                            "edge_ngram_filter",
                            "lowercase"
                        ],
                        "char_filter": [
                            "charconvert"
                        ]
                    },
                    "ngramSearchAnalyzer": {
                        "type": "custom",
                        "tokenizer": "keyword",
                        "filter": [
                            "lowercase"
                        ],
                        "char_filter": [
                            "charconvert"
                        ]
                    },
                    "ikIndexAnalyzer": {
                        "type": "custom",
                        "tokenizer": "ik_max_word",
                        "char_filter": [
                            "charconvert"
                        ]
                    },
                    "ikSearchAnalyzer": {
                        "type": "custom",
                        "tokenizer": "ik_max_word",
                        "char_filter": [
                            "charconvert"
                        ]
                    },
                    "simplePinyinIndexAnalyzer": {
                        "tokenizer": "keyword",
                        "filter": [
                            "simple_pinyin_filter",
                            "edge_ngram_filter",
                            "lowercase"
                        ]
                    },
                    "simplePinyinSearchAnalyzer": {
                        "tokenizer": "keyword",
                        "filter": [
                            "simple_pinyin_filter",
                            "lowercase"
                        ]
                    },
                    "fullPinyinIndexAnalyzer": {
                        "tokenizer": "keyword",
                        "filter": [
                            "full_pinyin_filter",
                            "edge_ngram_filter",
                            "lowercase"
                        ]
                    },
                    "fullPinyinSearchAnalyzer": {
                        "tokenizer": "keyword",
                        "filter": [
                            "full_pinyin_filter",
                            "lowercase"
                        ]
                    }
                }
            }
        }
    }
    ```

2. 在 $ES_HOME/config 目录下新建文件 char_filter_text.txt

3. 基于模板创建索引
    ```json
    PUT /goods_01
    {
        "mappings": {
            "doc": {
                "properties": {
                    "id": {
                        "type": "long"
                    },
                    "name": {
                        "type": "text",
                        "analyzer": "ikIndexAnalyzer",
                        "fields": {
                            "ngram": {
                                "type": "text",
                                "analyzer": "ngramIndexAnalyzer"
                            },
                            "SPY": {
                                "type": "text",
                                "analyzer": "simplePinyinIndexAnalyzer"
                            },
                            "FPY": {
                                "type": "text",
                                "analyzer": "fullPinyinIndexAnalyzer"
                            }
                        }
                    },
                    "update_time": {
                        "type": "date"
                    },
                    "deleted": {
                        "type": "boolean"
                    }
                }
            }
        }
    }
    ```

4. 查询
    ```json
    GET /goods_01/_search
    {
        "query": {
            "bool": {
                "must": [
                    {
                        "dis_max": { // 取相似度 score 最大的返回
                            "tie_breaker": 0,
                            "queries": [
                                {
                                    "match": {
                                        "name.ngram": {
                                            "query": "水果",
                                            "operator": "OR",
                                            "analyzer": "ngramSearchAnalyzer",
                                            "prefix_length": 0,
                                            "max_expansions": 50,
                                            "fuzzy_transpositions": true,
                                            "lenient": false,
                                            "zero_terms_query": "NONE",
                                            "auto_generate_synonyms_phrase_query": true,
                                            "boost": 5
                                        }
                                    }
                                },
                                {
                                    "term": {
                                        "name.SPY": {
                                            "value": "水果",
                                            "boost": 1
                                        }
                                    }
                                },
                                {
                                    "wildcard": {
                                        "name.SPY": {
                                            "wildcard": "*水果*",
                                            "boost": 0.8
                                        }
                                    }
                                },
                                {
                                    "match_phrase": {
                                        "name.FPY": {
                                            "query": "水果",
                                            "analyzer": "fullPinyinSearchAnalyzer",
                                            "slop": 0,
                                            "zero_terms_query": "NONE",
                                            "boost": 1
                                        }
                                    }
                                },
                                {
                                    "match": {
                                        "name": {
                                            "query": "水果",
                                            "operator": "OR",
                                            "analyzer": "ikSearchAnalyzer",
                                            "prefix_length": 0,
                                            "max_expansions": 50,
                                            "minimum_should_match": "100%",
                                            "fuzzy_transpositions": true,
                                            "lenient": false,
                                            "zero_terms_query": "NONE",
                                            "auto_generate_synonyms_phrase_query": true,
                                            "boost": 1
                                        }
                                    }
                                }
                            ],
                            "boost": 1
                        }
                    }
                ],
                "filter": [
                    {
                        "term": {
                            "deleted": {
                                "value": false,
                                "boost": 1
                            }
                        }
                    }
                ],
                "adjust_pure_negative": true,
                "boost": 1
            }
        }
    }
    ```

</details>

<details>
<summary>经纬度查询</summary>

1. 创建索引
    ```json
    PUT /myindex
    {
        "mappings": {
            "properties": {
                "name": {
                    "type": "text"
                },
                "location": {
                    "type": "geo_point"
                }
            }
        }
    }
    ```
2. 新增数据
    ```json
    PUT /myindex/_doc/1
    {
        "name": "天安门",
        "location": {
            "lon": 116.403981,
            "lat": 39.914492
        }
    }

    PUT /myindex/_doc/2
    {
        "name": "海淀公园",
        "location": {
            "lon": 116.302509,
            "lat": 39.991152
        }
    }

    PUT /myindex/_doc/3
    {
        "name": "北京动物园",
        "location": {
            "lon": 116.343184,
            "lat": 39.947468
        }
    }
    ```
3. 查询
    ```json
    // 查找索引内距离北京站(116.433733,39.908404)3000米内的点
    POST /myindex/_search
    {
        "query": {
            "geo_distance": {
                "location": {
                    "lon": 116.433733,
                    "lat": 39.908404
                },
                "distance": 3000,
                "distance_type": "arc"
            }
        }
    }
    ```
    ```json
    // 查找索引内位于中央民族大学(116.326943,39.95499)以及京站(116.433733,39.908404)矩形的点
    POST /myindex/_search
    {
        "query": {
            "geo_bounding_box": {
                "location": {
                    "top_left": {
                        "lon": 116.326943,
                        "lat": 39.95499
                    },
                    "bottom_right": {
                        "lon": 116.433446,
                        "lat": 39.908737
                    }
                }
            }
        }
    }
    ```
    ```json
    // 查找索引内位于西苑桥(116.300209,40.003423)，巴沟山水园(116.29561,39.976004)以及北京科技大学(116.364528,39.996348)三角形内的点
    POST /myindex/_search
    {
        "query": {
            "geo_polygon": {
                "location": {
                    "points": [
                        {
                            "lon": 116.29561,
                            "lat": 39.976004
                        },
                        {
                            "lon": 116.364528,
                            "lat": 39.996348
                        },
                        {
                            "lon": 116.300209,
                            "lat": 40.003423
                        }
                    ]
                }
            }
        }
    }
    ```

</details>

## 名词解释

- 召回率/recall  
    比如你搜索一个 java spark，总共有 100 个 doc，能返回多少个 doc 作为结果，就是召回率

- 精准度/precision  
    比如你搜索一个 java spark，能不能尽可能让包含 java spark，或者是 java 和 spark 离的很近的 doc，排在最前面
