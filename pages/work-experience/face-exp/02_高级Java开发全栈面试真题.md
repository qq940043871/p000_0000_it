# 高级Java开发全栈 - 面试真题（含详细答案）

> 适用岗位：高级Java开发工程师 / 全栈工程师 / AI应用开发工程师
> 面试时长：60-90 分钟

---

## 一、Java核心

### Q1: Java并发编程 - 线程池详解

**问题：** 请详细说明线程池的核心参数及其配置策略

**标准答案：**

```java
// ThreadPoolExecutor 核心参数
public ThreadPoolExecutor(
    int corePoolSize,      // 核心线程数
    int maximumPoolSize,   // 最大线程数
    long keepAliveTime,    // 空闲线程存活时间
    TimeUnit unit,         // 时间单位
    BlockingQueue<Runnable> workQueue,  // 工作队列
    ThreadFactory threadFactory,        // 线程工厂
    RejectedExecutionHandler handler    // 拒绝策略
)
```

**参数详解：**

| 参数 | 说明 | 配置建议 |
|------|------|---------|
| corePoolSize | 核心线程数，常驻线程 | CPU密集: CPU核数+1；IO密集: 2×CPU核数 |
| maximumPoolSize | 最大线程数 | 根据系统资源设置，避免OOM |
| keepAliveTime | 空闲线程存活时间 | 通常60-120秒 |
| workQueue | 工作队列 | 有界队列防止OOM |
| handler | 拒绝策略 | CallerRunsPolicy / 自定义 |

**拒绝策略：**

```java
// 1. AbortPolicy (默认): 抛异常
// 2. CallerRunsPolicy: 调用者线程执行
// 3. DiscardPolicy: 直接丢弃
// 4. DiscardOldestPolicy: 丢弃最老任务

// 自定义拒绝策略示例
public class CustomRejectedHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        // 记录日志
        log.warn("Task rejected, queue size: {}", e.getQueue().size());
        // 尝试放入缓存队列或告警
        alertService.sendAlert("线程池任务拒绝");
    }
}
```

**配置示例：**

```java
// IO密集型任务线程池
ThreadPoolExecutor ioPool = new ThreadPoolExecutor(
    Runtime.getRuntime().availableProcessors() * 2,  // core
    Runtime.getRuntime().availableProcessors() * 4,  // max
    60L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(1000),  // 有界队列
    new ThreadFactoryBuilder().setNameFormat("io-pool-%d").build(),
    new ThreadPoolExecutor.CallerRunsPolicy()
);

// CPU密集型任务线程池
ThreadPoolExecutor cpuPool = new ThreadPoolExecutor(
    Runtime.getRuntime().availableProcessors() + 1,  // core
    Runtime.getRuntime().availableProcessors() + 1,  // max = core
    0L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100),
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

---

### Q2: JVM 调优实战

**问题：** 线上服务频繁Full GC，如何排查和优化？

**标准答案：**

**排查步骤：**

```bash
# 1. 查看GC日志
jstat -gcutil <pid> 1000 10

# 2. 导出堆内存快照
jmap -dump:format=b,file=heap.hprof <pid>

# 3. 分析堆内存
# 使用 MAT 或 VisualVM 分析 heap.hprof

# 4. 查看线程栈
jstack <pid> > thread.txt
```

**常见问题与解决：**

| 问题 | 现象 | 解决方案 |
|------|------|---------|
| 内存泄漏 | Old区持续增长 | MAT分析泄漏对象，修复代码 |
| 大对象直接进老年代 | 频繁Full GC | 调大新生代，避免大对象 |
| System.gc()调用 | 意外Full GC | -XX:+DisableExplicitGC |
| 元空间不足 | Full GC | -XX:MaxMetaspaceSize=256m |

**JVM参数配置示例：**

```bash
# 生产环境推荐配置 (8G堆内存)
java -Xms8g -Xmx8g \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -XX:G1HeapRegionSize=16m \
     -XX:InitiatingHeapOccupancyPercent=45 \
     -XX:+ExplicitGCInvokesConcurrent \
     -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/logs/heap.hprof \
     -Xlog:gc*:file=/logs/gc.log:time,uptime,level,tags \
     -jar app.jar
