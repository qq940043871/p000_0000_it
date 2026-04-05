# Elasticsearch - 架构师学习笔记

## Elasticsearch

Elasticsearch是一个基于Apache Lucene构建的开源、分布式、RESTful搜索引擎。它能够快速地存储、搜索和分析大量数据，广泛应用于全文搜索、日志分析、指标监控等场景。

### Elasticsearch核心概念

#### Index（索引）

- 索引是具有相似特征的文档集合，类似于关系数据库中的数据库。

#### Document（文档）

- 文档是可被索引的基本单位，采用JSON格式表示，类似于关系数据库中的行。

#### Type（类型）

- 类型是索引中逻辑类别/分区的容器，但在7.x版本后已被废弃。

#### Shard（分片）

- 分片是索引的子集，用于水平分割数据，提高性能和可扩展性。

#### Replica（副本）

- 副本是分片的拷贝，提供高可用性和数据冗余。

#### Node（节点）

- 节点是Elasticsearch的单个服务器实例，存储数据并参与集群操作。

### 基本操作

#### 索引操作

```bash
# 创建索引
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}

# 删除索引
DELETE /my_index

# 查看索引信息
GET /my_index

# 打开/关闭索引
POST /my_index/_open
POST /my_index/_close
```

#### 文档操作

```bash
# 创建/更新文档
POST /my_index/_doc/1
{
  "title": "Elasticsearch入门",
  "content": "这是一篇关于Elasticsearch的文章",
  "author": "张三",
  "publish_date": "2023-01-01"
}

# 查询文档
GET /my_index/_doc/1

# 更新文档
POST /my_index/_update/1
{
  "doc": {
    "author": "李四"
  }
}

# 删除文档
DELETE /my_index/_doc/1
```

#### 搜索操作

```bash
# 基本搜索
GET /my_index/_search
{
  "query": {
    "match": {
      "title": "Elasticsearch"
    }
  }
}

# 复合搜索
GET /my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "Elasticsearch"}},
        {"range": {"publish_date": {"gte": "2023-01-01"}}}
      ]
    }
  }
}

# 聚合搜索
GET /my_index/_search
{
  "aggs": {
    "author_count": {
      "terms": {"field": "author.keyword"}
    }
  }
}
```

#### 映射操作

```bash
# 创建映射
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "author": {
        "type": "keyword"
      },
      "publish_date": {
        "type": "date"
      }
    }
  }
}

# 查看映射
GET /my_index/_mapping

# 更新映射（只能添加字段）
PUT /my_index/_mapping
{
  "properties": {
    "tags": {
      "type": "keyword"
    }
  }
}
```

### 查询DSL

#### 查询类型

| 查询类型 | 说明 | 示例 |
|---------|------|------|
| Match Query | 全文搜索 | {"match": {"title": "Elasticsearch"}} |
| Term Query | 精确匹配 | {"term": {"status": "published"}} |
| Range Query | 范围查询 | {"range": {"age": {"gte": 18, "lte": 65}}} |
| Bool Query | 复合查询 | {"bool": {"must": [...], "should": [...]}} |
| Wildcard Query | 通配符查询 | {"wildcard": {"title": "elastic*"}} |

#### 复合查询示例

```bash
# 复杂查询示例
GET /articles/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "Elasticsearch"}},
        {"range": {"publish_date": {"gte": "2023-01-01"}}}
      ],
      "must_not": [
        {"term": {"status": "draft"}}
      ],
      "should": [
        {"match": {"content": "search"}},
        {"match": {"content": "engine"}}
      ],
      "filter": [
        {"term": {"category": "technology"}}
      ]
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "content": {}
    }
  },
  "sort": [
    {"publish_date": {"order": "desc"}},
    {"_score": {"order": "desc"}}
  ]
}
```

### 分词器

#### 内置分词器

- Standard Analyzer：默认分词器，按词切分
- Simple Analyzer：按非字母切分，转小写
- Whitespace Analyzer：按空格切分
- Keyword Analyzer：不分词，整体作为关键词

#### 中文分词器

- IK Analyzer：最流行的中文分词器
- Smart Chinese Analyzer：ES内置中文分词器
- Jieba Analyzer：结巴分词器

```bash
# IK分词器配置示例
PUT /chinese_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ik_smart": {
          "type": "custom",
          "tokenizer": "ik_smart"
        }
      }
    }
  }
}
```

### 集群与分片

#### 集群架构

- **Master Node**：管理集群状态
- **Data Node**：存储数据
- **Client Node**：处理请求

#### 分片策略

- 分片数量在创建索引时确定，之后不能更改
- 每个分片建议保持在10-40GB之间
- 分片数量不应超过节点数量的20倍
- 合理设置副本数量以平衡性能和可靠性

### 聚合分析

#### 指标聚合

- Avg：平均值
- Max/Min：最大值/最小值
- Sum：求和
- Cardinality：基数统计

#### 桶聚合

- Terms：按字段值分组
- Range：按范围分组
- Date Histogram：按时间分组
- Filters：按过滤条件分组

### 性能优化

#### 查询优化

- 使用filter上下文代替query上下文
- 合理使用_source字段过滤
- 避免深度分页
- 使用索引别名

#### 内存优化

- 设置合适的堆内存大小（不超过32GB）
- 启用字段数据缓存
- 使用doc values代替fielddata
- 定期清理未使用的索引

### 监控与诊断

#### 监控API

| API | 功能 |
|-----|------|
| _cluster/health | 集群健康状态 |
| _nodes/stats | 节点统计信息 |
| _cat/indices | 索引信息 |
| _tasks | 任务信息 |

#### 常用监控命令

```bash
# 集群健康状态
GET /_cluster/health

# 节点信息
GET /_nodes/stats

# 索引状态
GET /_cat/indices?v

# 搜索慢日志
GET /_settings?include_defaults=true
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s"
}
```

### Elasticsearch最佳实践

- 合理设计索引和映射结构
- 选择合适的分词器处理文本数据
- 设置合适的分片和副本数量
- 实施监控和告警机制
- 定期备份重要数据
- 优化查询性能，避免慢查询
- 合理配置内存和存储资源
