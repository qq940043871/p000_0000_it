MongoDB - 架构师学习笔记
    
    


    
        
            
                MongoDB
            
            
            
                
                    MongoDB是一个基于分布式文件存储的开源数据库系统，属于NoSQL数据库的一种。它使用BSON（Binary JSON）格式存储数据，支持动态查询、索引、复制、分片等特性，适用于大数据量、高并发的Web应用。
                
                
                MongoDB核心概念
                
                
                    
                        
                            
                            Database（数据库）
                        
                        
                            数据库是集合的容器，一个MongoDB实例可以包含多个数据库。
                        
                    
                    
                    
                        
                            
                            Collection（集合）
                        
                        
                            集合类似于关系数据库中的表，是一组文档的集合。
                        
                    
                    
                    
                        
                            
                            Document（文档）
                        
                        
                            文档是MongoDB中基本的数据单元，类似于关系数据库中的行记录。
                        
                    
                    
                    
                        
                            
                            Field（字段）
                        
                        
                            字段是文档中的键值对，类似于关系数据库中的列。
                        
                    
                
                
                基本操作
                
                
                    
                        
                            
                            插入操作
                        
                        
// 插入单个文档
db.users.insertOne({
  name: "Alice",
  age: 25,
  email: "alice@example.com",
  hobbies: ["reading", "swimming"]
})

// 插入多个文档
db.users.insertMany([
  {name: "Bob", age: 30, email: "bob@example.com"},
  {name: "Charlie", age: 35, email: "charlie@example.com"}
])
                    
                    
                    
                        
                            
                            查询操作
                        
                        
// 基本查询
db.users.find({age: {$gte: 25}})

// 投影查询
db.users.find({age: {$gte: 25}}, {name: 1, email: 1})

// 排序和限制
db.users.find().sort({age: 1}).limit(5)

// 聚合查询
db.users.aggregate([
  {$match: {age: {$gte: 25}}},
  {$group: {_id: "$age", count: {$sum: 1}}},
  {$sort: {_id: 1}}
])
                    
                    
                    
                        
                            
                            更新操作
                        
                        
// 更新单个文档
db.users.updateOne(
  {name: "Alice"},
  {$set: {age: 26}}
)

// 更新多个文档
db.users.updateMany(
  {age: {$lt: 30}},
  {$set: {status: "young"}}
)

// 数组操作
db.users.updateOne(
  {name: "Alice"},
  {$push: {hobbies: "dancing"}}
)
                    
                    
                    
                        
                            
                            删除操作
                        
                        
// 删除单个文档
db.users.deleteOne({name: "Alice"})

// 删除多个文档
db.users.deleteMany({age: {$lt: 25}})

// 删除所有文档
db.users.deleteMany({})
                    
                
                
                索引优化
                
                
                    索引类型
                    
                        
                            
                                
                                    索引类型
                                    说明
                                    创建语法
                                
                            
                            
                                
                                    单字段索引
                                    为单个字段创建索引
                                    db.collection.createIndex({name: 1})
                                
                                
                                    复合索引
                                    为多个字段创建索引
                                    db.collection.createIndex({name: 1, age: -1})
                                
                                
                                    多键索引
                                    为数组字段创建索引
                                    db.collection.createIndex({tags: 1})
                                
                                
                                    文本索引
                                    为文本搜索创建索引
                                    db.collection.createIndex({content: "text"})
                                
                                
                                    地理空间索引
                                    为地理位置数据创建索引
                                    db.collection.createIndex({location: "2dsphere"})
                                
                            
                        
                    
                    
                    索引优化建议
                    
                        为经常查询的字段创建索引
                        复合索引遵循最左前缀原则
                        定期分析索引使用情况
                        删除未使用的索引
                        避免过多索引影响写入性能
                    
                
                
                复制集
                
                
                    
                        
                            
                            复制集架构
                        
                        
                            Primary节点：处理所有读写操作
                            Secondary节点：复制Primary数据，可处理读操作
                            仲裁节点：参与选举，不存储数据
                        
                    
                    
                    
                        
                            
                            故障转移
                        
                        
                            自动选举新的Primary节点
                            数据一致性通过oplog保证
                            支持读写分离
                            提高数据可用性
                        
                    
                
                
                分片集群
                
                
                    分片架构
                    
                        
                            Query Router
                            mongos
                        
                        
                            Config Server
                            配置数据
                        
                        
                            Shard 1
                            数据分片
                        
                        
                            Shard 2
                            数据分片
                        
                    
                    
                    分片键选择
                    
                        选择基数大的字段作为分片键
                        避免单调递增的字段
                        考虑查询模式
                        使用复合分片键提高分布均匀性
                    
                
                
                聚合管道
                
                
                    常用聚合阶段
                    
                        
                            
                                
                                    阶段
                                    功能
                                    示例
                                
                            
                            
                                
                                    $match
                                    过滤文档
                                    {$match: {age: {$gte: 18}}}
                                
                                
                                    $group
                                    分组聚合
                                    {$group: {_id: "$category", count: {$sum: 1}}}
                                
                                
                                    $sort
                                    排序
                                    {$sort: {age: 1}}
                                
                                
                                    $project
                                    投影
                                    {$project: {name: 1, age: 1}}
                                
                                
                                    $limit
                                    限制数量
                                    {$limit: 10}
                                
                            
                        
                    
                    
                    聚合示例
                    
// 计算每个类别的商品数量和平均价格
db.products.aggregate([
  {
    $group: {
      _id: "$category",
      count: {$sum: 1},
      avgPrice: {$avg: "$price"}
    }
  },
  {
    $sort: {count: -1}
  }
])

// 复杂聚合示例
db.orders.aggregate([
  {$match: {status: "completed"}},
  {$lookup: {
    from: "customers",
    localField: "customerId",
    foreignField: "_id",
    as: "customer"
  }},
  {$unwind: "$customer"},
  {$group: {
    _id: "$customer.region",
    totalSales: {$sum: "$amount"},
    orderCount: {$sum: 1}
  }}
])
                
                
                性能优化
                
                
                    
                        
                            
                            查询优化
                        
                        
                            使用explain()分析查询性能
                            避免全表扫描
                            合理使用索引
                            限制返回结果集大小
                        
                    
                    
                    
                        
                            
                            存储优化
                        
                        
                            使用压缩存储引擎
                            合理设计文档结构
                            避免大文档
                            定期清理无用数据
                        
                    
                
                
                监控与诊断
                
                
                    监控工具
                    
                        
                            MongoDB Compass
                            
                                官方图形化管理工具
                            
                        
                        
                            MongoDB Atlas
                            
                                云数据库监控
                            
                        
                        
                            第三方工具
                            
                                Prometheus, Grafana等
                            
                        
                    
                    
                    常用监控命令
                    
// 数据库状态
db.stats()

// 集合状态
db.collection.stats()

// 查询分析
db.collection.find({name: "Alice"}).explain("executionStats")

// 性能监控
db.currentOp()
db.killOp(opid)
                
                
                MongoDB最佳实践
                
                    
                        合理设计文档结构，避免过度嵌套
                        建立合适的索引策略
                        使用复制集提高可用性
                        根据数据量选择分片策略
                        定期备份重要数据
                        监控性能指标，及时优化
                        合理配置内存和存储资源