```

**G1 GC 调优要点：**

```
1. MaxGCPauseMillis: 目标暂停时间，默认200ms
2. G1HeapRegionSize: Region大小，1-32MB
3. InitiatingHeapOccupancyPercent: 触发并发GC的堆占用率
4. G1ReservePercent: 预留空间，默认10%

调优原则:
- 不要频繁调整，一次只改一个参数
- 先分析GC日志，再针对性优化
- 关注P99延迟而非平均延迟
```

---

### Q3: Spring Boot 自动装配原理

**问题：** Spring Boot 自动装配是如何实现的？

**标准答案：**

**核心流程：**

```
1. @SpringBootApplication
   └── @EnableAutoConfiguration
       └── @Import(AutoConfigurationImportSelector.class)
           └── selectImports() 方法
               └── 读取 META-INF/spring.factories
                   └── 加载所有 EnableAutoConfiguration 配置类
                       └── 根据条件注解筛选生效的配置
```

**关键注解：**

```java
// 条件装配注解
@ConditionalOnClass          // 类路径存在某类
@ConditionalOnMissingClass   // 类路径不存在某类
@ConditionalOnBean           // 容器中存在某Bean
@ConditionalOnMissingBean    // 容器中不存在某Bean
@ConditionalOnProperty       // 配置属性满足条件
@ConditionalOnWebApplication // Web应用环境
```

**自动配置示例：**

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    @ConditionalOnProperty(prefix = "spring.datasource", 
                          name = "url")
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

**自定义Starter步骤：**

```
1. 创建 autoconfigure 模块
   └── 编写自动配置类
   └── 添加条件注解

2. 创建 starter 模块
   └── 依赖 autoconfigure

3. 编写 spring.factories
   └── org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
       com.example.MyAutoConfiguration

4. 编写配置属性类
   └── @ConfigurationProperties
```

---

## 二、Spring 全家桶

### Q4: Spring Bean 生命周期

**标准答案：**

```
┌─────────────────────────────────────────────────────────┐
│                Bean 生命周期完整流程                      │
│                                                         │
│  1. 实例化 (Instantiation)                              │
│     └── 构造函数创建对象                                │
│                                                         │
│  2. 属性赋值 (Population)                               │
│     └── @Autowired / @Value 注入                       │
│                                                         │
│  3. 初始化前 (Initialization)                           │
│     └── BeanPostProcessor.postProcessBeforeInitialization│
│                                                         │
│  4. 初始化                                              │
│     ├── @PostConstruct                                 │
│     ├── InitializingBean.afterPropertiesSet()          │
│     └── init-method                                    │
│                                                         │
│  5. 初始化后                                            │
│     └── BeanPostProcessor.postProcessAfterInitialization│
│                                                         │
│  6. 使用                                                │
│     └── Bean正常工作                                   │
│                                                         │
│  7. 销毁                                                │
│     ├── @PreDestroy                                    │
│     ├── DisposableBean.destroy()                       │
│     └── destroy-method                                 │
└─────────────────────────────────────────────────────────┘
```

**代码示例：**

```java
@Component
public class MyBean implements InitializingBean, DisposableBean {
    
    @PostConstruct
    public void postConstruct() {
        log.info("1. @PostConstruct");
    }
    
    @Override
    public void afterPropertiesSet() {
        log.info("2. InitializingBean.afterPropertiesSet");
    }
    
    @PreDestroy
    public void preDestroy() {
        log.info("3. @PreDestroy");
    }
    
    @Override
    public void destroy() {
        log.info("4. DisposableBean.destroy");
    }
}
```

---

### Q5: Spring 事务传播机制

**标准答案：**

| 传播行为 | 说明 | 使用场景 |
|---------|------|---------|
| REQUIRED | 有则加入，无则新建 | 默认，最常用 |
| SUPPORTS | 有则加入，无则非事务运行 | 查询方法 |
| MANDATORY | 必须在事务中，否则抛异常 | 强制事务 |
| REQUIRES_NEW | 新建事务，挂起当前 | 独立事务 |
| NOT_SUPPORTED | 非事务运行，挂起当前 | 不需要事务 |
| NEVER | 非事务运行，有事务则抛异常 | 禁止事务 |
| NESTED | 嵌套事务，savepoint | 部分回滚 |

**代码示例：**

```java
@Service
public class OrderService {
    
