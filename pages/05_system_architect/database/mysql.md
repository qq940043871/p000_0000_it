MySQL优化 - 架构师学习笔记
    
    


    
        
            
                MySQL优化
            
            
            
                
                    MySQL是世界上最流行的开源关系型数据库管理系统之一。随着数据量和访问量的增长，数据库性能优化成为系统架构中的关键环节。合理的优化策略可以显著提升系统性能和用户体验。
                
                
                SQL优化
                
                
                    
                        
                            
                            EXPLAIN执行计划
                        
                        
                            通过EXPLAIN分析SQL语句的执行计划，找出性能瓶颈。
                        
                        
EXPLAIN SELECT * FROM users WHERE age > 25;

-- 关键字段说明
-- type: 访问类型（ALL、index、range、ref、eq_ref、const）
-- key: 实际使用的索引
-- rows: 扫描的行数
-- Extra: 额外信息
                    
                    
                    
                        
                            
                            SQL优化技巧
                        
                        
                            避免SELECT *，只查询需要的字段
                            使用LIMIT限制返回结果集大小
                            避免在WHERE子句中使用函数
                            使用JOIN代替子查询
                            合理使用UNION和UNION ALL
                        
                    
                
                
                索引优化
                
                
                    索引类型
                    
                        
                            
                                
                                    索引类型
                                    特点
                                    适用场景
                                
                            
                            
                                
                                    主键索引
                                    唯一且非空，一张表只能有一个
                                    主键字段
                                
                                
                                    唯一索引
                                    索引列的值必须唯一，允许空值
                                    需要唯一性约束的字段
                                
                                
                                    普通索引
                                    最基本的索引，没有任何限制
                                    频繁查询的字段
                                
                                
                                    复合索引
                                    多个字段组成的索引
                                    多条件查询
                                
                                
                                    全文索引
                                    用于全文搜索
                                    文本搜索
                                
                            
                        
                    
                    
                    索引优化原则
                    
                        选择区分度高的列作为索引
                        为经常出现在WHERE、ORDER BY、GROUP BY中的列创建索引
                        复合索引遵循最左前缀原则
                        避免过多索引，影响写入性能
                        定期分析和优化索引使用情况
                    
                
                
                表结构优化
                
                
                    
                        
                            
                            数据类型选择
                        
                        
                            选择合适的数据类型，避免过度分配
                            使用NOT NULL约束，避免NULL值
                            使用ENUM代替字符串，节省存储空间
                            使用TIMESTAMP代替DATETIME，节省存储空间
                        
                    
                    
                    
                        
                            
                            表拆分策略
                        
                        
                            垂直拆分：将表按列拆分为多个表
                            水平拆分：将表按行拆分为多个表
                            冷热数据分离：将不常用数据归档
                        
                    
                
                
                查询优化
                
                
                    优化技巧
                    
                        
                            避免全表扫描
                            
                                为WHERE条件字段创建索引
                                避免在索引列上使用函数
                                避免使用!=或操作符
                            
                        
                        
                            优化JOIN操作
                            
                                确保JOIN字段上有索引
                                优先过滤数据再JOIN
                                避免大表JOIN大表
                            
                        
                        
                            优化子查询
                            
                                使用JOIN代替子查询
                                使用EXISTS代替IN
                                避免嵌套过深的子查询
                            
                        
                        
                            优化GROUP BY
                            
                                为GROUP BY字段创建索引
                                使用ORDER BY NULL避免排序
                                合理使用WITH ROLLUP
                            
                        
                    
                
                
                存储引擎优化
                
                
                    
                        
                            
                            InnoDB存储引擎
                        
                        
                            支持事务和外键约束
                            支持行级锁和MVCC
                            支持崩溃恢复
                            适合高并发读写场景
                        
                        
-- InnoDB配置优化
innodb_buffer_pool_size = 70%内存
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
                    
                    
                    
                        
                            
                            MyISAM存储引擎
                        
                        
                            不支持事务和行级锁
                            查询性能高
                            支持全文索引
                            适合读多写少场景
                        
                        
-- MyISAM配置优化
key_buffer_size = 256M
myisam_sort_buffer_size = 128M
                    
                
                
                配置优化
                
                
                    关键配置参数
                    
                        
                            
                                
                                    参数
                                    说明
                                    推荐值
                                
                            
                            
                                
                                    innodb_buffer_pool_size
                                    InnoDB缓冲池大小
                                    物理内存的70-80%
                                
                                
                                    innodb_log_file_size
                                    InnoDB日志文件大小
                                    256M-2G
                                
                                
                                    query_cache_size
                                    查询缓存大小
                                    根据查询特点设置
                                
                                
                                    max_connections
                                    最大连接数
                                    根据并发需求设置
                                
                                
                                    tmp_table_size
                                    临时表大小
                                    64M-256M
                                
                            
                        
                    
                
                
                监控与诊断
                
                
                    
                        
                            
                            性能监控
                        
                        
                            SHOW PROCESSLIST查看当前连接
                            SHOW STATUS查看服务器状态
                            SHOW VARIABLES查看配置参数
                            慢查询日志分析
                        
                    
                    
                    
                        
                            
                            常见问题诊断
                        
                        
                            锁等待和死锁问题
                            索引失效问题
                            慢查询优化
                            连接数过多问题
                        
                    
                
                
                MySQL优化最佳实践
                
                    
                        定期分析和优化表结构
                        建立完善的索引策略
                        优化SQL查询语句
                        合理配置MySQL参数
                        实施监控和告警机制
                        定期备份和恢复测试
                        使用读写分离和分库分表
