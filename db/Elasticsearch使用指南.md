## 分析器 Tokenizer

### 默认分析器
standard 分析器默认用于全文字段，对于大部分西方语系来说是一个不错的选择，但是对中文不太友好
* 通过单词边界分割输入的文本，按照空格和标点进行切割
* 中文会被分在一起
* 词过滤器：
   - lowercase 语汇单元过滤器，转换所有的语汇单元为小写
   - stop 语汇单元过滤器，删除停用词（对搜索相关性影响不大的常用词），如 a， the， and， is

### 自定义分析器
一个**分析器**就是在一个包里面组合了三种函数的一个包装器，三种函数按照顺序被执行，详见：

### IK 分词器
elasticsearch-analysis-ik 分词器插件，IK分词器是基于正向匹配的分词算法
- ik_smart
   - ik_smart会做最粗粒度的拆分，输出认为最合理的分词结果
   - 参考：https://blog.csdn.net/zhaojianting/article/details/75502457
- ik_max_word 
   - 会将文本做最全粒度的拆分，将能够分出来的词进行排列组合全部输出

## 查询语句构建

### QueryString 关键词查询权重
* 查询query中，使用引号包裹关键词：`<keyword>`，是进行精确词匹配的前提。
* 通过`<keyword>`^n (n>1) 如：'营销'^3 '微信'^2，可提升特定关键词的查询权重。
```json
{
  "query": {
    "query_string" : {
      "default_field" : "*",
      "query" : "\"食品\"^4 \"健康\"^4 \"市场整合营销\"^2 \"营销\"^2 \"方案\"^2 \"策略\"^0.2 \"全国\"^0.2"
    }
  }
}
```

### multi_match 多字段查询
```json
{
  "multi_match": {
    "query": "\"食品\"^2.0 \"方案\"^2.0 \"健康\"^2.0 \"市场整合营销\"^2.0 \"营销\"^2.0 ",
    "fields": [
      "city^1.0",
      "company^1.0",
      "company_description^1.0",
      "department^1.0",
      "experience^1.0",
      "position^1.0",
      "profession^1.0",
      "skills^1.0",
      "works^1.0"
    ],
    "type": "most_fields",
    "operator": "OR",
    "slop": 0,
    "prefix_length": 0,
    "max_expansions": 50,
    "zero_terms_query": "NONE",
    "auto_generate_synonyms_phrase_query": true,
    "fuzzy_transpositions": true,
    "analyzer": "ik_max_word",
    "boost": 1
  }
}
```
相关属性：
* query 需要查询的关键词
* fields 需要匹配的字段
* type: 多次段匹配的类型
   - best_feilds 最佳字段模式 这表示它会为每个字段生成一个 match 查询，然后将它们的_score组合到最终查询。
   - most_feilds 多数字段匹配 返回所有相关的文档，以及准确率(Precision) - 不返回无关文档 缺点：
      - 它是为多数字段匹配 任意 词设计的，而不是在 所有字段 中找到最匹配的。
      - 它不能使用 operator 或 minimum_should_match 参数来降低次相关结果造成的长尾效应。
      - 词频对于每个字段是不一样的，而且它们之间的相互影响会导致不好的排序结果
* cross_fields  跨字段实体搜索
* phrase 短语匹配
* phrase_prefix 短语前缀匹配
* `analyzer` 设定分析器，如果不进行设置为`ik_max_word`，将不能按照词进行精确匹配。中文查询需要制定分析器analyzer字段，否则会按照默认分词进行分析，出现单字拆分的格式。
各种查询匹配模式参考：http://www.cnblogs.com/yjf512/p/4897294.html

### TF / IDF
```
DELETE /test*
PUT /test
{
  "settings": {
    "number_of_shards": 1
  }
}
```
* 定义索引的settings中，设置 number_of_shards 为 1，可以保证搜索得分的一致性。
* 如果 shards 设为多个，由于文档顺序添加，文档添加到的shard不同；即便不同文档相同字段的内容完全相同，其TF/IDF得分也会有所区别（基于shard内的统计值进行计算）；这将导致相同的查询条件，内容完全相同的文档得分会有差异。

### Function Score Query
使用[Function Score Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html Function Score Query)可以灵活地的调整文档在单词 query 的 score。一个Function Score Query中可以包含多个Function，每个Function可包含以下配置：
* weight
   - 对每份文档适用一个简单的提升，且该提升不会被归约：当weight为2时，结果为2 * _score。
* [field_value_factor](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-field-value-factor)
   - 对文档中特定字段的score进行修正