    // 主事务
    @Transactional
    public void createOrder(Order order) {
        orderDao.insert(order);
        // 调用子事务
        inventoryService.deductStock(order.getItemId(), order.getCount());
    }
}

@Service
public class InventoryService {
    
    // 独立事务，即使主事务回滚，库存扣减也生效
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void deductStock(Long itemId, int count) {
        inventoryDao.deductStock(itemId, count);
    }
}
```

**事务失效场景：**

```java
// 1. 方法不是public
@Transactional
private void method1() {}  // 失效

// 2. 同类调用
public void methodA() {
    this.methodB();  // methodB的事务失效
}
@Transactional
public void methodB() {}

// 解决: 注入自己或使用AopContext
public void methodA() {
    ((MyService) AopContext.currentProxy()).methodB();
}

// 3. 异常被catch
@Transactional
public void method() {
    try {
        // 业务代码
    } catch (Exception e) {
        // 异常被捕获，事务不回滚
    }
}

// 解决: 手动回滚或抛出异常
@Transactional
public void method() {
    try {
        // 业务代码
    } catch (Exception e) {
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}
```

---

### Q6: Spring Cloud 微服务架构

**标准答案：**

**核心组件：**

```
┌─────────────────────────────────────────────────────────┐
│                    微服务架构                            │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              API Gateway (Spring Cloud Gateway) │   │
│  │         路由 / 限流 / 熔断 / 鉴权 / 日志        │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                            │
│  ┌─────────────────────────┼───────────────────────┐   │
│  │                         │                       │   │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐    │   │
│  │  │ 服务A   │    │ 服务B   │    │ 服务C   │    │   │
│  │  │ Order   │    │ User    │    │ Product │    │   │
│  │  └────┬────┘    └────┬────┘    └────┬────┘    │   │
│  │       │              │              │          │   │
│  └───────┼──────────────┼──────────────┼──────────┘   │
│          │              │              │               │
│  ┌───────▼──────────────▼──────────────▼──────────┐   │
│  │            Nacos (注册中心 + 配置中心)          │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Sentinel (限流熔断) + Seata (分布式事务)       │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Gateway 配置示例：**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/order/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
            - name: CircuitBreaker
              args:
                name: default
                fallbackUri: forward:/fallback
```

---

## 三、大模型应用开发

### Q7: LangChain Java 开发实战

**标准答案：**

**LangChain4j 核心组件：**

```java
// 1. 模型配置
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey("your-api-key")
    .modelName("gpt-4o")
    .temperature(0.7)
    .maxTokens(1024)
    .build();

// 2. Prompt模板
PromptTemplate template = PromptTemplate.from(
    "根据以下上下文回答问题:\n" +
    "{{context}}\n\n" +
    "问题: {{question}}\n" +
    "答案:"
);

Map<String, Object> variables = Map.of(
    "context", context,
    "question", question
);
Prompt prompt = template.apply(variables);

// 3. RAG实现
EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
    .apiKey("your-api-key")
    .build();

EmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();

// 索引文档
String text = "文档内容...";
TextSegment segment = TextSegment.from(text);
Embedding embedding = embeddingModel.embed(segment).content();
embeddingStore.add(embedding, segment);

// 检索
Embedding queryEmbedding = embeddingModel.embed(question).content();
List<EmbeddingMatch<TextSegment>> matches = embeddingStore.findRelevant(
    queryEmbedding, 
    5  // top-k
);

// 4. 构建RAG Chain
String context = matches.stream()
    .map(m -> m.embedded().text())
    .collect(Collectors.joining("\n\n"));

String response = model.generate(prompt.text());
```

**与Spring Boot集成：**

```java
@Configuration
public class LangChainConfig {
    
    @Bean
    public ChatLanguageModel chatModel() {
        return OpenAiChatModel.builder()
            .apiKey("${openai.api-key}")
            .modelName("gpt-4o")
            .build();
    }
    
    @Bean
    public EmbeddingModel embeddingModel() {
        return OpenAiEmbeddingModel.builder()
            .apiKey("${openai.api-key}")
            .build();
    }
    
    @Bean
    public EmbeddingStore<TextSegment> embeddingStore() {
        // 使用Milvus
        return MilvusEmbeddingStore.builder()
            .host("localhost")
            .port(19530)
            .collectionName("documents")
            .build();
    }
}

@Service
public class RAGService {
    
    private final ChatLanguageModel chatModel;
    private final EmbeddingModel embeddingModel;
    private final EmbeddingStore<TextSegment> embeddingStore;
    
    public String chat(String question) {
        // 1. Embedding查询
        Embedding queryEmbedding = embeddingModel.embed(question).content();
        
        // 2. 检索相关文档
        List<EmbeddingMatch<TextSegment>> matches = 
            embeddingStore.findRelevant(queryEmbedding, 5);
        
        // 3. 构建上下文
        String context = matches.stream()
            .map(m -> m.embedded().text())
            .collect(Collectors.joining("\n\n"));
        
        // 4. 生成答案
        String prompt = String.format(
            "上下文:\n%s\n\n问题: %s\n答案:", 
            context, question
        );
        
        return chatModel.generate(prompt);
    }
}
```

---

### Q8: 向量数据库集成

**标准答案：**

**Milvus Java SDK 集成：**

```java
// 1. 依赖
// implementation 'io.milvus:milvus-sdk-java:2.3.4'

// 2. 连接配置
MilvusServiceClient milvusClient = new MilvusServiceClient(
    ConnectParam.newBuilder()
        .withHost("localhost")
        .withPort(19530)
        .build()
);

// 3. 创建Collection
String collectionName = "documents";
FieldType idField = FieldType.newBuilder()
    .withName("id")
    .withDataType(DataType.Int64)
    .withPrimaryKey(true)
    .withAutoID(true)
    .build();

FieldType embeddingField = FieldType.newBuilder()
    .withName("embedding")
    .withDataType(DataType.FloatVector)
    .withDimension(1024)  // BGE-M3向量维度
    .build();

FieldType contentField = FieldType.newBuilder()
    .withName("content")
    .withDataType(DataType.VarChar)
    .withMaxLength(65535)
    .build();

CreateCollectionParam createParam = CreateCollectionParam.newBuilder()
    .withCollectionName(collectionName)
    .withDescription("Document embeddings")
    .withShardsNum(2)
    .addFieldType(idField)
    .addFieldType(embeddingField)
    .addFieldType(contentField)
    .build();

milvusClient.createCollection(createParam);

// 4. 创建索引
CreateIndexParam indexParam = CreateIndexParam.newBuilder()
    .withCollectionName(collectionName)
    .withFieldName("embedding")
    .withIndexType(IndexType.HNSW)
    .withMetricType(MetricType.COSINE)
    .withExtraParam("{\"M\": 16, \"efConstruction\": 200}")
    .build();

milvusClient.createIndex(indexParam);

// 5. 插入数据
List<List<Float>> embeddings = Arrays.asList(embedding1, embedding2);
List<String> contents = Arrays.asList(content1, content2);

InsertParam insertParam = InsertParam.newBuilder()
    .withCollectionName(collectionName)
    .addField(Field.newBuilder()
        .withName("embedding")
        .withFloatVectors(embeddings)
        .build())
    .addField(Field.newBuilder()
        .withName("content")
        .withStrings(contents)
        .build())
    .build();

milvusClient.insert(insertParam);

// 6. 搜索
SearchParam searchParam = SearchParam.newBuilder()
    .withCollectionName(collectionName)
    .withMetricType(MetricType.COSINE)
    .withTopK(10)
    .withVectors(queryEmbedding)
    .withVectorFieldName("embedding")
    .withOutFields(Arrays.asList("content"))
    .withParams("{\"ef\": 100}")
    .build();

R<SearchResults> results = milvusClient.search(searchParam);
```

---

### Q9: 异步调用与流式响应

**标准答案：**

**Spring WebFlux 流式响应：**

```java
@RestController
public class ChatController {
    
    private final ChatLanguageModel chatModel;
    
    // 同步调用
    @PostMapping("/chat")
    public String chat(@RequestBody ChatRequest request) {
        return chatModel.generate(request.getMessage());
    }
    
    // 流式响应 (SSE)
    @GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> chatStream(@RequestParam String message) {
        return chatModel.generateStream(message);
    }
    
    // 使用WebFlux
    @PostMapping("/chat/webflux")
    public Mono<String> chatWebFlux(@RequestBody Mono<ChatRequest> request) {
        return request
            .map(ChatRequest::getMessage)
            .flatMap(msg -> Mono.fromCallable(() -> chatModel.generate(msg)))
            .subscribeOn(Schedulers.boundedElastic());
    }
}
```

**LangChain4j 流式调用：**

```java
// 流式模型配置
StreamingChatLanguageModel streamingModel = OpenAiStreamingChatModel.builder()
    .apiKey("your-api-key")
    .modelName("gpt-4o")
    .build();

// 流式调用
@PostMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> streamChat(@RequestBody ChatRequest request) {
    return Flux.create(emitter -> {
        streamingModel.generate(request.getMessage(), new StreamingResponseHandler<>() {
            @Override
            public void onNext(String token) {
                emitter.next(token);
            }
            
            @Override
            public void onComplete(Response<AiMessage> response) {
                emitter.complete();
            }
            
            @Override
            public void onError(Throwable error) {
                emitter.error(error);
            }
        });
    });
}
```

---

## 四、数据库与缓存

### Q10: MySQL 索引优化

**标准答案：**

**索引类型：**

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| 主键索引 | 聚簇索引，数据按主键存储 | 主键字段 |
| 唯一索引 | 唯一约束 + 索引 | 唯一约束字段 |
| 普通索引 | 非唯一索引 | 查询条件字段 |
| 复合索引 | 多列组合索引 | 多条件查询 |
| 全文索引 | 全文检索 | 文本搜索 |

**索引设计原则：**

```
1. 最左前缀原则
   索引 (a, b, c)
   ✅ WHERE a = 1
   ✅ WHERE a = 1 AND b = 2
   ✅ WHERE a = 1 AND b = 2 AND c = 3
   ❌ WHERE b = 2
   ❌ WHERE c = 3

2. 覆盖索引
   索引 (a, b)
   ✅ SELECT a, b FROM t WHERE a = 1  // 无需回表

3. 索引下推
   MySQL 5.6+ 自动优化
   WHERE a = 1 AND b LIKE '%x%'
   索引(a, b): 在索引层过滤b条件

4. 避免索引失效
   ❌ WHERE YEAR(create_time) = 2024  // 函数导致失效
   ✅ WHERE create_time >= '2024-01-01' AND create_time < '2025-01-01'
   
   ❌ WHERE name LIKE '%张%'  // 前缀%导致失效
   ✅ WHERE name LIKE '张%'
```

**Explain 分析：**

```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 100 AND status = 'PAID';

-- 关键字段:
-- type: ALL(全表扫描) < index < range < ref < eq_ref < const
-- key: 实际使用的索引
-- rows: 预估扫描行数
-- Extra: Using index(覆盖索引), Using filesort(需优化)
```

---

### Q11: Redis 缓存策略

**标准答案：**

**缓存模式：**

```java
// 1. Cache-Aside (旁路缓存)
public String getData(String key) {
    // 先查缓存
    String value = redis.get(key);
    if (value != null) {
        return value;
    }
    
    // 查数据库
    value = db.query(key);
    if (value != null) {
        // 写入缓存
        redis.set(key, value, 3600);
    }
    return value;
}

// 2. Write-Through (写穿透)
public void updateData(String key, String value) {
    // 先更新数据库
    db.update(key, value);
    // 再更新缓存
    redis.set(key, value, 3600);
}

// 3. Write-Behind (写回)
public void updateData(String key, String value) {
    // 先更新缓存
    redis.set(key, value, 3600);
    // 异步更新数据库
    asyncExecutor.execute(() -> db.update(key, value));
}
```

**缓存问题与解决：**

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| 缓存穿透 | 查询不存在的数据 | 布隆过滤器 / 缓存空值 |
| 缓存击穿 | 热点key过期 | 互斥锁 / 永不过期 |
| 缓存雪崩 | 大量key同时过期 | 随机过期时间 |

**代码示例：**

```java
// 防止缓存穿透 - 布隆过滤器
public String getDataWithBloom(String key) {
    // 先检查布隆过滤器
    if (!bloomFilter.mightContain(key)) {
        return null;  // 一定不存在
    }
    
    String value = redis.get(key);
    if (value != null) {
        return value;
    }
    
    value = db.query(key);
    if (value != null) {
        redis.set(key, value, 3600);
    } else {
        // 缓存空值，短过期时间
        redis.set(key, "", 60);
    }
    return value;
}

// 防止缓存击穿 - 互斥锁
public String getDataWithLock(String key) {
    String value = redis.get(key);
    if (value != null) {
        return value;
    }
    
    // 获取分布式锁
    String lockKey = "lock:" + key;
    boolean locked = redis.setnx(lockKey, "1", 10);
    
    if (locked) {
        try {
            // 双重检查
            value = redis.get(key);
            if (value != null) {
                return value;
            }
            
            value = db.query(key);
            if (value != null) {
                redis.set(key, value, 3600);
            }
            return value;
        } finally {
            redis.del(lockKey);
        }
    } else {
        // 等待并重试
        Thread.sleep(100);
        return getDataWithLock(key);
    }
}
```

---

## 五、系统设计

### Q12: 高并发系统设计

**问题：** 设计一个支持10万QPS的秒杀系统

**标准答案：**

```
┌─────────────────────────────────────────────────────────┐
│                    秒杀系统架构                          │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  用户请求 → CDN(静态资源) → Nginx(限流)          │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                            │
│  ┌─────────────────────────▼───────────────────────┐   │
│  │              API Gateway                         │   │
│  │      (鉴权/限流/熔断/降级)                       │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                            │
│  ┌─────────────────────────▼───────────────────────┐   │
│  │              Redis (库存预扣减)                  │   │
│  │         Lua脚本保证原子性                        │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                            │
│  ┌─────────────────────────▼───────────────────────┐   │
│  │              MQ (异步下单)                       │   │
│  │         Kafka / RocketMQ                        │   │
│  └─────────────────────────┬───────────────────────┘   │
│                            │                            │
│  ┌─────────────────────────▼───────────────────────┐   │
│  │              Order Service                       │   │
│  │      (消费MQ → 创建订单 → 扣减库存)              │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**核心代码：**

```java
// Redis Lua脚本 - 原子扣减库存
String luaScript = """
    local stock = redis.call('GET', KEYS[1])
    if not stock then
        return -1
    end
    if tonumber(stock) <= 0 then
        return 0
    end
    redis.call('DECR', KEYS[1])
    return 1
    """;

// 秒杀接口
@PostMapping("/seckill")
public Result seckill(@RequestParam Long userId, @RequestParam Long itemId) {
    // 1. 检查用户是否已购买
    if (redis.sismember("seckill:users:" + itemId, userId.toString())) {
        return Result.fail("已购买过");
    }
    
    // 2. 预扣减库存
    Long stock = redis.execute(luaScript, 
        Collections.singletonList("seckill:stock:" + itemId), 
        Collections.emptyList());
    
    if (stock == 0) {
        return Result.fail("库存不足");
    }
    
    // 3. 记录购买用户
    redis.sadd("seckill:users:" + itemId, userId.toString());
    
    // 4. 发送MQ异步下单
    SeckillMessage message = new SeckillMessage(userId, itemId);
    kafkaTemplate.send("seckill-order", message);
    
    return Result.success("下单成功，请等待处理");
}

// MQ消费者 - 异步创建订单
@KafkaListener(topics = "seckill-order")
public void createOrder(SeckillMessage message) {
    try {
        // 创建订单
        Order order = orderService.createOrder(message);
        // 扣减真实库存
        inventoryService.deductStock(message.getItemId(), 1);
        // 发送通知
        notificationService.notify(message.getUserId(), "秒杀成功");
    } catch (Exception e) {
        // 失败补偿：恢复库存
        redis.incr("seckill:stock:" + message.getItemId());
        redis.srem("seckill:users:" + message.getItemId(), 
                   message.getUserId().toString());
    }
}
```

---

## 六、评分标准

| 等级 | 分数 | 标准 |
|------|------|------|
| 卓越 | 90-100 | 技术全面、深度扎实、有实战经验 |
| 优秀 | 80-89 | 技术扎实、理解深入 |
| 良好 | 70-79 | 基础扎实、有一定深度 |
| 一般 | 60-69 | 了解基本概念、缺乏深度 |
| 不通过 | <60 | 基础薄弱 |

---

返回目录：面试问题大纲索引.md