* random_score
   - 使用一致性随机分值计算来对每个用户采用不同的结果排序方式，对相同用户仍然使用相同的排序方式。
* [script_score](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html#function-script-score)
   - 使用自定义的脚本来完全控制分值计算逻辑。如果你需要以上预定义函数之外的功能，可以根据需要通过脚本进行实现。
* 衰减函数 Decay Function
   - 分为linear、exp、gauss几种
   - 日期范围查询可以使用Decay Function实现模糊范围搜索。

使用function_score调整默认的socre，不要使用weight属性，weight属性会使匹配的所有文档权重全部一样。
```json
{
  "bool": {
    "disable_coord": false,
    "should": [
      {
        "function_score": {
          "boost": "1.0",
          "boost_mode": "replace",
          "query": {
            "query_string": {
              "query": "\"h5\"^2"
            }
          }
        }
      }
    ]
  }
}
```

参考：
* [How to complete disable tf/idf](https://discuss.elastic.co/t/how-to-complete-disable-tf-idf/70570)
* https://discuss.elastic.co/t/elasticsearch-java-api-for-function-score-query/18011
* https://stackoverflow.com/questions/24080771/elasticsearch-functionscore-query-using-java-api
* https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-compound-queries.html

## 内容评分
基于TF/IDF（词频/逆文档频率）算法默认评分公式，具体参见：[https://www.elastic.co/guide/cn/elasticsearch/guide/current/scoring-theory.html 相关度评分的背后理论]

计算文档得分需要考虑以下因子：
* 文档权重（document boost）：索引期赋予某个文档的权重值
* 字段权重（field boost）：查询期赋予某个字段的权重值
* 协调因子（coord）：基于文档中词项命中个数的协调因子，一个文档中命中了查询中的词项越多，得分越高
   - 查询关键词被分词为A和B，如果文档1命中了A和B，文档2命中了A，那么在这个项目上，文档1的分数更高
* 逆文档频率(inverse document frequency)：一个基于词项的因子，用来告诉评分公式该词项有多么罕见，逆文档频率越低，词项越罕见，评分公式利用该因子为包含罕见词项的文档加权
   - 查询关键词是A和B，如果文档1命中了A，文档2命中了B，但是在整个文档范围内，A出现的次数比B少，那么在这个项目中，文档1分数更高
* 字段长度归一值(length norm)：每个字段的基于词项个数的归一化因子，一个字段包含的词项越多，该因子的权重越低。
   - 这意味着lucene评分公式更“喜欢”包含更少词项的字段
   - 比如：查询关键词是A，文档1和2都匹配上了A，但是文档1内容长度比文档1短，那么在这个项目中，文档1分数更高
* 词频：一个基于词项的因子，用来表示一个词项在某个文档中出现多少次，词频越高，文档得分越高
   - 查询关键词是A，文档1和文档2都匹配上了，但是文档1中出现了2次A，文档2中出现了1次A，那么在这个项目中，文档1分数更高
* 查询范数（query norm）：一个基于查询的归一化因子，它等于查询中词项的权重平方和，查询范数使得不同查询的得分能相互比较，尽管这种比较通常是困难且不可行的

根据现有了解：
* 越多罕见的词项被匹配上，文档分数越高
* 文档字段越短，文档分数越高
* 权重越高（无论是索引期还查询期赋予的权重值），文档得分越高

### 词频
如果不在意词在某个字段中出现的频次，而只在意是否出现过，则可以在字段映射中禁用词频统计：
```
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type":          "string",
          "index_options": "docs"
        }
      }
    }
  }
}
```

### 逆向文档频率

### 字段长度归一值
字段越短，字段的权重越高 。如果词出现在类似标题 title 这样的字段，要比它出现在内容 body 这样的字段中的相关度更高。
```
PUT /my_index
{
  "mappings": {
    "doc": {
      "properties": {
        "text": {
          "type": "string",
          "norms": { "enabled": false }
        }
      }
    }
  }
}
```
### 权重集合的查询
* [Weighted Tags in Elasticsearch](https://conan.is/blogging/weighted-elasticsearch.html)

## 常见问题
### max file descriptors is too low
错误日志：
```shell
max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
```
追加配置 `vi /etc/security/limits.conf`
```shell
* hard nofile 65536
* soft nofile 65536
```

### max virtual memory too low
错误日志：
```shell
max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
追加配置 `vi /etc/sysctl.conf`
```shell
vm.max_map_count=262144
```
使用 `sysctl -p` 查看修改后的结果